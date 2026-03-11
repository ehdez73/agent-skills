---
name: gherkin-generator-skill
description: >
  Generates high-quality Gherkin (BDD) scenarios from functional requirements using a
  two-agent iterative cycle: a generator agent that creates/modifies the Gherkin and a
  reviewer agent that validates it and proposes improvements. The cycle repeats automatically
  until the Gherkin passes review.

  Use this skill whenever the user mentions: "generate Gherkin", "BDD scenarios",
  "Gherkin test cases", "Feature/Scenario/Given/When/Then", "requirements to Gherkin",
  "BDD specifications", or asks to transform functional requirements into behaviour tests.
  Also applies when the user brings a requirements document and wants test cases,
  acceptance criteria, or user stories with executable examples.
---

# Skill: Gherkin Generator (BDD)

Converts functional requirements into valid, high-quality Gherkin scenarios through an
iterative cycle with two specialised agents.

---

## System Architecture

```
Functional requirements
        │
        ▼
┌─────────────────────┐
│  gherkin-generator  │  ← Agent 1: generates/modifies the .feature file
│       -agent        │
└────────┬────────────┘
         │ Gherkin draft
         ▼
┌─────────────────────┐
│  gherkin-reviewer   │  ← Agent 2: validates and proposes improvements
│       -agent        │
└────────┬────────────┘
         │
    Approved?
    NO ──────────────────► returns to Agent 1 with feedback
    YES ─────────────────► delivers the final .feature file
```

---

## Step-by-step workflow

### Step 1 — Prepare the requirements

Before invoking the agents, extract from the user's input:
- **Module / feature name** (for the Feature name)
- **List of functional requirements** (numbered or in prose)
- **Business context** (actors, systems involved)
- **Example data** if available

If anything is ambiguous, ask the user before starting.

### Step 2 — Invoke the Generator Agent

Read the full instructions in `agents/generator-agent.md`.

Call the agent with:
```
INPUT:
  requirements:      <list of requirements>
  context:           <business context>
  previous_feedback: <empty on first iteration>
  previous_gherkin:  <empty on first iteration>
  iteration:         <current iteration number>
```

The agent returns a Gherkin block in `.feature` format.

### Step 3 — Invoke the Reviewer Agent

Read the full instructions in `agents/reviewer-agent.md`.

Call the agent with:
```
INPUT:
  gherkin:               <output from the generator agent>
  original_requirements: <user's original requirements>
  iteration:             <current iteration number>
```

The agent returns an object with:
- `approved: true/false`
- `score: 0-100`
- `issues: [list of problems]`
- `suggestions: [list of concrete improvements]`
- `annotated_gherkin: <the gherkin with inline comments>`

### Step 4 — Improvement loop

```
iteration = 1
maximum   = 5   ← prevents infinite loops

WHILE iteration <= maximum:
  gherkin  = generator_agent(requirements, feedback, previous_gherkin)
  review   = reviewer_agent(gherkin, requirements)

  IF review.approved == true OR review.score >= 85:
    BREAK

  feedback         = review.suggestions + review.issues
  previous_gherkin = gherkin
  iteration       += 1

IF iteration > maximum:
  deliver the best gherkin obtained with a warning note
```

### Step 5 — Delivery

Present to the user:
1. The final `.feature` file with syntax highlighting
2. The reviewer's final score
3. A summary of iterations performed
4. (Optional) Save the file if the user requests it

---

## Approval criteria (summary)

The Reviewer Agent approves when **all** of the following are met:

| Criterion | Minimum requirement |
|---|---|
| Requirements coverage | 100% of functional requirements covered |
| Gherkin syntax | No errors (Feature, Scenario, Given/When/Then) |
| Scenario independence | No order dependency between scenarios |
| Step clarity | No technical ambiguities |
| Example data | Scenario Outline + Examples tables when variants exist |
| Language | Consistent throughout the file |

---

## Implementation notes

- **Maximum iterations**: 5 (configurable by the user)
- **Language**: use the same language as the input requirements
- **Granularity**: one `.feature` file per module/feature
- **Tags**: add `@smoke`, `@regression`, `@happy-path`, `@edge-case` as appropriate
- **Background**: use only when ≥3 scenarios share the same Given steps

For implementation details of each agent, read:
- `agents/generator-agent.md`
- `agents/reviewer-agent.md`
- `references/gherkin-best-practices.md`
