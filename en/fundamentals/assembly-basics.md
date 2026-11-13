# Assembly Basics

Assembly language is the human-readable form of the machine instructions a CPU executes. It sits at the center of reverse engineering because compilers emit it and decompilers only approximate the code above it. You do not need to memorize the instruction set — you need to recognize patterns.

## Why Assembly Matters for Reversing

Source code tells you intent; the binary contains only what survived compilation. When the source is unavailable, assembly is the closest ground truth you have: a disassembler shows exactly what the CPU will do, while a decompiler's C-like output is an educated guess built on top of those instructions. Reading enough assembly to check the decompiler's work is the core skill of reversing.

## Registers (x86-64)

Registers are tiny, extremely fast storage slots inside the CPU — almost every instruction reads or writes one.

| Register | Role |
|---|---|
| `rax` | Accumulator: arithmetic results and return values |
| `rbx` | General-purpose, callee-saved (a function must preserve it) |
| `rcx` | Counter for loops and string operations |
| `rdx` | Data register: extended arithmetic and I/O |
| `rsi` | Source pointer for copies; 2nd argument on Linux/macOS |
| `rdi` | Destination pointer for copies; 1st argument on Linux/macOS |
| `rsp` | Stack pointer: top of the current stack |
| `rbp` | Frame base pointer: anchors the current function's stack frame |
| `rip` | Instruction pointer: address of the next instruction to execute |

Naming reflects width: `rax` is 64-bit, `eax` its lower 32 bits, `ax` 16 bits, `al` 8 bits. Writing to a 32-bit register such as `eax` zeroes the upper half of `rax` — a common source of confusion when reading optimized code. 32-bit binaries use the `e`-prefixed names throughout: `eax`, `esi`, `edi`, `ebp`, `esp`.

## The Instruction Vocabulary

A surprisingly small set of instructions covers most of what you will read:

| Instruction | Meaning | Notes |
|---|---|---|
| `mov dst, src` | Copy a value | By far the most common instruction; it copies, it does not print |
| `lea dst, [addr]` | Compute an address | Address math only — nothing is read from memory |
| `add` / `sub` | Add / subtract | Related arithmetic: `inc`, `dec`, `imul` |
| `cmp a, b` | Compare | Computes `a - b` and records it as CPU flags; nothing else changes |
| `jmp`, `je`, `jne` | Jump, jump-if-equal, jump-if-not-equal | Conditional jumps read the flags set by `cmp` |
| `push` / `pop` | Write / read the top of the stack | `push` puts data on the stack, `pop` takes it off |
| `call` / `ret` | Call a function / return from one | `call` pushes the return address; `ret` pops it back |

## Comparisons Drive Branches

The `if` of machine code is a pair: one instruction sets flags, another branches on them.

```asm
    cmp   al, 0x41        ; compare the value in al with 'A'
    jne   not_a           ; branch if the values differ
    ; fall through here only when al == 0x41 — this block is the "then"
not_a:
    ; this block is the "else"
```

`je` and `jne` are the everyday pair; `jg`, `jl`, `jge`, `jle` cover signed greater/less comparisons, and `ja`, `jb` and friends do the same for unsigned values. A chain of comparisons followed by jumps is usually a sequence of `if` / `else if` tests — password checks, command parsers, and state machines all look like this.

## The Stack

The stack is a last-in, first-out region of memory that functions use for return addresses, saved registers, and local variables. The most recently pushed value is always the first one popped, and the stack grows downward toward lower addresses:

```text
  higher addresses
  ┌─────────────────┐
  │  previous frame │
  ├─────────────────┤
  │ return address  │  pushed by `call`
  ├─────────────────┤
  │ saved rbp       │
  ├─────────────────┤
  │ local variables │  <- rsp (top of the stack)
  └─────────────────┘
  lower addresses (stack grows down)
```

When `call func` executes, the address of the next instruction is pushed so that `ret` can find its way back. That return address is why mapping a program's call graph is mostly a matter of following pushes and pops.

## Calling Conventions

A calling convention is the agreed layout for passing arguments. On x86-64 System V (Linux, macOS, most Unix-like systems) the first integer/pointer arguments go in `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`, the return value comes back in `rax`, and further arguments travel on the stack. Windows x64 uses different registers and reserves shadow space — see [Windows vs Linux Binaries](../techniques/windows-vs-linux.md).

## Endianness in One Line

x86 is little-endian: multi-byte values are stored least significant byte first, so the integer `0x01020304` appears in memory as `04 03 02 01` — keep this in mind whenever you read hex dumps, immediates, or addresses out of a file.

## How to Practice Reading

You do not need to memorize every instruction. In Ghidra and most other decompilers the C-like output sits next to the assembly: read the decompilation first, then spot-check it against the instructions underneath. Look up unfamiliar opcodes as they appear, keep a reference open, and focus on the recurring patterns — argument setup, comparison, branch, return — which repeat in every binary you will open.

### A minimal annotated example

This function takes one character and returns 1 if it equals `'A'`, otherwise 0. The C equivalent is `int check(char c) { return c == 'A'; }`.

```asm
check:
    push  rbp                   ; save the caller's frame pointer
    mov   rbp, rsp              ; start a new stack frame
    mov   [rbp-0x4], edi        ; keep the argument (a char in edi) locally
    cmp   byte [rbp-0x4], 0x41  ; compare the input with 'A'
    jne   fail                  ; not equal -> return 0
    mov   eax, 1                ; equal -> return value 1
    jmp   done
fail:
    xor   eax, eax              ; zero eax: return value 0
done:
    pop   rbp                   ; restore the caller's frame pointer
    ret                         ; return to the caller
```

The whole function is one comparison and one branch — the pattern from earlier, in context. Practice reading on binaries you own or are authorized to analyze: your own builds, provided lab binaries, crackmes, and CTF challenges.

## Further Reading

Curricula in the "NAND gates to operating systems" style build the right mental model from the hardware upward; for the language itself, any well-reviewed assembly primer will do — verify the current edition before buying, and prefer material matching your target architecture (x86-64 for most of this track). The vendor architecture manuals from Intel and AMD are authoritative but heavy; keep them for lookups rather than learning.

## See also

- [How Binaries Work](how-binaries-work.md)
- [Decompilers & Ghidra](../techniques/decompilers.md)
- [Windows vs Linux Binaries](../techniques/windows-vs-linux.md)
- [Practice & Labs](../practice/labs.md)
- [The Reversing Workflow](../practice/workflow.md)
