# 4-bit ALU

> A 4-bit Arithmetic Logic Unit with a register file, built entirely as a Quartus Prime schematic (BDF) — clocked datapath driven by a 10-bit instruction word.

Targeted at the Cyclone V `5CGXFC7C7F23C8` FPGA, built with **Quartus Prime 20.1 Lite Edition**.

## Features

- **Register file** (`regfile`) built around 74x379 quad D flip-flop registers (with output enable)
- ALU core (`alu_core`) with 4-bit **AND**, **OR** and ripple-carry **adder**
- 2-bit operation select, carry input and **carry flag**
- Sequential datapath: `clk` + 10-bit instruction `i[9..0]` → operands `A`, `B`, result `C`, carry flag
- Reusable, separately-documented component blocks

## Project structure

```
alu_project/
├── alu.bdf           # Top-level schematic (top entity: alu)
├── alu.qpf           # Project file
├── alu.qsf           # Settings & file assignments
├── Waveform.vwf      # ModelSim simulation waveform
├── components/       # Reusable building blocks (bdf + bsf)
│   ├── regfile.bdf / .bsf
│   ├── alu_core.bdf / .bsf
│   ├── 4bit_and.bdf    4bit_or.bdf
│   ├── 4bit_adder.bdf  1bit_adder.bdf
│   ├── 4bit_mux.bdf   fanout4.bdf
└── legacy/           # Archived old combinational ALU (for reference)
└── output_files/     # Generated compile outputs (alu.sof, reports)
```

> [!NOTE]
> `legacy/` holds the previous combinational-only ALU design, kept in the repo for reference. The active design tracked here is the register-based ALU.

## Prerequisites

- Quartus Prime 20.1 Lite Edition (or compatible) with Cyclone V device support.

> [!NOTE]
> `.bsf` symbol files live in `components/`, so the Quartus Block Editor renders blocks as generic boxes. Compilation and simulation are unaffected.

## Getting started

1. Open Quartus Prime → **File → Open Project** → select `alu.qpf`.
2. Run **Processing → Start Compilation** (or press <kbd>Ctrl</kbd>+<kbd>L</kbd>).
3. The compiled bitstream is written to `output_files/alu.sof`.

The full flow (Analysis & Synthesis → Fitter → Assembler) completes with 0 errors.

## Architecture

```
alu (top level)
├── regfile      # 74379-based register file (clk, instruction-driven)
│   ├── 2 × 74379 register (built-in maxplus2)
│   ├── 4bit_mux, fanout4
├── alu_core     # ALU: AND / OR / ripple add with 2-bit select + carry
│   ├── 4bit_and, 4bit_or
│   ├── 4bit_adder
│   │   └── 4 × 1bit_adder
│   └── 4bit_mux
└── (built-in) 74379, enadff, MUX41 — Quartus maxplus2 library parts
```

| Block          | Function                                  | Ports                                        |
|----------------|-------------------------------------------|----------------------------------------------|
| `regfile`      | 74379 register file (sequential)          | `clk`, `i`-decoded selects, `data[3..0]` |
| `alu_core`     | ALU core (AND/OR/add + mux + carry)       | `A[3..0]`, `B[3..0]`, `S[1..0]`, `c0` → `O[3..0]`, `cflag` |
| `4bit_and`     | 4-bit AND                                 | `A[3..0]`, `B[3..0]` → `O[3..0]`            |
| `4bit_or`      | 4-bit OR                                  | `A[3..0]`, `B[3..0]` → `O[3..0]`            |
| `4bit_adder`   | 4-bit ripple-carry adder                  | `A[3..0]`, `B[3..0]`, `c0` → `O[3..0]`, `cflag` |
| `1bit_adder`   | 1-bit full adder                          | `a`, `b`, `c` → `s`, `cout`                 |
| `4bit_mux`     | 4:1 multiplexer (4-bit)                   | `D0..D3[3..0]`, `s[1..0]` → `O[3..0]`       |
| `fanout4`      | 4-bit fan-out buffer                      | `inpute[3..0]` → `o[3..0]`                  |

## Inputs & outputs

| Signal       | Direction | Description                            |
|--------------|-----------|----------------------------------------|
| `clk`        | Input     | System clock for the register file     |
| `i[9..0]`    | Input     | Instruction / control bus              |
| `A[3..0]`    | Output    | Operand A (from register file)         |
| `B[3..0]`    | Output    | Operand B (from register file)         |
| `C[3..0]`    | Output    | ALU result                             |
| `c__Flag`    | Output    | Carry flag                             |

## Simulation

Open `Waveform.vwf` and run ModelSim (Verilog). The project is pre-configured in `alu.qsf`:

```tcl
set_global_assignment -name EDA_SIMULATION_TOOL "ModelSim (Verilog)"
set_global_assignment -name EDA_OUTPUT_DATA_FORMAT "VERILOG HDL" -section_id eda_simulation
```

## License

For academic and personal use.