# Kdex RISC-V FASM Documentation

Welcome to the official documentation for **Kdex (RISC-V FASM)**, a macro-enhanced assembler designed for the RV32I architecture. 

Kdex bridges the gap between low-level hardware control and high-level programming paradigms by providing stack-based control flow, compile-time memory layout (structs), variadic macros, and full GNU ELF compliance.

## Table of Contents

1. [Getting Started](getting-started.md)
   - Installation and Build Requirements
   - CLI Usage and Flags
   - First Program
2. [Syntax and Directives](syntax-and-directives.md)
   - Lexical Structure
   - Directives and Sections
   - Expressions and Compile-Time Evaluation
3. [Control Flow, Macros, and Structs](control-flow-and-macros.md)
   - Logic Stack (if, while)
   - Anonymous Labels
   - The Macro Engine
   - Memory Layout via Structs
4. [ELF Linking and C-Interop](elf-and-linking.md)
   - Dual-Mode Output (Flat vs ELF)
   - Relocations (HI20, LO12, JAL, CALL)
   - Interfacing with the GNU Toolchain

## Architecture Overview

Kdex is implemented entirely in C11 and follows a multi-pass architecture:

1. **Lexical Analysis & Preprocessing**: Resolves `.include` directives recursively, strips comments, and tokenizes strings and character literals safely.
2. **Macro Expansion**: Evaluates macro invocations and control-flow pseudo-instructions (`if`, `while`), generating scoped `.loop_%u` labels dynamically.
3. **Pass 1 (Symbol Resolution)**: Scans the codebase to evaluate mathematical expressions, track `.org` bounds, and build the global symbol table.
4. **Pass 2 (Encoding)**: Emits binary RV32I instruction encodings and applies local label offsets.
5. **Relocation & Emission (ELF Engine)**: If compiling in ELF mode, the engine skips local address resolution for external symbols, generates an ELF32 header, builds the `.symtab` and `.strtab`, and emits `.rela.text` sections for the GNU linker.
