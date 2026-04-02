# Show HN Draft — Ready to Post

**Target date:** Tuesday May 5, 2026, 9:00am PT

---

## Title

```
Show HN: Malwar – detect malware hidden in AI agent skill files
```

## Link

```
https://github.com/Ap6pack/malwar
```

---

## Founder Comment (post within 5 minutes)

```
Hey HN — built Malwar because I was auditing AI agent skill files and realized
nothing in my toolkit could catch what I was seeing.

The ClawHavoc campaign trojanized 341+ skills on ClawHub to deliver the AMOS
infostealer. A Snyk audit found 12% of all skills were malicious. The attacks
aren't binaries — they're Markdown files with natural language instructions
telling the AI to exfiltrate credentials, run obfuscated shell commands, or
download payloads from C2 infrastructure. VirusTotal ignores Markdown. Code
scanners don't understand intent. Nothing flagged any of them.

Malwar runs a 4-layer pipeline: rule engine (<50ms, fully offline) → URL
crawler → LLM analyzer → threat intel. The rule engine alone catches base64
payloads, prompt injection, credential harvesting, and known campaign IOCs.
The LLM layer catches social engineering attacks that regex can't touch —
like a "Prerequisites" section that casually asks you to curl | bash a
dropper.

It found 5 critical findings in a single skill file that was impersonating
the ClawHub CLI itself — ClawHavoc payload domain, base64-encoded dropper
decoding to a curl from 91.92.242.30, and SnykToxic campaign attribution.
Scan took 38ms.

Current limitations: 26 rules tuned primarily for ClawHub-style skill files.
The attack surface evolves fast and we need more rules for new patterns.
Would love feedback on false positive rates if you scan your own skills.

pip install malwar && malwar db init && malwar scan your-skill.md
```

---

## Response Strategy

**First 2 hours are critical** — respond to every comment.

Likely questions and prepared answers:

**"How is this different from YARA / Semgrep / CodeQL?"**
Those tools scan code. Skill files are Markdown — natural language instructions,
not executable code. The attacks are social engineering embedded in prose ("run
this prerequisite command"), base64 payloads inside inline code blocks, and
invisible Unicode characters for prompt injection. YARA doesn't parse Markdown
structure. Semgrep doesn't understand intent. Malwar is purpose-built for this
specific threat surface.

**"Why not just use an LLM to scan everything?"**
Cost, speed, and reliability. The rule engine runs in <50ms offline with zero
API calls. It catches the obvious stuff (base64 payloads, known C2 domains,
credential patterns) deterministically. The LLM layer is optional and handles
the long tail — social engineering, hidden intent, context-dependent attacks.
Most users run --no-llm for CI and only use the LLM for manual review.

**"12% seems high — is that real?"**
Sourced from Snyk's February 2026 audit: 341 malicious out of 2,857 skills
scanned. At peak scale Termdock reported 824 malicious across 10,700+ listings
(7.7%). Both numbers are published and independently verifiable. Links in
the README.

**"What about false positives?"**
We have a known issue (#1 on GitHub) where the rule engine flags documentation
that mentions file operations as suspicious. The LLM layer correctly identifies
these as FPs but currently can't suppress rule engine findings — that feedback
path is planned. For now, --no-llm gives clean, deterministic results on real
threats.

**"Can I use this for MCP servers / Cursor / other tools?"**
Yes. The AgentSkills spec is cross-platform — same SKILL.md format used by
Claude Code, Cursor, and others. Malwar scans any Markdown-based skill file
regardless of which agent runtime consumes it.
