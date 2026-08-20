# Control Flow, Macros, and Structs

Kdex abstracts away the boilerplate of low-level assembly by providing logical constructs built directly into the parser. These constructs evaluate to precise, deterministic machine code.

## The Logic Stack

Kdex maintains an internal logic stack to parse high-level structures like loops and conditionals. The assembler automatically manages label generation (`.loop_X`, `.done_X`) so you don't have to manually pollute your symbol table.

### Conditionals (If-Else)

The `if` block evaluates a logical comparison between two registers and conditionally executes the block.

```assembly
li t0, 5
li t1, 10

if lt, t0, t1
    # Executes because 5 < 10
    addi t0, t0, 1
endif
```

Supported comparators:
- `eq`: Equal
- `ne`: Not equal
- `lt`: Less than
- `ge`: Greater than or equal

### Loops (While)

The `while` block functions similarly to `if` but jumps back to the condition evaluation after hitting `endwhile`.

```assembly
li t0, 0
li t1, 10

# Loops 10 times
while lt, t0, t1
    addi t0, t0, 1
endwhile
```

## Anonymous Labels

For tight, local control flow that doesn't map well to `if` or `while`, Kdex supports anonymous labels (`@@`).

- `@@:` Defines an anonymous anchor.
- `@f`: Jumps forward to the nearest `@@:`.
- `@b`: Jumps backward to the nearest `@@:`.

```assembly
    mv t0, a0
@@:
    lb t1, 0(t0)
    beqz t1, @f      # Break loop if null byte
    addi t0, t0, 1
    j @b             # Continue loop
@@:
    sub a0, t0, a0   # Calculate length
```

## The Macro Engine

Macros allow you to parameterize large blocks of code. Kdex natively injects macro parameters using `%1`, `%2`, etc.

```assembly
macro memcpy dst, src, len
    mv t0, %1       # dst
    mv t1, %2       # src
    mv t2, %3       # len
    
  .copy_loop_%u:    # %u is a unique ID scoped to this macro instance
    lb t3, 0(t1)
    sb t3, 0(t0)
    addi t0, t0, 1
    addi t1, t1, 1
    addi t2, t2, -1
    bnez t2, .copy_loop_%u
endm
```
*Note: Using `%u` appends a unique internal counter to the label, ensuring that multiple invocations of `memcpy` will not result in label collisions.*

## Memory Layout via Structs

Managing byte offsets for complex data structures is error-prone. Kdex introduces the `struct` block, which computes and creates memory offsets for you.

```assembly
struct HTTPRequest
    field method, 4      # method = 0
    field status, 2      # status = 4
    array url, 256       # url = 6
endstruct                # HTTPRequest_SIZE = 262
```

This generates internal variables representing the structural offsets. You can then use them cleanly in your code:

```assembly
.data
    req_obj: .space HTTPRequest_SIZE

.text
    la t0, req_obj
    li t1, 200
    sh t1, HTTPRequest.status(t0)
```
