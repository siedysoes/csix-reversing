# What Is Reverse Engineering

Reverse engineering is the practice of understanding how a finished program works by examining the program itself instead of its source code. It matters because most software you meet in security work — firmware, malware, closed-source tools — rarely ships with readable code attached. This article defines reversing, shows where it is used legitimately, and sets the mindset for the rest of the CSIX Reversing fundamentals track.

## Definition

In software, reverse engineering means taking an artifact whose inner workings were not documented to you and building an accurate model of how it functions. In practice that means three things:

- reading the binary's bytes and decoding them into instructions
- tracing how data moves through the program
- observing behavior under controlled conditions and testing hypotheses about intent

The output of reversing is understanding: notes, diagrams, a description of a protocol, a finding about a vulnerability. Analysis like this does not require modifying the binary — in fact, your first sessions should not modify anything at all.

## The Reversing Loop

Every session follows roughly the same cycle:

1. **Observe** — run read-only inspection: `file`, `strings`, a disassembler.
2. **Hypothesize** — guess what a region of code or data does.
3. **Verify** — check the guess against the instructions, or against a debugger trace.
4. **Document** — write down what you proved, and clearly separate it from what you assumed.
5. **Repeat** — move to the next unknown.

Skepticism is built into the loop: assume nothing, verify everything.

## A Bit of History

Reverse engineering predates software — hardware engineers have always studied finished circuits, engines, and machines to learn how they work, and software reverse engineering simply inherited that approach: examine the finished artifact, observe it under controlled conditions, and rebuild your understanding of it.

## Who Does It

| Role | What they reverse |
|---|---|
| Security researchers | Applications, protocols, and firmware, to find vulnerabilities |
| Malware analysts | Malicious samples, to understand behavior and build defenses |
| Interoperability developers | Closed file formats and APIs, so separate products can communicate |
| Hardware and firmware engineers | Device firmware and drivers that bridge hardware and operating system |
| Students and CTF players | Crackmes and challenges, to build the skill itself |

## Legitimate Uses

| Use | What it looks like in practice |
|---|---|
| Security research | Determining whether a product's update mechanism can be tampered with |
| Malware analysis | Extracting configuration from a sample so its traffic can be blocked |
| Interoperability | Learning a file format so two programs can exchange data correctly |
| CTF and practice | Solving crackmes and reversing challenges on binaries you were given |
| License compliance | Verifying which open-source components are linked into a product |
| Recovering lost source | Rebuilding understanding of an in-house binary when the code is gone |

## What Reverse Engineering Is NOT

- **Not automatically illegal.** Legality depends on who owns the binary, what your license says, and which jurisdiction you are in. See [Legal & Ethics](legal-ethics.md).
- **Not piracy cracking.** Removing license checks from commercial software you do not own in order to use it for free is software piracy. That is a different activity with a different purpose, and it is not what this guide teaches.
- **Not necessarily modification.** A lot of reversing ends in a written report, not a patched binary.
- **Not magic.** It is patient, step-by-step reading — closer to debugging than to guessing a password on the third try.

## Reversing Next to Related Skills

| Activity | The question it answers |
|---|---|
| Reverse engineering | How does this program work? |
| Debugging | Why did my program do that, right now? |
| Code review | What is wrong with this source code? |
| Exploitation | How can I make the program do something unintended? |

## How It Connects to the CSIX Ecosystem

Reversing rarely happens in isolation. The companion CSIX sites cover the neighboring skills:

- [CSIX CTF](https://siedysoes.github.io/csix-ctf/) — where you practice these skills on challenges built for practice
- [CSIX Pentest](https://siedysoes.github.io/csix-pentest/) — how the weaknesses you find are turned into working exploits
- [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/) — defensive analysis of malicious binaries in a contained environment

## The Mindset

Three habits carry most sessions:

- **Curiosity.** Ask what a function does, then answer it with evidence.
- **Patience.** You will misread instructions, follow dead ends, and forget what a register does. That is normal: look it up and continue.
- **Skepticism.** Assume nothing, verify everything. A function named `check_auth` may not check anything, and a string reading "success" may be printed on the failure path.

### Your first reversing session

A typical first session on a simple crackme needs only two read-only commands:

```bash
strings crackme
strings crackme | grep -iE 'flag|correct'
```

`strings` extracts readable text from the binary; the grep narrows it to likely answers. If the second command prints something like `Correct! Flag{you_found_me}`, you have found an embedded flag without running or changing anything — plain observation of a challenge binary you were given. From here, continue with [Strings & Patterns](../techniques/strings-patterns.md) and the full [The Reversing Workflow](../practice/workflow.md).

## Authorization First

Everything in CSIX Reversing assumes one rule: analyze only binaries you own, were given, or are explicitly authorized to analyze — your own programs, lab exercises, crackmes, and CTF challenges. If you cannot honestly answer "am I allowed to look at this?" with yes, stop and get permission before continuing.

## See also

- [How Binaries Work](how-binaries-work.md)
- [Assembly Basics](assembly-basics.md)
- [Legal & Ethics](legal-ethics.md)
- [Strings & Patterns](../techniques/strings-patterns.md)
- [The Reversing Workflow](../practice/workflow.md)
