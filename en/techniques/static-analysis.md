# Static Analysis

Static analysis means studying a program without executing it: you read its bytes, structure, and code with tools that never run the binary. Because nothing executes, it is the safest way to start and the right place to build a map before any dynamic work. This article covers the standard workflow against binaries you are authorized to analyze.

## Why start static first

Running unknown code can trigger destructive or networked behavior; a disassembler cannot. Static analysis lets you identify the format, find the entry point, list imports, and locate interesting functions with zero execution risk — then decide what (if anything) is worth running later.

## The workflow

Start in the shell, finish in the disassembler:

```bash
file chall                  # identify format and architecture
strings chall | head -n 40  # see readable content
```

Then, in a disassembler such as Ghidra:

```
import the file -> accept auto-analysis -> open the Decompiler view
find main (Search > For Functions, or the Symbol Tree)
read main: note its arguments and what it calls
follow the interesting function one level at a time
record inputs (arguments, files, environment) and outputs (messages, return values)
```

The loop is: read, follow, annotate. You stop when you can explain how the input maps to the output.

## Imports as a map

Imported functions tell you what the program needs from the operating system and libraries. They are a quick profile of its behavior:

| Imported function | What it reveals |
| --- | --- |
| `printf` / `puts` | Produces output — look for the format strings |
| `strcmp` / `strncmp` | Compares something — likely a password or key check |
| `getenv` | Reads an environment variable — check which name |
| `socket` / `connect` | Networking — the program talks to something |
| `fopen` / `read` | File access — see which paths appear in strings |

If an import exists but its call site is hard to find, that call site becomes your next target.

## Cross-references (xrefs)

A cross-reference answers "who uses this?". Select a function or a string and list its references: strings point you to the code that prints or compares them, and functions point you to their callers. Xrefs are the breadcrumb trail through a binary — follow them instead of reading the disassembly linearly.

## Control flow basics

`if`/`else` becomes conditional jumps (`je`, `jne`, `jg`, ...) at the assembly level: one branch per outcome, joined later. Loops are backward jumps to the top of a block. When you read pseudocode, remember that every `if` you see was a conditional jump and every loop a jump backwards — same idea, different spelling.

A simple comparison reads like this in assembly:

```asm
cmp  eax, 0          ; compare the value with 0
je   branch_else     ; jump if equal -> the "if" took the other path
; ... "then" branch continues here ...
```

The pattern to learn is: **compare, then conditional jump, then two paths that rejoin**. Everything bigger — nested conditions, switches, loops — is built from that one pattern.

## Entry points: where reading starts

Every binary has at least one entry point, and it is not always the function you expect: a minimal ELF can start in `_start` (which calls `main`), a GUI program may use `WinMain`, and a DLL exports functions without any `main` at all. When the entry point looks like startup plumbing, follow its calls until you reach the code the author wrote — that is usually `main`.

## Constants and strings as anchors

The classic technique: find a string you care about ("wrong password"), jump to the code that references it, and read that function. Strings and numeric constants (`0x1337`, `1000000`) are stable landmarks — rename the functions around them and your map grows with every pass.

### Worked example

```bash
strings -n 6 chall | grep -i wrong
#   1f58 wrong password
```

Open `chall` in Ghidra, search memory for `wrong password`, list references, and you land in the comparison routine. Read it: an argument arrives from `main`, is compared against an expected value, and the failure branch prints that string. Rename it `check_password` and continue from there.

## Safe setup

Static tools only read the file — they do not execute it. Still, treat unknown binaries as untrusted: analyze them in a VM or a disposable environment, and never double-click a sample to "see what it happens". Carelessness, not the disassembler, is the usual accident.

## Limitations

Static analysis cannot see data that only exists at runtime — packed sections, decrypted strings, or values filled in after execution starts. When code hides its data behind decryption loops, switch to [Dynamic Analysis](dynamic-analysis.md) and observe it while it runs.

## Tool pointers

Ghidra (free), radare2/rizin, and Cutter all do this job — see [Reversing Tools](../practice/reversing-tools.md) for a comparison, and [Decompilers & Ghidra](decompilers.md) for the pseudocode side.

## Authorized use reminder

Everything here applies only to binaries you own or are authorized to analyze: lab exercises, crackmes, CTF challenges, and your own builds. Mapping someone else's proprietary program without permission is not covered by this material.

## See also

- [What Is Reverse Engineering](../fundamentals/what-is-reversing.md)
- [Dynamic Analysis](dynamic-analysis.md)
- [Decompilers & Ghidra](decompilers.md)
- [Reversing Tools](../practice/reversing-tools.md)
- [The Reversing Workflow](../practice/workflow.md)
