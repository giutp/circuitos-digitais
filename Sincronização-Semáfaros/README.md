<a id="readme-top"></a>

[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<br />
<div align="center">
  <h3 align="center">🚦 Sincronização de Semáforos</h3>

  <p align="center">
    Máquinas de estados sequenciais com flip-flops para simular semáforos sincronizados com botão de pedestre — trabalho da disciplina Circuitos Digitais (CI1068) na UFPR.
    <br />
    <a href="https://github.com/giutp/circuitos-digitais/issues/new?labels=bug">Reportar Bug</a>
    &middot;
    <a href="https://github.com/giutp/circuitos-digitais/issues/new?labels=enhancement">Sugerir Melhoria</a>
  </p>
</div>

---

<!-- SUMÁRIO -->
<details>
  <summary>Sumário</summary>
  <ol>
    <li><a href="#-sobre-o-projeto">Sobre o Projeto</a>
      <ul>
        <li><a href="#-construído-com">Construído com</a></li>
      </ul>
    </li>
    <li><a href="#-arquitetura-e-fundamentação">Arquitetura e Fundamentação</a></li>
    <li><a href="#-componentes-do-circuito">Componentes do Circuito</a></li>
    <li><a href="#-máquina-de-estados-e-fluxo">Máquina de Estados e Fluxo</a></li>
    <li><a href="#-estrutura-do-projeto">Estrutura do Projeto</a></li>
    <li><a href="#-como-simular">Como Simular</a></li>
    <li><a href="#-dificuldades-e-aprendizados">Dificuldades e Aprendizados</a></li>
    <li><a href="#-licença">Licença</a></li>
    <li><a href="#-contato">Contato</a></li>
    <li><a href="#-agradecimentos">Agradecimentos</a></li>
  </ol>
</details>

---

## 📖 Sobre o Projeto

**Sincronização de Semáforos** é um sistema sequencial que simula dois semáforos veiculares sincronizados com suporte a um botão de pedestre, implementado no simulador [Digital](https://github.com/hneemann/Digital) para a disciplina **Circuitos Digitais (CI1068)** da **Universidade Federal do Paraná (UFPR)**.

O desafio central foi projetar uma **máquina de estados finita (FSM)** para controlar a sequência de fases dos semáforos (Vermelho → Verde → Amarelo), garantindo que o acionamento do botão de pedestre interrompa o ciclo normal de forma segura e sincronizada — sem estados inválidos. Para isso, foram empregados **flip-flops D** como elementos de memória de estado, com as equações de próximo estado derivadas via tabelas-verdade e simplificadas com **mapas de Karnaugh**. Foram exploradas três variações de flip-flop: **D**, **S-R** e **J-K**.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

### 🛠 Construído com

* [![Digital][Digital-badge]][Digital-url]

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## ⏱ Arquitetura

O circuito é **sequencial síncrono**: todos os flip-flops compartilham o mesmo sinal de clock (gerado pelo `clk slow.dig`) e atualizam o estado a cada borda de subida. O estado atual é codificado em 4 bits (D3..D0), representando as fases dos dois semáforos.

```
        btn (pedestre)
           |
           v
     +-----+-------+
     |   Latch S-R  |   ← captura o pedido do pedestre e mantém até ser atendido
     +------+-------+
            |
     +------v-------+          clk slow
     |  FSM (D FFs) | <────────────────
     |  D3 D2 D1 D0 |
     +------+-------+
            |
     +------v-----------+
     | decoder_sem.dig  |   → R (vermelho), Y (amarelo), G (verde) para cada semáforo
     +------------------+
            |
     [LEDs Semáforo 1]  [LEDs Semáforo 2]
      🔴 🟡 🟢           🔴 🟡 🟢
```

As equações de próximo estado (D_FF inputs) foram derivadas via tabela-verdade e simplificadas com mapas de Karnaugh, gerando expressões AND/OR/XOR implementadas diretamente em portas lógicas.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 👥 Componentes do Circuito

| Arquivo | Entradas | Saídas | Descrição |
|---------|----------|--------|-----------|
| **`clk slow.dig`** | — | c sl | Divisor de frequência com 2 flip-flops D: gera clock lento para a FSM. |
| **`latch S-R.dig`** | S, E, R | Q, ~Q | Latch S-R habilitado (Enable) com 4 NANDs: captura e mantém o pedido do pedestre. |
| **`decoder_interno.dig`** | S0, S1 | S0i, S1i | Decodificador interno de estado: AND para detectar combinação de bits. |
| **`decoder_sem.dig`** | E0, E1 | R, Y, G | Decodificador de semáforo: converte 2 bits de estado em Vermelho, Amarelo e Verde (XNOR + ANDs). |
| **`Reset btn.dig`** | D0..D3, D2' | R | Lógica de reset do pedido: detecta quando a FSM chegou ao estado de atendimento ao pedestre. |
| **`circuito.dig`** | btn | E0S1, E1S1, ES2, E0S3, E1S3, ES4 | Núcleo da FSM: 4 flip-flops D + lógica de próximo estado + decodificadores de saída para cada semáforo. |
| **`main.dig`** | btn (botão), Clock | — | Toplevel: integra o circuito com os 6 LEDs de visualização (2 semáforos × 3 cores). |

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 🔄 Máquina de Estados e Fluxo

A FSM possui estados que representam as fases dos dois semáforos. Em operação normal, o ciclo segue automaticamente a cada pulso de clock. Ao pressionar `btn`, o latch S-R sinaliza o pedido e a FSM desvia para o estado de pedestre assim que a fase atual for segura para interrupção.

| Estado (D3..D0) | Semáforo 1 | Semáforo 2 | Observação |
|-----------------|-----------|-----------|------------|
| Fase normal 1 | 🟢 Verde | 🔴 Vermelho | Semáforo 1 com preferência |
| Fase normal 2 | 🟡 Amarelo | 🔴 Vermelho | Transição S1 |
| Fase normal 3 | 🔴 Vermelho | 🟢 Verde | Semáforo 2 com preferência |
| Fase normal 4 | 🔴 Vermelho | 🟡 Amarelo | Transição S2 |
| Fase pedestre | 🔴 Vermelho | 🔴 Vermelho | Ambos fechados para pedestres |

> Os arquivos de apoio em `Arquivos auxiliares/` documentam as tabelas-verdade completas (TV) e os mapas de Karnaugh (MK) para as três implementações: flip-flop **D**, **S-R** e **J-K**.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 📁 Estrutura do Projeto

```
Sincronização-Semáfaros/
├── Circuito/
│   ├── main.dig                  toplevel com LEDs e botão de pedestre
│   ├── circuito.dig              núcleo da FSM (4 flip-flops D + lógica de próximo estado)
│   ├── clk slow.dig              divisor de clock (frequência reduzida para visualização)
│   ├── latch S-R.dig             latch S-R com enable (captura pedido de pedestre)
│   ├── decoder_sem.dig           decodificador de estado → R/Y/G por semáforo
│   ├── decoder_interno.dig       decodificador auxiliar interno de estado
│   └── Reset btn.dig             lógica de liberação do latch após atendimento
├── Máquina de estados/
│   └── Máquina de estados.pdf    diagrama de estados da FSM
└── Arquivos auxiliares/
    ├── Perguntas e respostas - auxílio.pdf
    ├── Tabela-verdades/
    │   ├── Principal/             tabelas-verdade da FSM principal (D FFs)
    │   ├── S-R/                   tabelas-verdade para implementação com S-R
    │   └── J-K/                   tabelas-verdade para implementação com J-K
    └── Mapas de Karnaugh/
        ├── Principal/             mapas de Karnaugh e simplificações (implementação D)
        ├── S-R/                   mapas de Karnaugh (implementação S-R)
        └── J-K/                   mapas de Karnaugh (implementação J-K)
```

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 🚀 Como Simular

### 📦 Pré-requisitos

É necessário o simulador **Digital** (Java):

1. Baixe o [Digital](https://github.com/hneemann/Digital/releases/latest) (arquivo `.jar`).
2. Tenha o **Java 11+** instalado:
   ```sh
   sudo apt install default-jre -y
   ```

### ▶️ Executando

1. Clone o repositório:
   ```sh
   git clone https://github.com/giutp/circuitos-digitais.git
   cd "circuitos-digitais/Sincronização-Semáfaros/Circuito"
   ```
2. Abra o simulador Digital:
   ```sh
   java -jar /caminho/para/Digital.jar
   ```
3. Abra o arquivo `main.dig` pelo menu **File → Open**.
4. Clique em **▶ Start** (ou `F5`) para iniciar a simulação.
5. Observe os LEDs dos dois semáforos alternando automaticamente.
6. Pressione o botão **`btn`** para acionar o pedido de pedestre e observar a sincronização.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 📚 Dificuldades e Aprendizados

- **Entender tabelas-verdade e simplificações** — Derivar as equações de próximo estado para cada flip-flop a partir da tabela-verdade da FSM e então aplicar mapas de Karnaugh para minimizá-las foi o núcleo do trabalho — e o ponto mais desafiador, por envolver muitas variáveis simultaneamente.
- **Organização dos circuitos lógicos** — Com 4 bits de estado, um latch externo, decodificadores e 6 saídas de LED, a gestão dos fios no simulador rapidamente se tornava caótica. A decomposição em subcircuitos (`clk slow`, `decoder_sem`, `latch S-R`, `Reset btn`) foi fundamental para manter a clareza.
- **Sincronização com o botão de pedestre** — Garantir que o pedido do pedestre não causasse estados inválidos ou transições abruptas exigiu o uso do latch S-R para memorizar o pedido e da lógica de reset para liberá-lo apenas quando a FSM atingisse o estado correto.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 📄 Licença

O código-fonte deste projeto está distribuído sob a licença **MIT**. Consulte o arquivo `LICENSE` para mais informações.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 📬 Contato

giutp — [github.com/giutp](https://github.com/giutp)

Link do projeto: [https://github.com/giutp/circuitos-digitais](https://github.com/giutp/circuitos-digitais)

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 🙏 Agradecimentos

* [hneemann/Digital](https://github.com/hneemann/Digital) — simulador de circuitos lógicos utilizado
* [Best-README-Template](https://github.com/othneildrew/Best-README-Template) — template base deste README

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

<!-- MARKDOWN LINKS & IMAGES -->
[stars-shield]: https://img.shields.io/github/stars/giutp/circuitos-digitais.svg?style=for-the-badge
[stars-url]: https://github.com/giutp/circuitos-digitais/stargazers
[issues-shield]: https://img.shields.io/github/issues/giutp/circuitos-digitais.svg?style=for-the-badge
[issues-url]: https://github.com/giutp/circuitos-digitais/issues
[license-shield]: https://img.shields.io/github/license/giutp/circuitos-digitais.svg?style=for-the-badge
[license-url]: https://github.com/giutp/circuitos-digitais/blob/main/LICENSE
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/giutp/
[Digital-badge]: https://img.shields.io/badge/Digital-Simulator-blue?style=for-the-badge&logo=electron&logoColor=white
[Digital-url]: https://github.com/hneemann/Digital
