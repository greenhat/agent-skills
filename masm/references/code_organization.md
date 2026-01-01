# Miden Assembly Code Organization

## Table of Contents
- [Program Structure](#program-structure)
- [Procedures](#procedures)
- [Modules](#modules)
- [Constants](#constants)
- [Types](#types)
- [Execution Contexts](#execution-contexts)

---

## Program Structure

A Miden assembly program is a sequence of instructions separated by whitespace. Instructions are keywords with optional parameters: `keyword.param1.param2`.

```masm
# Program with procedures
proc helper
    push.1 add
end

begin
    push.5
    exec.helper
end
```

---

## Procedures

### Definition Syntax

```masm
@locals(N)                        # Optional: allocate N local memory elements
pub proc name(a: T, b: U) -> R    # Optional: visibility, types
    # instructions
end
```

### Components

1. **Attributes**: `@locals(N)` allocates N elements of procedure-local memory
2. **Visibility**: `pub` exports the procedure from the module
3. **Name**: starts with letter, contains letters/numbers/underscores
4. **Type signature**: optional, for tooling integration
5. **Body**: one or more instructions

### Invocation

| Instruction | Description | Context Switch |
| ----------- | ----------- | -------------- |
| `exec.name` | Inline execution, same context | No |
| `call.name` | Execute in new user context | Yes |
| `syscall.name` | Execute in root context (kernel procs) | Yes |
| `dynexec` | Dynamic execution via MAST root in memory | No |
| `dyncall` | Dynamic call via MAST root in memory | Yes |

### Dynamic Invocation

```masm
# Store procedure hash and execute dynamically
procref.foo mem_storew_be.ADDR dropw push.ADDR
dynexec
```

### Procedure Locals

- Access via `loc_load.i`, `loc_store.i`, `loc_loadw_be.i`, etc.
- Get absolute address via `locaddr.i`
- Max 2^16 locals per procedure, 2^31 - 1 total
- **Not 0-initialized** (unlike global memory)

---

## Modules

### Library Modules

Contain procedures that can be exported:

```masm
proc internal_helper
    # private procedure
end

pub proc public_api
    exec.internal_helper
end
```

### Executable Modules (Programs)

Have exactly one entry point:

```masm
proc helper
    # private
end

begin
    # program entry point
    exec.helper
end
```

### Importing

```masm
use foo::bar           # Import namespace
use foo::bar::baz      # Import specific item

begin
    exec.::foo::bar::baz  # Fully qualified (absolute)
    exec.bar::baz         # Via namespace import
    exec.baz              # Via item import
end
```

### Renaming on Import

```masm
use foo::bar->bar2     # Rename to avoid conflicts
```

### Re-exporting

```masm
use miden::core::math::u64

pub use u64::add                  # Re-export as 'add'
pub use u64::mul->mul64           # Re-export with new name
pub use ::miden::core::math::u64::div  # Absolute path
```

---

## Constants

### Scalar Constants

```masm
pub const EXPORTED = 100
const PRIVATE = 200
const COMPUTED = EXPORTED+50
const HEX_VAL = 0x1234
const ADDR = 3

pub proc example
    push.EXPORTED
    mem_store.ADDR
end
```

- Names: uppercase letters, numbers, underscores (max 100 chars)
- Values: 0 to 2^64 - 2^32 (decimal or hex)
- Arithmetic: `+`, `-`, `*`, `/` (field), `//` (integer)

### Word Constants

```masm
const WORD_ARRAY = [1, 2, 3, 4]
const WORD_HEX = 0x0200000000000000030000000000000004000000000000000500000000000000

begin
    push.WORD_ARRAY        # Pushes 1, 2, 3, 4
    push.WORD_HEX.6        # Pushes 2, 3, 4, 5, 6
end
```

### Constant Slices

```masm
const W = [5, 6, 7, 8]

begin
    push.W[1..3]   # Pushes 6, 7
    push.W[0]      # Pushes 5
end
```

---

## Types

### Built-in Types

| Type | Description |
| ---- | ----------- |
| `i1` | Boolean (0 or 1) |
| `iN`, `uN` | Signed/unsigned N-bit integers (N = power of 2) |
| `felt` | Field element (aligned to 4 bytes) |
| `word` | Array of 4 felt, equivalent to `[felt; 4]` |
| `ptr<T>` | Pointer to T (default felt address space) |
| `ptr<T, addrspace(byte)>` | Byte-addressed pointer |
| `[T; N]` | Array of N elements of type T |
| `struct { name: T }` | Named aggregate type |

### Type Aliases

```masm
type Id = u64

type Account = struct { id: Id, balance: u64 }
```

### Enum Types

```masm
enum Level : u8 {
    DEBUG = 1,
    WARN,
    ERROR,
}
```

### Procedure Signatures

```masm
# Single input, no output
proc foo(t: T)

# Multiple inputs, single output
proc bar(t: T, u: U) -> i1

# No input, multiple outputs
proc baz() -> (flag: i1, value: T)
```

---

## Execution Contexts

### Context Types

- **Root context**: Where program starts (`begin`)
- **User context**: Created by `call`/`dyncall`

### Memory Isolation

- Each context has independent memory space [0, 2^32)
- First 2^31 addresses: global memory
- Addresses >= 2^31: procedure locals

### Call Semantics

**`call`/`dyncall`/`syscall`:**
- Creates new context (or returns to root for syscall)
- Stack items beyond 16th are hidden
- Callee sees stack depth = 16
- Must return with exactly 16 stack items

**`exec`/`dynexec`:**
- Same context as caller
- Full stack access
- Locals share address space with caller

### Example Flow

```masm
# kernel
pub proc sys_handler
    caller       # Gets hash of calling procedure
end

# program
@locals(2)
proc bar
    syscall.sys_handler
end

@locals(3)
proc foo
    call.bar     # New context for bar
    exec.bar     # Same context as foo
end

begin
    call.foo     # New context for foo
end
```

---

## Comments

```masm
#! Documentation comment (before procedures)
pub proc documented_proc
    # Regular comment
    push.1
end
```
