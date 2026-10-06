# GitHub Actions Expressions — Lab Exercises

Below is a progressive hands-on lab set you can give to students, starting from basic expressions and ending with a production-style CI/CD pipeline.

## Lab 1 — Basic Expression

**Objective:** Understand `${{ }}` expressions and GitHub context.

Create:

```text
.github/workflows/expressions.yml
```

Use:

```yaml
name: Expression Basics

on:
  workflow_dispatch:

jobs:
  expressions:
    runs-on: ubuntu-latest

    steps:
      - name: Display GitHub Information
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Actor: ${{ github.actor }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Event: ${{ github.event_name }}"
          echo "Run Number: ${{ github.run_number }}"
          echo "Commit SHA: ${{ github.sha }}"
```

### Tasks

1. Run the workflow manually.
2. Identify the repository name.
3. Identify the user who triggered the workflow.
4. Identify the branch.
5. Identify the workflow run number.

---

# Lab 2 — Branch-Based Condition

**Objective:** Use `if` and comparison operators.

Create a workflow that prints different messages depending on the branch.

### Requirements

```text
main     → "Production branch"
develop  → "Development branch"
other    → "Feature branch"
```

### Challenge

Use expressions such as:

```yaml
if: ${{ github.ref_name == 'main' }}
```

and:

```yaml
if: ${{ github.ref_name == 'develop' }}
```

### Expected behavior

```text
main
  ↓
Production branch

develop
  ↓
Development branch

feature/*
  ↓
Feature branch
```

---

# Lab 3 — AND / OR Conditions

**Objective:** Combine multiple expressions.

Create a workflow that runs a deployment step only when:

```text
Event = push
AND
Branch = main
```

Use:

```yaml
if: ${{ github.event_name == 'push' && github.ref_name == 'main' }}
```

### Additional Challenge

Modify the condition so deployment is allowed on:

```text
main OR release/*
```

but only for a push event.

Hint:

```yaml
startsWith()
```

---

# Lab 4 — Workflow Inputs

**Objective:** Use `inputs` expressions.

Create a manually triggered workflow with:

```text
environment:
  dev
  staging
  production
```

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Select environment"
        required: true
        type: choice
        options:
          - dev
          - staging
          - production
```

### Tasks

Create three deployment steps:

```text
Deploy DEV
Deploy STAGING
Deploy PRODUCTION
```

Only the selected environment should execute.

### Expected logic

```yaml
if: ${{ inputs.environment == 'dev' }}
```

---

# Lab 5 — `contains()`

**Objective:** Use string functions.

Create a workflow that checks the commit message.

If the commit message contains:

```text
[deploy]
```

execute the deployment step.

Example:

```text
Fix login issue [deploy]
```

should trigger deployment.

Whereas:

```text
Fix login issue
```

should not.

### Hint

Use:

```yaml
contains()
```

---

# Lab 6 — `startsWith()` and `endsWith()`

**Objective:** Work with branch naming conventions.

Create conditions for the following branches:

```text
release/v1.0
release/v2.0
feature/login
feature/payment
hotfix/database
```

### Requirements

Execute:

```text
Release Build
```

for branches beginning with:

```text
release/
```

Execute:

```text
Feature Build
```

for branches beginning with:

```text
feature/
```

Execute:

```text
Hotfix Build
```

for branches beginning with:

```text
hotfix/
```

### Challenge

Create another condition that detects branches ending with:

```text
-production
```

---

# Lab 7 — Step Outputs

**Objective:** Pass information from one step to another.

Create a workflow with:

```text
Step 1 → Generate application version
Step 2 → Display version
Step 3 → Use version during deployment
```

Generate:

```text
1.0.<run-number>
```

For example:

```text
1.0.25
```

### Requirements

Use:

```yaml
$GITHUB_OUTPUT
```

and access the value using:

```yaml
steps.<step-id>.outputs.<output-name>
```

### Expected flow

```text
Generate Version
       |
       | 1.0.25
       ↓
Build Application
       |
       | 1.0.25
       ↓
Deploy Application
```

---

# Lab 8 — Job Outputs

**Objective:** Pass data between jobs.

Create two jobs:

```text
build
  ↓
deploy
```

The `build` job should generate:

```text
APP_VERSION=1.0.25
```

The `deploy` job should display:

```text
Deploying application version 1.0.25
```

### Requirements

Use:

```yaml
outputs:
```

and:

```yaml
needs.build.outputs.*
```

---

# Lab 9 — `success()` and `failure()`

**Objective:** Handle successful and failed jobs.

Create three jobs:

```text
Build
  ↓
Test
  ↓
Notification
```

Make the Test job intentionally fail:

```yaml
run: exit 1
```

Create a notification step that executes only when the workflow fails.

Use:

```yaml
if: ${{ failure() }}
```

### Expected result

```text
Build       → SUCCESS
Test        → FAILED
Notification → EXECUTED
```

---

# Lab 10 — `always()`

**Objective:** Perform cleanup even when previous steps fail.

Create:

```text
Build
Test
Cleanup
```

Make Test fail.

Cleanup must execute regardless of Test's result.

Use:

```yaml
if: ${{ always() }}
```

### Expected result

```text
Build
  ↓
Test → FAILED
  ↓
Cleanup → EXECUTED
```

---

# Lab 11 — Matrix Expressions

**Objective:** Use expressions with matrix strategies.

Create a matrix:

```yaml
strategy:
  matrix:
    environment:
      - dev
      - staging
      - production
```

Print:

```text
Deploying to dev
Deploying to staging
Deploying to production
```

Then create a step that executes **only for production**.

Hint:

```yaml
if: ${{ matrix.environment == 'production' }}
```

---

# Lab 12 — `toJSON()`

**Objective:** Understand contexts and JSON conversion.

Create a debugging workflow.

Display:

```yaml
github
```

using:

```yaml
toJSON()
```

Then display:

```yaml
github.event
```

### Questions

Ask students to identify:

1. Repository information
2. Actor
3. Event
4. Branch
5. Commit information

---

# Lab 13 — `hashFiles()`

**Objective:** Understand expression-based cache keys.

Create a Node.js project containing:

```text
package.json
package-lock.json
```

Create a cache key using:

```yaml
${{ hashFiles('**/package-lock.json') }}
```

Expected format:

```text
Linux-node-<hash>
```

### Challenge

Modify `package-lock.json` and run the workflow again.

Observe how the cache key changes.

---

# Lab 14 — Production Deployment Challenge ⭐

This is the **main lab** I recommend for your students.

### Scenario

Your organization has:

```text
develop → DEV
main    → PRODUCTION
```

You need to build a CI/CD pipeline.

### Pipeline

```text
                 ┌─────────────┐
                 │   Git Push  │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │    Build    │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │    Test     │
                 └──────┬──────┘
                        ↓
               ┌────────┴────────┐
               ↓                 ↓
          develop               main
               ↓                 ↓
          Deploy DEV       Deploy PROD
```

### Requirements

The workflow must:

1. Trigger on `push`.
2. Build the application.
3. Generate a version using the workflow run number.
4. Pass the version from the build job to the deployment job.
5. Run tests.
6. Deploy to DEV when branch is `develop`.
7. Deploy to PROD when branch is `main`.
8. Never deploy if tests fail.
9. Run a notification step when anything fails.
10. Run cleanup regardless of success or failure.

### Expressions students should use

```yaml
${{ github.ref_name }}
```

```yaml
${{ github.event_name }}
```

```yaml
${{ needs.build.outputs.version }}
```

```yaml
${{ needs.test.result }}
```

```yaml
${{ failure() }}
```

```yaml
${{ always() }}
```

---

# Lab 15 — Advanced Challenge: Dynamic Deployment

Create a manual workflow:

```text
Application:
  backend
  frontend
  database

Environment:
  dev
  staging
  production
```

The student must construct the deployment dynamically.

For example:

```text
Application = backend
Environment = production
```

should produce:

```text
Deploying backend to production
```

Use:

```yaml
inputs.*
```

and:

```yaml
format()
```

Example concept:

```yaml
${{ format('Deploying {0} to {1}', inputs.application, inputs.environment) }}
```

---

# Final Student Challenge 🚀

Ask students to build this complete pipeline without giving them the solution:

```text
                    GitHub Push
                         │
                         ▼
                  ┌─────────────┐
                  │    Build    │
                  └──────┬──────┘
                         │
                  Generate Version
                         │
                         ▼
                  ┌─────────────┐
                  │    Test     │
                  └──────┬──────┘
                         │
                ┌────────┴────────┐
                │                 │
             develop             main
                │                 │
                ▼                 ▼
          ┌───────────┐     ┌────────────┐
          │ Deploy DEV│     │ Deploy PROD│
          └───────────┘     └────────────┘
                                  │
                                  ▼
                         Production Approval
                                  │
                                  ▼
                            Notification
```

### Mandatory expressions

Students must use at least:

- `github.*`
- `inputs.*`
- `env.*`
- `vars.*`
- `if`
- `&&`
- `||`
- `contains()`
- `startsWith()`
- `format()`
- `steps.*.outputs.*`
- `needs.*.outputs.*`
- `matrix.*`
- `success()`
- `failure()`
- `always()`
- `toJSON()`

### Expected learning outcome

By completing these labs, students should be able to take a requirement such as:

> **“Deploy the application to production only when the code is pushed to main, tests pass, the generated version is available, and the production deployment condition is satisfied.”**

and translate it into GitHub Actions expressions and workflow logic.
