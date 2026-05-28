# C⏚ Examples

Example projects and tutorials for [C⏚](https://github.com/Neosyn-Logic/neosyn-studio) (C-Ground), a C-like hardware description language that compiles to Verilog and VHDL for FPGA development.

## Prerequisites

- [VS Code](https://code.visualstudio.com/) with the [C⏚ extension](https://marketplace.visualstudio.com/items?itemName=neosyn.neosyn-cg)

## Getting started

```bash
git clone https://github.com/Neosyn-Logic/cg-examples.git
```

Open any project folder in VS Code. The C⏚ extension provides:

- Syntax highlighting and error checking
- **Simulate** — run bytecode simulation with VCD waveform output
- **Generate HDL** — produce Verilog or VHDL from C⏚ source

## Project structure

```
examples/
  Arithmetic/       Counter pipeline, GCD algorithm
  HelloWorld/       7-segment display driver
  LED/              Blinking LED patterns
  StructDemo/       Nested structs, arrays of struct, struct ports
testing/
  SimpleCounter/    Basic counter with test property
  SimpleAdder/      Multi-input adder test
  SimpleDoubler/    Input/output push port test
  SimpleFSM/        Finite state machine with fence
  SimpleBoolTest/   Boolean flag output test
  SimpleMemTest/    RAM read/write with built-in entity
  DirectWriteTest/  Cross-task port communication
```

Each project follows the standard C⏚ layout:

```
ProjectName/
  src/
    com/neosyn/<package>/
      EntityName.cg
```

## Language overview

C⏚ has three building blocks:

| Keyword   | Purpose |
|-----------|---------|
| `task`    | A sequential or combinational hardware block with ports, state, and behavior |
| `network` | A structural container that instantiates tasks and connects them with dataflow |
| `bundle`  | A shared collection of constants, type aliases, and lookup tables |

### Tasks

A task describes a hardware block. It has ports for communication, state variables for internal storage, and `setup()` / `loop()` methods that define its behavior.

```cg
task BlankingLed {
    out bool led;

    bool switch;

    void loop() {
        switch = !switch;
        led.write(switch);
    }
}
```

Every clock cycle, `loop()` runs: it toggles the internal `switch` variable and writes the result to the `led` output port.

### Networks

A network instantiates tasks and connects them together using dataflow:

```cg
network HelloWorld {
    import com.neosyn.hello.WordToDisplay;
    import com.neosyn.hello.DriverSegment;

    wordToDisplay = new WordToDisplay();

    driverSegment = new DriverSegment();
    driverSegment.reads(wordToDisplay.character);

    printer = new task {
        void loop() {
            print(driverSegment.seg.read());
        }
    };
}
```

`driverSegment.reads(wordToDisplay.character)` connects the `character` output of `wordToDisplay` to the `character` input of `driverSegment`. The inline `printer` task reads `seg` directly for simulation output.

### Port interfaces

Ports can use different communication protocols:

| Keyword   | Behavior |
|-----------|----------|
| *(bare)*  | Directly wired, always available (default) |
| `push`    | Valid for one cycle, fire-and-forget |
| `stream`  | Holds valid until consumed, with backpressure |
| `confirm` | Delivery confirmation with acknowledgement |

```cg
task SimpleDoubler {
    sync {
        in u8 input;
        out u8 output;
    }

    void loop() {
        u8 v = input.read;
        output.write(v * 2);
    }
}
```

### Finite state machines

Tasks with blocking operations (port reads, `fence`, `idle()`) automatically become FSMs. The compiler infers states from the control flow:

```cg
task Gcd {
    in sync u16 a, sync b;
    out sync u16 z;

    u16 x, y;

    void loop() {
        x = a.read;
        y = b.read;

        while (y != 0) {
            if (x > y) {
                x -= y;
            } else {
                y -= x;
            }
        }

        z.write(x);
    }
}
```

This generates a multi-state FSM: wait for inputs, iterate the subtraction loop, then output the result.

## Examples

### HelloWorld

A 7-segment display driver demonstrating task composition:

- `WordToDisplay` outputs the characters of `"Hello world!"` one per cycle
- `DriverSegment` converts each character to a 7-segment code using a lookup table from the `Hello` bundle
- `HelloWorld` network connects them and prints the result

### LED

Two LED examples:

- `BlankingLed` toggles a boolean output every cycle
- `LedDriver` outputs an 8-bit LED chase pattern sequence
- `TestBlanking` wraps `BlankingLed` in a network testbench

### Arithmetic

**Counter pipeline** (`TopCounter` network):

```
Counter → Clip → Compare → Digit
```

- `Counter` outputs an incrementing 9-bit value
- `Clip` limits values to 6 bits (saturates at 63)
- `Compare` detects changes between raw and clipped values
- `Digit` maps counter values to 15-segment display patterns

Shared type definitions live in the `Definitions` bundle:

```cg
bundle Definitions {
    typedef u9 count_t;
    typedef u6 clipped;
}
```

**GCD** (`Gcd` task): computes the greatest common divisor of two 16-bit inputs using the subtractive algorithm. `Gcd_top` network provides a test harness with stimulus (64, 48) and expected result (16).

### StructDemo

A packet-pipeline tour of C⏚ **structs** (`driver` → `processor`, over a `stream` struct port):

- **nested structs** — `Packet` embeds a `Header` struct
- **arrays of struct** — `Packet batch[3]` with per-element field access (`batch[i].hdr.src`)
- **whole-struct copy** — `Packet copy = r;`
- **non-bare struct ports** — a `stream Packet` carried across two tasks, where one shared handshake gates the whole struct for atomic transfer

```cg
struct Header { u8 src; u8 dst; }
struct Packet { Header hdr; u16 payload; }
```

Run it with **Fast Sim**: `driver` builds a batch of packets, streams three of them through `processor`, and prints each round-tripped packet's nested fields before terminating.

## Simulation tests

Tests use the `test` property to define expected port values. The simulation framework automatically generates stimulus and checks outputs.

### SimpleCounter

Outputs an incrementing count and verifies the sequence:

```cg
task SimpleCounter {
    properties {
        test: { value: [1, 2, 3, 4, 5] }
    }

    sync {
        out u8 value;
    }

    u8 count;

    void setup() {
        count = 0;
    }

    void loop() {
        count = count + 1;
        value.write(count);
    }
}
```

### SimpleAdder

Tests a task with multiple inputs:

```cg
properties {
    test: {
        a: [1, 2, 3, 4, 5],
        b: [10, 20, 30, 40, 50],
        sum: [11, 22, 33, 44, 55]
    }
}
```

### SimpleDoubler

Reads input values and outputs their double. Demonstrates push port input/output with testbench verification.

### SimpleFSM

Uses a `while` loop and `fence` to output a sequence across multiple clock cycles:

```cg
void loop() {
    while (counter < 5) {
        value.write(counter);
        fence;
        counter++;
    }
}
```

`fence` inserts an explicit clock cycle boundary, creating a multi-state FSM.

### SimpleBoolTest

Outputs alternating `true`/`false` values. Verifies boolean port handling.

### SimpleMemTest

A network that instantiates a built-in RAM and tests read/write operations:

```cg
network SimpleMemTest {
    properties {
        test: { terminate: "simpleTester.finished" }
    }

    ram = new std.mem.SinglePortRAM({size: 8, width: 16});

    simpleTester = new task {
        // Writes to all 8 addresses, reads them back, then terminates
        ...
    };
}
```

The `test.terminate` property stops simulation when the `finished` flag becomes true.

### DirectWriteTest

Demonstrates cross-task port communication where one inline task writes directly to another's input port:

```cg
// tester writes to processor's input port
processor.input_val.write(5);

// Then reads the result
u8 result = processor.output_val.read();
```

## Key concepts

### `setup()` and `loop()`

- `setup()` runs once at initialization (cycle 0)
- `loop()` runs every clock cycle after setup

### Timing

- `fence;` — explicit single-cycle boundary between operations
- `idle(n);` — wait for `n` clock cycles
- Reading a push/stream port blocks until data is available

### Test properties

Two formats for defining simulation tests:

```cg
// Output-only: verify a single output port
properties {
    test: { portName: [val1, val2, val3] }
}

// Input/output: provide stimulus and verify results
properties {
    test: {
        inputPort: [in1, in2, in3],
        outputPort: [out1, out2, out3]
    }
}

// Network termination: stop when a flag becomes true
properties {
    test: { terminate: "instance.field" }
}
```

## License

[MIT](LICENSE)

## Links

- [Neosyn Studio](https://github.com/Neosyn-Logic/neosyn-studio) — C⏚ compiler and IDE
- [neosyn.io](https://neosyn.io) — Neosyn website
