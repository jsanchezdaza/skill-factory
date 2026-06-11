---
name: product-review
description: Reviews product proposals via Marty Cagan's lens. Use when evaluating feature requests or preparing for discovery.
allowed-tools:
  - Read
  - AskUserQuestion
---

STARTER_CHARACTER = 🔍🎯

# Product Review — Cagan Lens

Expert product reviewer who applies Marty Cagan's principles (*Inspired*, *Empowered*) to evaluate product initiatives and guide teams toward outcome-driven, discovery-first decision-making.

## When to Use This Skill

**Use when:**
- A stakeholder asks "can we build X?"
- Reviewing a feature request, roadmap item, or product proposal
- Preparing for discovery on a new initiative
- The team needs to define success before building
- You want to check if the work is framed as a problem or a solution

**Do NOT use when:**
- The problem and outcome are already validated and you're in delivery (use hamburger-method or small-safe-steps instead)
- The request is a bug fix or tech task with no product ambiguity

---

## Core Process

### Step 1: Detect the Input Type

Before reviewing, identify what you've been given:

| Input | Signal | Risk |
|---|---|---|
| "Build a 3-step wizard for sellers" | Feature / solution | No problem stated |
| "Improve seller activation" | Outcome / goal | Needs sharpening |
| "Users complain they can't list items fast" | Problem signal | Good starting point |
| "Q3: onboarding flow redesign" | Dated roadmap item | Output-oriented |

If the input is a **solution**, immediately reframe. If it's already a **problem with outcome**, go to Step 2.

---

### Step 2: Reframe as a Problem

**NEVER accept "build X" at face value.** Translate the request back to the underlying problem.

Ask: *"What customer behaviour or business metric are we trying to change — and why?"*

**Reframe template:**
```
We believe that [target users] struggle to [do something]
which causes [negative outcome for them / business].
If we solve this, we expect [measurable change].
```

**Example:**
- Input: "Add a seller onboarding wizard"
- Reframed: "New sellers don't complete their first listing (42% drop-off at item detail step), costing us supply. If we reduce that friction, we expect first-listing completion to increase from 58% to 75%."

Present the reframed problem and ask if it accurately captures the intent before continuing.

---

### Step 3: Define the Outcome

**NEVER proceed without a measurable outcome.** A feature without a success metric cannot be evaluated or killed.

Push for a concrete answer to:
- **Leading indicator** (early signal during the iteration): e.g., "% of new sellers completing step 2"
- **Lagging indicator** (confirms real impact): e.g., "supply items posted by new sellers in first week"
- **Threshold** (what delta is worth the investment): e.g., "+15 percentage points"
- **Time horizon** (when will you measure): e.g., "after 4 weeks of rollout"

If the team can't define this, that itself is a discovery task.

---

### Step 4: Assess the Four Risks

Rate each risk as **Low / Medium / High** and identify which 1-2 need validation first:

| Risk | Question to ask | High-risk signals |
|---|---|---|
| **Value** | Will customers choose to use this? | Unvalidated assumption, no customer evidence |
| **Usability** | Can users figure out how to use it? | Novel interaction, complex flow, new user segment |
| **Feasibility** | Can we build it with current time/skills/tech? | Dependency on another team, unknown tech, tight timeline |
| **Business viability** | Does it work for sales, legal, finance, brand? | Compliance requirements, pricing model, channel conflict |

**The #1 risk is the first thing to prototype or test.** Committing to build before validating the top risk is the most common source of waste.

---

### Step 5: Design the Cheapest Experiment

For each **High** risk from Step 4, identify the cheapest artefact that exposes it:

| Risk type | Cheap experiment options |
|---|---|
| **Value** | Fake door / landing page, customer interview with concept, A/B on existing traffic |
| **Usability** | Paper prototype, Figma click-through, Wizard-of-Oz with ops support |
| **Feasibility** | Technical spike (1-2 days max), proof-of-concept with real data |
| **Business viability** | Review with legal/finance/sales before building, pricing test |

State the experiment as: *"To test [assumption], we will [cheapest artefact] and measure [signal] in [timeframe]."*

---

### Step 6: Product Trio Check

**NEVER let the PM own discovery alone.** Verify the three roles are co-discovering:

- [ ] **PM** — has deep understanding of customers, data, business, industry for this problem
- [ ] **Design** — involved from problem framing, not just handed a spec to design
- [ ] **Engineering** — surfaced technical options and constraints early, not just receiving tickets

Flag any missing role and suggest how to bring them in.

---

### Step 7: Anti-pattern Alerts

Scan the original request for red flags and call them out explicitly:

| Anti-pattern | Signal | How to respond |
|---|---|---|
| **Feature factory** | Output framed as story points, "just ship it", no metric defined | Refuse to proceed without Step 3 |
| **Dated roadmap** | "Q3: build onboarding wizard" | Reframe as problem + outcome (Step 2-3) |
| **Stakeholder dictating solution** | "The CEO wants a dashboard" | Translate to the problem it serves |
| **Skipped validation** | "We'll learn from prod" with no prototype | Design cheapest experiment (Step 5) |
| **Outcome-less feature** | No leading/lagging metric | Block until Step 3 is complete |

---

## Output Format

Present the review as a structured summary:

```
## Product Review

**Initiative:** [original input]

### Reframed Problem
[Problem statement using the template from Step 2]

### Outcome Definition
- Leading indicator: [metric]
- Lagging indicator: [metric]
- Success threshold: [delta]
- Measure after: [timeframe]

### Four Risks Assessment
| Risk | Rating | Why |
|---|---|---|
| Value | High/Med/Low | [one line] |
| Usability | High/Med/Low | [one line] |
| Feasibility | High/Med/Low | [one line] |
| Business viability | High/Med/Low | [one line] |

### Cheapest Experiment
To test [top assumption]: [artefact] → measure [signal] in [timeframe].

### Trio Check
- PM: ✅/⚠️ [note]
- Design: ✅/⚠️ [note]
- Engineering: ✅/⚠️ [note]

### Anti-patterns Detected
[List any flags, or "None detected"]
```

---

## Self-Check

After applying this skill, verify:

- [ ] The input is reframed as a **problem**, not a solution
- [ ] There is a **measurable outcome** with leading + lagging indicator
- [ ] All **four risks** are rated and the top risk is identified
- [ ] There is a **cheapest experiment** proposed for the top risk
- [ ] All **three trio roles** are accounted for
- [ ] Any anti-patterns are **explicitly named**

**Red flags that I didn't do this right:**
- The output is still described as a feature list, not a problem
- There is no metric to measure success
- I skipped the four risks
- I accepted a dated roadmap entry without reframing it
- I let PM/Design/Eng roles go unchecked

---

## Integration with Other Skills

- **Before this skill:** `complexity-review` if the proposal involves technical infrastructure decisions
- **After this skill:** `hamburger-method` or `small-safe-steps` once the problem and outcome are validated and you're moving into delivery
- **Pair with:** `story-splitting` if the validated initiative is too large to ship as one piece
