# Windows vs Linux Binaries

The same reversing concepts apply everywhere, but file format, header layout, and calling conventions differ between Linux and Windows. Knowing which convention you are looking at prevents hours of misread arguments and wrong tool choices. This article compares the two platforms against binaries you are authorized to analyze.

## Why the difference matters

A register holding the first argument on Linux is not the first argument on Windows; a `strings` scan that finds everything in an ELF may find half of nothing in a PE. Learn the differences once and your workflow carries over almost unchanged.

## File formats

| Format | Typical home | What it contains (simplified) |
| --- | --- | --- |
| ELF | Linux, BSDs, most Unix systems | Program headers, sections, symbol table, dynamic linking info |
| PE | Windows (`.exe`, `.dll`) | DOS/NT headers, sections, import/export tables, resources |
| Mach-O | macOS (and iOS) | Load commands, segments/sections, symbol and relocation info |

One format per platform — identifying it is the first thing `file` does for you.

## PE structure essentials

PE files are divided into sections with fixed roles:

| Section | Role |
| --- | --- |
| `.text` | Executable code |
| `.data` | Initialized read/write data (globals) |
| `.rdata` | Read-only data: constants, import names, strings |
| `.rsrc` | Resources: icons, dialogs, version info — sometimes hidden data |

ELF has the same idea (`.text`, `.data`, `.rodata`) with slightly different names — once you know one layout, the other feels familiar.

## Headers: where the entry point lives

| | ELF | PE |
| --- | --- | --- |
| Read it with | `readelf -h chall` | Ghidra or a PE viewer (verify current tools) |
| Entry point field | `Entry point address` in the ELF header | `AddressOfEntryPoint` in the optional header |
| Extra detail | Program headers describe how segments load | Data directories point to imports, resources, and more |

`readelf -h` is the one-line ELF header dump worth memorizing; for PE, any decent inspector (including Ghidra) shows the equivalent fields.

## Imports: how external functions are resolved

Both formats need library functions they did not link in statically, but the mechanism differs.

### PLT/GOT (ELF)

Each imported call jumps through a stub in the `.plt` that resolves the real address on first use and caches it in the global offset table (`.got`). Later calls go straight to the cached entry.

### Import Address Table (PE)

The loader fills a table of addresses — the IAT — at load time, and calls go through it. Same problem, different plumbing: "call a function we do not own". Both mechanisms show up as a readable list of imported names early in your analysis (see [Static Analysis](static-analysis.md)).

## Calling conventions

| Convention | Argument registers | Notes |
| --- | --- | --- |
| SysV AMD64 (Linux) | `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9` | Return value in `rax`; first six integer arguments in registers |
| Windows x64 | `rcx`, `rdx`, `r8`, `r9` | First four integer arguments in registers; the caller reserves 32 bytes of shadow space for the callee |

This is the most consequential difference while reading disassembly: the same kind of call, two different places to look for arguments. Keep it to the register lists — no deep stack diagrams needed at this level.

## Tooling

| Platform | Common tools |
| --- | --- |
| Linux | `gdb`, `strace`, `readelf`, `objdump`, Ghidra |
| Windows | x64dbg, Process Monitor, Ghidra (verify current versions and licenses) |

Ghidra appears on both sides on purpose — it reads ELF, PE, and Mach-O alike.

## Strings: the encoding difference

Linux text is usually ASCII/UTF-8 and found by plain `strings`. Windows text is often UTF-16 (wide strings), which needs an encoding flag (verify your build's options):

```bash
strings -el application.exe | grep -i password
```

If a Windows binary looks string-empty, try `-el` before concluding anything is hidden.

## Runtime differences

Windows programs load DLLs; Linux programs load `.so` shared objects — same concept, different names, extensions, and linking conventions. One line to remember: **DLL on Windows, `.so` on Linux, both are "code someone else provides at runtime"**.

## Cross-platform note

Ghidra reads all three formats, so your core workflow barely changes: import, analyze, find `main` or `WinMain`, follow functions. What changes is the register conventions, the header commands, and which debugger you reach for.

## Practical advice

Learn Linux first if you are here for CTFs: most crackmes are ELF, and `gdb` + `strace` + `readelf` cover everything. Add Windows knowledge when you face real-world software — PE structure, x64dbg, and wide strings are the three things that transfer most often.

## Authorized use reminder

Apply all of this only to binaries you own or are authorized to analyze — lab exercises, crackmes, CTF challenges, and your own builds, on either platform.

## See also

- [How Binaries Work](../fundamentals/how-binaries-work.md)
- [Assembly Basics](../fundamentals/assembly-basics.md)
- [Strings & Patterns](strings-patterns.md)
- [Reversing Tools](../practice/reversing-tools.md)
- [The Reversing Workflow](../practice/workflow.md)
