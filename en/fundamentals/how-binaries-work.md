# How Binaries Work

A binary is the compiled form of a program: the instructions and data that a CPU executes directly. Understanding how one is built explains why it can be taken apart — every byte had a source, a purpose, and a place. This article walks the path from source code to a running executable and introduces the structures you will inspect in later exercises.

## From Source to Executable

```text
  hello.c                          (human-readable source)
      │  compiler (gcc, clang, cl)
      ▼
  hello.o                           (machine code for this file,
      │                              with unresolved references)
      │  linker (+ libraries such as libc)
      ▼
  hello                             (final executable: ELF, PE, or Mach-O)
```

The compiler translates each source file independently into an object file containing machine code and data plus unresolved references to things defined elsewhere. The linker merges those object files and the needed library code into one executable and resolves the references. What lands on disk is a container format holding code, data, and the bookkeeping the operating system needs in order to run it.

## Why Reversing Is Possible

Reversing works because of a simple fact: the CPU is the only intended audience of a shipped program, and the CPU is not a secret-keeping device. It executes a documented, fixed instruction encoding, so every byte the program will ever execute must be decodable by something — and that something is us with a disassembler. Compilation changes the shape of the code (names disappear, loops are unrolled, functions are inlined) but never encrypts it; obfuscation and packing raise the effort required without changing the underlying rule.

## File Formats

| Format | Used on | Typical extension | What it carries |
|---|---|---|---|
| ELF | Linux and most Unix-like systems | none required (for example `bash`) | Code, data, program and section headers, dynamic import/export tables |
| Mach-O | macOS and iOS | none required | The same basic content in Apple's layout, described by load commands |
| PE | Windows | `.exe`, `.dll` | Code, data, an import table naming the DLL functions used, and a header carrying the entry point |

All three solve the same problem: tell the loader where code and data live, what permissions each region needs, and where execution begins.

## Inside an ELF

An ELF file is a sequence of headers followed by data. The **ELF header** at byte zero identifies the file (the magic bytes `7f 45 4c 46`, i.e. `\x7fELF`), its class (32 or 64 bit), the target machine, and the entry point address. **Program headers** describe segments, which is what the loader cares about; **section headers** describe sections, which is what analysis tools and the linker care about.

### Sections

| Section | Contents |
|---|---|
| `.text` | Compiled machine code (executable) |
| `.data` | Initialized global and static variables (read/write) |
| `.bss` | Uninitialized globals — space is reserved and starts as zero, using no file space |
| `.rodata` | Read-only data: string literals, constants, jump tables |
| `.plt` / `.got` | Stubs and addresses used to call functions living in shared libraries |
| `.symtab` / `.dynsym` | Symbol tables: names and addresses of functions and variables, if not stripped |

### The entry point

The entry point is the address the processor jumps to when the program starts. It is usually not `main`: a small startup stub (commonly `_start` in ELF) runs the C runtime's initialization, sets up `argc` and `argv`, and only then calls your `main`.

## What the Loader Does at Run Time

Before a single instruction of your code runs, the operating system loader:

1. Reads the headers and maps each loadable segment into memory with the right permissions (code read/execute, data read/write).
2. Applies relocations, fixing up addresses that were not known at link time.
3. Loads the required shared libraries and resolves imported symbols — this is the dynamic linker: `ld.so` on Linux, `dyld` on macOS, the Windows loader for PE files.
4. Jumps to the entry point.

## Imports and Dynamic Linking

When a program calls `printf`, the binary does not contain `printf`; it contains a *reference* to `printf` recorded in the dynamic symbol tables, filled in by the loader at run time. That reference is your map: the import list tells you which library services the program uses — file I/O, network calls, crypto primitives, UI functions — often revealing what a binary does before you read a single instruction.

**Static vs dynamic linking, one line:** static linking copies library code into the executable at build time; dynamic linking keeps a reference that is resolved when the program loads.

## Symbols and Debug Information

| | Unstripped binary | Stripped binary |
|---|---|---|
| Function and variable names | Present in the symbol table | Removed |
| Recoverable code addresses | Yes | Yes — the instructions are unchanged |
| Debug info (line numbers, types) | Present if built with debug flags | Absent |
| Typical situation | Your own builds, debug packages | Installed production software |

Stripping removes names, not behavior. Reversing a stripped binary is simply slower, because you have to rename everything yourself.

## A Worked, Read-Only Example

Given `chall`, a challenge binary you were given to solve:

```bash
file chall
readelf -h chall
readelf -S chall
```

`file` identifies the format and architecture. `readelf -h` prints the ELF header; `readelf -S` lists the section headers. Example output, abridged — your values will differ:

```text
$ file chall
chall: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked,
      interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux, not stripped

$ readelf -h chall
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Type:                              DYN (Position-Independent Executable)
  Machine:                           Advanced Micro Devices X86-64
  Entry point address:               0x10a0
```

All three commands only read the file: nothing is executed, nothing is modified. The same restriction applies to every example in this guide — work only on binaries you own or are authorized to analyze. For PE (Windows) files, `file` still identifies them while the header-reading tools differ; that is covered in [Windows vs Linux Binaries](../techniques/windows-vs-linux.md).

## See also

- [What Is Reverse Engineering](what-is-reversing.md)
- [Assembly Basics](assembly-basics.md)
- [Strings & Patterns](../techniques/strings-patterns.md)
- [Windows vs Linux Binaries](../techniques/windows-vs-linux.md)
- [Reversing Tools](../practice/reversing-tools.md)
