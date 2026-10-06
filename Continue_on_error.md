# GitHub Actions Lab — `continue-on-error`

## Objective

Understand how GitHub Actions handles a failed step or job when `continue-on-error` is enabled.

Students will learn to:

- Use `continue-on-error: true`
- Understand the difference between **step-level** and **job-level** `continue-on-error`
- Continue workflow execution after failures
- Use `steps.<id>.outcome`
- Use `steps.<id>.conclusion`
- Build conditional logic based on a failed step

---

## Lab 1 — Basic `continue-on-error`

### Scenario

You have three steps:

```text
Step 1 → Build
Step 2 → Test
Step 3 → Deploy
```

The Test step is expected to fail, but you don't want the workflow to stop.

### Task

Create:

```text
.github/workflows/continue-error.yml
```

with:

```yaml
name: Continue on Error

on:
  workflow_dispatch:

jobs:
  demo:
    runs-on: ubuntu-latest

    steps:

      - name: Build
        run: |
          echo "Building application"
          echo "Build successful"

      - name: Test
        continue-on-error: true
        run: |
          echo "Running tests..."
          exit 1

      - name: Deploy
        run: |
          echo "Deploying application"
```

### Expected Result

```text
Build  → SUCCESS
Test   → FAILED
Deploy → SUCCESS
```

The overall job should continue despite the failed Test step.

---

# Lab 2 — Without `continue-on-error`

Remove:

```yaml
continue-on-error: true
```

Run the workflow again.

### Observe

```text
Build  → SUCCESS
Test   → FAILED
Deploy → SKIPPED
```

### Question

Why did the Deploy step not execute?

---

# Lab 3 — Step-Level vs Job-Level

## Part A — Step-Level

Create:

```yaml
steps:

  - name: Test
    continue-on-error: true
    run: exit 1

  - name: Deploy
    run: echo "Deploying"
```

Observe that the **job continues**.

---

## Part B — Job-Level

Now try:

```yaml
jobs:

  test:
    runs-on: ubuntu-latest
    continue-on-error: true

    steps:
      - name: Test
        run: |
          echo "Running tests"
          exit 1

  deploy:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying application"
```

### Question

Does the `deploy` job execute?

Investigate the behavior and explain the difference between:

```yaml
continue-on-error: true
```

at the **step level** and at the **job level**.

---

# Lab 4 — Check the Step Outcome

This is an important exercise.

Create:

```yaml
steps:

  - name: Run Tests
    id: tests
    continue-on-error: true
    run: |
      echo "Running tests..."
      exit 1

  - name: Check Test Result
    run: |
      echo "Outcome: ${{ steps.tests.outcome }}"
      echo "Conclusion: ${{ steps.tests.conclusion }}"
```

### Expected Observation

Students should investigate the values returned by:

```text
steps.tests.outcome
steps.tests.conclusion
```

### Questions

1. What is the value of `outcome`?
2. What is the value of `conclusion`?
3. Why can these values differ when `continue-on-error` is used?

---

# Lab 5 — Execute a Warning Step When Tests Fail

Modify the workflow so that a warning message is displayed only when the Test step failed.

Use:

```yaml
if: ${{ steps.tests.outcome == 'failure' }}
```

Example:

```yaml
- name: Test
  id: tests
  continue-on-error: true
  run: |
    echo "Running tests..."
    exit 1

- name: Test Warning
  if: ${{ steps.tests.outcome == 'failure' }}
  run: |
    echo "WARNING: Tests failed!"
    echo "Pipeline is continuing."
```

### Expected Flow

```text
Test
 ↓
FAILED
 ↓
continue-on-error
 ↓
Check outcome
 ↓
WARNING
 ↓
Continue pipeline
```

---

# Lab 6 — Conditional Deployment

### Scenario

You want to continue the pipeline even if tests fail, but you **must not deploy** when tests fail.

Create:

```text
Build
  ↓
Test
  ↓
Check Test Result
  ↓
Deploy only if tests passed
```

### Requirement

Test:

```yaml
continue-on-error: true
```

Deployment:

```yaml
if: ${{ steps.tests.outcome == 'success' }}
```

### Expected behavior

If tests pass:

```text
Build
 ↓
Test SUCCESS
 ↓
Deploy
```

If tests fail:

```text
Build
 ↓
Test FAILURE
 ↓
Pipeline continues
 ↓
Deploy SKIPPED
```

---

# Lab 7 — Optional Quality Check

### Scenario

Your pipeline contains:

- Unit tests
- SonarQube scan
- Security scan
- Build
- Deployment

You don't want a SonarQube warning to stop the deployment.

Create:

```yaml
- name: SonarQube Scan
  id: sonar
  continue-on-error: true
  run: |
    echo "Running SonarQube scan..."
    exit 1
```

Then:

```yaml
- name: Build
  run: echo "Building application"

- name: Deploy
  run: echo "Deploying application"
```

### Expected Result

```text
SonarQube → FAILURE
             ↓
       continue-on-error
             ↓
Build       → SUCCESS
             ↓
Deploy      → SUCCESS
```

### Discussion

Ask students:

> Should a production pipeline really allow a failed security scan to continue?

This leads into an important DevOps discussion about **which failures are allowed and which failures must block deployment**.

---

# Lab 8 — Production Pipeline Challenge ⭐

Build the following pipeline:

```text
                  Build
                    │
                    ▼
               Unit Tests
                    │
          ┌─────────┴─────────┐
          │                   │
       SUCCESS              FAILURE
          │                   │
          ▼                   ▼
      Security Scan      Warning Message
          │                   │
          └─────────┬─────────┘
                    ▼
                  Deploy
```

### Requirements

#### Build

Must stop the pipeline if it fails.

#### Unit Test

Use:

```yaml
continue-on-error: true
```

#### Security Scan

Use:

```yaml
continue-on-error: true
```

#### Deployment

Deploy only when:

```text
Unit tests succeeded
AND
Security scan succeeded
```

### Hint

You can use:

```yaml
if: ${{ steps.unit_tests.outcome == 'success' && steps.security.outcome == 'success' }}
```

---

# Lab 9 — `continue-on-error` + Matrix

Create a matrix:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest
```

Run a test on all three operating systems.

Make the Windows test fail intentionally.

Use:

```yaml
continue-on-error: true
```

### Expected

```text
Ubuntu  → SUCCESS
Windows → FAILURE → Continue
macOS   → SUCCESS
```

### Challenge

Display a message only for the failed matrix job.

---

# Lab 10 — Final Real-World Exercise 🚀

### Scenario

You are building a CI/CD pipeline for a microservice.

The pipeline contains:

```text
1. Checkout
2. Build
3. Unit Test
4. Code Quality
5. Security Scan
6. Docker Build
7. Deploy
```

### Business Requirements

| Step | Failure behavior |
|---|---|
| Checkout | Stop |
| Build | Stop |
| Unit Test | Continue |
| Code Quality | Continue |
| Security Scan | Continue |
| Docker Build | Stop |
| Deploy | Conditional |

### Deployment rule

Deploy only if:

```text
Build = SUCCESS
AND
Docker Build = SUCCESS
AND
Unit Test = SUCCESS
```

Code Quality and Security Scan failures should **not automatically stop the pipeline**.

### Students must use

```yaml
continue-on-error
```

```yaml
steps.<id>.outcome
```

```yaml
if
```

```yaml
&&
```

```yaml
|| 
```

### Expected architecture

```text
                 ┌──────────────┐
                 │   Checkout   │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │    Build     │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │ Unit Testing │
                 └──────┬───────┘
                        │
                  continue-error
                        │
                        ↓
                 ┌──────────────┐
                 │ Code Quality │
                 └──────┬───────┘
                        │
                  continue-error
                        │
                        ↓
                 ┌──────────────┐
                 │Security Scan │
                 └──────┬───────┘
                        │
                  continue-error
                        │
                        ↓
                 ┌──────────────┐
                 │ Docker Build │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │   Deploy     │
                 └──────────────┘
```

### Final Question for Students

**Why would you use `continue-on-error` for Code Quality or Security Scan, but still prevent production deployment when critical tests fail?**

This question helps students understand that `continue-on-error` is not simply **“ignore errors”**—it is a mechanism for allowing the workflow to continue while still retaining the failed step's status for subsequent decision-making.
