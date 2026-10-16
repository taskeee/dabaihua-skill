# 大白话 SKILL —— 防止 AI 汇报"修好了"但实际没修好 / dabaihua SKILL

> **一句话**：一个给"不写代码、只看界面"的用户用的 AI 行为约束技能（SKILL），把"改完之后有据可查、真的复测过"钉成硬约束。
> One-liner: a behavioral constraint skill for AI agents, designed for users who can't read code — pinning "verifiable, actually retested after the change" as a hard rule.

[English](README.en.md) | 简体中文

---

## 中文

### 这是什么

`dabaihua` 是一个 [Claude/DSH 形态的 SKILL](https://github.com/anthropics/skills)——一个可以放进 agent 技能目录的 `SKILL.md` 指令集。它管一件事：**让 AI 汇报"修好了"之前，先拿出用户自己能验证的证据。**

设计给这样的用户：

- 靠口述交代任务，不写代码、看不懂代码
- 看不见任何后端证据（日志、接口返回、数据库）
- 判断"修没修好"的唯一方式：**在那个界面上点一下，毛病还在不在**

### 解决什么痛点

**痛点一：验错层。**
AI 验收时本能挑自己能通过的那一层——接口返回 200、脚本退出码 0、日志干净、单测过。这些它自己就能跑、自己就能看。但用户验收的是另一层：界面上那个毛病还在不在。两层不是一回事，所以**"后台跑通了，报完成"和"用户看到完全没修"可以同时为真**。这不是撒谎，是省事——便宜的验收就在手边，贵的验收要开界面。

**痛点二：补丁糊弄。**
加个判断跳过、try/catch 吞掉、把出问题的入口藏起来、只改显示文案不改数据——报成"修好了"，最后修成屎山。

**痛点三：读资料只读标题。**
搜索结果的标题和摘要就在手边，读原文要再点一次——AI 就凭标题下结论，用户按结论去用，用错了才知道它压根没读过。

**痛点四：反射式反驳。**
对方观点是对的时，AI 为了"输出内容"而强行反驳，用"显得在思考"替代"真的在思考"。

### 用了之后有什么效果

| 改动前 | 改动后 |
|---|---|
| "改好了，接口返回 200" | "你要看见的是：在 XX 界面点 XX，看到 XX。改前截图 + 改后截图都给你" |
| 拿脚本跑通当验收 | 脚本结果只当附加证据，主验收 = 用户能自己操作的那条 |
| 出问题就打补丁 | 每次改动自报：改的哪层 / 为什么症状会消失 / 是不是绕过 |
| 只读标题就下结论 | 每条结论跟原始链接；链接打不开就明说"没读到原文" |
| 反射式反驳正确观点 | 对方对就承认；真有不同意见先给证据 |

技能里还有一套**"他的说法 → 该交的东西"翻译表**：把用户真实会话里的原话（"帮我弄一下""你后台跑通了有什么用""根本没修好""你就读了个标题吧"……27 条）逐条对上真实意图和必须交出的产物。这张表是从 275 个真实会话里挖出来的。

### 怎么使用

1. **安装**：把 [SKILL.md](SKILL.md) 放进你的 agent 技能目录（Claude Code 用户放 `~/.claude/skills/dabaihua/`；DSH 用户放 `$DSH_HOME/skills/dabaihua/`）。
2. **触发**：技能靠 description 语义匹配。对话里说"按大白话来""大白话模式"，或者出现任何一条触发词就会命中——改活类（"帮我弄一下""改一下""你看看这个""继续""到哪一步了""你给我测试好了再给我"）、验错层类（"根本没修好""还是老样子""界面还是没变""你后台跑通了有什么用""又改坏了""别打补丁"）、核实类（"瞎说 你去搜""你都出现幻觉了""口径不一样""你读全了吗""你就读了个标题吧""瞎反驳"）、停与撤（"不用测了 我截图给你""别看了""我发错了 不用管"）、流程类（"更新档案""交接""弄完记得关电脑""标记为废案""别用子进程""干重活"）。中文口语怎么说都能触发，不用记全。
3. **验收 AI 的活**：收活时问它三句——
   - 他能在哪看见它好了？
   - 改前那一次对照在哪？
   - 有没有悄悄多做或少做？

三条都能答，才算交活。

### 技能的核心机制（四条）

1. **验收钉在用户能看见的那一层**——动手前先写"他要看见的是：在哪个界面做什么，看到什么"，这句话里禁止出现接口/日志/数据库这些用户看不见的词
2. **前后对照才叫复测**——同一条复现步骤改前跑一次、改后跑一次，两次结果摆一起；只贴改后 = 没复测
3. **补丁必须自报**——是绕过就明说"这是绕过，不是修好"；同一毛病第二次修必须回到干净状态
4. **查资料先读原文**——每条结论跟原始链接，没读过原文就明说

### 适用 / 不适用

**适用**：修 bug、改界面、改配置、查资料下结论——一切"AI 替你干活、你要验收结果"的场景。

**不适用**：纯知识问答（不涉及执行）；写新软件时的代码结构约束（那种情况用结构类技能，本技能管"怎么证明真修了"，不管"怎么写不崩"）。

---

## English

### What is this

`dabaihua` is a [SKILL](https://github.com/anthropics/skills) (an instruction set for AI agents) that does one thing: **forces the agent to hand over evidence the user can verify themselves before it says "done".**

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
