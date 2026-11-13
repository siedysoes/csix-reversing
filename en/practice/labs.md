# Practice & Labs

Reverse engineering is a skill you build by solving small, sanctioned targets repeatedly — not by reading about it. Labs give you legal, intentionally-designed binaries to cut your teeth on before you ever touch real-world work. This page maps the lab landscape, shows how to pick your first target, and gives you a repeatable practice loop.

> Everything here happens on sanctioned challenges and binaries you own — never on proprietary software you have no right to analyze.

## Why Practice on Sanctioned Targets First

Real-world targets are messy, hostile, and legally sensitive. Sanctioned targets (crackmes, CTF challenges, wargames) are designed to be solved, so you learn the workflow — triage, anchoring, decompiling, verifying — without legal risk or wasted time on unsolvable randomness. Skills transfer: the same loop works on an authorized engagement later.

## The Lab Landscape

| Platform | Level | Notes |
| --- | --- | --- |
| crackmes.one | Beginner to advanced | Community-uploaded crackmes, free — verify current availability and rules |
| picoCTF | Beginner | Annual CTF plus a practice problem set — verify current access |
| OverTheWire (Narnia-style wargames) | Beginner | Wargame basics: shell, scripting, simple reversing — verify current game names |
| HackTheBox — Reverse Engineering boxes | Intermediate | Some content on the free tier — verify current tiers and pricing |
| TryHackMe — RE rooms | Beginner to intermediate | Guided rooms with walkthroughs — verify current free/paid access |
| CTFtime-listed events | Intermediate to advanced | Real competitions with published write-ups — verify current event listings |

Pricing, free tiers, and platform features change — check each platform's current terms before committing.

## How to Choose Your First Target

| Experience | Target |
| --- | --- |
| Absolute beginner | picoCTF strings/rev problems — find a flag with `strings` and basic tooling |
| Comfortable with the tools | A beginner-level crackme — guess/validate a password or find a hidden key |
| Ready for more difficulty | An HTB reverse engineering box or a CVE-analysis exercise (authorized write-up reproductions) |

Pick one level below where you think you are. Finishing an easy target builds more skill than abandoning a hard one.

## The Drill Loop

Every target follows the same loop:

1. **Pick** a target you are authorized to solve.
2. **Timebox** it (see the schedule below) — decide in advance when to stop.
3. **Triage** — `file`, `checksec`, `strings` before touching a disassembler.
4. **Document** — write down commands, findings, and dead ends as you go.
5. **Compare** with the published write-up — note what you missed and why.

The comparison step is where most of the learning happens: it shows you which question you failed to ask.

### A Sample 30-Minute Session

| Minutes | Activity |
| --- | --- |
| 0–5 | Triage: `file`, `checksec`, `strings`; write the session question |
| 5–15 | Anchor and read: find the xref, decompile the target function |
| 15–25 | Hypothesize and verify: trace the logic, test your reading |
| 25–30 | Document findings and the next question before closing the file |

If triage alone consumes the whole slot, that is a normal result — record it and pick the session back up next time.

## Treat It Like a Workout

Consistency beats marathons. Short, regular sessions with a clear question each time outperform occasional all-nighters.

| Session | Focus |
| --- | --- |
| 30 min | One triage + one targeted question (e.g., "what does this string do?") |
| 30 min | One decompile-and-read pass on a single function |
| 30 min | Compare your notes with a write-up and update your checklist |
| Weekly | Re-solve an old target without looking at your notes |

Three 30-minute sessions a week will beat one 6-hour binge every month.

## Write Up Every Solve

Treat each solve as the first draft of your personal notes. A short write-up — target, protections, approach, key commands, what worked, what didn't — becomes searchable material for the next time you meet a similar pattern. Keep them alongside your workflow notes so your process improves with every target.

## Self-Drills With Your Own Binaries

You do not always need an online platform. Compile a small C program you wrote, then reverse your own work — it is 100% legal and instantly repeatable.

| Drill | Setup | Goal |
| --- | --- | --- |
| Hidden string | `char *s = "correct-horse";` print nothing | Find the string with `strings`, then locate its xref in Ghidra |
| Password check | `strcmp(input, "letmein")` | Find the comparison, confirm with gdb |
| Arithmetic key | `if (x * 31 + 7 == 1234)` | Solve for `x` from the decompiled code |
| Menu binary | `switch` on 1–4 choices | Map each branch to its function |
| Anti-tamper lite | Exit if a value differs from expected | Break on `exit`/early return, find the check |

Compile with debug symbols first (`gcc -g`), then rebuild stripped (`strip`) and redo the drill to feel the difference:

```bash
gcc -g -o drill drill.c    # symbols kept — easier first pass
strip drill                # rebuild without symbols — redo the drill
```

## The CSIX Ecosystem

- [CTF categories and practice pointers](https://siedysoes.github.io/csix-ctf/) — where reversing fits among CTF categories.
- [Home lab guide](https://siedysoes.github.io/csix-wiki/#/en/practical/hands-on-lab) — how to build the isolated environment these drills should run in.

## Safety Note

Run every practice binary in a VM with no shared folders and (for crackmes) no network — see the home lab guide above. Only use binaries provided by the platform or compiled by yourself; if a target's origin or authorization is unclear, leave it.

## See also

- [Reversing Tools](reversing-tools.md)
- [The Reversing Workflow](workflow.md)
- [Strings & Patterns](../techniques/strings-patterns.md)
- [Legal & Ethics](../fundamentals/legal-ethics.md)
