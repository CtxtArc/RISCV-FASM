<div align="center">

# Kdex RISC-V FASM

**A Macro-Assembly Language and Toolchain with Compiler-Like Capabilities**

![C](https://img.shields.io/badge/Language-C-blue.svg)
![Architecture](https://img.shields.io/badge/Architecture-RV32I-red.svg)
![Output](https://img.shields.io/badge/Output-ELF%20%7C%20Flat%20Binary-green.svg)
![License](https://img.shields.io/badge/License-MIT-purple.svg)

*Bridging the gap between raw hardware control and high-level programming logic.*

</div>

---

## Overview

Kdex (RISC-V FASM) is a RISC-V assembler and toolchain written entirely in C. It features a stack-based macro engine, an ELF relocation engine, and a minimal standard library. 

The project aims to provide high-level programming constructs—such as loops, nested conditionals, and dynamic data structures—while maintaining precise control over the CPU. Kdex supports both bare-metal environments via flat binaries and GNU Linker integration via ELF object files.

---

## Features

- **Dual-Mode Output:**
  - **Relocatable ELF (`-f elf`)**: Generates industry-standard object files. The relocation engine handles RISC-V specific relocations (`R_RISCV_HI20`, `LO12_I`, `CALL`, `JAL`) for integration with `gcc` and `ld`.
  - **Flat Binary (`-f flat`)**: Generates headerless machine code for embedded microcontrollers, bootloaders, or direct QEMU execution starting from a defined `.org` origin.
- **Macro System**: Supports logic nesting, variadic arguments, and scoped labels (`.loop_%u`) to prevent naming collisions.
- **Control-Flow Stack**: Provides high-level loop (`while`, `endwhile`) and conditional (`if`, `endif`) statements.
- **Kdex Standard Library (`kstdlib`)**: Includes register-preserving wrappers for Linux syscalls, covering printing (`kstdio`), file I/O (`kfile`), and memory operations (`kstring`).
- **Preprocessor**: Evaluates math expressions and character literals at compile-time, handles mutable variables (`=`), and supports recursive file inclusion (`.include`).

---

## Build & Requirements

### Dependencies
- `make` and a standard C compiler (`gcc` or `clang`)
- `riscv64-elf-gcc` / `riscv64-elf-ld` (for ELF linking)
- `qemu-system-riscv32` / `qemu-riscv32` (for execution testing)

### Build Commands
| Command | Description |
| --- | --- |
| `make` | Builds the `riscv-fasm` executable. |
| `make run FILE=test.s` | Assembles (flat) and executes a file in QEMU. |
| `make dump FILE=test.s` | Dumps the symbol table and binary layout for debugging. |
| `make test` | Runs the automated Kdex test suite. |

---

## CLI Usage

```bash
./riscv-fasm [options] input.s
```

| Flag | Long Flag | Description |
| --- | --- | --- |
| `-q` | `--quiet` | Suppress UI output. |
| `-f` | `--format` | Set output format (`elf` or `flat`). |
| `-o` | `--output` | Specify output filename. |

**Example (C-Interop Workflow):**
```bash
./riscv-fasm -f elf math.s -o math.o
riscv64-elf-gcc -march=rv32i -mabi=ilp32 -nostdlib main.c math.o -o program
```

---

## Syntax & Language Features

### Control-Flow Stack
The internal logic stack enables high-level conditionals and loops:

```assembly
    li t0, 20
    li t1, 20
    
    if eq, t0, t1
        print_str match_msg
    endif 
```

### Data Modeling & Structs
The mutable variable system allows for automatic memory layout management.

```assembly
struct Packet
    field ID, 4          ; ID = 0
    array DATA, 128      ; DATA = 4
    field CRC, 4         ; CRC = 132
endstruct Packet         ; Packet_SIZE = 136

.data
    my_packet: .space Packet_SIZE
```

### Anonymous Labels (`@@`, `@f`, `@b`)
Anonymous labels provide an alternative for short, local jumps to avoid symbol table pollution.

```assembly
    mv t0, a0         
@@:                   # Anchor
    lb t1, 0(t0)      
    beqz t1, @f       # Jump forward to the next @@
    addi t0, t0, 1    
    j @b              # Jump backward to the previous @@
@@:                   
    sub a0, t0, a0    
    ret
```

### Memory Directives
- **Sections:** `.text` (R/X), `.data` (R/W), `.bss`.
- **Data Types:** `.word`, `.half`, `.byte`, `.asciz`, `.space`, `.fill`.
- **Visibility:** `.global`, `.extern` for the GNU Linker.

```assembly
.equ BUFFER_SIZE, 1024 * 2
li a0, BUFFER_SIZE + 16      # Math is evaluated at compile-time
print_char '\n'              # Lexer handles escape characters
```

---

## Development Roadmap

### Completed
- [x] Relocatable ELF Output (GNU-compliant).
- [x] Section Separation (`.text` and `.data` isolation).
- [x] Relocation Math (handling addends in ELF mode).
- [x] Advanced Lexer (character literals and zero-initialized buffers).
- [x] Recursive Inclusion (`.include`).
- [x] Pseudo-Instruction Expansion (`bltz`, `bgez`, `neg`).

### Planned
- [ ] **Documentation**: Compile-time vs runtime rules, macro expansion order, relocation model, struct memory layout rules.
- [ ] **Debug mode**: `--dump-macros`, `--dump-relocations`, `--trace-expansion`.
- [ ] **Heap memory**: `kmalloc` / `kfree` standard library functions.
