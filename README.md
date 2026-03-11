# CPU-MIPS-Pipeline
Implementação parcial de uma CPU RISC baseada em MIPS com pipeline de 4 estágios em Verilog HDL.

## Status
Trabalho em andamento — apenas instrução ADD implementada e testada.

## O que funciona
- Estrutura completa do pipeline (IF, ID, EX, WB)
- Instrução ADD com WriteBack no RegisterFile
- NOP tratado automaticamente

## Em desenvolvimento
- Demais instruções (SUB, AND, OR, MUL, LW, SW, BNE, ADDI, ORI, JMP)
- DataMemory
- Multiplicador
- Extend
