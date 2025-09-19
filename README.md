# RISCV_Pipeline_Core
A comprehensive 5-stage pipelined RISC-V processor implementation in Verilog, featuring hazard detection, forwarding, and support for the RV32I instruction set.

**Author**: Pranjal Agrawal ([@agrpranjal07](https://github.com/agrpranjal07))


# TOP Architecture
![Top Architecture](doc/Top%20Architecture.png)

## Features

- ✅ Complete RV32I instruction set support
- ✅ 5-stage pipeline (IF, ID, EX, MEM, WB)
- ✅ Data forwarding for hazard mitigation
- ✅ Branch prediction and control hazard handling
- ✅ Comprehensive testbench with waveform analysis

## Quick Start

```bash
# Navigate to source directory
cd src

# Compile and simulate
iverilog -o sim pipeline_tb.v && ./sim

# View waveforms  
gtkwave dump.vcd
```

## Documentation

- 📖 [Architecture Guide](doc/ARCHITECTURE.md) - Detailed design documentation
- 🚀 [Getting Started](doc/GETTING_STARTED.md) - Setup and simulation guide
- 📋 [Instruction Set](doc/INSTRUCTION_SET.md) - Supported RISC-V instructions
- 🧪 [Testing Guide](doc/TESTING.md) - Verification and testing procedures

## Project Structure

```
├── src/                    # Verilog source files
│   ├── Pipeline_Top.v      # Top-level processor
│   ├── Fetch_Cycle.v       # Instruction fetch stage
│   ├── Decode_Cycle.v       # Instruction decode stage
│   ├── Execute_Cycle.v     # Execution stage
│   ├── Memory_Cycle.v      # Memory access stage
│   ├── Writeback_Cycle.v   # Write-back stage
│   ├── Control_Unit_Top.v  # Control unit
│   ├── Hazard_unit.v       # Hazard detection unit
│   ├── ALU.v               # Arithmetic Logic Unit
│   ├── ALU_Decoder.v       # ALU control decoder
│   ├── Main_Decoder.v      # Main instruction decoder
│   ├── Register_File.v     # Register file implementation
│   ├── PC.v                # Program counter
│   ├── PC_Adder.v          # PC increment logic
│   ├── Data_Memory.v       # Data memory module
│   ├── Instruction_Memory.v # Instruction memory module
│   ├── Sign_Extend.v       # Sign extension unit
│   ├── Mux.v               # Multiplexer modules
│   ├── pipeline_tb.v       # Testbench
│   ├── pipeline.gtkw       # GTKWave save file
│   └── memfile.hex         # Test program memory
└── doc/                    # Documentation
    ├── ARCHITECTURE.md     # Detailed architecture guide
    ├── GETTING_STARTED.md  # Setup and usage guide
    ├── INSTRUCTION_SET.md  # Instruction set documentation
    ├── TESTING.md          # Testing and verification guide
    └── Top Architecture.png # Architecture diagram
```

## License

Licensed under the Apache License, Version 2.0. See LICENSE for full terms.
