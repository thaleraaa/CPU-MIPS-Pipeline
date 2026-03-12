# CPU-MIPS-Pipeline
Implementação parcial de uma CPU RISC baseada em MIPS com pipeline de 5 estágios em Verilog HDL.

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
| LW        | I       | 42           | ⏳ Pendente  |
| SW        | I       | 43           | ⏳ Pendente  |
| BNE       | I       | 44           | ⏳ Pendente  |

## O que funciona
- Estrutura completa do pipeline (IF, ID, EX, WB)
- Todas as instruções tipo R (ADD, SUB, MUL, AND, OR) com WriteBack no RegisterFile
- Instrução JMP com desvio incondicional validado por armadilha
- Instruções tipo I imediatas (ADDI, ORI) com SignExtend e MUX no estágio EX
- NOP tratado automaticamente pelo módulo Control
- Data hazard resolvido por NOPs (bubbles)

## Arquitetura
Pipeline de 5 estágios:
```
IF → ID → EX → MEM → WB
```

## Limitações Conhecidas
- InstructionMemory hardcoded no testbench — será substituída por `Code.hex` via IP `altsyncram`
- DataMemory ainda não implementada
- Instruções tipo I com memória (LW, SW) e desvio (BNE) pendentes

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
