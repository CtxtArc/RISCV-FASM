# Getting Started

## Prerequisites

To build and use Kdex, your environment requires:

- **Build System**: GNU Make
- **Compiler**: A standard C11 compiler (`gcc` or `clang`)
- **RISC-V Toolchain**: `riscv64-elf-gcc` and `riscv64-elf-ld` (required for linking ELF outputs and standard library testing)
- **Emulator**: `qemu-system-riscv32` and `qemu-riscv32` (required for test execution)

## Building from Source

To compile the assembler, run `make` from the repository root:

```bash
make
```

This will produce the `riscv-fasm` executable. To run the automated test suite and ensure your toolchain is correctly configured:

```bash
make test
```

## Command Line Interface

Kdex is invoked from the command line, accepting a primary input file and output configuration.

```bash
./riscv-fasm [OPTIONS] <input.s>
```

### Options

| Flag | Long Flag | Description |
| ---- | --------- | ----------- |
| `-f` | `--format` | Specifies the output architecture format. Acceptable values are `elf` (relocatable object file) or `flat` (raw binary blob). Defaults to `flat`. |
| `-o` | `--output` | Specifies the output filename. |
| `-q` | `--quiet` | Suppresses the UI banner, compilation summary, and non-error output. Recommended for CI/CD environments. |

## Your First Program

Kdex programs can be built as raw memory blobs. By default, flat binaries assemble sequentially from an origin address.

**hello.s:**
```assembly
.org 0x80000000

_start:
    li a0, 0        # Load immediate
    li a1, 42
    add a2, a0, a1
    j _start        # Infinite loop
```

Assemble this into a binary:
```bash
./riscv-fasm -f flat -o hello.bin hello.s
```

You can view the resulting machine code via a cross-objdump:
```bash
riscv64-elf-objdump -D -b binary -m riscv:rv32 hello.bin
```
