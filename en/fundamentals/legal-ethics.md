# Legal & Ethics

Reverse engineering is a powerful skill, and with it comes an obligation: the same techniques that protect users can be used against them. This article comes first in the fundamentals track because a technical mistake can be corrected, while analyzing a binary you had no right to touch cannot be undone. Treat authorization as part of the skill, not paperwork around it.

## Why This Comes First

Tools and techniques are neutral; what separates security research from wrongdoing is authorization, scope, and intent. Every command and disassembler session later in CSIX Reversing rests on the foundation laid in this article — if that foundation is wrong, nothing built on top of it is defensible.

Two questions resolve most cases: were you given this binary with the expectation that you would look at it, and does someone with authority agree? If both answers are yes, you can work in good faith and document it. If either answer is unclear, taking a minute to ask now is always cheaper than explaining later — you cannot un-read a binary you had no right to open.

## The Core Principle

Your right to analyze a binary comes from one of exactly three sources:

- **Ownership.** You wrote it, or you hold the rights that come with owning it.
- **Permission.** A license, challenge description, or rules-of-engagement document grants analysis rights.
- **Authorization.** Someone entitled to grant access explicitly allows it: an employer, a client under contract, a bug-bounty scope.

If none of the three applies, do not analyze the binary. Note that owning a *copy* of software does not automatically include the right to reverse it — that is where the laws below come in.

## Where Activities Fall

| Activity | Typical status | Notes |
|---|---|---|
| Reverse engineering a program you wrote or own | Generally OK | Your binary, your analysis |
| Solving crackmes and CTF challenges you were given | Generally OK | Analysis is exactly what the author intended |
| Testing a binary assigned by an employer or client | Generally OK within scope | Stay inside the written scope and rules of engagement |
| Clean-room interoperability work | Gray area | Often permitted for interoperability, but it varies by jurisdiction — verify local law |
| Decompiling a proprietary app in violation of its ToS | Gray area | Frequently a contract matter; statutes can change that — verify |
| Bypassing DRM or license checks on software you do not own | Off-limits | Circumvention is restricted in many jurisdictions |
| Analyzing a malware sample in an isolated lab | Gray, tending to OK with care | Recognized defensive research when containment is solid |

## Laws at a Glance

This is orientation, not legal advice — verify the rules that actually apply to you:

- **United States — DMCA.** The Digital Millennium Copyright Act contains anti-circumvention provisions restricting the bypass of technological protections such as DRM, alongside exemptions that have been revised repeatedly, including ones relevant to security research. Read the current text before relying on any exemption.
- **European Union — Software Directive.** Directive 2009/24/EC allows, under defined conditions, studying a program and decompulating it to achieve interoperability. The conditions are specific; confirm how they apply to your situation.
- **Your jurisdiction.** Copyright, computer-misuse, and trade-secret laws differ widely by country and change over time.
- **Verify what applies to you.** Exemptions expire, statutes get amended, and no tutorial — including this one — is a substitute for qualified advice in your own jurisdiction.

### Gray areas deserve care

Anything marked gray in the table above depends on facts and location: which country you are in, what the license says, what you actually did, and who was harmed. When a situation is genuinely important to you, slow down and get qualified advice rather than reasoning it out alone.

## Terms of Service vs. Law

Breaching a Terms of Service agreement is usually a contract issue: the vendor may terminate your account or pursue a civil claim. A crime requires a statute — a broken contract does not by itself make conduct criminal, and a clean ToS record never overrides the law. Both can apply at once, which is why ToS questions sit in the gray zone; check your license terms *and* the law, not one or the other.

## Professional Norms

- **Employer authorization.** Analyze only the binaries and systems named in your assignment or scope, and keep that scope in writing.
- **NDAs and confidentiality.** Findings from a job stay within the job; know what your contract covers before discussing anything publicly.
- **Responsible disclosure.** Vulnerabilities in products you do not own go to the vendor first — see below.
- **Documentation.** Keep records of what you were authorized to do; when a question comes up later, records answer it.
- **Proportionality.** Use the least intrusive method that answers your question — a `strings` pass before a debugger session, a local copy before a live target.
- **When unsure, ask.** A short question to a manager, instructor, or program owner costs minutes; an unauthorized test can cost a contract or a job.

## Malware Analysis Is Legitimate Research

Defenders have to understand malicious binaries in order to write signatures, block infrastructure, and warn victims, and the research culture around this is well established. It rests on containment: analyze samples in an isolated lab — a disposable virtual machine with snapshots, no shared folders, and network access disabled or tightly controlled — never on your everyday machine. The companion [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/) site covers that environment in more detail. Receiving a sample by accident is not unlimited permission to study it; know why you have it, and contain it regardless.

## The "Did I Get This Binary Legitimately?" Test

Before you open anything, answer these honestly — if any answer is no, stop and get permission first:

1. Do I own this program, or hold the rights to analyze it?
2. Was I given this binary expecting analysis — a challenge, a lab exercise, an assignment?
3. Does someone entitled to grant access — employer, client, vendor — permit this exact activity?
4. Am I inside the scope, license terms, and jurisdiction that apply to me?

A yes to one of the first three, together with number four, is your green light. Otherwise the correct move is to ask, not to proceed quietly.

## Reporting Vulnerabilities Responsibly

If analysis turns up a vulnerability in a product you do not own:

- Contact the vendor first, through its published security channel (`security.txt`, a `security@` address, or a bug-bounty program).
- Give enough detail to reproduce the issue, and stop at proof of concept — do not access real user data or keep access you were not granted.
- Agree a reasonable fix window, then coordinate publication (coordinated disclosure). Timelines vary by program; weeks rather than hours is common practice — verify current guidance for the specific program.
- If the vendor is unresponsive, involve a recognized disclosure intermediary instead of jumping straight to full public disclosure.

## Red Lines

```text
Never:
  - analyze a binary you do not own and are not authorized to test
  - bypass DRM or license enforcement on third-party commercial software
  - exfiltrate real user data as "proof" of a vulnerability
  - publish exploit details before the vendor has had a chance to fix them
  - move malware out of containment, or test anything outside your written scope
```

If a proposed action lands on this list, no amount of good intent changes the answer. The whole CSIX Reversing track is written for binaries you own or are authorized to analyze — crackmes, lab exercises, CTF challenges, and systems inside your professional scope.

## See also

- [What Is Reverse Engineering](what-is-reversing.md)
- [Practice & Labs](../practice/labs.md)
- [The Reversing Workflow](../practice/workflow.md)
- [Dynamic Analysis](../techniques/dynamic-analysis.md)
