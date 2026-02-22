# Contributing to C⏚ Examples

Thank you for your interest in contributing to the C⏚ example projects!

## Getting Started

1. Install the [C⏚ VS Code extension](https://marketplace.visualstudio.com/items?itemName=neosyn.neosyn-cg) (or build from [neosyn-studio](https://github.com/Neosyn-Logic/neosyn-studio))
2. Clone this repository
3. Open any project folder in VS Code

## Project Structure

Each example is a standalone C⏚ project with this layout:

```
ProjectName/
  src/
    com/neosyn/.../
      EntityName.cg    # C⏚ source files
```

Generated files (`.ir/`, `verilog-gen/`, `sim/`, `*.vcd`) are gitignored.

## Adding a New Example

1. Create a new folder under the appropriate category (`examples/` or `testing/`)
2. Follow the package convention: `com.neosyn.<category>.<project>`
3. Include comments explaining the design
4. Make sure the example compiles and simulates correctly

## Submitting Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b add-my-example`)
3. Commit your changes
4. Open a Pull Request with a clear description

## Code Style

- Use meaningful names for tasks, networks, and ports
- Add comments for non-obvious logic
- Use the new port interface keywords: `push`, `stream`, `confirm`
- Include test properties where applicable (`properties { test: {...} }`)

## Questions?

Open an issue or contact us at info@neosyn.io.
