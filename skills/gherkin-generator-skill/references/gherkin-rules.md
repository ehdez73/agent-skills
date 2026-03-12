# Gherkin Structure & Writing Rules

Applies to both agents. Generator uses these to create valid Gherkin; Reviewer uses these to validate.

---

## Mandatory Structure

### Feature Header
```gherkin
Feature: <Feature Name>
  As a <actor/role>
  I want <action/capability>
  So that <business benefit>
```

### Scenarios
- **One Feature per file**
- Scenarios are independent (no order dependencies)
- Scenario names describe **expected behaviour**, not the action sequence
  - ✅ `Scenario: Login fails with invalid credentials`
  - ❌ `Scenario: User enters wrong password and clicks login`

### Background (optional but recommended)
- Use **only if 3+ scenarios share identical Given steps**
- Consolidates setup code: system state, authentication, data fixtures
- Example:
  ```gherkin
  Background:
    Given the system is operational
    And the admin user is authenticated
    And 5 test products exist in inventory
  ```

### Scenario Outline + Examples
- Use when a single scenario has **multiple data variants**
- Each row of the Examples table tests the same flow with different values
- Example:
  ```gherkin
  Scenario Outline: Validation messages appear for empty fields
    Given I am on the registration form
    When I leave the "<field>" empty and submit
    Then I see the error "<error_message>"

    Examples:
      | field | error_message         |
      | name  | Name is required      |
      | email | Email is required     |
  ```

---

## Given / When / Then Semantics

### Given (Preconditions)
- Describes **system state** before the action
- Sets up context, data, or configuration
- ✅ `Given a user with email "ana@company.com" already exists`
- ✅ `Given the checkout total is $150`
- ❌ `Given I click the login button` (this is an action, not state)

### When (Actor's Action)
- **One action per When step** (avoid multi-action steps)
- Describes what the actor does in plain language, not UI details
- ✅ `When I submit the registration form`
- ✅ `When I delete the account`
- ❌ `When I click the blue "Delete" button in the bottom-right corner`
- ❌ `When I fill in the form, click save, and wait for confirmation`

### Then (Observable Results)
- Describes **verifiable outcomes** (results visible to the actor)
- Avoid implementation or technical details
- ✅ `Then the user "Ana Garcia" appears in the active users list`
- ✅ `Then a welcome email is sent to "ana@company.com"`
- ❌ `Then the database user record has status='ACTIVE'` (implementation detail)
- ❌ `Then the REST API response code is 201` (technical artifact, not user-visible)

### And / But (Continuations)
- Used to chain multiple steps of the **same semantic type**
- ✅ `Given user X exists` / `And user Y exists` (both Given)
- ✅ `Then I see message A` / `And I see message B` (both Then)
- ❌ `Given I am logged in` / `And I click the save button` (mixing Given and When)

---

## Data and Examples

### Realistic Values
- Use real-world, representative data (not "test123" or "foo")
- ✅ `"ana@company.com"`, `"John Garcia"`, `"2025-03-12"`
- ❌ `"test"`, `"abc"`, `"user1"`

### Inline Values
- Wrap in double quotes when appearing inside step text
```gherkin
When I enter the name "Ana Garcia" and email "ana@company.com"
Then the user "Ana Garcia" appears in the list
```

### Table Data (Scenario Outline)
- Column names must be clear and match the step placeholders
```gherkin
Scenario Outline: Payment validation
  When I process a payment of <amount> currency <currency>
  Then the transaction status is "<status>"

  Examples:
    | amount | currency | status      |
    | -50    | USD      | rejected    |
    | 0      | EUR      | rejected    |
    | 10.50  | GBP      | approved    |
```

---

## Tags (for test execution and filtering)

Use consistently to categorize scenarios:

| Tag | Usage |
|---|---|
| `@smoke` | Critical path scenarios; quick validation of core functionality |
| `@happy-path` | Standard successful flow (without edge cases) |
| `@edge-case` | Boundary values, extreme cases, unusual but valid inputs |
| `@error-handling` | Error flows, validation failures, exception handling |
| `@regression` | Non-regression check; ensures old bugs don't resurface |

**Example:**
```gherkin
@smoke @happy-path
Scenario: User logs in with valid credentials
  ...

@edge-case
Scenario: Login with username containing special characters
  ...

@error-handling
Scenario: Login fails with invalid email format
  ...
```

---

## Language Consistency

- **Detect the language** of the original requirements
- **Use it throughout** the entire `.feature` file
- Do not mix languages within a single feature
- Translate feature names consistently:
  - EN: `Feature: User Management`
  - ES: `Característica: Gestión de Usuarios`
  - FR: `Fonctionnalité: Gestion des Utilisateurs`

---

## Approval Criteria (Reviewer Checklist)

| Dimension | Minimum standard |
|---|---|
| **Requirements coverage** | 100% of functional requirements have at least one happy-path scenario; error/validation flows covered |
| **Gherkin syntax** | Valid Feature, Scenario/Scenario Outline, Given/When/Then structure |
| **Scenario independence** | Each scenario executes in isolation; no dependencies between scenarios |
| **Step clarity** | No UI-specific language; no implementation details; clear semantic role (Given/When/Then) |
| **Example data** | Realistic values; Scenario Outline used for variants |
| **Language** | Consistent throughout; matches original requirements |

---

## Common Anti-patterns (Things to Avoid)

### ❌ UI-Specific Steps
```gherkin
# BAD — couples test to UI implementation
When I click the blue "Save" button located at the bottom right
When I scroll down and find the checkbox labeled "Accept Terms"

# GOOD — describes the action, not the UI
When I save the form
When I accept the terms and conditions
```

### ❌ Scenarios with Dependencies
```gherkin
# BAD — Scenario 2 assumes state from Scenario 1
Scenario: Create a user account
  ...

Scenario: Edit the user we just created
  ...

# GOOD — each is self-contained
Scenario: Create a user account
  Given a clean system
  ...

Scenario: Edit an existing user account
  Given a user "Ana Garcia" already exists
  ...
```

### ❌ Multiple Actions in One When
```gherkin
# BAD
When I fill the form, click save, and wait for confirmation

# GOOD
When I fill the form with the product details
And I click "Save"
Then I see the confirmation message
```

### ❌ Mixing Step Types (Given + When in one And)
```gherkin
# BAD
Given the user is logged in
And I click the logout button

# GOOD
Given the user is logged in
When I click the logout button
```

### ❌ Implementation Details in Then
```gherkin
# BAD — reveals database/API structure
Then the database record has status='ACTIVE'
Then the REST API returns status code 201

# GOOD — observable user-facing outcome
Then the user appears in the active list
Then the account is successfully created
```

### ❌ Non-Descriptive Scenario Names
```gherkin
# BAD
Scenario: Test login
Scenario: Feature X

# GOOD
Scenario: User successfully logs in with valid credentials
Scenario: Login fails with an invalid email format
```

---

## Generator Instructions

When you receive requirements:
- **Read all rules above** before generating scenarios
- **Structure the Feature** with proper header and user story context
- **Create 1–2 happy-path scenarios** for the main flow
- **Create error/validation scenarios** for each error condition mentioned
- **Use Scenario Outline + Examples** when there are data variants
- **Tag appropriately** (@smoke, @happy-path, @edge-case, @error-handling)
- **Use Background** only if 3+ scenarios share Given steps
- **Return only the Gherkin block**; no explanations before or after

---

## Reviewer Instructions

When you receive Gherkin:
- **Map each requirement** to scenarios (note gaps)
- **Validate syntax** (Feature, Scenario, Given/When/Then structure)
- **Check independence** (no scenario depends on another)
- **Scan steps** for UI coupling, implementation details, ambiguous language
- **Verify data** is realistic and consistent
- **Review tags** (appropriate categorization)
- **Score** each evaluation dimension
- **Return JSON** with issues (sorted by severity) and suggestions
- **Annotate original Gherkin** with `# ⚠️ ISSUE:` comments before problematic lines
