# CPU-MIPS-Pipeline

Implementação de uma CPU RISC baseada em MIPS com pipeline em Verilog HDL para FPGA Cyclone IV GX.

📄 [Relatório do Projeto](./relatorio.pdf) | 📦 [Código-fonte](./CPU.RAR)

> Família FPGA: Cyclone IV GX

---

## Configuração do Grupo

O projeto é **totalmente parametrizável por grupo**. Basta alterar o `parameter GRUPO` no módulo top `CPU.v` e todos os endereços, opcodes e configurações se ajustam automaticamente:

```verilog
module CPU #(parameter GRUPO = 14) ( ... );
```

Todos os submódulos recebem `GRUPO` via instanciação `#(.GRUPO(GRUPO))` — não há nenhum valor fixo espalhado pelo código.

---

## Requisitos do Trabalho — Checklist

### Requisitos obrigatórios

| Requisito | Status |
|-----------|--------|
| Word de 32 bits Big Endian | ✅ |
| Pipeline implementado | ✅ |
| Instruções de 4 bytes | ✅ |
| 32 registradores (r0 hard-wired em 0) | ✅ |
| Memória de programa: 1kWord a partir de `GRUPO × 0x120` | ✅ |
| Memória de dados: 1kWord a partir de `GRUPO × 0x125` | ✅ |
| PC aponta para endereço inicial no Reset | ✅ |
| Simulação RTL funcionando | ✅ |
| FPGA família Cyclone IV GX | ✅ |
| Simulação Gate Level | ❌ Não funcional (timeout — ver notas) |
| Multiplicador com CLK_MUL separado (via PLL) | ✅ |
| ADDRDecoding — seleção memória interna/externa (dados) | ✅ |
| ADDRDecoding_Prog — seleção memória interna/externa (programa) | ✅ |

### ISA implementado

| Instrução | Formato | Opcode (grupo 14) | Status |
|-----------|---------|-------------------|--------|
| ADD | R | 24, funct=32 | ✅ Concluído |
| SUB | R | 24, funct=34 | ✅ Concluído |
| MUL | R | 24, funct=50 | ✅ Concluído |
| AND | R | 24, funct=36 | ✅ Concluído |
| OR  | R | 24, funct=37 | ✅ Concluído |
| JMP | J | 2 | ✅ Concluído |
| LW  | I | 46 | ✅ Concluído |
| SW  | I | 47 | ✅ Concluído |
| BNE | I | 48 | ✅ Concluído |
| ADDI | I | 49 | ✅ Concluído |
| ORI  | I | 50 | ✅ Concluído |
| NOP  | — | opcode 0 | ✅ Concluído |

---

## Arquitetura
Pipeline de 5 estágios (IF, ID, EX, MEM, WB) com 4 registradores de fronteira explícitos. As memórias síncronas BRAM da Altera absorvem o estágio IF — cujo registrador de saída serve como IF/ID —, conforme figura 1b do roteiro:
```
IF/ID → ID/EX → EX/MEM → MEM/WB
```
---

## Programa de Teste

O programa implementa a expressão exigida pelo roteiro:

```
MemDados[última posição] ← (Σ Mem[i + GRUPO×0x0125], i=0..31) × 255
```

Duas versões implementadas conforme o roteiro:

**Versão com data hazard** — sem NOPs, serve para evidenciar o problema.

**Versão com bubbles** — NOPs inseridos manualmente para resolver data hazard, conforme exigido.

### Resultado validado

| Registrador | Valor | Descrição |
|-------------|-------|-----------|
| r30 | 32 | Tamanho do vetor |
| r13 | 255 (0x00FF) | Constante multiplicadora |
| r10 | 496 | Soma de mem[0..31] = 0+1+…+31 |
| r20 | 126480 | 496 × 255 |

---

## Testes Unitários

| Teste | Resultado |
|-------|-----------|
| ADD r3 = r1+r2 (10+5=15) | ✅ |
| ADD r6 = r3+r1 com NOPs (15+10=25) | ✅ |
| SUB r4 = r1-r2 (10-5=5) | ✅ |
| MUL r5 = r1*r2 (10*5=50) | ✅ |
| AND r7 = r1&r2 (10&5=0) | ✅ |
| OR  r8 = r1\|r2 (10\|5=15) | ✅ |
| JMP — destino correto | ✅ |
| ADDI r10 = r1+7 (10+7=17) | ✅ |
| ADDI r11 = r2+(-1) com SignExt negativo (5-1=4) | ✅ |
| ORI  r12 = r1\|6 (10\|6=14) | ✅ |
| ORI  r13 = r2\|0xFFF0 com SignExt | ✅ |
| BNE tomado: r1≠r2 → desvia | ✅ |
| BNE não tomado: r1==r1 → sequencial | ✅ |
| SW r1→mem[0]=10, SW r2→mem[4]=5 | ✅ |
| LW r18←mem[0]=10, LW r19←mem[4]=5 | ✅ |
| Loop sem bubble: soma mem[0..31] = 496 | ✅ |
| Loop com bubble: 496×255 = 126480 | ✅ |

---

## Resultados TimeQuest (Fast 1200mV −40°C, EP4CGX150DF31I7AD)

| Clock | Fmax |
|-------|------|
| CLK_SYS — pipeline (clk[0]) | 78.55 MHz |
| Multiplicador interno | 332.67 MHz |
| CLK_MUL (clk[1]) | 376.51 MHz |

---

## Nota — Simulação Gate Level

A simulação gate-level com SDF não produziu resultados válidos.
---
