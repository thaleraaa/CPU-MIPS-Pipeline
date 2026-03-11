# CPU-MIPS-Pipeline
Implementação parcial de uma CPU RISC baseada em MIPS com pipeline de 4 estágios em Verilog HDL.

## Status Atual
| Instrução | Formato | Opcode       | Status      |
|-----------|---------|--------------|-------------|
| ADD       | R       | 20, funct=32 | ✅ Concluído |
| SUB       | R       | 20, funct=34 | ✅ Pendente  |
| MUL       | R       | 20, funct=50 | ⏳ Pendente  |
| AND       | R       | 20, funct=36 | ⏳ Pendente  |
| OR        | R       | 20, funct=37 | ⏳ Pendente  |
| LW        | I       | 42           | ⏳ Pendente  |
| SW        | I       | 43           | ⏳ Pendente  |
| BNE       | I       | 44           | ⏳ Pendente  |
| ADDI      | I       | 45           | ⏳ Pendente  |
| ORI       | I       | 46           | ⏳ Pendente  |
| JMP       | J       | 2            | ⏳ Pendente  |

## O que funciona
- Estrutura completa do pipeline (IF, ID, EX, WB)
- Instrução ADD com WriteBack no RegisterFile
- NOP tratado automaticamente

## Arquitetura
Pipeline de 4 estágios:
```
IF → ID → EX → WB
```

## Limitações Conhecidas
- InstructionMemory hardcoded — será substituída por arquivo .hex
- Data hazard resolvido por NOPs (bubbles) — sem forwarding
- DataMemory, Extend, Multiplicador, BNE/JMP ainda não implementados


## Testado
- `ADD r3 = r1+r2 (10+5=15)` ✅
- `ADD r6 = r3+r1 (15+10=25) com NOPs` ✅
