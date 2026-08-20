# Syntax and Directives

Kdex follows standard GNU-assembler-like syntax for its core instruction set while introducing powerful features for compile-time constants and expressions.

## Lexical Structure

- **Comments**: Supported via `#` or `;`. Both indicate the start of a line comment.
- **Registers**: Standard RISC-V ABI register names (`zero`, `ra`, `sp`, `a0`-`a7`, `t0`-`t6`, `s0`-`s11`) and hardware names (`x0`-`x31`) are natively understood.
- **Immediate Values**: Immediate values can be provided in base-10 (`42`), base-16 (`0x2A`), or as ASCII character literals (`'A'`, `'\n'`).

## Directives

Memory and structural directives inform the assembler how to align, place, and export data.

| Directive | Arguments | Description |
| --------- | --------- | ----------- |
| `.org` | `address` | Sets the internal virtual address counter. Required for `flat` binaries. |
| `.text` | None | Defines the beginning of executable machine code. |
| `.data` | None | Defines the beginning of initialized memory. |
| `.bss` | None | Defines the beginning of zero-initialized memory. |
| `.global` | `symbol` | Exports a symbol for use by the GNU linker (ELF mode only). |
| `.extern` | `symbol` | Declares a symbol that is resolved at link time (ELF mode only). |
| `.include` | `"path"` | Recursively includes and evaluates the specified file inline. |
| `.align` | `bytes` | Pads the current section to ensure the next byte is aligned to a multiple of `bytes`. |

## Data Emission

Data can be statically compiled into the binary using type-specific directives:

```assembly
.data
.align 4
my_word:   .word 0xDEADBEEF        # 32-bit integer
my_half:   .half 0xFFFF            # 16-bit integer
my_byte:   .byte 0x01              # 8-bit integer
my_string: .asciz "Hello, World!"  # Null-terminated string
my_buffer: .space 256              # Reserves 256 bytes
```

## Expressions and Variables

Kdex provides a compile-time math evaluator that simplifies address calculations and constant management. 

### Constants
Constants are defined using `.equ`. They are resolved instantly and substituted inline.

```assembly
.equ BUFFER_SIZE, 1024 * 4
.equ OFFSET, BUFFER_SIZE + 16

li a0, OFFSET
```

### Mutable Variables
Unlike `.equ`, variables can be reassigned during the assembly process using the `=` operator. This is heavily utilized by the macro and struct engine.

```assembly
CURSOR = 0
CURSOR = CURSOR + 4
```

### Math Resolution
The mathematical parser handles addition (`+`), subtraction (`-`), multiplication (`*`), and division (`/`). It also handles label arithmetic:

```assembly
# Calculates the size of a string dynamically
msg: .asciz "Dynamic Length\n"
msg_end:

li a0, msg_end - msg
```
