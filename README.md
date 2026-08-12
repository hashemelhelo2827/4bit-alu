# 4-bit ALU

> A compact 4-bit Arithmetic Logic Unit built entirely as a Quartus Prime schematic (BDF), combining AND, OR and add operations behind a 2-bit MUX select.

Targeted at the Cyclone V `5CGXFC7C7F23C8` FPGA, built with **Quartus Prime 20.1 Lite Edition**.

## Features

- 4-bit **AND** and **OR** logic units
- 4-bit ripple-carry **adder** with carry-out flag
- Operation select via a 2-bit control input (`s[1..0]`)
- Carry input (`c0`) for chaining / subtraction support
- Reusable, separately-documented component blocks

## Project structure

```
alu_project/
├── alu.bdf           # Top-level schematic
├── alu.qpf           # Project file
├── alu.qsf           # Settings & file assignments
├── Waveform.vwf      # ModelSim simulation waveform
├── components/       # Reusable building blocks (bdf + bsf)
│   ├── 4bit_and.bdf / .bsf
│   ├── 4bit_or.bdf  / .bsf
│   ├── 4bit_adder.bdf / .bsf
│   └── 1bit_adder.bdf / .bsf
└── output_files/     # Generated compile outputs (alu.sof, reports)
```

## Prerequisites

- Quartus Prime 20.1 Lite Edition (or compatible) with Cyclone V device support.

> [!NOTE]
> The `.bsf` symbol files live in `components/`, so the Quartus Block Editor renders these blocks as generic boxes. Compilation and simulation are unaffected.

## Getting started

1. Open Quartus Prime → **File → Open Project** → select `alu.qpf`.
2. Run **Processing → Start Compilation** (or press <kbd>Ctrl</kbd>+<kbd>L</kbd>).
3. The compiled bitstream is written to `output_files/alu.sof`.

The full flow (Analysis & Synthesis → Fitter → Assembler) completes with 0 errors.

## Architecture

```
alu (top level)
├── 4bit_and     # A[3..0] AND B[3..0]
├── 4bit_or      # A[3..0] OR  B[3..0]
├── 4bit_adder   # 4-bit ripple-carry adder
│   └── 4 × 1bit_adder
└── 4 × MUX41    # operation select (Quartus maxplus2 library part)
```

| Block          | Function                        | Ports                                     |
|----------------|---------------------------------|-------------------------------------------|
| `4bit_and`     | 4-bit AND                       | `A[3..0]`, `B[3..0]` → `o1`..`o4`         |
| `4bit_or`      | 4-bit OR                        | `A[3..0]`, `B[3..0]` → `o1`..`o4`         |
| `4bit_adder`   | 4-bit ripple-carry adder        | `A[3..0]`, `B[3..0]`, `pin_name1` (cin) → `o1`..`o4`, `cflag` |
| `1bit_adder`   | 1-bit full adder                | `a`, `b`, `c` → `s`, `cout`               |
| `MUX41`        | 4:1 multiplexer (built-in)      | `D0`..`D3`, `S0`, `S1`, `INH` → `Q`       |

## Inputs & outputs

| Signal     | Direction | Description                              |
|------------|-----------|------------------------------------------|
| `s[1..0]`  | Input     | Operation select                         |
| `A[3..0]`  | Input     | Operand A                                |
| `b[3..0]`  | Input     | Operand B                                |
| `c0`       | Input     | Carry-in                                 |
| `pin_name2..5` | Output | 4-bit result                          |
| `ccflag`   | Output    | Carry flag                               |

## Simulation

Open `Waveform.vwf` and run ModelSim (Verilog). The project is pre-configured in `alu.qsf`:

```tcl
set_global_assignment -name EDA_SIMULATION_TOOL "ModelSim (Verilog)"
set_global_assignment -name EDA_OUTPUT_DATA_FORMAT "VERILOG HDL" -section_id eda_simulation
```

## License

For academic and personal use.