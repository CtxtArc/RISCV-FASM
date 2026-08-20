# ELF Linking and C-Interop

Kdex natively supports outputting raw machine code for bare-metal execution (`-f flat`) and relocatable object files (`-f elf`) for seamless interoperation with standard C/C++ toolchains via GNU Linker (`ld`).

## Dual-Mode Compilation

### Flat Binary Mode (`-f flat`)
A flat binary is a headerless image containing exactly the byte-for-byte machine code written in the assembly file. It relies heavily on the `.org` directive to know where it will be loaded in memory (e.g., `0x80000000` for QEMU `virt` machine).

**Use case**: Bootloaders, microcontroller firmware, OS kernels.

### ELF Object Mode (`-f elf`)
The Executable and Linkable Format (ELF) provides a container that separates code (`.text`) from data (`.data`, `.bss`), lists external symbols, and provides a relocation table.

**Use case**: Linking assembly routines into a larger C/C++ project, utilizing the C Standard Library.

## The Relocation Engine

When compiling with `-f elf`, Kdex detects when a symbol is unresolved or marked `.extern` / `.global`. Instead of failing, the assembler defers the resolution to the Linker by embedding relocation records.

Kdex supports standard RISC-V 32-bit relocations:

1. **`R_RISCV_HI20` / `R_RISCV_LO12_I`**: Used when loading an address via the `la` pseudo-instruction.
2. **`R_RISCV_CALL` / `R_RISCV_JAL`**: Used for jumping to external functions.

### Relocation Math

Kdex correctly evaluates addends (offsets) against external symbols at compile-time and encodes them into the `.rela.text` section.

```assembly
# my_buffer is located in another file
.extern my_buffer

la t0, my_buffer + 1024
```
This correctly delegates the base address of `my_buffer` to the linker, while adding `1024` to the relocation addend.

## C-Interop Example

You can write a high-performance routine in Kdex and call it directly from a C program.

**math.s:**
```assembly
.global fast_add

.text
fast_add:
    # a0 = param 1, a1 = param 2
    add a0, a0, a1
    ret
```

**main.c:**
```c
#include <stdio.h>

extern int fast_add(int a, int b);

int main() {
    printf("Result: %d\n", fast_add(10, 20));
    return 0;
}
```

**Build Process:**
```bash
# 1. Assemble with Kdex
./riscv-fasm -f elf math.s -o math.o

# 2. Compile and Link with GCC
riscv64-elf-gcc main.c math.o -o program

# 3. Run
qemu-riscv32 ./program
```
