# CPU-MIPS-Pipeline
Implementação de uma CPU RISC baseada em MIPS com pipeline em Verilog HDL para FPGA Cyclone IV GX.

## Status Atual
| Instrução | Formato | Opcode (grupo 0) | Status       |
|-----------|---------|------------------|--------------|
| ADD       | R       | 10, funct=32     | ✅ Concluído |
| SUB       | R       | 10, funct=34     | ✅ Concluído |
| MUL       | R       | 10, funct=50     | ✅ Concluído |
| AND       | R       | 10, funct=36     | ✅ Concluído |
| OR        | R       | 10, funct=37     | ✅ Concluído |
| JMP       | J       | 2                | ✅ Concluído |
| LW        | I       | 32               | ✅ Concluído |
| SW        | I       | 33               | ✅ Concluído |
| BNE       | I       | 34               | ✅ Concluído |
| ADDI      | I       | 35               | ✅ Concluído |
| ORI       | I       | 36               | ✅ Concluído |

## O que funciona
- Estrutura completa do pipeline (IF, ID, EX, WB) — 4 estágios efetivos devido às memórias síncronas da FPGA absorverem um estágio
- Todas as instruções tipo R (ADD, SUB, MUL, AND, OR) com WriteBack no RegisterFile
- Instrução JMP com desvio incondicional validado
- Instruções tipo I imediatas (ADDI, ORI) com SignExtend e MUX no estágio EX
- Instrução BNE com desvio condicional validado (tomado e não tomado)
- Instruções de memória LW e SW com DataMemory síncrona (altsyncram)
- Branch hazard resolvido por NOPs (2 delay slots)
- Data hazard resolvido por NOPs (bubbles)
- NOP tratado automaticamente pelo módulo Control (opcode 0)
- Loop com acumulador validado: soma de mem[0..31] = 496, MUL 496×255 = 126480
- Apenas memória interna (grupo 0, sem acesso a memória externa)

## Arquitetura
Pipeline de 4 estágios efetivos (memórias síncronas absorvem estágio de busca):
```
IF/ID → ID/EX → EX/MEM → WB
```

### Módulos Implementados
- `PC` — Program Counter com suporte a JMP e BNE, endereço base 0x0 (grupo 0)
- `InstructionMemory` — ROM síncrona altsyncram, inicializada via Code.hex, indexada por PC[11:2]
- `Control` — decodificação de instruções R, I e J; NOP (opcode 0) gera sinais neutros
- `RegisterFile` — banco de 32 registradores síncronos, r0 hard-wired em 0
- `ALU` — operações ADD, SUB, MUL, AND, OR com zeroFlag
- `Extend` — sign extension de 16 para 32 bits
- `DataMemory` — RAM síncrona altsyncram, inicializada via Data.hex (mem[0..31] = 0..31)
- `IMM` — registrador de pipeline para imediato extendido e flag isIMM
- `A, B` — registradores de estágio ID/EX
- `CTRL, CTRL2` — registradores de controle propagando sinais pelo pipeline
- `D` — registrador de estágio EX/WB

## Resultado do Programa de Teste
```
r30  = 32   (tamanho do vetor)         ✅
r13  = 255  (constante multiplicadora) ✅
r10  = 496  (soma mem[0..31])          ✅
r20  = 126480 (496 × 255)             ✅
```

## Limitações Conhecidas
- Multiplicador com clock separado (CLK_MUL via PLL) pendente
- ADDRDecoding e ADDRDecoding_Prog pendentes (não necessários para grupo 0 — sem acesso a memória externa)
- Simulação Gate Level pendente

## Testado
- `ADD r3 = r1+r2 (10+5=15)` ✅
- `ADD r6 = r3+r1 (15+10=25) com NOPs` ✅
- `SUB r4 = r1-r2 (10-5=5)` ✅
- `MUL r5 = r1*r2 (10*5=50)` ✅
- `AND r7 = r1&r2 (10&5=0)` ✅
- `OR  r8 = r1|r2 (10|5=15)` ✅
- `JMP → destino correto validado` ✅
- `ADDI r10 = r1+7 (10+7=17)` ✅
- `ADDI r11 = r2+(-1) (5-1=4) com SignExt negativo` ✅
- `ORI  r12 = r1|6 (10|6=14)` ✅
- `ORI  r13 = r2|0xFFF0 com SignExt` ✅
- `BNE tomado: r1!=r2 → desvia` ✅
- `BNE não tomado: r1==r1 → sequencial` ✅
- `SW r1→mem[0]=10, SW r2→mem[4]=5` ✅
- `LW r18←mem[0]=10, LW r19←mem[4]=5` ✅
- `Loop sem bubble: soma mem[0..31] = 496` ✅
- `Loop com bubble: soma 496×255 = 126480` ✅
