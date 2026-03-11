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
| LW        | I       | 42           | ⏳ Pendente  |
| SW        | I       | 43           | ⏳ Pendente  |
| BNE       | I       | 44           | ⏳ Pendente  |
| ADDI      | I       | 45           | ⏳ Pendente  |
| ORI       | I       | 46           | ⏳ Pendente  |

## O que funciona
- Estrutura completa do pipeline (IF, ID, EX, WB)
- Todas as instruções tipo R (ADD, SUB, MUL, AND, OR) com WriteBack no RegisterFile
- Instrução JMP com desvio incondicional validado por armadilha
- NOP tratado automaticamente pelo módulo Control
- Data hazard resolvido por NOPs (bubbles)

## Arquitetura
Pipeline de 5 estágios:
```
IF → ID → EX → MEM → WB
```

### Módulos Implementados
- `PC` — Program Counter com suporte a JMP
- `InstructionMemory` — memória de programa indexada por byte (compatível com BRAM Altera)
- `Control` — decodificação de instruções R e J
- `RegisterFile` — banco de 32 registradores, r0 hard-wired em 0
- `ALU` — operações ADD, SUB, MUL, AND, OR
- `A, B` — registradores de estágio ID/EX
- `CTRL, CTRL2` — registradores de controle de pipeline
- `D` — registrador de estágio EX/WB

## Limitações Conhecidas
- InstructionMemory hardcoded no testbench — será substituída por `Code.hex` via IP `altsyncram`
- DataMemory, Extend, BNE ainda não implementados
- Instruções tipo I (LW, SW, BNE, ADDI, ORI) pendentes

## Testado
- `ADD r3 = r1+r2 (10+5=15)` ✅
- `ADD r6 = r3+r1 (15+10=25) com NOPs` ✅
- `SUB r4 = r1-r2 (10-5=5)` ✅
- `MUL r5 = r1*r2 (10*5=50)` ✅
- `AND r7 = r1&r2 (10&5=0)` ✅
- `OR  r8 = r1|r2 (10|5=15)` ✅
- `JMP → destino correto validado por armadilha` ✅

