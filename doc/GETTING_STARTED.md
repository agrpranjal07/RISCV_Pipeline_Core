# Getting Started Guide

This guide will help you set up, compile, and run the RISC-V pipeline processor on your sys### 3. Compile and Run
```bash
# Navigate to src directory
cd RISCV_Pipeline_Core/src

# Compile the testbench (includes all necessary files)
iverilog -o pipeline_sim pipeline_tb.v

# Run simulation
./pipeline_sim

# Expected output:
# VCD info: dumpfile dump.vcd opened for output.
# Simulation completed successfully.
```able of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Project Structure](#project-structure)
5. [Compilation and Simulation](#compilation-and-simulation)
6. [Customizing Test Programs](#customizing-test-programs)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software

#### Verilog Simulator
Choose one of the following:
- **Icarus Verilog** (Open source, recommended for beginners)
- **ModelSim** (Commercial, advanced features)
- **Vivado** (Xilinx, includes synthesis)
- **Verilator** (High-performance, C++ based)

#### Waveform Viewer
- **GTKWave** (Open source, works with all simulators)
- **ModelSim** (Built-in waveform viewer)
- **Vivado** (Built-in waveform viewer)

#### Text Editor/IDE (Optional)
- **VS Code** with Verilog extensions
- **Vim/Emacs** with Verilog syntax highlighting
- **Any text editor**

### System Requirements
- **OS**: Linux, macOS, or Windows (WSL recommended)
- **RAM**: 2GB minimum, 4GB recommended
- **Storage**: 100MB for project files
- **CPU**: Any modern processor

---

## Installation

### Ubuntu/Debian Systems
```bash
# Update package list
sudo apt update

# Install Icarus Verilog and GTKWave
sudo apt install iverilog gtkwave

# Verify installation
iverilog -V
gtkwave --version
```

### CentOS/RHEL/Fedora Systems
```bash
# For CentOS/RHEL (enable EPEL first)
sudo yum install epel-release
sudo yum install iverilog gtkwave

# For Fedora
sudo dnf install iverilog gtkwave
```

### macOS Systems
```bash
# Using Homebrew
brew install icarus-verilog gtkwave

# Using MacPorts
sudo port install iverilog gtkwave
```

### Windows Systems
```bash
# Using WSL (Windows Subsystem for Linux)
wsl --install Ubuntu
# Then follow Ubuntu instructions above

# Alternative: Download Windows binaries
# Icarus Verilog: http://bleyer.org/icarus/
# GTKWave: http://gtkwave.sourceforge.net/
```

---

## Quick Start

### 1. Get the Project
```bash
# Clone repository (if using Git)
git clone <repository-url>
cd RISCV_Pipeline_Core

# Or download and extract ZIP file
unzip RISCV_Pipeline_Core.zip
cd RISCV_Pipeline_Core-main
```

### 2. Verify File Structure
```bash
# Check that all files are present
ls -la src/
ls -la doc/

# Key files should include:
# src/Pipeline_Top.v, src/pipeline_tb.v, src/memfile.hex
```

### 3. Compile and Run
```bash
# Navigate to project root
cd RISCV_Pipeline_Core

# Compile all Verilog files
iverilog -o pipeline_sim src/pipeline_tb.v src/*.v

# Run simulation
./pipeline_sim

# Expected output:
# VCD info: dumpfile pipeline.vcd opened for output.
# Simulation completed successfully.
```

### 4. View Waveforms
```bash
# Open GTKWave with saved configuration
gtkwave pipeline.gtkw

# Or open VCD file directly  
gtkwave dump.vcd
```

---

## Project Structure

### Directory Layout
```
RISCV_Pipeline_Core/
├── README.md                   # Project overview
├── LICENSE                     # License information
├── src/                        # Verilog source files
│   ├── Pipeline_Top.v          # Top-level module
│   ├── Fetch_Cycle.v           # IF stage
│   ├── Decode_Cycle.v          # ID stage  
│   ├── Execute_Cycle.v         # EX stage
│   ├── Memory_Cycle.v          # MEM stage
│   ├── Writeback_Cycle.v       # WB stage
│   ├── Control_Unit_Top.v      # Control unit
│   ├── Hazard_unit.v           # Hazard detection
│   ├── ALU.v                   # Arithmetic logic unit
│   ├── ALU_Decoder.v           # ALU control
│   ├── Main_Decoder.v          # Main control decoder
│   ├── Register_File.v         # Register file
│   ├── PC.v                    # Program counter
│   ├── PC_Adder.v              # PC increment logic
│   ├── Data_Memory.v           # Data memory
│   ├── Instruction_Memory.v    # Instruction memory
│   ├── Sign_Extend.v           # Sign extension
│   ├── Mux.v                   # Multiplexers
│   ├── pipeline_tb.v           # Main testbench
│   ├── memfile.hex             # Test program
│   └── pipeline.gtkw           # GTKWave save file
└── doc/                        # Documentation
    ├── ARCHITECTURE.md         # Architecture details
    ├── INSTRUCTION_SET.md      # Supported instructions
    ├── TESTING.md              # Testing guide
    ├── GETTING_STARTED.md      # This file
    └── Top Architecture.png    # Architecture diagram
```

### Key File Descriptions

| File | Purpose |
|------|---------|
| `Pipeline_Top.v` | Top-level module connecting all components |
| `pipeline_tb.v` | Testbench for simulation |
| `memfile.hex` | Test program in hexadecimal format |
| `pipeline.gtkw` | GTKWave configuration for waveform viewing |
| `Control_Unit_Top.v` | Generates control signals for instructions |
| `Hazard_unit.v` | Detects and resolves pipeline hazards |

---

## Compilation and Simulation

### Basic Compilation
```bash
# Simple compilation
iverilog -o sim src/pipeline_tb.v src/*.v

# Compilation with debug information
iverilog -g2005-sv -o sim src/pipeline_tb.v src/*.v

# Check for syntax errors only
iverilog -t null src/*.v
```

### Advanced Compilation Options
```bash
# Include specific directories
iverilog -I src/ -o sim src/pipeline_tb.v src/*.v

# Define preprocessor macros
iverilog -D DEBUG=1 -o sim src/pipeline_tb.v src/*.v

# Generate dependency information
iverilog -M src/pipeline_tb.v src/*.v > dependencies.txt
```

### Running Simulations

#### Basic Simulation
```bash
# Run with default settings
./sim

# Run with VCD output (for waveforms)
./sim +dump
```

#### Simulation with Parameters
```bash
# Modify testbench for longer simulation
# Edit pipeline_tb.v:
# Change: #1000; 
# To:     #5000;  // Run for more cycles

# Recompile and run
iverilog -o sim src/pipeline_tb.v src/*.v
./sim
```

#### Batch Mode Simulation
```bash
# Create script for automated testing
cat > run_sim.sh << 'EOF'
#!/bin/bash
echo "Starting RISC-V Pipeline Simulation..."
iverilog -o sim src/pipeline_tb.v src/*.v
if [ $? -eq 0 ]; then
    echo "Compilation successful, running simulation..."
    ./sim
    echo "Simulation completed."
else
    echo "Compilation failed!"
    exit 1
fi
EOF

chmod +x run_sim.sh
./run_sim.sh
```

---

## Customizing Test Programs

### Understanding memfile.hex

The test program is stored in `src/memfile.hex` in hexadecimal format:

```hex
20020005    // addi x2, x0, 5      # x2 = 5
20030003    // addi x3, x0, 3      # x3 = 3
00230133    // add x2, x2, x3      # x2 = x2 + x3 = 8
00000013    // addi x0, x0, 0      # nop (no operation)
```

### Creating Custom Test Programs

#### Method 1: Hand Assembly
```bash
# Create a simple test program
cat > custom_test.hex << 'EOF'
20020005    # addi x2, x0, 5
20030003    # addi x3, x0, 3  
00230133    # add x2, x2, x3
40300133    # sub x2, x0, x3
EOF

# Replace default test
cp custom_test.hex src/memfile.hex
```

#### Method 2: Using RISC-V Assembler
```bash
# Install RISC-V toolchain (if available)
sudo apt install gcc-riscv64-unknown-elf

# Create assembly file
cat > test.s << 'EOF'
.text
.globl _start
_start:
    addi x2, x0, 5      # x2 = 5
    addi x3, x0, 3      # x3 = 3
    add x4, x2, x3      # x4 = x2 + x3
    sw x4, 0(x0)        # Store result
    lw x5, 0(x0)        # Load result
EOF

# Assemble and convert to hex
riscv64-unknown-elf-as -o test.o test.s
riscv64-unknown-elf-objcopy -O binary test.o test.bin
hexdump -v -e '1/4 "%08x" "\n"' test.bin > src/memfile.hex
```

### Test Program Examples

#### Arithmetic Operations
```hex
# Test all arithmetic operations
20020005    # addi x2, x0, 5      # x2 = 5
20030003    # addi x3, x0, 3      # x3 = 3
00230133    # add x4, x2, x3      # x4 = x2 + x3 = 8
40230233    # sub x4, x2, x3      # x4 = x2 - x3 = 2
00237333    # and x6, x2, x3      # x6 = x2 & x3 = 1
00236333    # or x6, x2, x3       # x6 = x2 | x3 = 7
```

#### Memory Operations
```hex
# Test load/store
20020064    # addi x2, x0, 100    # x2 = 100 (address)
2003002A    # addi x3, x0, 42     # x3 = 42 (data)
00302023    # sw x3, 0(x2)        # mem[100] = 42
00002203    # lw x4, 0(x2)        # x4 = mem[100] = 42
```

#### Branch Operations
```hex
# Test conditional branch
20020005    # addi x2, x0, 5      # x2 = 5
20030005    # addi x3, x0, 5      # x3 = 5
00230463    # beq x2, x3, 8       # if x2 == x3, jump ahead 8 bytes
20040001    # addi x4, x0, 1      # x4 = 1 (skipped if branch taken)
20050002    # addi x5, x0, 2      # x5 = 2 (target of branch)
```

---

## Troubleshooting

### Common Issues and Solutions

#### Compilation Errors

**Error**: `command not found: iverilog`
```bash
# Solution: Install Icarus Verilog
sudo apt install iverilog
```

**Error**: `syntax error in module`
```bash
# Solution: Check Verilog syntax
# Common issues:
# - Missing semicolons
# - Incorrect module declarations
# - Undefined signals

# Debug specific file:
iverilog -t null src/specific_file.v
```

**Error**: `undefined module`
```bash
# Solution: Ensure all files are included
iverilog -o sim src/pipeline_tb.v src/*.v

# Check that all .v files are in src/ directory
ls src/*.v
```

#### Simulation Issues

**Issue**: Simulation runs but produces no output
```bash
# Solution: Check testbench timing
# Edit pipeline_tb.v, increase simulation time:
#1000 → #5000

# Ensure VCD dumping is enabled
$dumpfile("pipeline.vcd");
$dumpvars(0, pipeline_tb);
```

**Issue**: GTKWave shows no signals
```bash
# Solution: Check VCD file generation
ls -la *.vcd

# If no VCD file, modify testbench:
initial begin
    $dumpfile("pipeline.vcd");
    $dumpvars(0, pipeline_tb);
end
```

**Issue**: Wrong simulation results
```bash
# Solution: Verify test program
# Check memfile.hex format and content
# Ensure instructions are properly encoded
hexdump -C src/memfile.hex
```

#### Waveform Viewing Issues

**Issue**: GTKWave not opening
```bash
# Solution: Install GTKWave
sudo apt install gtkwave

# Try alternative viewers:
# - Use built-in simulator viewer
# - Convert VCD to other formats
```

**Issue**: Signals not visible in GTKWave
```bash
# Solution: 
# 1. Expand hierarchy in SST window
# 2. Add signals from the design
# 3. Use the included .gtkw save file:
gtkwave src/pipeline.gtkw
```

### Performance Tuning

#### Slow Simulation
```bash
# Use Verilator for faster simulation
sudo apt install verilator

# Compile with Verilator
verilator --cc --exe --build -j 0 -Wall pipeline_tb.v src/*.v
```

#### Memory Issues
```bash
# Reduce simulation time for large programs
# Edit testbench to run fewer cycles
# Use smaller test programs for debugging
```

### Getting Help

#### Documentation
- Read the [Architecture Guide](ARCHITECTURE.md)
- Check the [Instruction Set Reference](INSTRUCTION_SET.md)  
- Review [Testing Procedures](TESTING.md)

#### Online Resources
- **Icarus Verilog Wiki**: http://iverilog.wikia.com/
- **GTKWave Documentation**: http://gtkwave.sourceforge.net/
- **RISC-V ISA Manual**: https://riscv.org/specifications/

#### Debug Strategies
1. Start with simple test programs
2. Use $display statements for debugging
3. Check intermediate signals in waveforms
4. Verify each pipeline stage independently
5. Compare with expected behavior

---

## Next Steps

After successfully running the basic simulation:

1. **Explore the Architecture**: Read [ARCHITECTURE.md](ARCHITECTURE.md) for detailed design information
2. **Study Instructions**: Review [INSTRUCTION_SET.md](INSTRUCTION_SET.md) for supported operations
3. **Run Tests**: Follow [TESTING.md](TESTING.md) for comprehensive verification
4. **Modify Design**: Experiment with additional features or optimizations
5. **Synthesis**: Try synthesizing the design for FPGA implementation

### Advanced Topics
- Add cache memory
- Implement branch prediction
- Add floating-point support
- Optimize for higher clock frequencies
- Add debug interfaces

This guide should help you get started with the RISC-V pipeline processor. For detailed technical information, refer to the other documentation files in the `doc/` directory.
