# Testing and Verification Guide

This document describes the testing methodology, testbenches, and verification procedures for the RISC-V pipeline processor.

---

## Table of Contents
1. [Overview](#overview)
2. [Test Environment Setup](#test-environment-setup)
3. [Testbench Architecture](#testbench-architecture)
4. [Test Cases](#test-cases)
5. [Simulation Instructions](#simulation-instructions)
6. [Verification Checklist](#verification-checklist)
7. [Performance Analysis](#performance-analysis)

---

## Overview

The RISC-V pipeline processor includes comprehensive verification to ensure correct functionality across all supported instructions and pipeline scenarios.

### Testing Objectives
- ✅ Verify all RV32I instructions execute correctly
- ✅ Validate pipeline hazard detection and resolution
- ✅ Confirm data forwarding functionality
- ✅ Test branch prediction and control flow
- ✅ Validate memory interface operations
- ✅ Ensure proper reset and initialization

### Verification Methodology
- **Unit Testing**: Individual module verification
- **Integration Testing**: Full pipeline functionality
- **Regression Testing**: Continuous validation of changes
- **Waveform Analysis**: Visual verification using GTKWave

---

## Test Environment Setup

### Required Tools
```bash
# For Ubuntu/Debian systems
sudo apt update
sudo apt install iverilog gtkwave

# For other systems, install:
# - Verilog simulator (ModelSim, Vivado, etc.)
# - Waveform viewer (GTKWave recommended)
```

### Project Structure
```
RISCV_Pipeline_Core/
├── src/
│   ├── *.v              # Verilog source files
│   ├── pipeline_tb.v    # Main testbench
│   ├── memfile.hex      # Test program memory
│   └── pipeline.gtkw    # GTKWave save file
└── doc/
    └── *.md             # Documentation
```

---

## Testbench Architecture

### Main Testbench (`pipeline_tb.v`)

The primary testbench instantiates the complete pipeline and provides:
- Clock generation
- Reset sequence
- Test program loading
- Result monitoring

```verilog
module pipeline_tb();
    // Test signals
    reg clk = 0;
    reg rst;
    
    // Clock generation (10ns period = 100MHz)
    always #5 clk = ~clk;
    
    // DUT instantiation
    Pipeline_top uut(
        .clk(clk),
        .rst(rst)
    );
    
    // Test sequence
    initial begin
        // Initialize
        rst = 1;
        #10 rst = 0;
        
        // Run test program
        #1000;
        
        // Finish simulation
        $finish;
    end
endmodule
```

### Test Program Memory (`memfile.hex`)

The test program is loaded into instruction memory in hexadecimal format:

```hex
20020005    // addi x2, x0, 5      # x2 = 5
20030003    // addi x3, x0, 3      # x3 = 3  
00230133    // add x2, x2, x3      # x2 = x2 + x3 = 8
00500113    // addi x2, x0, 5      # x2 = 5 (test overwrite)
00300193    // addi x3, x0, 3      # x3 = 3
40208133    // sub x2, x1, x2      # x2 = x1 - x2
00210233    // add x4, x2, x2      # x4 = x2 + x2 (test forwarding)
```

---

## Test Cases

### 1. Basic Instruction Tests

#### Arithmetic Instructions
```assembly
# Test ADD instruction
addi x1, x0, 10     # x1 = 10
addi x2, x0, 5      # x2 = 5
add x3, x1, x2      # x3 = 15
Expected: x3 = 15

# Test SUB instruction  
sub x4, x1, x2      # x4 = 10 - 5 = 5
Expected: x4 = 5

# Test AND instruction
andi x1, x0, 0xFF   # x1 = 255
andi x2, x0, 0x0F   # x2 = 15
and x3, x1, x2      # x3 = 255 & 15 = 15
Expected: x3 = 15
```

#### Memory Instructions
```assembly
# Test store/load
addi x1, x0, 100    # x1 = 100 (address)
addi x2, x0, 42     # x2 = 42 (data)
sw x2, 0(x1)        # Store x2 to address 100
lw x3, 0(x1)        # Load from address 100
Expected: x3 = 42
```

#### Branch Instructions
```assembly
# Test branch equal
addi x1, x0, 5      # x1 = 5
addi x2, x0, 5      # x2 = 5
beq x1, x2, target  # Should branch (5 == 5)
addi x3, x0, 1      # Should be skipped
target:
    addi x4, x0, 2  # x4 = 2
Expected: x3 = 0, x4 = 2
```

### 2. Hazard Detection Tests

#### Data Hazard with Forwarding
```assembly
# RAW hazard - forward from EX/MEM
addi x1, x0, 10     # x1 = 10
add x2, x1, x1      # x2 = x1 + x1 (needs x1 from previous instruction)
Expected: Forwarding occurs, x2 = 20
```

#### Load-Use Hazard
```assembly
# Load-use hazard requiring stall
lw x1, 0(x0)        # Load x1 from memory
add x2, x1, x1      # Use x1 immediately (stall required)
Expected: Pipeline stalls 1 cycle
```

#### Multiple Forwarding
```assembly
# Chain of dependencies
addi x1, x0, 5      # x1 = 5
add x2, x1, x1      # x2 = 10 (forward x1)
add x3, x2, x1      # x3 = 15 (forward x2 and x1)
Expected: All forwards correctly, x3 = 15
```

### 3. Control Hazard Tests

#### Branch Prediction
```assembly
# Test branch taken
addi x1, x0, 10     
addi x2, x0, 5
blt x2, x1, taken   # 5 < 10, should branch
addi x3, x0, 1      # Should be flushed
taken:
    addi x4, x0, 2
Expected: x3 = 0, x4 = 2
```

#### Jump Instructions
```assembly
# Test JAL
jal x1, jump_target # x1 = PC + 4, jump to target
addi x2, x0, 1      # Should be skipped
jump_target:
    addi x3, x0, 3
Expected: x1 = return address, x2 = 0, x3 = 3
```

### 4. Edge Cases

#### Register x0 Testing
```assembly
# x0 should always be zero
addi x0, x0, 100    # Try to write to x0
add x1, x0, x0      # x1 should be 0
Expected: x0 = 0, x1 = 0
```

#### Pipeline Flush
```assembly
# Test complete pipeline flush
addi x1, x0, 1
addi x2, x0, 2  
beq x1, x1, flush   # Always taken
addi x3, x0, 3      # Should be flushed
addi x4, x0, 4      # Should be flushed
flush:
    addi x5, x0, 5
Expected: x3 = 0, x4 = 0, x5 = 5
```

---

## Simulation Instructions

### Quick Start
```bash
# Navigate to project directory
cd RISCV_Pipeline_Core

# Compile all Verilog files
iverilog -o pipeline_sim src/pipeline_tb.v src/*.v

# Run simulation
./pipeline_sim

# View waveforms
gtkwave src/pipeline.gtkw
```

### Detailed Simulation Steps

#### 1. Compilation
```bash
# Compile with debugging information
iverilog -g2005-sv -o pipeline_sim src/pipeline_tb.v src/*.v

# Check for compilation errors
echo $?  # Should return 0 for success
```

#### 2. Execution
```bash
# Run simulation with VCD output
./pipeline_sim

# Simulation generates pipeline.vcd for waveform viewing
```

#### 3. Waveform Analysis
```bash
# Open GTKWave with save file
gtkwave src/pipeline.gtkw

# Or open VCD directly
gtkwave pipeline.vcd
```

### Custom Test Programs

#### Creating Test Programs
1. Write assembly code
2. Assemble to machine code
3. Convert to hex format
4. Update `memfile.hex`

```bash
# Example: Custom test program
echo "20020005" > custom_test.hex  # addi x2, x0, 5
echo "20030003" >> custom_test.hex # addi x3, x0, 3
echo "00230133" >> custom_test.hex # add x2, x2, x3

# Copy to memfile.hex
cp custom_test.hex src/memfile.hex
```

---

## Verification Checklist

### Functional Verification

#### ✅ Instruction Set Compliance
- [ ] All R-type instructions execute correctly
- [ ] All I-type instructions execute correctly  
- [ ] All S-type instructions execute correctly
- [ ] All B-type instructions execute correctly
- [ ] All U-type instructions execute correctly
- [ ] All J-type instructions execute correctly

#### ✅ Pipeline Functionality
- [ ] Instructions progress through all 5 stages
- [ ] Pipeline registers update correctly
- [ ] Control signals propagate properly
- [ ] Reset initializes all state correctly

#### ✅ Hazard Handling
- [ ] Data forwarding works for EX/MEM → EX
- [ ] Data forwarding works for MEM/WB → EX
- [ ] Load-use hazards detected and stalled
- [ ] Branch hazards flush pipeline correctly
- [ ] No false hazard detections

#### ✅ Memory Interface
- [ ] Instruction memory reads correctly
- [ ] Data memory loads work properly
- [ ] Data memory stores work properly
- [ ] Memory timing meets requirements

### Performance Verification

#### ✅ Timing Analysis
- [ ] Critical path meets timing requirements
- [ ] Clock frequency achieves target
- [ ] Setup/hold times satisfied
- [ ] No combinational loops present

#### ✅ Throughput Analysis
- [ ] CPI close to 1.0 for normal operation
- [ ] Hazard penalties within expected range
- [ ] Branch prediction effectiveness measured
- [ ] Memory latency impact quantified

---

## Performance Analysis

### Key Metrics

#### Cycles Per Instruction (CPI)
```
CPI = Total_Cycles / Total_Instructions
Target: 1.0 - 1.3 (including hazard penalties)
```

#### Instruction Throughput
```
IPC = 1 / CPI = Instructions / Total_Cycles
Target: 0.77 - 1.0 instructions per cycle
```

#### Hazard Impact
```
Hazard_Penalty = Stall_Cycles / Total_Cycles
Target: < 20% for typical programs
```

### Benchmark Programs

#### Simple Arithmetic Test
```assembly
# 10 instructions, minimal hazards
addi x1, x0, 1
addi x2, x0, 2
add x3, x1, x2
addi x4, x0, 4
add x5, x3, x4
# Expected: ~10-12 cycles, CPI ≈ 1.2
```

#### Memory-Intensive Test
```assembly
# Load/store with dependencies
addi x1, x0, 100
sw x1, 0(x0)
lw x2, 0(x0)
add x3, x2, x1
# Expected: Higher CPI due to load-use hazards
```

### Waveform Analysis Points

When reviewing waveforms, verify:
1. **Clock edges**: All state changes on positive edge
2. **Reset behavior**: All registers initialize to zero
3. **Pipeline progression**: Instructions move through stages
4. **Control signals**: Correct generation and timing
5. **Data paths**: Values flow correctly through pipeline
6. **Hazard detection**: Stalls and forwards occur when needed
7. **Branch handling**: Pipeline flushes on branch taken

### Common Issues and Debugging

#### Typical Problems
1. **Incorrect forwarding**: Check hazard unit logic
2. **Pipeline stalls**: Verify stall conditions
3. **Control signal timing**: Check pipeline register updates
4. **Memory interface**: Verify address/data timing
5. **Branch prediction**: Check flush logic

#### Debugging Techniques
1. Add debug outputs to key modules
2. Use $display statements for state monitoring
3. Compare expected vs actual register values
4. Trace instruction execution through pipeline
5. Verify control signal generation

This comprehensive testing framework ensures the RISC-V pipeline processor operates correctly across all supported instructions and operating conditions.
