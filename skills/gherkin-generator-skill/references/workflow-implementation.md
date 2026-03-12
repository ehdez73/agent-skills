# Workflow: Two-Agent Iterative Cycle

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

## Complete Step-by-step Process

### Step 1: Validate and Prepare Requirements

Before invoking agents, confirm you have:
- **Module/feature name** (for the Feature header)
- **List of functional requirements** (numbered or prose)
- **Business context** (actors, systems, workflows)
- **Example data** (if available)

**If anything is missing or ambiguous, ask the user explicitly before proceeding.**

### Step 2: Invoke Generator Agent (Iteration P)

Read full instructions in `agents/generator-agent.md`.

**Input:**
```
requirements:      <list of functional requirements>
context:           <business context and actors>
previous_feedback: <empty on first iteration>
previous_gherkin:  <empty on first iteration>
iteration:         <current iteration number (starts at 1)>
```

**Output:** A valid Gherkin `.feature` code block.

### Step 3: Invoke Reviewer Agent

Read full instructions in `agents/reviewer-agent.md`.

**Input:**
```
gherkin:               <output from generator agent>
original_requirements: <user's original requirements>
iteration:             <current iteration number>
```

**Output:** JSON object with:
```json
{
  "approved": true|false,
  "score": 0-100,
  "summary": "...",
  "issues": [...],
  "additional_suggestions": [...],
  "annotated_gherkin": "..."
}
```

### Step 4: Improvement Loop

```
iteration = 1
maximum   = 5   ← prevents infinite loops

WHILE iteration <= maximum:
  gherkin  = invoke_generator_agent(...)
  review   = invoke_reviewer_agent(...)

  IF review.approved == true OR review.score >= 85:
    BREAK

  feedback         = review.issues + review.additional_suggestions
  previous_gherkin = gherkin
  iteration       += 1

IF iteration > maximum:
  deliver best gherkin obtained (with warning about max iterations reached)
```

### Step 5: Delivery

Present to the user:

1. The final `.feature` file (syntax highlighted in Gherkin)
2. Review score and summary
3. Number of iterations performed
4. Offer to save the file or refine further

---

## Configuration

| Setting | Value |
|---|---|
| Max iterations | 5 (adjustable per user request) |
| Input language | Detect from requirements; use same throughout |
| Output granularity | One `.feature` file per module/feature |
| Approval threshold | `approved: true` OR `score >= 85` |
| Early exit | If approved before max iterations, stop immediately |
