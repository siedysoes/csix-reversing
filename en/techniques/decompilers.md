# Decompilers & Ghidra

A decompiler turns machine code back into C-like pseudocode, so you read `if (strcmp(a, b))` instead of a pile of conditional jumps. It is an approximation — never the original source — but it is the biggest speedup available to a beginner-to-intermediate reverser. This article covers how to read decompiler output honestly and use Ghidra on binaries you are authorized to analyze.

## What a decompiler does

A disassembler shows instructions; a decompiler reconstructs structure on top of them: statements, loops, conditions, function calls, and variables. The output is readable but lossy — names are gone, types are guessed, and some structure is inferred rather than proven. Treat it as a strong hint about the original code.

## Why it speeds everything up

Assembly control flow is a graph of jumps; pseudocode restores the shape a human wrote. You scan for the comparison, the loop, and the branch that prints success at the speed of C instead of the speed of opcodes.

## Pseudocode honesty: what you see vs reality

| What the pseudocode shows | What is actually true |
| --- | --- |
| Clean `if`/`else` blocks | The compiler may merge, reorder, or duplicate branches |
| Variable names like `local_8h` | Original names are gone — rename them yourself |
| Clear types | Types are guessed from usage and can be wrong |
| Struct field access | Sometimes raw byte offsets the decompiler pattern-matched |
| One tidy function | Optimization may have inlined or constant-folded parts of it |

Read pseudocode as a hypothesis, and confirm anything critical against the disassembly.

## Ghidra basics for CTF

Launch it from the distribution (the wrapper script is named `ghidraRun`):

```bash
ghidraRun
```

### First session, step by step

```
File > New Project -> choose a directory -> Non-Shared Project
File > Import File -> select chall -> OK (accept auto-analysis)
Window > Decompiler (the Pseudocode view in the Code Browser)
Search > For Functions -> main        (or use the Symbol Tree)
read the pseudocode for main
```

Auto-analysis does the heavy lifting on a first pass; accept the defaults until you know why you would change them.

## Ghidra workflow tips

- Search for a string, right-click it, and list its references — that lands you in the code that uses it.
- Rename constantly: functions (`check_password`), variables (`given_pin`), data labels. Renaming is your annotation layer.
- Fix obviously wrong data types: right-click a variable or field and set the type; correct types improve both pseudocode and your reasoning.
- Keep the disassembly view open beside the pseudocode to verify anything suspicious.

### Renaming in practice

The same function before and after thirty seconds of annotation:

```c
// fresh decompile
undefined8 FUN_004011a0(char *param_1) {
  int iVar1;
  iVar1 = strcmp(param_1, "s3cret");
  if (iVar1 == 0) {
    puts("Access granted");
  }
  // ...
}

// after renaming
int check_password(char *given) {
  int result;
  result = strcmp(given, "s3cret");
  if (result == 0) {
    puts("Access granted");
  }
  // ...
}
```

Nothing about the logic changed — but the second version reads like code you can reason about. Rename early and rename often.

## Other decompilers

| Tool | Cost / model | Notes |
| --- | --- | --- |
| IDA | Free version available; its limits vary by version (verify current) | Long-standing standard; commercial editions exist |
| rizin / Cutter | Free and open source | rizin is the radare2 fork; Cutter is its GUI |
| Binary Ninja | Paid (verify current editions and pricing) | Strong intermediate representation; cloud/log-in options |

## Common pseudocode patterns

| You see | It usually means |
| --- | --- |
| `strcmp(a, b)` | A password or key comparison |
| `atoi` / `strtol` | Numeric input being converted |
| `strlen(s)` | A length check or bounds guard |
| A loop over a buffer | Parsing, decoding, or transforming input |

Recognizing one of these in five seconds saves minutes of scrolling.

## Limits

Heavily optimized or obfuscated code stays hard no matter how good the decompiler: junk code, opaque predicates, and aggressive inlining blur everything. When pseudocode looks bizarre, fall back to the disassembly and the fundamentals — the machine always has the final word.

### When pseudocode disagrees

Pseudocode and disassembly can appear to disagree — a missing branch, an impossible condition, a variable that seems to hold two values at once. The disassembly is the authority: it is what the CPU executes. Use the disagreement as a signal to slow down, read the instructions, and check your assumptions about types and calling conventions. And remember that a decompiler never runs the code, so runtime-only data still needs [Dynamic Analysis](dynamic-analysis.md).

## Authorized use reminder

Use decompilers only on binaries you own or are authorized to analyze — lab exercises, crackmes, CTF challenges, and your own builds. Decompiling proprietary software you have no rights to examine is outside the scope of this material.

## See also

- [Static Analysis](static-analysis.md)
- [Assembly Basics](../fundamentals/assembly-basics.md)
- [Dynamic Analysis](dynamic-analysis.md)
- [Reversing Tools](../practice/reversing-tools.md)
