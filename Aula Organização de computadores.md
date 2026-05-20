# Organização Estruturada de Computadores — Tanenbaum
## Capítulo 2: Organização dos Sistemas de Computadores
**Anotações de Aula — 3 horas**

---

## Sumário

1. [Visão Geral do Computador](#1-visão-geral-do-computador)
2. [Arquitetura de Von Neumann](#2-arquitetura-de-von-neumann)
3. [CPU — Unidade Central de Processamento](#3-cpu--unidade-central-de-processamento)
4. [Clock](#4-clock)
5. [Ciclo de Instrução](#5-ciclo-de-instrução)
6. [Paralelismo](#6-paralelismo)
7. [Memória e Cache](#7-memória-e-cache)
8. [RAM e Memória Virtual](#8-ram-e-memória-virtual)
9. [Entrada e Saída (E/S)](#9-entrada-e-saída-es)
10. [DMA e Interrupções](#10-dma-e-interrupções)
11. [Barramentos](#11-barramentos)
12. [Armazenamento](#12-armazenamento)
13. [Caminho Completo dos Dados](#13-caminho-completo-dos-dados)

---

## 1. Visão Geral do Computador

### O que é um computador?

Um computador é uma máquina que executa instruções armazenadas na memória, operando sobre dados para produzir resultados. Ao contrário de máquinas específicas (como uma calculadora de bolso), o computador é **de propósito geral**: o mesmo hardware pode executar um editor de texto, um jogo ou um sistema de controle de aviões — dependendo apenas do software carregado.

### Principais subsistemas

Todo computador moderno é composto por quatro grandes subsistemas que interagem entre si:

| Subsistema | Função |
|---|---|
| **CPU (Processador)** | Busca instruções, decodifica e executa operações |
| **Memória** | Armazena instruções e dados que a CPU usa durante a execução |
| **E/S (Entrada/Saída)** | Interface com o mundo externo: teclado, tela, disco, rede |
| **Barramento** | Via de comunicação que conecta todos os subsistemas |

### Como eles se relacionam?

```
┌─────────────┐       Barramento do Sistema        ┌─────────────┐
│     CPU     │◄──────────────────────────────────►│   Memória   │
│  (processa) │                                     │ (armazena)  │
└──────┬──────┘                                     └─────────────┘
       │
       │ Barramento de E/S
       ▼
┌─────────────────────────────────────────┐
│  Controladores de E/S                   │
│  (disco, teclado, tela, rede, USB...)   │
└─────────────────────────────────────────┘
```

### Níveis de abstração

Tanenbaum descreve o computador como uma **máquina multinível**: cada nível esconde os detalhes do nível inferior e oferece uma interface mais simples ao nível superior.

```
Nível 5: Linguagem de alto nível (C, Java, Python)
Nível 4: Linguagem de montagem (Assembly)
Nível 3: Sistema operacional
Nível 2: Arquitetura do conjunto de instruções (ISA)
Nível 1: Microprogramação / microarquitetura
Nível 0: Lógica digital (transistores, portas lógicas)
```

> **Analogia:** É como uma empresa: o diretor (nível alto) dá ordens em linguagem humana, que são traduzidas por gerentes e supervisores até chegar ao operário (transistor) que realiza a ação física.

---

## 2. Arquitetura de Von Neumann

### Contexto histórico

Em 1945, John von Neumann publicou um relatório descrevendo a arquitetura que se tornaria o padrão para praticamente todos os computadores construídos desde então. Antes disso, computadores como o ENIAC eram programados fisicamente — mudando cabos e interruptores.

A grande inovação foi o **programa armazenado**: instruções e dados residem na **mesma memória**, e a CPU os busca sequencialmente para executar.

### Os cinco componentes de Von Neumann

```
         ┌──────────────────────────────────────┐
         │           Memória Principal           │
         │   [ Instruções | Dados ]             │
         └───────────────┬──────────────────────┘
                         │
         ┌───────────────▼──────────────────────┐
         │           Unidade de Controle         │
         │   (busca, decodifica, coordena)       │
         └───────────────┬──────────────────────┘
                         │
    ┌────────────────────▼───────────────────────┐
    │         Unidade Lógica e Aritmética (ULA)  │
    │         (+, -, *, /, AND, OR, NOT, ...)    │
    └────────────────────┬───────────────────────┘
                         │
         ┌───────────────▼──────────────────────┐
         │         Dispositivos de E/S           │
         │   (entrada: teclado) (saída: tela)   │
         └──────────────────────────────────────┘
```

### Características fundamentais

- **Memória única e endereçável:** instruções e dados ocupam o mesmo espaço de memória, distinguidos apenas pelo contexto de uso.
- **Execução sequencial:** o contador de programa (PC) avança instrução por instrução, exceto em desvios e chamadas de função.
- **Representação binária:** todos os dados — números, texto, imagens, instruções — são armazenados como sequências de 0s e 1s.
- **Barramento compartilhado:** CPU e memória se comunicam pelo mesmo canal.

### O gargalo de Von Neumann

Uma limitação clássica da arquitetura: a CPU e a memória competem pelo **mesmo barramento** para buscar instruções e dados. Isso cria um gargalo, pois a CPU é muito mais rápida que a memória.

**Soluções modernas para o gargalo:**
- Memória cache (aproxima dados rápidos da CPU)
- Memória de largura de banda alta (GDDR, HBM)
- Arquitetura Harvard modificada (caches separadas de instrução e dado)

### Arquitetura Harvard

Uma variação da Von Neumann que usa **memórias e barramentos separados** para instruções e dados. Comum em microcontroladores (Arduino, AVR, PIC) e internamente nos processadores modernos para os caches L1.

| | Von Neumann | Harvard |
|---|---|---|
| Memória | Única | Separada (instrução / dado) |
| Barramento | Compartilhado | Duplo |
| Complexidade | Menor | Maior |
| Uso | Computadores gerais | Microcontroladores, DSPs |

---

## 3. CPU — Unidade Central de Processamento

### Visão geral

A CPU é o componente que efetivamente **executa** as instruções de um programa. Ela busca cada instrução da memória, interpreta o que ela significa e realiza a operação correspondente — seja um cálculo, uma comparação, um desvio ou uma movimentação de dados.

### Componentes internos da CPU

#### Unidade de Controle (UC)

Coordena todas as operações da CPU. Ela:
- Busca a próxima instrução da memória (usando o PC)
- Decodifica o código binário da instrução
- Gera os sinais de controle para os demais componentes
- Controla o fluxo de dados pelo barramento interno da CPU

#### Unidade Lógica e Aritmética (ULA)

Realiza todas as operações computacionais:
- **Aritméticas:** adição, subtração, multiplicação, divisão
- **Lógicas:** AND, OR, NOT, XOR
- **Comparações:** igual, maior que, menor que
- **Deslocamento de bits:** shift left, shift right

A ULA opera sobre **operandos** fornecidos pelos registradores e devolve o resultado para um registrador de destino.

#### Registradores

São memórias **dentro da própria CPU** — os mais rápidos do sistema (acesso em frações de nanossegundo). Cada registrador armazena um único valor (tipicamente 32 ou 64 bits).

| Registrador | Função |
|---|---|
| **PC (Program Counter)** | Endereço da próxima instrução a buscar |
| **IR (Instruction Register)** | Armazena a instrução atual em execução |
| **SP (Stack Pointer)** | Aponta para o topo da pilha de chamadas |
| **MAR (Memory Address Register)** | Endereço de memória a ser acessado |
| **MDR (Memory Data Register)** | Dado lido ou a ser escrito na memória |
| **PSW (Program Status Word)** | Flags de estado: zero, negativo, overflow, carry |
| **Registradores gerais** | R0–R31 (RISC) ou EAX, EBX, ECX, EDX (x86) — uso geral |

#### Flags (PSW)

Após operações na ULA, bits de estado são atualizados:
- **Zero (Z):** resultado foi zero
- **Negativo (N):** resultado foi negativo
- **Carry (C):** houve transporte no bit mais significativo
- **Overflow (V):** resultado excedeu a capacidade do registrador

Esses flags são usados pelas instruções de desvio condicional (`BEQ`, `BNE`, `BGT`...).

### Conjunto de Instruções (ISA)

A ISA (Instruction Set Architecture) é o contrato entre o hardware e o software: define quais instruções o processador entende, seus formatos binários e seus efeitos.

#### Categorias de instruções

- **Transferência de dados:** `LOAD`, `STORE`, `MOV` — movem dados entre registradores e memória
- **Aritméticas/lógicas:** `ADD`, `SUB`, `MUL`, `AND`, `OR`, `NOT`
- **Desvio:** `JMP`, `BEQ`, `BNE` — alteram o PC (condicionais ou incondicionais)
- **Chamada de sub-rotina:** `CALL`, `RET` — salvam/restauram o PC na pilha
- **E/S:** `IN`, `OUT` — transferem dados de/para dispositivos

#### RISC vs CISC

**RISC (Reduced Instruction Set Computer)**
- Poucas instruções, todas simples e de tamanho fixo
- Cada instrução executa em exatamente 1 ciclo de clock
- Operações sobre dados acontecem apenas em registradores; memória só é acessada por LOAD/STORE
- Compiladores são mais complexos mas o hardware é simples e rápido
- Exemplos: ARM (celulares), MIPS, RISC-V, PowerPC

**CISC (Complex Instruction Set Computer)**
- Muitas instruções, algumas muito complexas e de tamanho variável
- Uma única instrução pode acessar memória e fazer aritmética ao mesmo tempo
- Hardware mais complexo (microcódigo para traduzir instruções complexas)
- Código gerado pelo compilador é mais compacto
- Exemplo principal: x86 / x86-64 (Intel e AMD)

| Característica | RISC | CISC |
|---|---|---|
| Tamanho das instruções | Fixo (ex.: 32 bits) | Variável (1–15 bytes no x86) |
| Ciclos por instrução | 1 (ideal) | Vários |
| Acesso à memória | Apenas LOAD/STORE | Qualquer instrução |
| Nº de registradores | Muitos (32+) | Poucos (8 no x86 de 32 bits) |
| Exemplos | ARM, MIPS, RISC-V | x86, x86-64 |

> **Curiosidade:** Os processadores x86 modernos (Intel Core, AMD Ryzen) são internamente RISC. A unidade de decodificação traduz as instruções x86 (CISC) em micro-ops internas (RISC) antes da execução. O melhor dos dois mundos.

---

## 4. Clock

### O que é o clock?

O clock é o **sinal de sincronismo** do processador — um oscilador de cristal de quartzo que gera pulsos elétricos regulares, alternando entre 0 e 1 em frequência constante.

```
        _   _   _   _   _   _
       | | | | | | | | | | | |
  _____|_| |_| |_| |_| |_| |_|_____

  ◄──────────────────────────────►
               tempo
```

Cada pulso completo (0 → 1 → 0) é chamado de **ciclo de clock**. A frequência indica quantos ciclos ocorrem por segundo:

- **1 MHz** = 1.000.000 ciclos/segundo
- **1 GHz** = 1.000.000.000 ciclos/segundo
- Processadores modernos: 3–6 GHz

### O que acontece em cada ciclo?

Cada ciclo de clock é a menor unidade de tempo em que o processador realiza uma operação elementar: ler um registrador, enviar um valor para a ULA, escrever um resultado. Operações mais complexas levam **múltiplos ciclos**.

### Frequência não é tudo

Dobrar o clock **não** dobra necessariamente o desempenho, porque:

- Instruções diferentes levam números diferentes de ciclos (CPI — Cycles Per Instruction)
- A memória pode não conseguir fornecer dados na velocidade do processador
- O pipeline pode ser interrompido por dependências entre instruções

**Fórmula do desempenho:**

```
Tempo de execução = Nº de instruções × CPI × Período do clock

                  = Nº de instruções × CPI
                    ──────────────────────
                        Frequência do clock
```

### Overclocking e limites térmicos

Aumentar artificialmente a frequência do clock além do especificado chama-se **overclocking**. O problema: a dissipação de calor cresce rapidamente com a frequência (aproximadamente proporcional ao cubo da frequência em algumas situações). Por isso, os chips modernos têm limites térmicos rígidos.

### Clock e sincronismo no barramento

Diferentes partes do computador operam em diferentes frequências:
- CPU: 3–6 GHz
- Cache L3 / barramento interno: ~1–2 GHz
- RAM DDR5: ~3–6 GHz (mas com latência maior)
- PCIe: ~16 GHz por lane

Circuitos divisores e multiplicadores de clock sincronizam essas partes.

---

## 5. Ciclo de Instrução

### Definição

O ciclo de instrução é a sequência de passos que a CPU repete continuamente desde que é ligada até ser desligada. Para cada instrução do programa, a CPU realiza este ciclo completo.

### As fases do ciclo

#### Fase 1 — Busca (Fetch)

A CPU usa o endereço contido no **PC** para ler a próxima instrução da memória:

1. O valor do PC é copiado para o MAR
2. A memória é ativada para leitura no endereço do MAR
3. O dado retorna pelo MDR e é copiado para o IR
4. O PC é incrementado (aponta para a próxima instrução)

```
PC → MAR → [Memória] → MDR → IR
PC = PC + tamanho_da_instrução
```

#### Fase 2 — Decodificação (Decode)

A Unidade de Controle interpreta o padrão de bits no IR:
- Identifica o **opcode** (código da operação — ex.: ADD, LOAD, JMP)
- Identifica os **operandos** (registradores ou endereços envolvidos)
- Gera os sinais de controle para as próximas fases

#### Fase 3 — Busca de operandos (Operand Fetch)

Se a instrução precisa de dados que estão na memória (e não apenas em registradores), eles são lidos agora. Esta fase pode ser omitida em instruções que operam só em registradores.

#### Fase 4 — Execução (Execute)

A ULA (ou outra unidade funcional) realiza a operação:
- Aritmética: `R1 = R2 + R3`
- Lógica: `R1 = R2 AND R3`
- Desvio: atualiza o PC se a condição for verdadeira

#### Fase 5 — Escrita do resultado (Write-back)

O resultado da operação é gravado no registrador destino ou na memória. Alguns processadores combinam esta fase com a de execução.

### Diagrama completo

```
         ┌──────────┐
   ┌────►│  BUSCA   │  PC → MAR → Memória → MDR → IR; PC++
   │     └────┬─────┘
   │          ▼
   │     ┌──────────────┐
   │     │ DECODIFICAÇÃO│  UC interpreta o IR
   │     └──────┬───────┘
   │            ▼
   │     ┌──────────────────┐
   │     │ BUSCA OPERANDOS  │  (se necessário: lê da memória)
   │     └──────┬───────────┘
   │            ▼
   │     ┌──────────┐
   │     │ EXECUÇÃO │  ULA opera; desvios atualizam PC
   │     └────┬─────┘
   │          ▼
   │     ┌──────────────┐
   └─────│  WRITE-BACK  │  resultado → registrador ou memória
         └──────────────┘
```

### Exemplo concreto: `ADD R1, R2, R3`

| Fase | Ação |
|---|---|
| Busca | Lê a instrução `ADD R1, R2, R3` da memória via PC |
| Decodificação | UC identifica: opcode=ADD, dest=R1, src1=R2, src2=R3 |
| Operandos | Lê os valores de R2 e R3 (já nos registradores, fase rápida) |
| Execução | ULA calcula R2 + R3 |
| Write-back | Resultado gravado em R1 |

### Impacto de desvios condicionais

Instruções de desvio (`if`, `while`, `for`) fazem o PC pular para outro endereço. Isso é especialmente problemático para o pipeline, pois o processador precisa "adivinhar" o destino — técnica chamada **predição de desvio**.

---

## 6. Paralelismo

### Por que paralelismo?

O clock físico tem limites: aumentar a frequência gera calor excessivo e consome energia demais. A solução para continuar melhorando o desempenho foi explorar o paralelismo — executar múltiplas operações ao mesmo tempo.

### 6.1 Pipelining

O pipeline divide a execução de uma instrução em estágios independentes e processa múltiplas instruções simultaneamente — como uma linha de montagem industrial.

**Sem pipeline:** uma instrução por vez

```
Tempo:    1  2  3  4  5  6  7  8  9  10  11  12
Instr 1: [F][D][E][W]
Instr 2:              [F][D][E][W]
Instr 3:                           [F][D][E][W]
                                                   ← 12 ciclos para 3 instruções
```

**Com pipeline de 4 estágios:** (F=Fetch, D=Decode, E=Execute, W=Write-back)

```
Tempo:    1  2  3  4  5  6  7
Instr 1: [F][D][E][W]
Instr 2:    [F][D][E][W]
Instr 3:       [F][D][E][W]
                              ← 7 ciclos para 3 instruções (ganho de ~1,7×)
```

Em regime estacionário (pipeline cheio), **uma instrução é concluída por ciclo**, independente do número de estágios.

#### Hazards (conflitos) no pipeline

**Hazard de dados:** uma instrução precisa do resultado de uma instrução anterior que ainda não terminou.

```asm
ADD R1, R2, R3    ; calcula R1 = R2 + R3
SUB R4, R1, R5    ; precisa de R1, que ainda está sendo calculado!
```

Soluções: **stall** (inserir bolhas/NOPs) ou **forwarding** (encaminhar o resultado diretamente entre estágios).

**Hazard de controle:** instrução de desvio condicional — o PC correto só é conhecido após a execução do desvio, mas o pipeline já buscou instruções adiante.

Solução: **predição de desvio** (branch prediction). Processadores modernos acertam 95–99% das previsões usando histórico de execuções anteriores.

**Hazard estrutural:** dois estágios tentam usar o mesmo recurso ao mesmo tempo (ex.: memória única para busca de instrução e acesso a dados). Solução: caches separadas de instrução e dado (arquitetura Harvard interna).

### 6.2 Superescalar

Um processador superescalar possui **múltiplas ULAs e unidades funcionais** e pode emitir mais de uma instrução por ciclo, desde que não haja dependências entre elas.

```
Ciclo 5:  [ADD R1, R2, R3]  ←── executam ao mesmo tempo
           [MUL R4, R5, R6]  ←── se não há dependência entre elas
```

O hardware analisa dinamicamente as instruções em busca de independências — isso é chamado de **execução fora de ordem (out-of-order execution)**.

### 6.3 Execução Fora de Ordem (OoO)

Em vez de executar instruções estritamente na ordem do programa, a CPU reordena para aproveitar unidades funcionais livres:

```asm
; Ordem original no programa:
LOAD R1, [mem]    ; lento — vai buscar na memória
ADD  R2, R1, R3  ; depende de R1 — precisa esperar
MUL  R4, R5, R6  ; independente! pode executar enquanto LOAD acontece
```

A CPU executa `MUL` enquanto espera o `LOAD`, mas **apresenta os resultados na ordem correta** ao programador (commit em ordem).

### 6.4 Multithreading

Enquanto uma thread espera dados da memória (cache miss → acesso à RAM), a CPU pode alternar para outra thread e aproveitar os ciclos ociosos.

**Multithreading por bloqueio (coarse-grained):** troca de thread quando há uma espera longa (ex.: acesso à RAM).

**Multithreading simultâneo (SMT / Hyper-Threading da Intel):** múltiplas threads são intercaladas no mesmo ciclo. Um core físico aparece como 2 cores lógicos para o sistema operacional.

```
Core físico com SMT:
Ciclo:  1    2    3    4    5    6
Thread A: [F] [--] [E] [--] [F] [E]
Thread B: [--] [F] [--][E] [--] [F]
```

### 6.5 Multicore

Múltiplos núcleos completos e independentes dentro de um único chip. Cada core possui suas próprias unidades de controle, ULA e caches L1/L2. Compartilham a cache L3 e o acesso à RAM.

```
┌─────────────────────────────────────────┐
│                  Chip                   │
│  ┌──────────┐  ┌──────────┐            │
│  │  Core 0  │  │  Core 1  │  ...       │
│  │ L1I | L1D│  │ L1I | L1D│            │
│  │    L2    │  │    L2    │            │
│  └────┬─────┘  └────┬─────┘            │
│       └──────┬───────┘                 │
│           L3 Cache (compartilhada)      │
│              │                          │
│         Controlador de memória          │
└─────────────────────────────────────────┘
```

**Diferença importante:** multicore e multithreading (SMT) são ortogonais — um processador pode ter os dois. Um Intel Core i9 com 8 cores e Hyper-Threading apresenta 16 threads lógicas ao sistema operacional.

### 6.6 GPU e paralelismo massivo

GPUs (Graphics Processing Units) levam o paralelismo ao extremo: possuem milhares de núcleos simples operando em paralelo. São ideais para tarefas altamente paralelas (renderização gráfica, redes neurais, criptografia), onde o mesmo cálculo é aplicado a milhões de dados independentes (**SIMD — Single Instruction, Multiple Data**).

---

## 7. Memória e Cache

### 7.1 A Hierarquia de Memória

Não existe uma tecnologia de memória que seja simultaneamente **rápida, grande e barata**. Por isso, os computadores usam múltiplos níveis de memória, cada um com diferentes compromissos:

| Nível | Tipo | Velocidade | Capacidade típica | Custo relativo |
|---|---|---|---|---|
| 0 | Registradores | ~0,3 ns | Dezenas de bytes | Altíssimo |
| 1 | Cache L1 | ~1 ns | 32–64 KB | Muito alto |
| 2 | Cache L2 | ~3–5 ns | 256 KB – 1 MB | Alto |
| 3 | Cache L3 | ~10–20 ns | 4–64 MB | Moderado |
| 4 | RAM (DRAM) | ~60–100 ns | 4 GB – 512 GB | Baixo |
| 5 | SSD (NVMe) | ~100 µs | 256 GB – 4 TB | Muito baixo |
| 6 | HD (magnético) | ~5–10 ms | 500 GB – 20 TB | Baixíssimo |
| 7 | Fita magnética | Segundos | Centenas de TB | Mínimo |

> **Princípio chave:** quanto mais próximo da CPU, mais rápido e menor. O sistema operacional e o hardware gerenciam automaticamente a movimentação de dados entre os níveis.

### 7.2 Memória Cache

A cache é uma memória intermediária que guarda cópias de dados da RAM que foram usados recentemente, evitando que a CPU espere os ~100 ns de latência da RAM para cada acesso.

#### Por que a cache funciona? Princípios de localidade

**Localidade temporal:** se um dado foi acessado agora, provavelmente será acessado novamente em breve.

```c
int soma = 0;
for (int i = 0; i < 1000; i++) {
    soma += vetor[i];   // 'soma' é acessada 1000 vezes → localidade temporal
}
```

**Localidade espacial:** se um dado foi acessado, os dados vizinhos na memória também serão acessados em breve.

```c
for (int i = 0; i < 1000; i++) {
    soma += vetor[i];   // vetor[0], vetor[1], vetor[2]... são contíguos → localidade espacial
}
```

Por isso, quando ocorre um **cache miss**, a CPU não busca apenas o byte solicitado — ela busca um **bloco inteiro (linha de cache)** de 64 bytes, antecipando os acessos futuros.

#### Funcionamento básico

```
CPU solicita dado no endereço X
        │
        ▼
    Cache Hit? ──── SIM ────► Retorna dado em ~1–5 ns ✓
        │
       NÃO
        │
        ▼
Busca linha de 64 B na RAM (~60–100 ns)
        │
        ▼
Carrega linha na cache + retorna dado à CPU
```

#### Taxa de acerto e taxa de falha

**Taxa de acerto (hit rate):** fração dos acessos atendidos pela cache. Caches bem projetadas atingem 95–99% de hit rate.

**Taxa de falha (miss rate):** = 1 − hit rate

**Tempo médio de acesso:**

```
T_médio = T_hit + miss_rate × T_penalidade

Exemplo: T_hit = 1 ns, miss_rate = 5%, T_penalidade = 100 ns
T_médio = 1 + 0,05 × 100 = 1 + 5 = 6 ns
```

#### Organização da cache: mapeamento

**Cache de mapeamento direto:** cada bloco de RAM mapeia para exatamente uma linha de cache. Simples, mas pode causar conflitos se dois blocos frequentes mapearem para a mesma linha.

**Cache totalmente associativa:** qualquer bloco pode ir para qualquer linha. Flexível, mas hardware complexo (compara todos os tags em paralelo).

**Cache associativa por conjuntos (set-associative):** compromisso entre os dois. A cache é dividida em conjuntos; cada bloco mapeia para um conjunto específico, mas pode ocupar qualquer linha dentro desse conjunto. Padrão atual: 8-way set-associative.

#### Políticas de substituição

Quando a cache está cheia e um novo bloco precisa entrar, qual linha expulsar?

- **LRU (Least Recently Used):** expulsa a linha menos recentemente usada. Mais eficiente na prática; difícil de implementar para associatividade alta.
- **FIFO:** expulsa a linha mais antiga. Simples, mas ignora padrões de uso.
- **LFU (Least Frequently Used):** expulsa a linha menos acessada no total.
- **Aleatório (Random):** expulsa uma linha ao acaso. Surpreendentemente eficaz e fácil de implementar.
- **Pseudo-LRU:** aproximação do LRU com hardware mais simples, usada na prática.

#### Políticas de escrita

**Write-through:** a escrita vai simultaneamente para a cache e para a RAM.
- Vantagem: a RAM sempre está atualizada (consistência simples)
- Desvantagem: cada escrita gera tráfego no barramento

**Write-back:** a escrita vai apenas para a cache. O bloco é marcado como "sujo" (dirty bit). Só é escrito na RAM quando a linha é substituída.
- Vantagem: muito mais rápido (múltiplas escritas para o mesmo bloco custam apenas 1 acesso à RAM)
- Desvantagem: complexidade adicional; risco de perda de dados em falha de energia

**Write-allocate:** numa falha de escrita, o bloco é carregado da RAM para a cache antes da escrita. Combinado com write-back.

**No-write-allocate:** numa falha de escrita, escreve diretamente na RAM sem carregar na cache. Combinado com write-through.

### 7.3 Coerência de Cache em Multicore

Em sistemas multicore, cada core tem sua própria cache L1/L2. Se dois cores têm cópias do mesmo dado e um deles modifica sua cópia, as outras ficam desatualizadas (**problema de coerência**).

**Protocolo MESI:** o mais usado. Cada linha de cache pode estar em 4 estados:

| Estado | Significado |
|---|---|
| **M (Modified)** | Dado modificado, só existe nesta cache (dirty) |
| **E (Exclusive)** | Dado não modificado, só existe nesta cache |
| **S (Shared)** | Dado não modificado, existe em múltiplas caches |
| **I (Invalid)** | Dado inválido (foi modificado por outro core) |

Quando um core modifica um dado, os outros cores são notificados via barramento e marcam sua cópia como **Invalid**.

---

## 8. RAM e Memória Virtual

### 8.1 SRAM vs DRAM

**SRAM (Static RAM)**
- Usa circuito flip-flop com 4–6 transistores por bit
- Mantém o dado enquanto houver energia, sem necessidade de refresh
- Muito rápida (~1 ns), muito cara, ocupa muito espaço no chip
- Usada em: registradores, caches L1, L2, L3

**DRAM (Dynamic RAM)**
- Usa 1 transistor + 1 capacitor por bit
- O capacitor descarrega naturalmente → precisa ser **refrescado** a cada ~64 ms
- Mais lenta (~60–100 ns), muito mais barata, alta densidade
- Usada em: memória principal (RAM do computador)

**DDR SDRAM (Double Data Rate Synchronous DRAM)**
- Variante síncrona (sincronizada com o clock do barramento)
- **Double Data Rate:** transfere dados na borda de **subida E descida** do clock, dobrando a taxa de transferência
- Evolução: DDR → DDR2 → DDR3 → DDR4 → DDR5
- DDR5 (2023): até ~6400 MT/s, largura de banda de ~50 GB/s

| Geração | Velocidade típica | Largura de banda (dual channel) |
|---|---|---|
| DDR3 | 1333–2133 MT/s | ~21–34 GB/s |
| DDR4 | 2133–3600 MT/s | ~34–57 GB/s |
| DDR5 | 4800–7200 MT/s | ~76–115 GB/s |

### 8.2 Organização da DRAM

A DRAM é organizada como uma **matriz de células** com linhas (rows) e colunas (columns):

1. **RAS (Row Address Strobe):** ativa uma linha inteira → carrega na buffer de linha
2. **CAS (Column Address Strobe):** seleciona a coluna desejada dentro da linha ativada

O tempo para ativar uma linha é chamado **latência RAS** (~10–15 ns). Acessos consecutivos à mesma linha são muito mais rápidos (apenas CAS).

### 8.3 Memória Virtual

#### O problema que ela resolve

Em sistemas multitarefa, vários programas precisam rodar ao mesmo tempo. Problemas:

1. **Isolamento:** um programa não pode acessar a memória de outro (segurança)
2. **Espaço insuficiente:** os programas juntos podem precisar de mais RAM do que existe fisicamente

A **memória virtual** resolve ambos: cada processo enxerga um espaço de endereçamento próprio e contíguo, mesmo que a memória física esteja fragmentada ou seja insuficiente.

#### Paginação

A memória (virtual e física) é dividida em blocos de tamanho fixo chamados **páginas** (tipicamente 4 KB).

- **Página virtual:** divisão do espaço de endereçamento do processo
- **Quadro de página (frame):** divisão da RAM física
- **Tabela de páginas:** estrutura mantida pelo SO que mapeia páginas virtuais → quadros físicos

```
Endereço virtual: [ número da página | deslocamento dentro da página ]
                        ↓ (tabela de páginas)
Endereço físico:  [ número do quadro  | deslocamento dentro da página ]
```

#### TLB (Translation Lookaside Buffer)

A tradução de endereços envolve consultar a tabela de páginas na memória — custoso. Para acelerar, os processadores possuem um **cache da tabela de páginas** chamado TLB:

- TLB hit: tradução em ~1 ciclo
- TLB miss: acessa a tabela de páginas na memória (~10–100 ciclos)

A TLB normalmente tem 64–1024 entradas e resolve 99%+ dos acessos.

#### Page Fault

Se a página solicitada **não está na RAM** (foi movida para o disco), ocorre um **page fault**:

1. A CPU gera uma exceção
2. O SO suspende o processo
3. O SO localiza a página no disco (arquivo de paginação / swap)
4. O SO copia a página do disco para um quadro livre na RAM
5. O SO atualiza a tabela de páginas e retoma o processo

Este processo leva **milissegundos** — extremamente lento comparado ao acesso normal.

#### Thrashing

Se há page faults frequentes demais — o SO passa mais tempo transferindo páginas entre disco e RAM do que executando processos —, o sistema entra em **thrashing**. Sintoma: computador extremamente lento, disco 100% ativo. Solução: adicionar RAM ou reduzir o número de processos ativos.

#### Segmentação

Alternativa (ou complemento) à paginação: divide a memória em **segmentos** de tamanho variável com significado semântico (segmento de código, de dados, de pilha). O x86 usa segmentação historicamente; sistemas modernos a usam apenas para proteção de memória.

---

## 9. Entrada e Saída (E/S)

### 9.1 Estrutura geral de E/S

Dispositivos de E/S são inerentemente mais lentos que a CPU e a memória. Para gerenciar essa diferença, cada dispositivo conta com um **controlador** — um chip dedicado que faz a ponte entre o dispositivo lento e o barramento rápido do sistema.

```
CPU ◄──── Barramento ────► Controlador ◄──► Dispositivo (disco, teclado, etc.)
                                │
                          Registradores:
                          - Status (estado atual)
                          - Controle (comandos)
                          - Dados (buffer)
```

A CPU comunica-se com o controlador lendo e escrevendo em seus registradores.

### 9.2 Como a CPU acessa registradores de dispositivos

**E/S mapeada em portas (Port-mapped I/O):** registradores do dispositivo têm endereços separados do espaço de memória. Acessados por instruções especiais (`IN`, `OUT` no x86). Espaço de endereços separado.

**E/S mapeada em memória (Memory-mapped I/O):** registradores do dispositivo ocupam endereços no espaço de memória normal. Acessados pelas mesmas instruções de `LOAD`/`STORE` usadas para RAM. Mais simples para o software; padrão em sistemas RISC (ARM).

### 9.3 Software de E/S: camadas

```
Aplicação (usuário)
     │   open(), read(), write()
     ▼
Sistema de arquivos (SO)
     │
     ▼
Driver do dispositivo (SO — específico por hardware)
     │
     ▼
Controlador do dispositivo (hardware)
     │
     ▼
Dispositivo físico (disco, teclado, NIC...)
```

Os **drivers** são programas que traduzem as chamadas genéricas do SO (ex.: "leia 512 bytes") em comandos específicos do hardware (ex.: séquencia de registradores a escrever no controlador SATA).

---

## 10. DMA e Interrupções

### 10.1 Técnicas de E/S

#### E/S Programada (Polling / Busy-wait)

A CPU executa um loop consultando repetidamente o registrador de status do controlador até que o dispositivo esteja pronto.

```c
// Pseudo-código de polling
while (controlador.status != PRONTO) { }  // CPU presa aqui
dados = controlador.dados;
```

- **Vantagem:** simples de implementar, latência mínima quando o dispositivo responde rapidamente
- **Desvantagem:** CPU 100% ocupada esperando; impossível executar outro processo enquanto espera

> **Analogia:** ficar olhando para o forno a cada segundo verificando se a pizza está pronta.

#### E/S por Interrupção

O controlador avisa a CPU quando termina, via um sinal elétrico chamado **interrupção (IRQ — Interrupt ReQuest)**. Enquanto isso, a CPU faz outro trabalho.

```
CPU inicia operação de E/S
         │
         ▼
CPU executa outros processos
         │
         │ ← IRQ chega (dispositivo terminou)
         ▼
CPU salva contexto (registradores, PC) na pilha
         │
         ▼
CPU executa ISR (Interrupt Service Routine / rotina de tratamento)
         │
         ▼
CPU restaura contexto e retoma o que estava fazendo
```

**Vetor de interrupções:** tabela na memória que mapeia cada número de IRQ para o endereço da ISR correspondente.

**Prioridades:** interrupções têm prioridades. Uma interrupção de alta prioridade pode interromper o tratamento de uma de baixa prioridade (**interrupção aninhada**).

**Tipos de interrupções:**
- **Hardware (externas):** geradas por dispositivos (disco terminou, tecla pressionada, pacote de rede chegou)
- **Software (traps):** geradas pelo próprio programa, intencionalmente (chamada de sistema) ou por erro (divisão por zero, acesso inválido à memória → **exceção**)
- **NMI (Non-Maskable Interrupt):** interrupção que não pode ser desabilitada; usada para falhas críticas de hardware

> **Analogia:** usar o temporizador do forno — você cozinha outras coisas e o alarme avisa quando a pizza está pronta.

### 10.2 DMA (Direct Memory Access)

Para transferências grandes (blocos do disco, frames de vídeo, pacotes de rede), usar interrupção a cada byte seria inviável. O **DMA** permite que o controlador transfira dados diretamente entre o dispositivo e a RAM, **sem envolver a CPU** a cada byte.

#### Como funciona o DMA

```
1. CPU configura o chip DMA:
   - Endereço de origem (dispositivo ou porta)
   - Endereço de destino na RAM
   - Quantidade de bytes a transferir
   - Direção (leitura ou escrita)

2. DMA assume o controle do barramento e realiza a transferência
   (CPU pode continuar executando com restrições de acesso ao barramento)

3. Ao terminar, DMA gera uma interrupção para avisar a CPU
```

**Ciclo-stealing:** o DMA "rouba" ciclos do barramento da CPU para transferir dados. A CPU ainda funciona, mas pode ser levemente retardada se precisar do barramento.

**Burst mode:** o DMA toma controle total do barramento por um burst completo — mais eficiente mas bloqueia a CPU por mais tempo.

| Técnica | CPU durante E/S | Latência | Uso típico |
|---|---|---|---|
| Polling | 100% ocupada esperando | Mínima | Dispositivos simples/rápidos |
| Interrupção | Livre, interrompida ao fim | Baixa | Teclado, mouse, rede |
| DMA | Livre (com ciclo-stealing) | Menor ainda | Disco, placa de vídeo, rede |

---

## 11. Barramentos

### 11.1 Conceito

Um barramento é um conjunto de condutores elétricos compartilhados que permite a comunicação entre múltiplos componentes. É, essencialmente, a "estrada" por onde trafegam dados, endereços e sinais de controle no computador.

### 11.2 Tipos de linha

**Barramento de endereços (address bus):**
- Unidirecional: CPU → Memória/Dispositivo
- Indica qual posição de memória ou dispositivo será acessado
- Largura determina o espaço de endereçamento: 32 bits → 4 GB, 64 bits → 16 EB

**Barramento de dados (data bus):**
- Bidirecional
- Transporta os dados propriamente ditos (leitura ou escrita)
- Largura típica: 64 bits (8 bytes por transferência)

**Barramento de controle (control bus):**
- Sinais como: Read/Write, Clock, Reset, IRQ, BUSREQ, BUSACK
- Coordena quem está usando o barramento e em que direção

### 11.3 Transferência no barramento: protocolo

**Barramento síncrono:** todas as transferências são sincronizadas por um clock global. Simples, mas o clock deve ser lento o suficiente para o componente mais lento.

**Barramento assíncrono:** usa handshaking (MSYN/SSYN) para sincronizar sem clock fixo. Mais flexível, permite dispositivos de velocidades diferentes.

**Handshaking assíncrono:**

```
Mestre coloca endereço → ativa MSYN
Escravo reconhece e coloca dado → ativa SSYN
Mestre lê dado → desativa MSYN
Escravo desativa SSYN
Mestre retira endereço
```

### 11.4 Arbitragem do barramento

Quando múltiplos dispositivos querem usar o barramento ao mesmo tempo, é preciso arbitrar quem tem prioridade.

**Arbitragem centralizada:** um árbitro (chip dedicado) decide quem pode usar o barramento. Simples, ponto único de falha.

**Daisy chain:** os dispositivos estão em série; o sinal de concessão passa de um para o próximo. Quem capturar primeiro ganha. Prioridade pelo posicionamento físico.

**Arbitragem distribuída:** cada dispositivo negocia com os outros via protocolo. Sem ponto único de falha; mais complexo.

### 11.5 Largura de banda do barramento

```
Largura de banda = Largura do barramento (bits) × Frequência / 8

Exemplo: DDR4 com barramento de 64 bits a 3200 MHz (DDR → × 2):
Banda = 64 × 3200 × 10⁶ × 2 / 8 = 51,2 GB/s
```

### 11.6 Barramentos modernos

**PCIe (PCI Express):**
- Substitui o barramento PCI compartilhado por **links ponto a ponto** dedicados
- Organizado em lanes (× 1, ×4, ×8, ×16); cada lane bidirecional
- PCIe 4.0 × 16: ~32 GB/s; PCIe 5.0 × 16: ~64 GB/s
- Usado por: GPU, SSD NVMe, placas de rede de alta velocidade

**USB (Universal Serial Bus):**
- Barramento de propósito geral para periféricos externos
- USB 2.0: 480 Mbps; USB 3.2: 20 Gbps; USB4: 40–80 Gbps
- Suporta hot-plug e fornecimento de energia (USB-PD: até 240 W)

**SATA (Serial ATA):**
- Interconexão interna para HDs e SSDs tradicionais
- SATA III: 6 Gbps (~550 MB/s real)
- Sendo substituído pelo NVMe via PCIe em SSDs de alto desempenho

**NVMe (Non-Volatile Memory Express):**
- Protocolo otimizado para SSDs sobre PCIe
- Latências de ~20–100 µs vs ~50–100 µs do SATA
- PCIe 4.0 NVMe: até ~7 GB/s

| Barramento | Velocidade máx. | Uso |
|---|---|---|
| SATA III | ~550 MB/s | HDs, SSDs SATA |
| USB 3.2 Gen 2×2 | ~2,5 GB/s | Periféricos externos |
| PCIe 4.0 ×4 (NVMe) | ~7 GB/s | SSDs NVMe |
| PCIe 5.0 ×16 | ~64 GB/s | GPUs de alto desempenho |
| DDR5 (dual channel) | ~100 GB/s | RAM principal |

---

## 12. Armazenamento

### 12.1 Discos Magnéticos (HDs)

#### Estrutura física

Um HD contém vários **pratos** (platters) de material magnético girando em alta velocidade. Cabeças de leitura/escrita flutuam micrometricamente acima da superfície.

```
                  ┌─────────┐
         ─────────┤  Prato  │──────────  ◄── superfície magnética
                  │         │
         ─────────┤  Prato  │──────────
                  └─────────┘
                      │
                   Spindle (eixo)
                      │
                   Motor (RPM: 5400, 7200, 10k, 15k)
```

**Trilhas (tracks):** círculos concêntricos em cada superfície do prato.

**Setores (sectors):** divisões de cada trilha. Tradicionalmente 512 bytes; modernos: 4096 bytes (4K).

**Cilindro (cylinder):** conjunto de todas as trilhas na mesma posição radial em todos os pratos. Acessar um cilindro inteiro não exige movimento da cabeça.

**Pista (track) vs Cilindro:** uma trilha é em um único prato; um cilindro é a mesma trilha em todos os pratos simultaneamente.

#### Tempo de acesso

```
T_acesso = T_busca + T_latência_rotacional + T_transferência
```

**Tempo de busca (seek time):** tempo para mover a cabeça até a trilha correta. ~1–10 ms (típico: 5 ms em HDs de 7200 RPM).

**Latência rotacional (rotational latency):** tempo de espera para o setor girar até a cabeça. No pior caso, uma rotação completa; em média, metade. Para 7200 RPM: 60/7200 = 8,33 ms por rotação → latência média ~4,2 ms.

**Tempo de transferência:** tempo para ler/escrever os dados depois que a cabeça está posicionada. ~100–200 MB/s para HDs modernos.

**Exemplo:** ler um setor aleatório = 5 ms (busca) + 4 ms (rotação) + ~0,1 ms (transferência) ≈ **9 ms total**

Por isso, leituras **sequenciais** (dados contíguos) são muito mais rápidas que leituras **aleatórias** em HDs — a busca e a rotação dominam o tempo.

#### Escalonamento de disco (Disk Scheduling)

O SO pode reordenar as requisições de leitura/escrita para minimizar o tempo de busca:

- **FCFS:** atende na ordem de chegada — simples mas ineficiente
- **SSTF (Shortest Seek Time First):** atende sempre o pedido mais próximo da posição atual — pode causar starvation de pedidos distantes
- **SCAN (elevador):** a cabeça varre o disco de um lado ao outro, atendendo pedidos no caminho — comportamento de elevador
- **C-SCAN (Circular SCAN):** como SCAN mas retorna rapidamente ao início sem atender pedidos na volta — mais uniforme

### 12.2 SSDs (Solid State Drives)

SSDs armazenam dados em **memória flash NAND** — sem partes móveis. Elétrons são aprisionados em células de armazenamento por tunelamento quântico.

#### Tipos de célula NAND

| Tipo | Bits/célula | Velocidade | Durabilidade (ciclos P/E) | Custo |
|---|---|---|---|---|
| SLC | 1 bit | Máxima | ~100.000 | Muito alto |
| MLC | 2 bits | Alta | ~10.000 | Alto |
| TLC | 3 bits | Moderada | ~3.000 | Moderado |
| QLC | 4 bits | Menor | ~1.000 | Baixo |
| PLC | 5 bits | Menor ainda | ~300 | Muito baixo |

**Ciclos P/E (Program/Erase):** cada célula suporta um número limitado de escritas antes de degradar. Por isso, SSDs usam técnicas de **wear leveling** (distribuição uniforme de escritas) para prolongar a vida útil.

#### Operações na flash NAND

- **Leitura:** rápida (~25–100 µs), pode ser feita bit a bit
- **Escrita (programação):** só pode escrever 0→1 (apaga) ou 1→0 (programa), uma página por vez (~256 KB – 1 MB)
- **Apagamento:** a flash só pode ser **apagada em blocos** (~256 páginas de uma vez), operação lenta (~1–5 ms) que reseta todas as células do bloco para 1

Este comportamento cria a necessidade do **garbage collection** (coleta de lixo): quando um bloco tem algumas páginas inválidas, o SSD move as páginas válidas para outro lugar e apaga o bloco inteiro.

#### Comparação HD vs SSD

| Característica | HD | SSD (NVMe) |
|---|---|---|
| Velocidade sequencial | ~150 MB/s | ~3.000–7.000 MB/s |
| Velocidade aleatória (4K) | ~0,5–2 MB/s | ~500–1.000 MB/s |
| Latência de acesso | ~5–10 ms | ~0,02–0,1 ms |
| Ruído | Sim (mecânico) | Não |
| Resistência a impacto | Baixa | Alta |
| Durabilidade de escrita | Praticamente ilimitada | Limitada (ciclos P/E) |
| Custo por TB (2024) | ~$15–25 | ~$50–80 |

### 12.3 Mídia Óptica

CDs, DVDs e Blu-rays armazenam dados em uma espiral de **pits** (depressões) e **lands** (superfícies planas) gravados em material reflexivo. Um laser lê as transições pit/land como bits.

A diferença entre os formatos é o comprimento de onda do laser — menor comprimento de onda → foco mais preciso → dados mais densos → maior capacidade.

| Mídia | Comprimento de onda | Capacidade | Taxa de leitura |
|---|---|---|---|
| CD | 780 nm (infravermelho) | ~700 MB | ~150 KB/s (1×) |
| DVD | 650 nm (vermelho) | ~4,7 GB (camada simples) | ~1,38 MB/s (1×) |
| Blu-ray | 405 nm (azul-violeta) | ~25 GB (camada simples) / 100 GB (4 camadas) | ~4,5 MB/s (1×) |

### 12.4 Fita Magnética

Ainda amplamente usada para **backup e arquivamento** de grandes volumes:

- Capacidade por cartucho: ~6–45 TB nativos (LTO-9: 45 TB)
- Custo por TB: ~$5–10 (o mais barato de todos)
- Acesso: estritamente sequencial (pode demorar minutos para acessar um arquivo específico)
- Durabilidade: 30 anos se armazenadas corretamente
- Padrão atual: LTO (Linear Tape-Open)

---

## 13. Caminho Completo dos Dados

### Da instrução ao resultado: um exemplo end-to-end

Vamos rastrear o que acontece, nos mínimos detalhes, quando o computador executa esta linha de código:

```c
resultado = A + B;
```

Assumindo que `A` e `B` estão na memória RAM e que `resultado` também precisa ser escrito na RAM.

---

### Passo 1 — O programa está no disco

Antes de qualquer coisa, o programa está armazenado no **disco (SSD ou HD)**. Quando o usuário clica para executar:

1. O SO cria um processo, aloca memória virtual
2. O **loader** copia as páginas do executável do disco → RAM (via controlador de armazenamento + DMA)
3. O **PC é configurado** para o endereço inicial do programa na RAM

---

### Passo 2 — Busca da instrução (Fetch)

A CPU lê a instrução de máquina correspondente a `resultado = A + B`.

```
PC contém o endereço da instrução
         │
         ▼
TLB: traduz endereço virtual → endereço físico
         │
         ▼
Cache L1 de instruções: instrução está aqui? (hit ~99% do tempo)
         │ (se miss → L2 → L3 → RAM)
         ▼
Instrução carregada no IR
PC incrementado
```

---

### Passo 3 — Decodificação

A Unidade de Controle interpreta o IR:
- Opcode: `LOAD` (para buscar A)
- Registrador destino: R1
- Endereço: `[endereço de A]`

---

### Passo 4 — Busca do operando A (LOAD A → R1)

```
Endereço de A → TLB → endereço físico
         │
         ▼
Cache L1 de dados: A está aqui? 
  SIM (hit): retorna valor em ~4 ciclos (~1 ns)
  NÃO (miss): procura em L2 (~5 ns), L3 (~20 ns), RAM (~100 ns)
         │
         ▼
Valor de A → MDR → R1
```

Se A estiver na RAM, o controlador de memória aciona os chips DRAM:
- Endvia RAS (ativa a linha correta no array)
- Envia CAS (seleciona a coluna)
- Dado percorre o barramento de dados (64 bits por transferência)

---

### Passo 5 — Busca do operando B (LOAD B → R2)

Mesmo processo para B → R2. Se B está próximo de A na memória (localidade espacial), provavelmente já está na cache do mesmo bloco carregado no passo anterior.

---

### Passo 6 — Execução (ADD R1, R2 → R3)

A ULA recebe R1 e R2 como operandos e executa a adição. O resultado é colocado no registrador R3. As flags (zero, carry, overflow) são atualizadas.

```
R1 (valor de A)  ─┐
                   ├──► ULA (ADD) ──► R3 (resultado)
R2 (valor de B)  ─┘
```

---

### Passo 7 — Escrita do resultado (STORE R3 → [resultado])

O valor de R3 precisa ser escrito na variável `resultado` na memória.

Com **write-back**: o dado é escrito na cache L1 (marcado como dirty). Só irá para a RAM quando essa linha de cache for substituída.

Com **write-through**: o dado é escrito na cache e simultaneamente no barramento → controlador de memória → DRAM.

---

### Passo 8 — Saída (se resultado for exibido na tela)

Se o programa eventualmente exibe o resultado:

1. O valor chega ao **driver da placa de vídeo** via chamada de sistema
2. O driver escreve no **framebuffer** (região de memória mapeada para a GPU)
3. A GPU lê o framebuffer e gera os sinais analógicos/digitais para o monitor
4. O monitor apresenta os pixels ao usuário

---

### Diagrama do caminho completo

```
 DISCO           RAM              CACHE           CPU
 ┌─────┐       ┌───────┐       ┌─────────┐     ┌──────────────┐
 │     │ DMA   │       │◄─────►│  L3     │     │              │
 │     │──────►│       │       │         │     │  ┌────────┐  │
 │SSD/ │       │Código │       │  L2     │◄───►│  │  UC    │  │
 │ HD  │       │Dados  │       │         │     │  └────────┘  │
 │     │       │       │       │  L1-I   │◄───►│  ┌────────┐  │
 └─────┘       └───────┘       │  L1-D   │     │  │  ULA   │  │
    ▲              ▲            └─────────┘     │  └────────┘  │
    │              │                            │  ┌────────┐  │
    └──────────────┴────────────────────────────┤  │  Regs  │  │
              Barramento do sistema             │  └────────┘  │
                                                └──────────────┘
                                                       │
                                                       │ Barramento PCIe
                                                       ▼
                                               ┌───────────────┐
                                               │      GPU      │
                                               │  Framebuffer  │
                                               └───────┬───────┘
                                                       │ DisplayPort / HDMI
                                                       ▼
                                                   Monitor
```

---

### Resumo do caminho de dados

| Etapa | Componente | Velocidade típica |
|---|---|---|
| Instrução buscada | Cache L1-I | ~1 ns |
| Operando na cache | Cache L1-D | ~1–4 ns |
| Operando no cache L2 | Cache L2 | ~5 ns |
| Operando no cache L3 | Cache L3 | ~20 ns |
| Operando na RAM | DRAM | ~60–100 ns |
| Operando no SSD | NVMe SSD | ~100 µs |
| Operando no HD | HD magnético | ~5–10 ms |
| Resultado na tela | GPU + Monitor | ~8–16 ms (60–120 Hz) |

> **Conclusão:** um simples `A + B` pode ser executado em menos de 1 ns se os dados estiverem nos registradores — ou levar centenas de milissegundos se precisarem ser lidos do disco. A hierarquia de memória e o projeto cuidadoso do software (localidade de acesso, uso eficiente da cache) fazem toda a diferença no desempenho real.

---

*Fim das anotações — Capítulo 2 — Tanenbaum, Organização Estruturada de Computadores*
