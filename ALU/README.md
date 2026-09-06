<a id="readme-top"></a>

[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<br />
<div align="center">
  <h3 align="center">⚙️ ALU — Unidade Lógico-Aritmética</h3>

  <p align="center">
    Implementação de uma ALU de 6 bits em portas lógicas, capaz de somar, subtrair, multiplicar e comparar — trabalho da disciplina Circuitos Digitais (CI1068) na UFPR.
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
    <li><a href="#-operações-e-fluxo-de-dados">Operações e Fluxo de Dados</a></li>
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

**ALU** é uma Unidade Lógico-Aritmética de **6 bits** construída inteiramente com portas lógicas primitivas no simulador [Digital](https://github.com/hneemann/Digital), desenvolvida para a disciplina **Circuitos Digitais (CI1068)** da **Universidade Federal do Paraná (UFPR)**.

O objetivo foi projetar um circuito capaz de realizar quatro operações sobre dois operandos de 6 bits (A e B), selecionadas por um seletor de 2 bits (O1, O0): **soma**, **subtração**, **multiplicação** (4 bits efetivos) e **comparação de igualdade**. O circuito foi construído de forma hierárquica, compondo subcircuitos menores e reutilizáveis — desde meio-somadores (HA) até o multiplicador completo.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

### 🛠 Construído com

* [![Digital][Digital-badge]][Digital-url]

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## ⏱ Arquitetura

A ALU é puramente **combinacional** (sem elementos de memória ou clock). Os operandos A e B são apresentados em paralelo como entradas individuais bit a bit, e a saída é determinada pelo seletor de operação:

```
          A[5:0]   B[5:0]   O[1:0]
             |        |        |
      +-------+--------+--------+-------+
      |                                 |
      |   Adder   Subtractor  Mux       |
      |   6-bit   6-bit    (Op select)  |
      |      \       /    Multiplier    |
      |       \     /     Comparator    |
      |        v   v          |         |
      |       Circuito.dig    |         |
      |              \        |         |
      +---------------+-------+---------+
                      |
                 S[7:0]  +  F (overflow/flag)
```

A seleção de operação é feita por uma rede de **multiplexadores** de portas `AND`/`OR` que roteiam a saída do bloco ativo para os bits de resultado S[7:0] e o flag F.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 👥 Componentes do Circuito

| Arquivo | Entradas | Saídas | Descrição |
|---------|----------|--------|-----------|
| **`HA.dig`** | A, B | S, Co | Meio-somador (*Half Adder*): XOR para soma, AND para carry. |
| **`FA.dig`** | A, B, Ci | S, Co | Somador completo (*Full Adder*): XOR + AND + OR para carry propagado. |
| **`Adder's 6 bits.dig`** | A[5:0], B[5:0] | S[5:0], F | Somador ripple-carry de 6 bits composto de HA + FAs. F indica overflow. |
| **`2's complement.dig`** | n[5:0] | c[5:0] | Complemento de dois: inverte bits (XOR) e some 1 via OR/AND em cascata. |
| **`Subtractor's 6 bits.dig`** | A[5:0], B[5:0] | S[5:0], F | Subtrator: aplica complemento de dois em B e usa o somador. |
| **`HM.dig`** | A, B | P | Meio-multiplicador: AND entre dois bits (produto parcial 1-bit). |
| **`FM.dig`** | A[3:0], B | p[3:0] | Linha de produto: multiplica 4 bits de A por 1 bit de B via HMs. |
| **`Multiplier's 4 bits.dig`** | A[5:0], B[5:0] | S[7:0], F | Multiplicador 4×4 bits usando FAs, HAs e demux para somar produtos parciais. |
| **`Comparator.dig`** | A, B | S | Comparador 1-bit de igualdade via XNOR. |
| **`Comparator's 6 bits.dig`** | A[5:0], B[5:0] | S | Comparador 6-bit: AND de 6 comparadores de 1 bit. S=1 se A==B. |
| **`Demux.dig`** | E, O | S0, S1 | Demultiplexador 1:2 com AND/NOT. |
| **`Mux.dig`** | E[3:0], O[1:0] | S | Multiplexador de saída para 4 operações (AND + OR por seletor). |
| **`Mux without Comparator.dig`** | E[1:0], E3, O[1:0] | S | Mux para bits que não incluem a saída do comparador. |
| **`Mux only E3 (multiplier).dig`** | O[1:0], E3 | S | Mux exclusivo para roteamento do bit extra do multiplicador. |
| **`Circuito.dig`** | A[5:0], B[5:0], O[1:0] | S[7:0], F | Circuito principal: integra todos os blocos e roteamento de MUX. |
| **`main.dig`** | A[5:0], B[5:0], O[1:0] | S[7:0], F | Toplevel da simulação com entradas e saídas visíveis. |

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 🔄 Operações e Fluxo de Dados

| Seletor (O1 O0) | Operação | Bloco Ativo | Resultado |
|-----------------|----------|-------------|-----------|
| `0 0` | **Soma** (A + B) | `Adder's 6 bits.dig` | S[5:0] com F = carry out (overflow) |
| `0 1` | **Subtração** (A − B) | `Subtractor's 6 bits.dig` | S[5:0] com F = borrow/overflow |
| `1 0` | **Multiplicação** (A × B) | `Multiplier's 4 bits.dig` | S[7:0] com F = overflow de produto |
| `1 1` | **Comparação** (A == B?) | `Comparator's 6 bits.dig` | S[0] = 1 se igual, 0 caso contrário |

> O roteamento é feito pelos multiplexadores `Mux.dig` e `Mux without Comparator.dig`: cada seletor habilita via AND apenas o bloco correspondente, e os resultados são combinados por OR na saída.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 📁 Estrutura do Projeto

```
ALU/
├── Circuitos/
│   ├── main.dig                            toplevel da simulação
│   ├── Circuito.dig                        integrador principal da ALU
│   ├── Adder's 6 bits.dig                  somador ripple-carry de 6 bits
│   ├── Subtractor's 6 bits.dig             subtrator (complemento de dois + somador)
│   ├── Multiplier's 4 bits.dig             multiplicador 4x4 bits por soma de parciais
│   ├── Comparator's 6 bits.dig             comparador de igualdade de 6 bits
│   ├── 2's complement.dig                  conversor para complemento de dois
│   ├── FA.dig                              full adder (1 bit)
│   ├── HA.dig                              half adder (1 bit)
│   ├── FM.dig                              linha de produto do multiplicador
│   ├── HM.dig                              half multiplier (produto 1-bit)
│   ├── HA B and Cin (for multiplier).dig   half adder variante para o multiplicador
│   ├── Mux.dig                             mux 4:1 de seleção de operação
│   ├── Mux without Comparator.dig          mux sem entrada do comparador
│   ├── Mux only E3 (multiplier).dig        mux para bits extras do multiplicador
│   ├── Demux.dig                           demultiplexador 1:2
│   ├── Comparator.dig                      comparador de igualdade 1 bit (XNOR)
│   ├── Only A (for Multiplier).dig         buffer passthrough para A
│   ├── Only B (for Multiplier).dig         buffer passthrough para B
│   └── Only Cin (for Multiplier).dig       buffer passthrough para Cin
├── Demonstrações/
│   └── Funcionamento e demonstrações.docx
└── README.md
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
   cd circuitos-digitais/ALU/Circuitos
   ```
2. Abra o simulador Digital:
   ```sh
   java -jar /caminho/para/Digital.jar
   ```
3. Abra o arquivo `main.dig` pelo menu **File → Open**.
4. Clique em **▶ Start** (ou `F5`) para iniciar a simulação.
5. Ajuste as entradas A[5:0], B[5:0] e o seletor O[1:0] para testar as operações.

<p align="right">(<a href="#readme-top">voltar ao topo</a>)</p>

---

## 📚 Dificuldades e Aprendizados

- **Entender tabelas-verdade e simplificações** — O processo de derivar expressões booleanas mínimas para cada operação exigiu bastante prática com mapas de Karnaugh e álgebra booleana até a simplificação fazer sentido intuitivo.
- **Organização dos circuitos lógicos** — Com a quantidade de fios e componentes crescendo rapidamente (especialmente no multiplicador), manter o diagrama legível e sem cruzamentos foi um desafio constante. A estratégia de hierarquizar em subcircuitos (`HA`, `FA`, `FM`…) foi essencial.
- **Multiplicador por soma de produtos parciais** — Implementar a multiplicação sem operadores aritméticos de alto nível, usando apenas HAs e FAs para acumular produtos parciais bit a bit, foi a parte mais complexa do projeto.

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
