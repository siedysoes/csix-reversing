# The Reversing Workflow

Random poking at a binary produces random results — you cannot reproduce a lucky find, and you never learn what you missed. A fixed workflow makes every analysis repeatable, comparable, and teachable. This page lays out the end-to-end process from first `file` command to finished notes.

> The workflow is for binaries you own, were given, or are authorized to analyze — sanctioned challenges and your own builds. Nothing here is a license to reverse software you have no rights to.

## Why a Method Beats Random Opening a Disassembler

- **Reproducible:** the same steps yield the same findings, so you can retry a failed analysis systematically.
- **Teaches you what you missed:** a documented sequence makes gaps obvious — you can see exactly which step you skipped.
- **Fast triage first:** most time is wasted reading the wrong function; the early steps deliberately narrow the search before you read any code.

## The End-to-End Workflow

```text
1. Triage   — file, checksec, strings
2. Map      — sections, imports, entry point
3. Anchor   — find the interesting string/function via xref
4. Read     — decompile the target function
5. Hypothesize — what does it check, and how?
6. Verify   — test your understanding (static, or dynamic in VM if authorized)
7. Document — commands, findings, screenshots
```

Work the steps in order. The discipline is in not skipping ahead to step 4 before triage has told you where to look.

### Step 1: Triage

Three commands answer "what am I dealing with?":

```bash
file target            # format, architecture, stripped?
checksec --file=target # protections — verify flags for your version
strings -n 6 target | less   # readable hints: prompts, paths, errors
```

Triage tells you whether the binary is packed, stripped, protected, or full of low-hanging strings before you invest in a disassembler.

### Step 2: Map

Open the binary in Ghidra (or your disassembler of choice) and build a mental map: the sections (`.text`, `.data`, `.rodata`), the imports (what external functions does it call?), and the entry point. Imports are a cheat sheet — a call to `strcmp` or `strlen` often points straight at the interesting logic.

### Step 3: Anchor

Never read the binary linearly. Find one anchor — an interesting string from triage, a suspicious import, a function name — and follow its cross-reference (xref) to the code that uses it. One good anchor collapses a 10,000-line binary to a single function.

### Step 4: Read

Decompile the anchored function and read it as if it were source code. Focus on control flow: what is compared, what happens on success, what happens on failure. Ignore compiler-generated noise on the first pass.

### Step 5: Hypothesize

State your understanding explicitly: "This function compares the input against a stored value and prints 'Access granted' only on match." Writing the hypothesis turns vague intuition into something you can prove wrong.

### Step 6: Verify

Test the hypothesis: trace the logic statically through the callers and branches, or — if you are authorized to run the binary and are in your VM — confirm dynamically with `gdb` or `strace`. An untested hypothesis is a guess.

### Step 7: Document

Record the commands you ran, the anchor you followed, the decompiled logic (with screenshots if useful), and the verdict. Your future self and your write-up both start from this file.

## Triage Decision Table

| Finding | Direction |
| --- | --- |
| Interesting string ("wrong password") | Find the xref to that string — the check is nearby |
| Import: `strcmp` / `strncmp` | Likely a password/key check — read the calling function |
| Import: `fopen` / `open` | It reads a file — trace what it expects as input |
| No strings at all, odd sections | Possibly packed → see [Anti-Debug & Packing](../techniques/anti-debug.md) |
| Anti-debug signs (`int3`, `IsDebuggerPresent`) | Timing/debug checks — plan a dynamic-analysis countermeasure |
| Stripped, tiny binary | Start from the entry point and follow calls outward |

## The "One Question at a Time" Rule

Every session starts by writing one question: **what do I want to learn?** — "Does this binary check a password?" beats "let's see what's in here." When the question is answered, write the next one or end the session. One question keeps sessions focused and finishable.

## Timeboxing

| Situation | Action |
| --- | --- |
| Same spot for 20+ minutes with no new information | Switch approach (static ↔ dynamic) or re-triage |
| Making steady progress on one function | Push through — finish the read before switching |
| Fatigued, re-reading the same lines | Sleep — fresh eyes find anchors faster |
| Target clearly beyond your current skill | Shelve it, pick an easier one, come back later |

Stop-loss prevents the classic failure mode: burning an evening inside one wrong function.

## Common Dead Ends

| Dead end | Correction |
| --- | --- |
| Down a rabbit hole in an unrelated function | Re-check your anchor — is this function actually reached from the input path? |
| Over-reading compiler noise (stack setup, copies) | Read control flow first; ignore mechanics on pass one |
| Ignoring the binary's actual input path | Trace backward from how input arrives (`argv`, `stdin`, file) — the check is on that path |
| Arguing with optimized assembly | Switch to the decompiler view, or rebuild your own test binary unoptimized |

## Documenting As You Go

Keep a scratch file open for the whole session. A minimal format:

```markdown
# target: login-check (authorized crackme)
date: 2026-01-10
question: what does the check compare against?

## triage
- file: ELF 64-bit, stripped, not packed (checksec: NX, PIE, canary)
- strings: "Enter key:", "Access granted" at ...

## findings
- xref from "Access granted" -> sub_401234
- decompile: strcmp(input, s) at ...

## verdict / next question
- hypothesis confirmed statically; dynamic test pending in VM
```

## Quality Checklist for a Finished Analysis

- [ ] The original question is answered with evidence, not a guess.
- [ ] Key findings are traced from input to the decision point.
- [ ] Commands and versions needed to reproduce are recorded.
- [ ] Hypothesis was verified (static reasoning, or dynamic run in a VM where authorized).
- [ ] What remains unknown is listed honestly.

## When to Go Dynamic

Static analysis hits a wall when the interesting code is computed at runtime, obfuscated, or unpacked on the fly. At that point, move to a snapshot-ready VM and debug the running process with `gdb` — see [Dynamic Analysis](../techniques/dynamic-analysis.md) for the handoff. Dynamic work still applies only to binaries you are authorized to run.

## See also

- [Reversing Tools](reversing-tools.md)
- [Practice & Labs](labs.md)
- [Static Analysis](../techniques/static-analysis.md)
- [Dynamic Analysis](../techniques/dynamic-analysis.md)
