# RTL Design and Synthesis

A collection of Verilog designs and experiments covering the RTL-to-gate-level flow: writing RTL, simulating it, synthesizing it, and comparing the synthesized netlist against the original design. The focus is on understanding how coding style affects what the synthesizer generates.

## Toolchain

| Stage | Tool |
|---|---|
| RTL simulation | Icarus Verilog (`iverilog`) |
| Waveform viewing | GTKWave |
| Logic synthesis | Yosys |
| Standard-cell library | SKY130 (`library/`) |
| GLS (gate-level simulation) | Icarus Verilog with the synthesized netlist |

## Repository Structure

Each folder holds the Verilog design, its testbench, and the related synthesis output for the experiment.

### Combinational logic and multiplexers
| Folder | Description |
|---|---|
| `good_mux` | 2:1 mux written with a clean `if/else` style |
| `bad_mux` | Mux with a problematic coding style, kept for comparison |
| `ternary_operator_mux` | Mux implemented with the `?:` operator |
| `mux_generate` | Parameterized mux built with a `generate` loop |
| `demux_case` | Demux using a `case` statement |
| `demux_generate` | Demux built with a `generate` loop |

### Conditional coding styles (latch inference)
| Folder | Description |
|---|---|
| `incomplete_if`, `incomplete_if2` | `if` statements without a full `else`, leading to inferred latches |
| `incomplete_case` | `case` with missing branches |
| `partial_case` | `case` where only some outputs are assigned |
| `complete_case` | Fully specified `case` (no latches) |
| `bad_case` | Poorly written `case` for comparison |

### Sequential logic
| Folder | Description |
|---|---|
| `dff_asyncres` | D flip-flop with asynchronous reset |
| `dff_async_set` | D flip-flop with asynchronous set |
| `dff_syncres` | D flip-flop with synchronous reset |
| `dff_asyncres_syncres` | D flip-flop with both asynchronous and synchronous reset |
| `dff_const` | Flip-flop with constant inputs, used to observe synthesis optimization |
| `counter` | Counter design |
| `blocking_caveat` | Blocking vs. non-blocking assignment pitfalls |

### Arithmetic
| Folder | Description |
|---|---|
| `mult_2`, `mult_8` | Multiplication by constants (2 and 8), showing how synthesis reduces them to wiring/shifts |
| `ripple_carry_adder` | Ripple-carry adder |

### Optimization and hierarchy
| Folder | Description |
|---|---|
| `opt_check`, `opt_check2`, `opt_check3`, `opt_check4` | Small designs used to observe combinational/sequential optimizations in Yosys |
| `multiple_modules` | Hierarchical design with several modules |
| `multiple_module_opt` | Hierarchical design used to study optimization across module boundaries |

### Library
| Folder | Description |
|---|---|
| `library` | Standard-cell liberty file(s) used for technology mapping |

## How to Run

### 1. Simulate the RTL

```bash
cd <experiment_folder>
iverilog <design>.v <testbench>.v
./a.out
gtkwave <dumpfile>.vcd
```

### 2. Synthesize with Yosys

```bash
yosys
read_liberty -lib ../library/<liberty_file>.lib
read_verilog <design>.v
synth -top <top_module>
dfflibmap -liberty ../library/<liberty_file>.lib
abc -liberty ../library/<liberty_file>.lib
clean
flatten          # optional, for hierarchical designs
write_verilog -noattr <design>_net.v
show             # view the synthesized netlist
```

### 3. Gate-level simulation

```bash
iverilog <primitives>.v <sky130_cells>.v <design>_net.v <testbench>.v
./a.out
gtkwave <dumpfile>.vcd
```

Compare the GLS waveform with the RTL waveform to confirm the netlist matches the intended behavior.

## Key Takeaways

- Incomplete `if` and `case` statements infer latches; fully specify all branches or assign defaults.
- Use non-blocking assignments (`<=`) for sequential logic and blocking (`=`) for combinational logic.
- Synthesis simplifies constant logic aggressively, e.g. multiplication by a power of two becomes plain wiring.
- Yosys optimizes across the design when flattened, which changes the hierarchy of the resulting netlist.
- RTL and gate-level simulation should match; a mismatch usually points to a coding-style issue.

## Author

**Deva Prassad S**
