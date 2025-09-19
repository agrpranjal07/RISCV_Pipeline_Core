# RISC-V Instruction Set Support

This document details the RISC-V instructions supported by the pipeline processor core.

---

## Table of Contents
1. [Overview](#overview)
2. [Instruction Format](#instruction-format)
3. [Supported Instructions](#supported-instructions)
4. [Instruction Encoding](#instruction-encoding)
5. [Control Signal Generation](#control-signal-generation)

---

## Overview

This RISC-V processor core implements the **RV32I** (32-bit Integer) base instruction set architecture. RV32I provides a complete instruction set sufficient for general-purpose computing.

### Key Features
- 32-bit instruction width
- 32 general-purpose registers (x0-x31)
- Load-store architecture
- Fixed-length instruction encoding
- Support for all RV32I instruction types

---

## Instruction Format

### RISC-V Instruction Types

| Type | Format | Usage |
|------|--------|-------|
| **R-Type** | `funct7[6:0] rs2[4:0] rs1[4:0] funct3[2:0] rd[4:0] opcode[6:0]` | Register-register operations |
| **I-Type** | `imm[11:0] rs1[4:0] funct3[2:0] rd[4:0] opcode[6:0]` | Immediate operations, loads |
| **S-Type** | `imm[11:5] rs2[4:0] rs1[4:0] funct3[2:0] imm[4:0] opcode[6:0]` | Store operations |
| **B-Type** | `imm[12,10:5] rs2[4:0] rs1[4:0] funct3[2:0] imm[4:1,11] opcode[6:0]` | Branch operations |
| **U-Type** | `imm[31:12] rd[4:0] opcode[6:0]` | Upper immediate |
| **J-Type** | `imm[20,10:1,11,19:12] rd[4:0] opcode[6:0]` | Jump operations |

### Register Convention

| Register | ABI Name | Description | Saved by |
|----------|----------|-------------|----------|
| x0 | zero | Hard-wired zero | - |
| x1 | ra | Return address | Caller |
| x2 | sp | Stack pointer | Callee |
| x3 | gp | Global pointer | - |
| x4 | tp | Thread pointer | - |
| x5-x7 | t0-t2 | Temporary registers | Caller |
| x8 | s0/fp | Saved register/Frame pointer | Callee |
| x9 | s1 | Saved register | Callee |
| x10-x11 | a0-a1 | Function arguments/return values | Caller |
| x12-x17 | a2-a7 | Function arguments | Caller |
| x18-x27 | s2-s11 | Saved registers | Callee |
| x28-x31 | t3-t6 | Temporary registers | Caller |

---

## Supported Instructions

### R-Type Instructions (Register-Register)

| Instruction | Opcode | funct3 | funct7 | Description | Operation |
|-------------|--------|--------|--------|-------------|-----------|
| **ADD** | 0110011 | 000 | 0000000 | Add | rd = rs1 + rs2 |
| **SUB** | 0110011 | 000 | 0100000 | Subtract | rd = rs1 - rs2 |
| **SLL** | 0110011 | 001 | 0000000 | Shift Left Logical | rd = rs1 << rs2[4:0] |
| **SLT** | 0110011 | 010 | 0000000 | Set Less Than | rd = (rs1 < rs2) ? 1 : 0 |
| **SLTU** | 0110011 | 011 | 0000000 | Set Less Than Unsigned | rd = (rs1 < rs2) ? 1 : 0 (unsigned) |
| **XOR** | 0110011 | 100 | 0000000 | Exclusive OR | rd = rs1 ^ rs2 |
| **SRL** | 0110011 | 101 | 0000000 | Shift Right Logical | rd = rs1 >> rs2[4:0] |
| **SRA** | 0110011 | 101 | 0100000 | Shift Right Arithmetic | rd = rs1 >> rs2[4:0] (sign-extend) |
| **OR** | 0110011 | 110 | 0000000 | Bitwise OR | rd = rs1 \| rs2 |
| **AND** | 0110011 | 111 | 0000000 | Bitwise AND | rd = rs1 & rs2 |

### I-Type Instructions (Immediate)

#### Arithmetic Immediate
| Instruction | Opcode | funct3 | Description | Operation |
|-------------|--------|--------|-------------|-----------|
| **ADDI** | 0010011 | 000 | Add Immediate | rd = rs1 + sign_extend(imm) |
| **SLTI** | 0010011 | 010 | Set Less Than Immediate | rd = (rs1 < sign_extend(imm)) ? 1 : 0 |
| **SLTIU** | 0010011 | 011 | Set Less Than Immediate Unsigned | rd = (rs1 < imm) ? 1 : 0 (unsigned) |
| **XORI** | 0010011 | 100 | XOR Immediate | rd = rs1 ^ sign_extend(imm) |
| **ORI** | 0010011 | 110 | OR Immediate | rd = rs1 \| sign_extend(imm) |
| **ANDI** | 0010011 | 111 | AND Immediate | rd = rs1 & sign_extend(imm) |

#### Shift Immediate
| Instruction | Opcode | funct3 | funct7 | Description | Operation |
|-------------|--------|--------|--------|-------------|-----------|
| **SLLI** | 0010011 | 001 | 0000000 | Shift Left Logical Immediate | rd = rs1 << imm[4:0] |
| **SRLI** | 0010011 | 101 | 0000000 | Shift Right Logical Immediate | rd = rs1 >> imm[4:0] |
| **SRAI** | 0010011 | 101 | 0100000 | Shift Right Arithmetic Immediate | rd = rs1 >> imm[4:0] (sign-extend) |

#### Load Instructions
| Instruction | Opcode | funct3 | Description | Operation |
|-------------|--------|--------|-------------|-----------|
| **LB** | 0000011 | 000 | Load Byte | rd = sign_extend(mem[rs1 + imm][7:0]) |
| **LH** | 0000011 | 001 | Load Halfword | rd = sign_extend(mem[rs1 + imm][15:0]) |
| **LW** | 0000011 | 010 | Load Word | rd = mem[rs1 + imm] |
| **LBU** | 0000011 | 100 | Load Byte Unsigned | rd = zero_extend(mem[rs1 + imm][7:0]) |
| **LHU** | 0000011 | 101 | Load Halfword Unsigned | rd = zero_extend(mem[rs1 + imm][15:0]) |

#### Jump and Link Register
| Instruction | Opcode | funct3 | Description | Operation |
|-------------|--------|--------|-------------|-----------|
| **JALR** | 1100111 | 000 | Jump and Link Register | rd = PC + 4; PC = (rs1 + imm) & ~1 |

### S-Type Instructions (Store)

| Instruction | Opcode | funct3 | Description | Operation |
|-------------|--------|--------|-------------|-----------|
| **SB** | 0100011 | 000 | Store Byte | mem[rs1 + imm][7:0] = rs2[7:0] |
| **SH** | 0100011 | 001 | Store Halfword | mem[rs1 + imm][15:0] = rs2[15:0] |
| **SW** | 0100011 | 010 | Store Word | mem[rs1 + imm] = rs2 |

### B-Type Instructions (Branch)

| Instruction | Opcode | funct3 | Description | Operation |
|-------------|--------|--------|-------------|-----------|
| **BEQ** | 1100011 | 000 | Branch if Equal | if (rs1 == rs2) PC = PC + imm |
| **BNE** | 1100011 | 001 | Branch if Not Equal | if (rs1 != rs2) PC = PC + imm |
| **BLT** | 1100011 | 100 | Branch if Less Than | if (rs1 < rs2) PC = PC + imm |
| **BGE** | 1100011 | 101 | Branch if Greater or Equal | if (rs1 >= rs2) PC = PC + imm |
| **BLTU** | 1100011 | 110 | Branch if Less Than Unsigned | if (rs1 < rs2) PC = PC + imm (unsigned) |
| **BGEU** | 1100011 | 111 | Branch if Greater or Equal Unsigned | if (rs1 >= rs2) PC = PC + imm (unsigned) |

### U-Type Instructions (Upper Immediate)

| Instruction | Opcode | Description | Operation |
|-------------|--------|-------------|-----------|
| **LUI** | 0110111 | Load Upper Immediate | rd = imm << 12 |
| **AUIPC** | 0010111 | Add Upper Immediate to PC | rd = PC + (imm << 12) |

### J-Type Instructions (Jump)

| Instruction | Opcode | Description | Operation |
|-------------|--------|-------------|-----------|
| **JAL** | 1101111 | Jump and Link | rd = PC + 4; PC = PC + imm |

---

## Instruction Encoding

### Immediate Encoding

Different instruction types encode immediate values differently:

#### I-Type Immediate
```
imm[11:0] = instruction[31:20]
sign_extended_imm = {{20{instruction[31]}}, instruction[31:20]}
```

#### S-Type Immediate
```
imm[11:5] = instruction[31:25]
imm[4:0] = instruction[11:7]
sign_extended_imm = {{20{instruction[31]}}, instruction[31:25], instruction[11:7]}
```

#### B-Type Immediate
```
imm[12] = instruction[31]
imm[10:5] = instruction[30:25]
imm[4:1] = instruction[11:8]
imm[11] = instruction[7]
imm[0] = 0 (always zero)
sign_extended_imm = {{19{instruction[31]}}, instruction[31], instruction[7], instruction[30:25], instruction[11:8], 1'b0}
```

#### U-Type Immediate
```
imm[31:12] = instruction[31:12]
imm[11:0] = 12'b0
```

#### J-Type Immediate
```
imm[20] = instruction[31]
imm[10:1] = instruction[30:21]
imm[11] = instruction[20]
imm[19:12] = instruction[19:12]
imm[0] = 0 (always zero)
sign_extended_imm = {{11{instruction[31]}}, instruction[31], instruction[19:12], instruction[20], instruction[30:21], 1'b0}
```

---

## Control Signal Generation

### Main Decoder Logic

Based on the instruction opcode, the main decoder generates:

```verilog
// RegWrite: Enable register file write
RegWrite = (Op == 7'b0000011 | Op == 7'b0110011 | Op == 7'b0010011) ? 1'b1 : 1'b0;

// ImmSrc: Immediate source selection
ImmSrc = (Op == 7'b0100011) ? 2'b01 :  // S-type
         (Op == 7'b1100011) ? 2'b10 :  // B-type
                              2'b00;   // I-type

// ALUSrc: ALU source selection (register vs immediate)
ALUSrc = (Op == 7'b0000011 | Op == 7'b0100011 | Op == 7'b0010011) ? 1'b1 : 1'b0;

// MemWrite: Memory write enable
MemWrite = (Op == 7'b0100011) ? 1'b1 : 1'b0;

// ResultSrc: Result source (ALU vs Memory)
ResultSrc = (Op == 7'b0000011) ? 1'b1 : 1'b0;

// Branch: Branch instruction indicator
Branch = (Op == 7'b1100011) ? 1'b1 : 1'b0;

// ALUOp: ALU operation type
ALUOp = (Op == 7'b0110011) ? 2'b10 :  // R-type
        (Op == 7'b1100011) ? 2'b01 :  // Branch
                             2'b00;   // Load/Store/I-type
```

### ALU Control Logic

The ALU decoder generates specific operation codes:

```verilog
ALUControl = (ALUOp == 2'b00) ? 3'b000 :  // ADD for load/store
             (ALUOp == 2'b01) ? 3'b001 :  // SUB for branch
             (ALUOp == 2'b10) ? 
                 ((funct3 == 3'b000) ? 
                     ((funct7 == 7'b0000000) ? 3'b000 :  // ADD
                      (funct7 == 7'b0100000) ? 3'b001 :  // SUB
                                                3'b000) : // default ADD
                  (funct3 == 3'b111) ? 3'b010 :          // AND
                  (funct3 == 3'b110) ? 3'b011 :          // OR
                                        3'b000) :         // default ADD
                                        3'b000;           // default ADD
```

---

## Assembly Examples

### Basic Arithmetic
```assembly
addi x1, x0, 10     # x1 = 10
addi x2, x0, 5      # x2 = 5
add x3, x1, x2      # x3 = x1 + x2 = 15
sub x4, x1, x2      # x4 = x1 - x2 = 5
```

### Memory Operations
```assembly
addi x1, x0, 100    # x1 = 100 (base address)
addi x2, x0, 42     # x2 = 42 (data to store)
sw x2, 0(x1)        # mem[100] = 42
lw x3, 0(x1)        # x3 = mem[100] = 42
```

### Control Flow
```assembly
addi x1, x0, 10     # x1 = 10
addi x2, x0, 20     # x2 = 20
blt x1, x2, target  # if x1 < x2, jump to target
addi x3, x0, 1      # x3 = 1 (not executed if branch taken)
target:
    addi x4, x0, 2  # x4 = 2
```

This instruction set provides a solid foundation for implementing various algorithms and applications on the RISC-V pipeline processor.
