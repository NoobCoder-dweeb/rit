# ritsim

`ritsim` is the C++17 foundation of a small x86-64 instruction simulator. This
first increment provides a deterministic random-program generator, a headless
interpreter, and a simulated four-level page table. It does not decode or run
native x86 machine-code bytes.

## Build and test

```sh
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

No third-party libraries are required.

## Generate a test

```sh
./build/ritsim --seed 0x12345 --count 0x100 --output test.rit
```

Without `--seed`, the program obtains a seed from `std::random_device`; the
seed is included in the generated program so the test can be reproduced.
Supported generated forms are 64-bit `mov`, `add`, `xor`, and memory `mov`
operations. The simulator has eight general-purpose registers and models the
carry, zero, sign, and overflow flags for the supported arithmetic operations.
The `add` and `xor` register-immediate forms use sign-extended 32-bit
immediates, matching their 64-bit x86 encodings.
Execution is bounded by the generated instruction count and the supplied
maximum-step limit.

Optional reports are always written to the file path passed to the option:

```sh
./build/ritsim --seed 0x12345 --count 0x100 \
  --disassemble disassembly.txt \
  --physical-addresses page-walks.txt
```

`--show-physical-addresses FILE` is an alias for
`--physical-addresses FILE`. The physical-address report describes this
simulator's deterministic, simulated 4-level page walk and physical frames; it
does not represent host machine addresses or claim hardware-level fidelity.
Only aligned 64-bit memory accesses are supported in this increment.
The disassembly option writes an assembly-style rendering of the generated
instruction representation; it does not decode native x86 machine-code bytes.

## Instruction architecture

- `InstructionType` identifies an instruction independently of its behavior.
- Each supported instruction has a handler under `src/instructions/`. A
  handler owns its operand generation, execution semantics, and disassembly.
- `InstructionHandler` is the common interface; the registry resolves an
  `InstructionType` to its handler. The simulator, generator, and formatter
  delegate to that interface instead of switching on instruction types.
- To add an instruction, add its handler and factory registration, add the
  implementation source to CMake, and cover generation, execution, and
  disassembly in the tests.
- `AddressTranslator` is the translation strategy interface;
  `SimulatedPageTable` provides the current four-level implementation.
- `ProgramGenerator` uses a fixed SplitMix64 sequence. Generator version is
  incremented when generated test sequences change.

## Tests

The dependency-free test executable is registered with CTest. It covers
handler registration, deterministic generation, instruction semantics,
disassembly, page walks, and report files.
