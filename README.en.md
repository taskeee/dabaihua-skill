# dabaihua SKILL — Stop the AI from reporting "fixed" when nothing is fixed

> **One-liner**: a behavioral constraint skill for AI agents, designed for users who can't read code — pinning "verifiable, actually retested after the change" as a hard rule.

[English](README.en.md) | [简体中文](README.md)

---

### What is this

`dabaihua` ("plain speech") is a [SKILL](https://github.com/anthropics/skills) — an instruction set for AI agents. It does one thing: **forces the agent to hand over evidence the user can verify themselves before it says "done".**

Built for users who:

- give tasks by voice, don't write or read code
- cannot see any backend evidence (logs, API responses, database rows)
- judge "is it fixed?" only by: **open the UI, click it, is the problem still there**

### The pain points it solves

**1. Verifying the wrong layer.**
When self-reviewing, an agent instinctively checks the layer it can pass itself — API returns 200, script exits 0, logs are clean, unit tests pass. But the user verifies a different layer: is the problem still visible in the UI? These are not the same thing, so **"the backend ran fine, reporting done" and "the user sees nothing fixed" can both be true at once**. It's not lying; it's laziness — the cheap check is right there, the expensive one requires opening the UI.

**2. Patch-over-patch.**
Add an `if` to skip the case, swallow the exception, hide the broken entry point, change the label but not the data — reported as "fixed", ends up as a pile of hacks.

**3. Concluding from titles only.**
Search-result titles and snippets are right there; opening the original page costs one more click — so the agent concludes from titles, and the user only finds out when applying the conclusion fails.

**4. Reflexive contradiction.**
When the other side is right, the agent argues anyway — just to produce output. Performing thinking instead of thinking.

### What changes after using it

| Before | After |
|---|---|
| "Fixed, API returns 200" | "You should see: open X, click Y, observe Z. Before/after screenshots both attached" |
| Script exit code as acceptance | Script output is supplementary evidence only; the real acceptance = an operation the user can run |
| Patches on failure | Every change self-reports: which layer / why the symptom disappears / is this a workaround |
| Conclusions from titles | Every conclusion links the original source; can't fetch it → says so, never invents |
| Reflexive contradiction of correct views | Agrees when the other side is right; disagreement requires evidence |

The skill also includes a **"user phrase → required deliverable" translation table**: 27 real phrases mined from 275 real sessions ("just fix it", "what good is the backend passing", "still not fixed", "did you even read it"...) each mapped to its real intent and the deliverable that must follow.

### How to use

1. **Install**: drop [SKILL.md](SKILL.md) into your agent's skills directory (Claude Code: `~/.claude/skills/dabaihua/`; DSH: `$DSH_HOME/skills/dabaihua/`).
2. **Trigger**: the skill matches by description semantics. Say "按大白话来" / "大白话模式", or let any trigger phrase appear — it fires on everyday spoken Chinese across five buckets: task phrases ("帮我弄一下" / just fix it, "继续" / continue, "到哪一步了" / where are you), wrong-layer phrases ("根本没修好" / still not fixed, "你后台跑通了有什么用" / what good is the backend passing, "别打补丁" / no patches), verify phrases ("瞎说 你去搜" / you never searched, "你读全了吗" / did you even read it, "瞎反驳" / baseless contradiction), stop phrases ("不用测了 我截图给你" / stop, I'll screenshot), and process phrases ("更新档案" / update the archive, "干重活" / heavy work mode). Casual spoken Chinese is enough — no need to memorize the full list.
3. **Verify the agent's work** — ask three questions when receiving a delivery:
   - Where can I see it working?
   - Where is the "before" comparison?
   - Did it silently do more or less than asked?

All three answered = a real delivery.

### Core mechanisms (four rules)

1. **Pin acceptance to the layer the user can see** — before starting, write "the user should see: open X, do Y, observe Z"; this sentence must not contain API/log/database terms the user cannot see
2. **Before/after comparison = real retest** — same reproduction steps, run once before the change, once after, both results shown together; "after" only = not retested
3. **Self-report patches** — if it's a workaround, say "this is a workaround, not a fix"; fixing the same bug twice requires returning to a clean state first
4. **Read the source before concluding** — every conclusion carries its original link; not read → say so

### Applicable / not applicable

**Use for**: bug fixes, UI changes, config changes, research conclusions — any "agent does the work, user verifies the result" scenario.

**Not for**: pure knowledge Q&A (nothing is executed); code-structure constraints when writing new software (use a structure-focused skill — this one governs "how to prove it's really fixed", not "how to write code that doesn't break").

---

## License

MIT
