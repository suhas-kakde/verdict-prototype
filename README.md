# Verdict
### An AI reviewer inside Jira that tells you whether "done" is actually *done*.

> **▶ Try the live prototype:** **[REPLACE-WITH-YOUR-GITHUB-PAGES-URL]**
> Opens in any browser — no login, no setup. Best viewed on desktop. Takes about 3 minutes.
> *(After you enable GitHub Pages, paste your https://suhas-kakde.github.io/verdict-prototype/ link above.)*

---

## The problem we kept hitting

Every team marks stories "done" and epics "complete." But **"done" and "actually meets the acceptance criteria" are not the same thing** — and the difference has a habit of surfacing at the worst possible moment: in a demo, in regression, or in production.

Why it keeps happening:

- **The Product Owner is the only real backstop, and the backstop doesn't scale.** With dozens of stories in flight, nobody can deeply re-check every one against its criteria *and* its evidence. Review quietly becomes shallow or selective — and gaps slip through precisely because there's too much to look at.
- **The most expensive miss hides at the epic level.** An epic can have *every* child story closed and still fail its goal — because the gap was in how the epic was broken into stories, not in any single story. Each story did its job; the hole is in the plan. Nobody notices until release.
- **The cost compounds:** rework, late defects, and a slow erosion of trust in the word "done" — plus Product time spent re-reviewing work instead of making decisions.

In short: the gap between *what a story asked for* and *what actually got built* is found too late and reviewed too thinly.

## The idea

**Verdict** is an AI reviewer that lives inside Jira. The moment a story or epic is moved to a **Validate** status, it reads everything the issue already contains — the acceptance criteria (**including any added later in a comment**), the linked pull request, the QA evidence, and the analysis / regression sub-tasks — and answers two questions the team currently answers by hand, late, and inconsistently:

1. **How much of what was asked for was actually delivered?**
2. **Is the result sound?**

The key insight is that these are **two different signals**:

- **Coverage** is a **count** — *"8 of 8 criteria met."* Precise, because it's just counting.
- **Verdict** is a **judgment** — and it can **override** the count.

**A story can be 100% covered and still get a Risk verdict** — because a regression the criteria never named slipped in. That one idea is the whole product: a number can't catch what nobody thought to ask for; a judgment can. So the colour you see always follows the **verdict**, never the percentage.

And every claim Verdict makes is backed by **evidence you can click** — a file and line, a test name, a QA case, a sub-task id — so a reviewer verifies it in seconds instead of re-reviewing the whole story.

## Powered by AI

Verdict uses an LLM to do the part humans find tedious and error-prone: **semantically relating the full Jira context to the actual code and evidence, and judging whether each requirement is genuinely met** — not merely whether a box was ticked or a file was attached.

Two AI behaviours are worth calling out, because a checklist can't do them:

- **It reads criteria added mid-flight.** Requirements often get added in a comment *after* the story starts. Verdict reads those too, and tags them so everyone can see where they came from.
- **It investigates when evidence is missing.** If a story has *no* regression / impact sub-task, Verdict doesn't shrug — it runs its **own impact analysis on the pull request** and reports what it finds.

And when it genuinely can't be sure, it says so — a distinct **Unverified** state — instead of defaulting to a reassuring green. That honesty is what makes it trustworthy enough to eventually gate on.

> The hardest part is already de-risked: an LLM relating full Jira context to a PR and judging *"is this met?"* has already been demonstrated in a working review agent. This prototype shows the **product** built around that capability.

## See it yourself — a guided demo (~3 min)

The prototype opens on a realistic example: a work item about exposing a business unit's **User Sync mode** (`INT-57550`), part of a larger epic (`INT-57495`). You don't need to know the domain — just follow along.

**1. Run a verdict — and meet the "100% yet Risk" moment.**
The item opens on the status **Ready for Dev**. Click that status pill (near the top of the page) and choose **Validate**. Watch the AI work through its steps, then land on the result:

> **⛔ Risk — 8 / 8 criteria met (100% coverage).**

Read that again: *every* listed criterion is met, each with evidence — and the verdict is still **Risk**. Scroll down to see why. One criterion (*"return `sync_mode` everywhere"*) was **added later in a comment** (look for the **"added in comment"** tag). It was implemented by changing a *shared* piece of code — which also leaked data into a **different, out-of-scope endpoint** that other services don't expect. There was no regression sub-task on the story, so **the AI ran its own impact analysis on the pull request and caught it.** That's the catch a coverage number would have sailed straight past.

**2. Stay in control — act on the verdict.**
A verdict is a recommendation, not a silent gate. Under the result are three buttons:

- **Looks correct** — you agree. Verdict accepts it and **notifies the developer** to fix the flagged issue (it posts a comment and @-mentions them).
- **Correct this verdict** — you (the reviewer, or the developer who did the work) *disagree*. Click it, type **why** you think it's wrong, and Verdict **re-runs** — weighing your reasoning against the pull request and the sub-tasks — then shows a **"Reviewer challenge"** section with its response. In the demo it holds its ground with a specific reason *and* tells you exactly how to change its mind (e.g. add an acceptance criterion) — so it reasons with you instead of rubber-stamping.
- **Create fix PR** — open a pull request to fix the discrepancy, right from the panel.

Try **Correct this verdict**, type any objection, and watch it re-evaluate.

**3. Roll it up to the epic — and catch the miss nobody else catches.**
In the left sidebar, click **Verdict** (or the epic `INT-57495` in the breadcrumb), then click **Validate epic**. Verdict reads all six child reports at once and, instead of one blended bar, gives you an **honest breakdown** (*"6 children · 5 validated · 1 not run"*). Most importantly, it flags a **planning gap**:

> One epic criterion (*"refresh the config cache"*) has **no story delivering it.**

Every story is closed against its own criteria — but a required piece was never broken out into a work item. **The gap isn't in the code; it's in the plan** — the exact class of miss usually found only at release, when it's most expensive. Verdict finds it now, and names it.

**4. The board, and reset.**
Left sidebar → **HI Sinhagad Scrum Board** shows the work as verdict-coloured cards, with Verdict running in the **Validate** column. To restart the demo at any time, click **Reset demo** (top bar).

## What makes it different

1. **A verdict, not just a percentage.** Coverage counts; the verdict judges; the judgment can override the count — the "100% yet Risk" moment.
2. **Scope-drift detection.** In the same pass it surfaces work that shipped but was never asked for, and work that was asked for but nobody touched.
3. **Honest by construction.** Where evidence is thin it says **Unverified** and routes to a human — it never hides uncertainty behind a green light.
4. **A human stays in the loop.** Accept, challenge, or fix — every decision is deliberate and recorded, never an unaccountable automated gate.

## Why it matters

- **Catches the acceptance gap before release,** when it's cheapest to fix.
- **Scales review capacity without adding people** — Product reviews the few flagged items, not everything.
- **Makes "done" mean something** — a validated done, backed by citable evidence.
- **Catches the planning-level miss at the epic** — the expensive defect class usually found only at release.
- **Turns review into a feedback loop** — the accept / challenge actions become honest labels for measuring and improving accuracy over time.

## Where it goes next (roadmap)

- **State locks** — a story can't advance to the next status until its verdict is finished and confident. Turns Verdict from advisory into a safe gate, once accuracy is proven.
- **"Ignore verdict" with a required, recorded reason** — override and move ahead, but only with a written justification saved on the issue, so every override is deliberate and auditable.
- Deliver the verdict as a **pull-request comment** too (the developer's home turf), and auto-recompute the epic rollup when a child changes.

## About this prototype

This is a **clickable prototype**: the verdicts are authored to tell one coherent, realistic story end-to-end, so you can experience the product without wiring up a live model. The underlying AI judgment — reading Jira context and a PR and deciding whether requirements are met — is the part that's already proven. Everything here runs in your browser; there's no backend, no data leaves the page, and the scenario is a mock.

## Files in this repo

- **`index.html`** — the live prototype (this is what's hosted and linked above).

---

*Built for Sparkathon 2026.*
