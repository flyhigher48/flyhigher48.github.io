---
title: "Stop Prompting. Start Context Engineering"
description: "From eager-intern AI to coachable collaborator: context engineering, reverse prompting, bias guardrails, and the adjacent possible—with copy-paste prompts."
date: 2025-09-25
categories: [llm-ml]
tags: [llm, engineering, productivity, prompts, workflows, mcp, agents]
published: true
---

In the first post—**“Chatbots Can’t Think—But You Can”**—we framed LLMs as probability machines and shared a practical prompt template. This sequel levels you up from *prompting* to *context engineering* so the model stops acting like an overeager intern and starts behaving like a reliable teammate.

---

## The “eager intern” problem (and why coaching beats clever prompts)

LLMs default to **yes**. They won’t push back, ask for resources, or set boundaries unless you *teach them how*. Treat the model like a smart but junior colleague:

* **Delegate clearly**: scope, sources, constraints, definition of done.
* **Invite dissent**: insist on challenges and risk calls.
* **Reward honesty**: ask for “unsure” instead of confident guesses.

Your job isn’t to outsmart the model—it’s to **manage it**.

---

## Prompting vs. Context Engineering

**Prompting** says *what you want.*
**Context engineering** adds *how to do it, with what inputs, and to what standard.*

> **Prompting**: “Write me a sales email.”
> **Context engineering**: “Write a sales email using our brand voice (below), quote the call transcript (below), and align with these product specs. Return 140–170 words, subject line + preview text, and end with a single CTA.”

### A drop-in scaffold you can reuse

```
Role: You are a [seniority + domain].
Task: [exact deliverable].
Audience & Goal: [who, why].
Inputs: 
- Brand voice: [paste]
- Source snippets: [paste]
- Product specs: [paste]
Constraints: [length, tone, format, compliance notes].
Quality Bar:
- If info is missing, ask up to 3 questions first.
- Cite which input lines you used.
Definition of Done:
- Meets [objective metric], passes [checklist], no ungrounded claims.
```

**Rule of thumb:** If a human colleague couldn’t execute from your instructions, neither can the model. Make the implicit **explicit**.

---

## Reverse prompting: give it permission to ask first

Most failures come from unstated assumptions. End big tasks with:

```
Before you begin:
1) List what’s unclear or risky (max 5 bullets).
2) Ask 2–3 focused questions to close gaps.
Wait for my answers, then proceed.
```

This turns guessing into **requirements gathering**.

---

## Turn flattery into feedback: demand hard grades

Models tend to be nice. Niceness doesn’t help you improve. Ask for **tough scoring with fixes**:

```
Act as a stern Olympic judge.
Grade this draft on a 0–100 scale. 
Return ONLY:
- Score: [number]
- 3 biggest weaknesses (plain language)
- 3 specific edits that would raise the score by 10+ points
- One example sentence that demonstrates the improved style
```

That cheeky “**Score: 42**” moment beats “Looks good!” every time.

---

## Bias guardrails: design against your own blind spots

Jeremy’s core warning: AI mirrors **100% of human biases**. Don’t let it amplify yours.

1. **Request dissent by default**

```
Play devil’s advocate in one short section:
- Which assumption is most fragile?
- What evidence would falsify my claim?
```

2. **Role-rotate perspectives**

* “Now respond as: a skeptical peer reviewer → a compliance auditor → a hostile customer.”

3. **Counterexample few-shots**

* Provide one **bad** example alongside a good one so the model learns the pitfalls, not just the pattern.

4. **Verification lane**

* Require the model to flag uncertain claims and mark anything that lacks a cited input.

---

## “Reasoning,” but right-sized

You don’t need the model’s entire inner monologue. Ask for **concise, checkable scaffolding**:

```
Before the final answer, give:
- Assumptions (3 bullets)
- Plan (3–5 steps)
- Risks & mitigations (max 3)
Then deliver the final output.
```

Short, auditable reasoning beats sprawling step-by-step walls of text.

---

## Few-shot: examples over adjectives

Adjectives (“make it punchy”) are vague; **examples** are concrete.

```
Style Pack
Good:
- “Subject: The 7-minute fix your invoices need”
- “Body: One insight, one stat, one CTA.”
Bad:
- “Subject: Unlock your potential today!”
- “Body: Generic claims, no proof, 3 CTAs.”
Copy Rules
- Short subject (≤6 words). 
- One stat with a source line.
- Exactly one CTA.
```

Paste this “mini-dataset” with your task. The model imitates patterns it sees.

---

## Flight simulator for tough conversations

Roleplay lets you rehearse and debrief high-stakes chats (negotiations, performance reviews).

```
Phase 1 — Profile
From this brief: [paste notes], build a persona with goals, fears, likely objections.

Phase 2 — Sim
Simulate a 10-minute dialogue. Increase difficulty gradually (3 rounds). Keep my lines short.

Phase 3 — Debrief
Score my performance (0–100). 
Call out missed opportunities and give 3 rewrites I should try next time.
```

Run the loop twice: once friendly, once adversarial.

---

## Numbers to keep perspective

Jeremy cites stats from the AI-assisted coding world: **~2.5M builders** producing **~100,000 products/day**. Whether those exact numbers fluctuate, the takeaway holds—barriers to execution are collapsing. The bottleneck is shifting from **ability** to **imagination**.

---

## The Adjacent Possible (why imagination is the real limiter)

Innovation expands one door at a time: today’s capabilities unlock tomorrow’s ideas. The more diverse people who experiment—teachers, operators, scientists, founders—the wider our collective **adjacent possible**. Your experiments this week become someone else’s starting point next week.

**Weekly practice (15 minutes):**

1. Pick one task you repeat.
2. Write a context pack (role, inputs, constraints, DoD).
3. Add one good + one bad example.
4. Run reverse prompting.
5. Ship the improved output and save the prompt for reuse.

---

## Copy-paste toolkit

**1) Coach Mode (boundaries + honesty)**

```
You are my AI teammate. If my request is vague, risky, or missing inputs, pause and ask up to 3 clarifying questions. 
If you’re uncertain, say “unsure” and ask for a source. 
Your goal is accuracy over speed and usefulness over flattery.
```

**2) Context Pack Header**

```
Context:
- Brand voice: [paste]
- Source snippets: [paste with identifiers]
- Product/specs: [paste]
Constraints: [format, length, tone, compliance]
Definition of Done: [objective tests]
```

**3) Tough Grader**

```
Act as a rigorous reviewer.
Output:
- Score 0–100
- Top 3 weaknesses
- 3 edits to raise score by 10+
- One rewoven example paragraph
```

**4) Bias Spotter**

```
Scan for bias and blind spots.
Return:
- The most questionable assumption
- A counter-hypothesis
- What data would change your mind
```

**5) Requirements First (reverse prompting)**

```
Before drafting, list missing info and ask 2–3 specific questions.
Wait for answers, then produce the deliverable.
```

---

## A tiny Definition-of-Done (DoD) you can reuse

* ✅ Uses only supplied sources (quotes labeled)
* ✅ Meets length/tone/format constraints
* ✅ Flags uncertainty and open risks
* ✅ Contains one measurable outcome or success metric
* ✅ Passes “one-screen skim” (executive-readable in 30 seconds)

---

### Bring it home

LLMs aren’t minds. They’re mirrors and amplifiers. With **context engineering**, **reverse prompting**, and **hard-nosed feedback**, you turn a yes-bot into a thinking partner—one that challenges you, not flatters you. Do that consistently and you’ll keep stepping into the **adjacent possible**—one clear, well-scoped task at a time.
