# Strings & Patterns

Strings is the fastest first look at an unknown binary: it extracts runs of printable characters, and programs usually carry their messages, flags, formats, and error text as plain literals. That is why almost every reversing session starts with the same few commands. This article covers the standard, read-only way to read a binary's text before opening a disassembler.

## Why strings is always the first command

When a developer writes `printf("Access denied")`, those bytes end up in the file. Compilers rarely hide them. A ten-second scan can therefore tell you what the program says, what it asks for, and sometimes the exact shape of the answer it expects.

## The first 10 seconds

Run these on a binary you are authorized to analyze (a lab file, crackme, or CTF challenge):

| Command | What it tells you |
| --- | --- |
| `file chall` | File format and CPU architecture (ELF x86-64, PE32, ...) |
| `strings chall \| head` | The first readable content: messages, formats, paths |
| `strings -n 8 chall \| grep -i flag` | Longer strings that look like flags or keys |
| `xxd chall \| head` | Magic bytes at offset 0 — confirms what `file` reported |

Always start with `file`: it decides which toolchain you reach for next.

## How strings works

`strings` scans the file for consecutive printable ASCII bytes (letters, digits, punctuation, spaces) and prints each run it finds. It executes nothing and does not need to understand the file format — it just reads bytes, which makes it safe and fast.

### The minimum length flag

Short runs produce noise (`a`, `OK`, `if`). The `-n` flag sets the minimum length of a run to print:

```bash
strings -n 8 chall | head -n 30   # only runs of 8+ characters
```

Use a lower value (for example `-n 5`) when you expect short tokens, and a higher one to cut noise.

## Grep patterns worth trying

`strings` prints everything; `grep` filters it. Each pattern suggests a different part of the program:

| Pattern | What it suggests |
| --- | --- |
| `flag{` | A CTF flag literal, or the format of one |
| `password` | A credential comparison nearby |
| `usage:` | Command-line argument parsing |
| `error` | Failure paths — often reveal hidden states |
| `admin` | Roles, privileges, or hidden commands |

Combine flags and search case-insensitively:

```bash
strings -n 6 chall | grep -i 'password'
```

### Extended regex one-liner

For several alternatives in one pass, use `grep -E`:

```bash
strings -n 6 chall | grep -E 'flag\{|secret|admin|root'
```

Escape `{` there, because in extended regex it starts a repetition operator.

## ASCII vs UTF-16 strings

Linux binaries mostly store text as single-byte ASCII/UTF-8, so plain `strings` finds it. Windows programs often store text as UTF-16 (one character = two bytes, low byte first), which plain `strings` misses because every other byte is `0x00`.

GNU `strings` supports encodings; little-endian 16-bit is the usual one for Windows text (verify the option set of your build):

```bash
strings -el application.exe | grep -i password
```

Verify what you find: a string existing in the file does not mean it is used — confirm with a cross-reference in your disassembler (see [Static Analysis](static-analysis.md)).

## Reading surrounding context

Print the file offset of each string with `-t x` (hexadecimal offsets):

```bash
strings -t x chall | grep -i 'password'
# example output:   1f30 usage: ./chall <password>
# example output:   1f58 wrong password
```

With that offset you can:

- seek to it in a hex editor and read the bytes around it;
- jump to the address in Ghidra (for a non-PIE ELF the file offset often equals the virtual address — confirm with `readelf -l` before trusting it);
- notice that related strings are grouped together — developers put message tables in one place.

## When strings is NOT enough

Strings only sees what is stored in plain form. Packed or encrypted binaries yield little more than fragments, and stripped binaries hide function names. If the output looks like garbage or is nearly empty, move on to packing and anti-debug concepts in [Anti-Debug & Packing](anti-debug.md).

## Worked example: a fictional crackme

Supplied file: `crackme1` (a lab exercise you are authorized to analyze).

```bash
file crackme1
# crackme1: ELF 64-bit LSB executable, x86-64 ...

strings -n 6 crackme1 | grep -iE 'flag\{|password|usage'
# usage: ./crackme1 <password>
# wrong password
# Access granted: flag{...}

strings -t x crackme1 | grep -i password
#   1f30 usage: ./crackme1 <password>
#   1f58 wrong password
```

Conclusions: the program takes one argument, compares it against something, and on success prints a flag-shaped string (placeholder `flag{...}` above). The next step is to open the binary in a disassembler, find the string at offset `0x1f58`, and follow its cross-reference to the comparison code.

## Authorized use reminder

Run these commands only on binaries you own or are explicitly authorized to analyze — your own builds, provided lab files, crackmes, and CTF challenges. Reading strings out of proprietary software you have no right to examine is out of scope for this material.

## See also

- [How Binaries Work](../fundamentals/how-binaries-work.md)
- [Static Analysis](static-analysis.md)
- [Anti-Debug & Packing](anti-debug.md)
- [Reversing Tools](../practice/reversing-tools.md)
