# Reference: Gherkin / BDD Best Practices

## Table of Contents
1. [Anatomy of a .feature file](#anatomy)
2. [Step rules](#steps)
3. [Scenario Outline and Examples](#outline)
4. [Recommended tags](#tags)
5. [Common anti-patterns](#anti-patterns)
6. [Quality checklist](#checklist)

---

## 1. Anatomy of a .feature file {#anatomy}

```gherkin
Feature: <Module name>                  ← REQUIRED, one single Feature per file
  As a <role/actor>                     ← Who benefits
  I want <functionality>                ← What they want to do
  So that <business goal>               ← Why they need it

  Background:                           ← OPTIONAL — shared preconditions (≥3 scenarios)
    Given ...
    And ...

  @tag1 @tag2                           ← Optional per-scenario tags
  Scenario: <description of expected behaviour>
    Given <initial context>
    When <actor's action>
    Then <observable result>
    And <additional result>

  Scenario Outline: <description with variants>
    Given <context with <variable>>
    When <action with <variable>>
    Then <result with <variable>>

    Examples:
      | variable | other_field |
      | value1   | data1       |
      | value2   | data2       |
```

---

## 2. Step rules {#steps}

### Given — Context / Precondition
✅ Describes the **system state** before the action
✅ Can include data already existing in the system
✅ Can be a user state (authenticated, with permissions, etc.)
❌ Does not describe user actions
❌ Does not include business logic

```gherkin
# ✅ Correct
Given an order in "pending" status with id "ORD-001" exists
Given the user "Mary" has the "administrator" role

# ❌ Incorrect
Given the user logs in and navigates to the orders module
```

### When — Action / Event
✅ Describes **one single action** by the actor or a system event
✅ Intent-oriented, not implementation-oriented
✅ Uses business verbs (requests, cancels, approves, submits)
❌ No references to UI elements (buttons, colours, positions)
❌ Does not chain multiple actions

```gherkin
# ✅ Correct
When the customer cancels order "ORD-001"
When the system detects that stock has reached zero

# ❌ Incorrect
When I click the red "Cancel" button in the confirmation popup
When I fill in the form and click save and wait for the confirmation
```

### Then — Result / Consequence
✅ Describes **observable results** from the business perspective
✅ Can verify multiple results with `And`
✅ Includes error/success messages, state changes, notifications
❌ No implementation details (SQL, API calls, DB values)
❌ No UI details unless they are the business result (e.g., "I see message X")

```gherkin
# ✅ Correct
Then order "ORD-001" appears as "cancelled" in the history
And the customer receives a cancellation confirmation email

# ❌ Incorrect
Then the "orders" table has a record with status=CANCELLED and updated_at != null
```

---

## 3. Scenario Outline and Examples {#outline}

Use `Scenario Outline` when the same flow repeats with different data:

```gherkin
Scenario Outline: Registration with invalid data shows specific error
  Given I am on the registration form
  When I enter "<email>" as email and "<password>" as password
  And I attempt to register
  Then I see the error message "<message>"

  Examples:
    | email             | password | message                                      |
    | not-an-email      | Abc123!  | Invalid email format                         |
    | user@test.com     | abc      | Password must be at least 8 characters long  |
    | user@test.com     |          | Password is required                         |
    |                   | Abc123!  | Email is required                            |
```

**When NOT to use Scenario Outline:**
- When there is only 1 example → use a regular Scenario
- When the examples have very different logic → create separate Scenarios

---

## 4. Recommended tags {#tags}

| Tag | Purpose | When to add |
|---|---|---|
| `@smoke` | Smoke tests for fast CI | Critical business flows (1–3 per Feature) |
| `@happy-path` | Standard successful flow | Main success scenario |
| `@edge-case` | Boundary and extreme values | Min/max values, empty strings, null |
| `@error-handling` | Error management | Validations, error messages |
| `@regression` | Non-regression | Previous bugs that must not recur |
| `@wip` | Work in progress | Scenarios not yet implemented |
| `@skip` | Exclude from execution | Temporarily pending or broken |

Feature-level tags (apply to all scenarios in the file):
```gherkin
@user-module @sprint-12
Feature: User Management
```

---

## 5. Common anti-patterns {#anti-patterns}

### ❌ Narrative Given-When-Then (not executable)
```gherkin
# BAD: too vague to implement step definitions
Scenario: The system works correctly
  Given everything is configured
  When the user does things
  Then the result is as expected
```

### ❌ God scenario (does too much)
```gherkin
# BAD: one scenario should not verify 10 different behaviours
Scenario: Complete purchase process
  Given ...
  When I add a product to the cart
  And I add another product
  And I go to checkout
  And I enter a shipping address
  And I select a payment method
  And I confirm the order
  Then I see confirmation
  And I receive an email
  And stock is reduced
  And the order appears in my account
  And the admin receives a notification
```

### ❌ Ambiguous shared step definitions
```gherkin
# BAD: "the user is authenticated" is ambiguous — which user? what role?
Given the user is authenticated
# GOOD: specific
Given the user "carlos@company.com" with role "manager" is authenticated
```

### ❌ Relying on execution order
```gherkin
# BAD: Scenario 2 assumes Scenario 1 left data in the system
Scenario: Create category
  ...

Scenario: Assign product to the created category
  Given the category from the previous scenario exists  ← NEVER do this
```

---

## 6. Quality checklist {#checklist}

Before considering the Gherkin complete, verify:

**Coverage**
- [ ] Each functional requirement has at least one scenario
- [ ] Happy path scenarios exist for the main flows
- [ ] Error scenarios exist for the important validations

**Structure**
- [ ] One single Feature per file
- [ ] Background used only if ≥3 scenarios need it
- [ ] Scenario Outline for data variants (with ≥2 examples)
- [ ] All scenarios have descriptive names

**Steps**
- [ ] Given = context (not actions)
- [ ] When = one single action, no UI references
- [ ] Then = observable result, no implementation details
- [ ] Data values in double quotes when appearing inline

**Tags**
- [ ] @smoke on the most critical scenarios
- [ ] @happy-path / @edge-case / @error-handling as appropriate

**Independence**
- [ ] Each scenario can run on its own
- [ ] No scenario depends on state left by another

**Language and consistency**
- [ ] The entire file uses the same language
- [ ] Business terminology is consistent (same words for the same concepts)
