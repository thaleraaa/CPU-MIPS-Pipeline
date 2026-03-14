# CPU-MIPS-Pipeline
Implementação de uma CPU RISC baseada em MIPS com pipeline de 5 estágios em Verilog HDL.

## Status Atual
| Instrução | Formato | Opcode       | Status       |
|-----------|---------|--------------|--------------|
| ADD       | R       | 20, funct=32 | ✅ Concluído |
| SUB       | R       | 20, funct=34 | ✅ Concluído |
| MUL       | R       | 20, funct=50 | ✅ Concluído |
| AND       | R       | 20, funct=36 | ✅ Concluído |
| OR        | R       | 20, funct=37 | ✅ Concluído |
| JMP       | J       | 2            | ✅ Concluído |
| ADDI      | I       | 45           | ✅ Concluído |
| ORI       | I       | 46           | ✅ Concluído |
| BNE       | I       | 44           | ✅ Concluído |
| LW        | I       | 42           | ✅ Concluído |
| SW        | I       | 43           | ✅ Concluído |

## O que funciona
- Estrutura completa do pipeline (IF, ID, EX, MEM, WB)
- Todas as instruções tipo R (ADD, SUB, MUL, AND, OR) com WriteBack no RegisterFile
- Instrução JMP com desvio incondicional validado por armadilha
- Instruções tipo I imediatas (ADDI, ORI) com SignExtend e MUX no estágio EX
- Instrução BNE com desvio condicional validado em dois cenários (tomado e não tomado)
- Instruções de memória LW e SW com DataMemory síncrona
- Branch hazard resolvido por NOPs (2 delay slots, offset = destino - (BNE + 8))
- NOP tratado automaticamente pelo módulo Control
- Data hazard resolvido por NOPs (bubbles)
- Loop com acumulador validado (soma de 32 valores, resultado correto com e sem bubbles)

## Arquitetura
Pipeline de 5 estágios:
```
IF → ID → EX → MEM → WB
```

### Módulos Implementados
- `PC` — Program Counter com suporte a JMP e BNE
- `InstructionMemory` — memória de programa indexada por byte
- `Control` — decodificação de instruções R, I e J
- `RegisterFile` — banco de 32 registradores, r0 hard-wired em 0
- `ALU` — operações ADD, SUB, MUL, AND, OR com zeroFlag
- `Extend` — sign extension de 16 para 32 bits
- `DataMemory` — memória de dados síncrona com suporte a LW e SW
- `IMM` — registrador de pipeline para imediato e flag isIMM
- `A, B` — registradores de estágio ID/EX
- `CTRL, CTRL2` — registradores de controle de pipeline
- `D` — registrador de estágio EX/WB

### Cálculo de Offset BNE
O pipeline tem 2 ciclos de atraso entre o BNE e a decisão do PC:
```
offset = destino - (endereço_do_BNE + 8)

Exemplos:
  Para frente: BNE em mem[160], destino mem[176] → offset = 176 - 168 = +8
  Para trás:   BNE em mem[24],  destino mem[12]  → offset = 12  - 32  = -20
```

## Limitações Conhecidas
- InstructionMemory hardcoded no testbench — será substituída por `Code.hex` via IP `altsyncram`
- DataMemory hardcoded no testbench — será substituída por `Data.hex` via IP `altsyncram`
- Multiplicador com clock separado (CLK_MUL via PLL) pendente
- ADDRDecoding e ADDRDecoding_Prog pendentes
- Simulação Gate Level pendente

## Testado
- `ADD r3 = r1+r2 (10+5=15)` ✅
- `ADD r6 = r3+r1 (15+10=25) com NOPs` ✅
- `SUB r4 = r1-r2 (10-5=5)` ✅
- `MUL r5 = r1*r2 (10*5=50)` ✅
- `AND r7 = r1&r2 (10&5=0)` ✅
- `OR  r8 = r1|r2 (10|5=15)` ✅
- `JMP → destino correto validado por armadilha` ✅
- `ADDI r10 = r1+7 (10+7=17)` ✅
- `ADDI r11 = r2+(-1) (5-1=4) com SignExt negativo` ✅
- `ORI  r12 = r1|6 (10|6=14)` ✅
- `ORI  r13 = r2|0xFFF0 (5|0xFFFFFFF0=0xFFFFFFF5) com SignExt` ✅
- `BNE tomado: r1!=r2 → desvia, armadilha não executou` ✅
- `BNE não tomado: r1==r1 → sequencial, armadilha não executou` ✅
- `SW r1→mem[0]=10, SW r2→mem[4]=5` ✅
- `LW r18←mem[0]=10, LW r19←mem[4]=5` ✅
- `Loop sem bubble: soma mem[0..31] = 528, SW em mem[1023]` ✅
- `Loop com bubble: soma mem[0..31] = 528, MUL 528*255 = 134640` ✅
