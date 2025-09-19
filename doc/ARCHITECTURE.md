# RISC-V Pipeline Core Architecture

This document provides a comprehensive overview of the 5-stage pipelined RISC-V processor architecture, including implementation details, module specifications, and hazard handling mechanisms.

---

## Table of Contents
1. [Overview](#overview)
2. [Pipeline Architecture](#pipeline-architecture)
3. [Pipeline Stages](#pipeline-stages)
4. [Module Specifications](#module-specifications)
5. [Control Unit](#control-unit)
6. [Hazard Handling](#hazard-handling)
7. [Instruction Set Support](#instruction-set-support)

---

## Overview

### Pipelining Concept

This RISC-V processor core implements a classic 5-stage pipeline architecture that allows multiple instructions to be in different stages of execution simultaneously. The five stages are:

- **IF (Instruction Fetch)**: Fetch instruction from memory
- **ID (Instruction Decode)**: Decode instruction and read registers
- **EX (Execute)**: Perform ALU operations
- **MEM (Memory)**: Access data memory for loads/stores
- **WB (Writeback)**: Write results back to register file

### Key Benefits

- **Increased Throughput**: Up to 5 instructions can execute simultaneously
- **Higher Clock Frequency**: Each stage contains only a fraction of total logic
- **Improved Performance**: Approximately 5x performance improvement over single-cycle design

### Architecture Diagram

![Top Architecture](Top%20Architecture.png)

---

## Pipeline Architecture

### Pipeline Registers

The pipeline uses four sets of registers between stages to store intermediate data:

| Register Set | Purpose | Key Signals |
|--------------|---------|-------------|
| **IF/ID** | Fetch → Decode | `InstrD`, `PCD`, `PCPlus4D` |
| **ID/EX** | Decode → Execute | Control signals, register data, immediate values |
| **EX/MEM** | Execute → Memory | `ALU_ResultM`, `WriteDataM`, control signals |
| **MEM/WB** | Memory → Writeback | `ALU_ResultW`, `ReadDataW`, `WriteRegW` |

### Top-Level Module Interface

```verilog
module Pipeline_top(
    input clk,           // Clock signal
    input rst            // Reset signal
);
```

---

## Pipeline Stages

### 1. Instruction Fetch (IF)

**Module**: `Fetch_Cycle.v`

**Functionality**:
- Fetches instructions from instruction memory
- Updates program counter (PC)
- Handles branch target calculation

**Key Components**:
- Program Counter (PC)
- PC Adder (+4)
- Instruction Memory
- PC Source Multiplexer

**Signals**:
```verilog
// Inputs
input clk, rst, PCSrcE
input [31:0] PCTargetE

// Outputs  
output [31:0] InstrD, PCD, PCPlus4D
```

### 2. Instruction Decode (ID)

**Module**: `Decode_Cycle.v`

**Functionality**:
- Decodes fetched instructions
- Reads source registers from register file
- Generates immediate values
- Produces control signals

**Key Components**:
- Control Unit
- Register File
- Sign Extension Unit
- ID/EX Pipeline Registers

**Control Signals Generated**:
- `RegWriteE`: Enable register write
- `ALUSrcE`: ALU source selection
- `MemWriteE`: Memory write enable
- `ResultSrcE`: Result source selection
- `BranchE`: Branch instruction indicator

### 3. Execute (EX)

**Module**: `Execute_Cycle.v`

**Functionality**:
- Performs arithmetic and logic operations
- Calculates branch targets
- Handles data forwarding
- Determines branch conditions

**Key Components**:
- Arithmetic Logic Unit (ALU)
- Branch Target Adder
- Forwarding Multiplexers
- EX/MEM Pipeline Registers

**Forwarding Logic**:
```verilog
// ForwardAE and ForwardBE control ALU input selection
// 00: From register file
// 01: Forward from MEM/WB stage  
// 10: Forward from EX/MEM stage
```

### 4. Memory Access (MEM)

**Module**: `Memory_Cycle.v`

**Functionality**:
- Handles load and store operations
- Accesses data memory
- Passes through ALU results for non-memory instructions

**Key Components**:
- Data Memory Interface
- MEM/WB Pipeline Registers

**Memory Operations**:
- **Load**: Read data from memory to register
- **Store**: Write register data to memory

### 5. Writeback (WB)

**Module**: `Writeback_Cycle.v`

**Functionality**:
- Selects final result (ALU output or memory data)
- Writes result back to register file

**Key Components**:
- Result Multiplexer
- Register File Write Port

---

## Module Specifications

### Core Pipeline Modules

#### Pipeline_Top.v
Top-level module instantiating all pipeline stages and control logic.

```verilog
// Key internal signals
wire PCSrcE, RegWriteW, RegWriteE, ALUSrcE, MemWriteE;
wire [2:0] ALUControlE;
wire [31:0] PCTargetE, InstrD, ResultW;
wire [1:0] ForwardBE, ForwardAE;
```

#### Control_Unit_Top.v
Generates all control signals based on instruction opcode and function fields.

```verilog
module Control_Unit_Top(
    input [6:0] Op, funct7,
    input [2:0] funct3,
    output RegWrite, ALUSrc, MemWrite, ResultSrc, Branch,
    output [1:0] ImmSrc,
    output [2:0] ALUControl
);
```

#### Hazard_unit.v
Implements data forwarding logic to resolve data hazards.

```verilog
module hazard_unit(
    input rst, RegWriteM, RegWriteW,
    input [4:0] RD_M, RD_W, Rs1_E, Rs2_E,
    output [1:0] ForwardAE, ForwardBE
);
```

### Supporting Modules

| Module | Description | Key Features |
|--------|-------------|--------------|
| `ALU.v` | Arithmetic Logic Unit | 8 operations (ADD, SUB, AND, OR, etc.) |
| `Register_File.v` | 32-register file | Dual read ports, single write port |
| `PC.v` | Program Counter | Synchronous update with reset |
| `Data_Memory.v` | Data memory interface | Load/store operations |
| `Instruction_Memory.v` | Instruction memory | Read-only memory for instructions |
| `Sign_Extend.v` | Immediate generation | Supports I, S, B type immediates |

---

## Control Unit

### Main Decoder

The main decoder generates primary control signals based on the instruction opcode:

| Opcode | Instruction Type | RegWrite | ALUSrc | MemWrite | ResultSrc | Branch | ALUOp |
|--------|------------------|----------|--------|----------|-----------|--------|-------|
| 0110011 | R-type | 1 | 0 | 0 | 0 | 0 | 10 |
| 0010011 | I-type (ALU) | 1 | 1 | 0 | 0 | 0 | 00 |
| 0000011 | Load | 1 | 1 | 0 | 1 | 0 | 00 |
| 0100011 | Store | 0 | 1 | 1 | X | 0 | 00 |
| 1100011 | Branch | 0 | 0 | 0 | X | 1 | 01 |

### ALU Control

The ALU decoder generates specific ALU operation codes:

| ALUOp | funct3 | funct7 | ALUControl | Operation |
|-------|--------|--------|------------|-----------|
| 00 | XXX | XXXXXXX | 000 | ADD |
| 01 | XXX | XXXXXXX | 001 | SUB |
| 10 | 000 | 0000000 | 000 | ADD |
| 10 | 000 | 0100000 | 001 | SUB |
| 10 | 111 | 0000000 | 010 | AND |
| 10 | 110 | 0000000 | 011 | OR |

---

## Hazard Handling

### Data Hazards

**Problem**: An instruction depends on the result of a previous instruction still in the pipeline.

**Solution**: Data forwarding (bypassing) from later pipeline stages to earlier ones.

### Forwarding Logic

```verilog
// Forward from EX/MEM stage
assign ForwardAE = ((RegWriteM == 1'b1) & (RD_M != 5'h00) & (RD_M == Rs1_E)) ? 2'b10 : ...

// Forward from MEM/WB stage  
assign ForwardAE = ((RegWriteW == 1'b1) & (RD_W != 5'h00) & (RD_W == Rs1_E)) ? 2'b01 : ...
```

### Forwarding Conditions

| ForwardAE/BE | Source | Description |
|--------------|--------|-------------|
| 00 | Register File | Normal register read |
| 01 | MEM/WB | Forward from memory/earlier ALU result |
| 10 | EX/MEM | Forward from current ALU result |

### Control Hazards

**Problem**: Branch instructions change PC, affecting instruction fetch.

**Solution**: 
- Pipeline flush on branch taken
- Branch resolution in Execute stage
- PC source multiplexer for branch targets

---

## Instruction Set Support

This processor supports the RV32I (RISC-V 32-bit Integer) base instruction set:

### Supported Instruction Types

- **R-Type**: Register-register operations (ADD, SUB, AND, OR, etc.)
- **I-Type**: Immediate operations and loads (ADDI, LW, etc.)
- **S-Type**: Store operations (SW, SH, SB)
- **B-Type**: Branch operations (BEQ, BNE, BLT, etc.)
- **U-Type**: Upper immediate (LUI, AUIPC)
- **J-Type**: Jump operations (JAL, JALR)

### Performance Characteristics

- **Ideal CPI**: 1.0 cycles per instruction
- **Actual CPI**: ~1.2 (including hazard penalties)
- **Clock Frequency**: Up to 5x single-cycle implementation
- **Throughput**: Near 1 instruction per cycle with forwarding

---

## Testing and Verification

The design includes comprehensive testbenches for:
- Individual module testing
- Pipeline integration testing
- Hazard scenario verification
- Instruction set compliance

See [TESTING.md](TESTING.md) for detailed testing procedures and results.
