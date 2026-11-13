# Reversing Tools

A reversing toolkit is a set of programs for reading, disassembling, decompiling, and debugging binaries. The tools themselves are neutral — their legality depends entirely on the target you point them at. This page lists the core tools by task, how to set them up safely, and which tool answers which question.

> **Golden rule:** a tool is only as legal as the target you point it at. Use every tool on binaries you own, were given, or are explicitly authorized to analyze — never to bypass license checks or protections on software you do not own.

## Tools by Task

| Task | Tool | Notes |
| --- | --- | --- |
| Identify a file | `file`, `checksec` | `file` shows type/format; `checksec` shows protections |
| Extract strings | `strings`, `grep` | First look for readable hints: paths, prompts, error text |
| Disassemble | `objdump`, Ghidra, rizin | Linear vs. recursive disassembly; Ghidra and rizin add navigation |
| Decompile | Ghidra (free), IDA (free version limits — verify current), Cutter | Ghidra is free and open source; Cutter is rizin-based |
| Debug | `gdb` (Linux), x64dbg (Windows) | Set breakpoints, step, inspect registers and memory |
| Trace syscalls | `strace` / `ltrace` | See what a program actually does at runtime (syscalls / library calls) |
| Hex view | `xxd`, hex editors | Read raw bytes, headers, encrypted or packed regions |
| Unpack | UPX CLI (on files you may analyze — verify flags) | Common packer; run only on binaries you are allowed to examine |

Verify current pricing and access for IDA's free version and any hosted platform — offerings change over time.

## Ghidra Setup (Quick Note)

Ghidra is a free reverse engineering suite released by the NSA, available from the official Ghidra site — verify current releases there before installing.

- Requires a Java Development Kit (JDK); check the release notes for the exact supported version.
- Download from the official site or the official GitHub repository, not third-party mirrors.
- Create a project, import a binary you are authorized to analyze, and let Ghidra run its auto-analysis.
- Start with the Listing view (disassembly) and the Decompiler window side by side.

## The gdb Ecosystem

Plain `gdb` works out of the box on most Linux systems, but plugin front-ends make it far more usable:

- **GEF** and **pwndbg** add register/memory panes, heap views, and nicer breakpoints — verify current install instructions on their official repositories.
- A typical workflow: run the binary under gdb, `break` at a function of interest, `run` with input, inspect registers and stack at each step.

## checksec One-Liner

`checksec` (often shipped with gdb plugins or as a standalone tool) prints a binary's protections at a glance:

```bash
checksec --file=./target      # typical usage — verify flags for your version
```

What it reports and why it matters:

| Protection | What it means |
| --- | --- |
| NX (no-execute) | Stack/memory cannot run shellcode directly |
| Canary | Stack-smash detection value before return addresses |
| ASLR | Randomized memory layout each run |
| PIE | Executable itself is loaded at a random address |

These change how exploitable a bug is — and how hard dynamic analysis is — so check them before you plan an approach.

## Research Environment

| Practice | Why |
| --- | --- |
| Dedicated VM for reversing | Malware and unknown binaries never touch your host OS |
| Take snapshots before running anything | Restore a clean state in seconds after a crash or infection |
| No shared folders with real data | A hostile binary must not reach your personal files |
| Network off for crackmes | Crackmes should not need the network; isolation removes exfiltration risk |

Only run binaries you are authorized to analyze, and only inside this isolated environment.

## Tool Hygiene

- Install from official repositories or official project sites — never from random download mirrors.
- Verify checksums or signatures where the project publishes them (verify current instructions per tool).
- Keep tools updated, but read release notes: disassembler and decompiler updates can change analysis output.
- Prefer package-manager installs on Linux so security updates flow through automatically.

## Automation Habit

Repetitive parsing (scanning thousands of strings, walking structures, extracting fields) should be scripted, not done by hand. Python is the default choice in this field, and **pwntools** is the standard helper library in the CTF context (context handling, ELF inspection, ROP helpers). Learn it on sanctioned challenges and your own binaries first.

## What NOT to Do

- Do not use these tools to bypass license checks, DRM, or activation on software you do not own.
- Do not analyze a company's proprietary product "for practice" without authorization.
- Do not download and run unknown binaries outside a VM — and do not analyze anything you were not given permission to touch.

Practice only on sanctioned challenges and binaries you own — the legal and ethics rules apply to every tool on this page.

## Which Tool for Which Question

| Your question | Reach for |
| --- | --- |
| "What kind of file is this?" | `file` |
| "What protections does it have?" | `checksec` |
| "What readable text is inside?" | `strings` + `grep` |
| "Where is this string referenced?" | Ghidra / rizin cross-references |
| "What does this function do?" | Ghidra decompiler |
| "What does it do when I run it?" | `strace` / `ltrace`, then `gdb` |
| "What do these raw bytes mean?" | `xxd` or a hex editor |
| "Is this packed?" | `file`, high-entropy check, strings showing packer signatures |

## See also

- [Static Analysis](../techniques/static-analysis.md)
- [Decompilers & Ghidra](../techniques/decompilers.md)
- [Legal & Ethics](../fundamentals/legal-ethics.md)
- [Practice & Labs](labs.md)
