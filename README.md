# C⏚ Examples

Example projects and tutorials for the [C⏚ hardware description language](https://github.com/Neosyn-Logic/neosyn-studio).

C⏚ (C-Ground) is a C-like language for designing digital hardware that compiles to Verilog/VHDL for FPGA development.

## Prerequisites

- [VS Code](https://code.visualstudio.com/) with the [C⏚ extension](https://marketplace.visualstudio.com/items?itemName=neosyn.neosyn-cg)

## Examples

### Getting Started

| Project | Description |
|---------|-------------|
| **HelloWorld** | 7-segment display driver — basic task and network composition |
| **LED** | Blinking LED with configurable timing — combinational and synchronous tasks |
| **Arithmetic** | Counter and GCD — FSM patterns, multi-digit arithmetic |

### Simulation Tests

| Project | Description |
|---------|-------------|
| **SimpleCounter** | Counter with testbench — basic `test: {port: [values]}` syntax |
| **SimpleAdder** | 8-bit adder — input/output port testing |
| **SimpleDoubler** | Value doubler — push port communication |
| **SimpleFSM** | Finite state machine — multi-state task with transitions |
| **SimpleBoolTest** | Boolean logic — flag-based control flow |
| **SimpleMemTest** | RAM read/write — built-in entity (`std.mem.SinglePortRAM`) usage |
| **DirectWriteTest** | Cross-task write — inline task port communication |

## Quick Start

```bash
git clone https://github.com/Neosyn-Logic/cg-examples.git
```

Open any project folder in VS Code. The C⏚ extension provides:
- Syntax highlighting and error checking
- **Simulate** — run bytecode simulation with VCD waveform output
- **Generate HDL** — produce Verilog/VHDL from C⏚ source

## Example: SimpleCounter

```cg
package com.neosyn.test.counter;

task SimpleCounter {
    properties {
        test: { value: [1, 2, 3, 4, 5] }
    }

    out u8 value;

    u8 count;

    void loop() {
        count++;
        value.write(count);
    }
}
```

## Project Layout

```
examples/
  Arithmetic/       Counter, GCD
  HelloWorld/       7-segment display
  LED/              Blinking LED
testing/
  SimpleCounter/    Basic counter test
  SimpleMemTest/    RAM test
  ...
```

Each project follows the standard C⏚ structure:

```
ProjectName/
  src/
    com/neosyn/<package>/
      EntityName.cg
```

## License

[MIT](LICENSE)

## Links

- [Neosyn Studio](https://github.com/Neosyn-Logic/neosyn-studio) — C⏚ compiler and IDE
- [neosyn.io](https://neosyn.io) — Neosyn website
