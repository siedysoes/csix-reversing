# Anti-Debug & Packing

Anti-debug techniques detect or hinder debuggers, and packing compresses or hides a binary's contents — both are common ways a program resists casual inspection. Commercial software uses them to protect itself, and CTF authors use them to add difficulty, so recognizing them is a core reversing skill. This article explains what these techniques are and how to recognize them on authorized targets.

## What anti-debug is

A program that checks whether it is being observed and then reacts — exiting early, lying about results, or altering its behavior. Your goal at this level is recognition: spot the check, understand what it asks, and reason about what its result controls.

## Common detection techniques

| Technique | Concept |
| --- | --- |
| `isDebuggerPresent` | Windows API that reads a flag set when a debugger is attached |
| `PTRACE_TRACEME` | Linux process asks the kernel "am I being traced?" |
| Timing checks | `rdtsc` or clock reads measure how long code took — debuggers slow it down |
| Hardware breakpoint checks | Inspects debug registers (DR0-DR3 on x86) that a debugger may have set |
| Parent-process checks | Looks at the parent process name and compares it to an expectation |

These are detection ideas to understand, not a catalog of countermeasures — the educational goal is knowing what each check asks and why it works.

### Why programs do this

Commercial software uses these checks to make casual tampering harder; CTF authors use them to force you to read code instead of stopping at the first breakpoint. Either way the check is ordinary code: it reads a value, compares it, and branches — exactly the pattern you already know how to read.

## How they appear in static analysis

You can often see the whole check without running anything: a call to `isDebuggerPresent`, a `ptrace` request, or a timestamp read, followed by a conditional branch. Follow that branch and you find the reaction — an early `exit`, a different message, a bogus result. The structure is always "check, then branch": find the check, then ask what the branch does.

The pseudocode shape is predictable:

```c
if (isDebuggerPresent()) {
    exit(-1);
}
```

Read the condition first, then both branches: one path continues the program, the other path is the reaction you are documenting.

## How they appear dynamically

The recognizable pattern: the program behaves differently under a debugger. When a session dies right at a breakpoint, anti-debug is the first hypothesis — not a mysterious crash.

| Symptom observed | Likely reading |
| --- | --- |
| The program quits as soon as you attach | A presence check fired early |
| Tracing output stops abruptly (`strace ./chall` ends early) | The process detected tracing or exited on purpose |
| Results differ with and without a debugger | The check sits in the main decision path |

## What to do about it — conceptually

Recognize and understand first. The reading order is:

1. Identify the check — which API call, which instruction.
2. Read the branch it controls — what changes when the check fires?
3. Decide what that result means for the rest of your analysis.

In CTF settings, standard approaches — patching the check, or making the program observe the state it expects — are documented in course material and write-ups; study them in that lab context rather than treating this article as a recipe. The skill being taught here is reading the check, not memorizing bypasses.

## Packing explained

A packed binary stores most of its sections compressed or encrypted; at startup a small stub decompresses the real code into memory. On disk it looks like garbage: `strings` returns fragments, imports may be hidden behind the stub, and the code you see first is the unpacker rather than the program. UPX is the classic example and is very common in crackmes and CTFs.

### Unpacking a UPX file

```bash
file packed_chall               # often reports "UPX compressed"
upx -d packed_chall -o chall    # decompress (verify options for your UPX build)
upx -l packed_chall             # list packed contents (verify)
```

Unpacking a file you are authorized to analyze is standard practice, like running `strings` — you are only reading it.

### What a packed file looks like

```bash
file packed_chall
# ELF 64-bit LSB executable ... (some versions report "UPX compressed")
strings packed_chall | grep -i upx
# UPX!
```

The `UPX!` marker and near-empty string output are the signal that the real code is not on disk in plain form.

## Why packing changes the workflow

Your normal sequence fails in a specific order: `file` says compressed, `strings` shows noise, the disassembler maps a stub instead of real code. The fix is to unpack first, then restart the standard workflow:

1. Unpack (`upx -d` for UPX files you are authorized to analyze).
2. Re-run `file` and `strings` on the unpacked result.
3. Open the unpacked file in the disassembler and find `main` as usual.

Packing changes the entry point of your process, not the concepts.

## Legal and ethical note

Anti-debug and packing exist to protect software. Bypassing protection on software you do not own and are not authorized to analyze — commercial products, games, licensed applications — is off-limits. Your targets in this material are binaries you own or are authorized to analyze: crackmes, CTF challenges, lab files, and your own builds.

## Packed binaries and malware

Packers are heavily used by malware authors to delay inspection, so packed samples are routine in defensive work, which requires isolated, purpose-built environments. Defensive analysis of such samples is covered at the [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/); never run unknown packed files outside that kind of isolation.

## Authorized use reminder

Everything above — recognizing checks, unpacking, and reading results — applies only to binaries you own or are explicitly authorized to analyze.

## See also

- [Strings & Patterns](strings-patterns.md)
- [Dynamic Analysis](dynamic-analysis.md)
- [Legal & Ethics](../fundamentals/legal-ethics.md)
- [What Is Reverse Engineering](../fundamentals/what-is-reversing.md)
