# Dynamic Analysis

Dynamic analysis is running the program while watching what it does — which syscalls it makes, which functions it calls, and what values sit in registers at each step. It shows the truth of runtime behavior that static reading can only guess. This article covers the standard observation tools against binaries you are authorized to analyze.

## When to use it

Use it after static triage has given you the map, or when static analysis hits a wall: unpacking a packed binary, tracing data that is decrypted only at runtime, or confirming a hypothesis quickly. A good default order is: static first to plan, dynamic second to verify.

## The golden safety rule

Only run binaries you trust or own, inside an isolated VM — never on your daily machine, and with the network disabled unless you deliberately need it. Most crackmes and CTF challenges need no network at all, so a disconnected VM is enough. The authorized targets for this material are your own builds, provided lab files, crackmes, and CTF challenges.

## GDB basics

`gdb` is the standard Linux debugger. These are the commands you will use constantly:

| Command | Purpose |
| --- | --- |
| `gdb ./chall` | Start the debugger with the binary |
| `break main` | Set a breakpoint at `main` |
| `run` | Execute the program (stops at breakpoints) |
| `next` | Step over the current line or function call |
| `step` | Step into the current function call |
| `info registers` | Show the general-purpose registers |
| `x/20x $rsp` | Examine 20 hex words starting at the stack pointer |
| `continue` | Resume execution until the next event |
| `quit` | Leave the debugger |

### A typical session

```bash
gdb ./chall
(gdb) break main
(gdb) run                # or: run secret123  (pass arguments after run)
(gdb) next
(gdb) info registers
(gdb) quit
```

Note that arguments go after `run`, not on the `gdb` command line — the debugger starts the program itself.

## Breakpoints: pausing at the right place

A breakpoint pauses execution when it reaches an address, so you can inspect state at exactly the moment you care about — before a comparison, right after an input is read. Break on `main` first, then on the function you identified statically, then on the comparison itself.

## Watching comparisons

Break at the comparison call (for example `strcmp`) and look at its arguments: `info registers` plus `x/s $rdi` (first argument on SysV AMD64) prints the string being checked. You see the expected value and the supplied value at the same instant — the fastest way to confirm a password check.

## Tracing syscalls and library calls

Two one-liners that need no breakpoint setup:

```bash
strace ./chall   # system calls: open, read, write, exit
ltrace ./chall   # library calls: printf, strcmp, malloc
```

`strace` shows what the program does to the outside world — a `write` of "wrong password" tells you the failure path without reading any assembly. `ltrace` shows library-level logic. Both are standard on Linux distributions; verify availability on your system.

## printf-style debugging vs a debugger

If you control the source, you would just add print statements — that is ordinary debugging, not reversing, and it is out of scope here. When you only have the binary, the debugger and the trace tools are what you have.

## Worked example: a fictional crackme

Supplied file: `crackme1` (a lab exercise you are authorized to analyze).

```bash
strace ./crackme1 test
# ... write(1, "wrong password\n", 15) = 16
# ... exit_group(1)

gdb ./crackme1
(gdb) break check_password
(gdb) run test
(gdb) info registers
```

The trace alone tells you the program prints a fixed message and exits — so the logic is a comparison, not a lookup. Breaking on the checking function confirms it: you watch the input arrive and see where it diverges from the expected value. The static map said *where*; the trace said *what actually happens*.

## Dynamic vs static

| Dimension | Static analysis | Dynamic analysis |
| --- | --- | --- |
| Risk | None — code never runs | Runs untrusted code (use a VM) |
| Runtime data | Cannot see it | Sees decrypted and unpacked data |
| Speed to first answer | Fast for triage | Slower setup, fast confirmation |
| Coverage | All code, even unreachable paths | Only the paths actually executed |

Use both: static to find the interesting spot, dynamic to confirm what happens there.

## When NOT to dynamic

Do not run malware samples casually — that belongs to defensive analysis in properly isolated environments; see the [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/) for how defensive work is structured. In this course, dynamic analysis targets are binaries you own or are authorized to analyze: lab files, crackmes, and CTF challenges.

## See also

- [Static Analysis](static-analysis.md)
- [Anti-Debug & Packing](anti-debug.md)
- [Legal & Ethics](../fundamentals/legal-ethics.md)
- [Practice & Labs](../practice/labs.md)
