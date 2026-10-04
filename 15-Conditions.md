# Conditions in GitHub Actions

Conditions allow you to control **whether a job or step should execute**.

The main mechanism is the `if:` expression.

For example:

```yaml
if: ${{ github.ref_name == 'main' }}
```

means:

> Run this job/step only when the current branch is `main`.

Conditions become extremely powerful when combined with **contexts, expressions, `needs`, matrix, inputs, and workflow events**.

---

# 1. Basic `if` condition

```yaml
name: Conditions Demo

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Always execute
        run: echo "This always runs"

      - name: Production step
        if: ${{ github.ref_name == 'main' }}
        run: echo "Running production step"
```

The second step executes only when:

```text
github.ref_name == main
```

---

# 2. Condition on a job

You can put `if` at the job level.

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Building application"


  deploy:
    if: ${{ github.ref_name == 'main' }}
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying to production"
```

Execution:

```text
                  Build
                    │
                    ▼
            Is branch main?
              /          \
            YES           NO
             │             │
             ▼             ▼
          Deploy         Skip
```

---

# 3. `if` with branch

One of the most common use cases:

```yaml
if: ${{ github.ref_name == 'main' }}
```

Example:

```yaml
- name: Deploy production
  if: ${{ github.ref_name == 'main' }}
  run: |
    echo "Deploying production"
```

For `develop`:

```yaml
- name: Deploy development
  if: ${{ github.ref_name == 'develop' }}
  run: |
    echo "Deploying development"
```

---

# 4. Multiple conditions using `&&`

You can combine conditions using logical AND:

```yaml
if: ${{ github.ref_name == 'main' && github.event_name == 'push' }}
```

Meaning:

```text
Branch = main
       AND
Event = push
       │
       ▼
     Deploy
```

Example:

```yaml
- name: Production deployment
  if: >
    github.ref_name == 'main' &&
    github.event_name == 'push'
  run: |
    echo "Production deployment"
```

Both conditions must be true.

---

# 5. OR condition using `||`

Use `||` when either condition can be true.

```yaml
if: ${{ github.ref_name == 'main' || github.ref_name == 'develop' }}
```

This runs for:

```text
main     → YES
develop  → YES
feature  → NO
```

---

# 6. NOT condition

Use `!` to negate a condition:

```yaml
if: ${{ github.ref_name != 'main' }}
```

This means:

> Run when the branch is anything other than `main`.

Example:

```yaml
- name: Development step
  if: ${{ github.ref_name != 'main' }}
  run: echo "Not production"
```

---

# 7. Conditions based on event

You can use:

```yaml
github.event_name
```

Example:

```yaml
- name: Run on push
  if: ${{ github.event_name == 'push' }}
  run: echo "Push detected"
```

Pull request:

```yaml
- name: Run PR validation
  if: ${{ github.event_name == 'pull_request' }}
  run: echo "Pull request detected"
```

Manual execution:

```yaml
- name: Manual execution
  if: ${{ github.event_name == 'workflow_dispatch' }}
  run: echo "Manually triggered"
```

---

# 8. Combining branch and event

A very common production condition:

```yaml
if: >
  github.ref_name == 'main' &&
  github.event_name == 'push'
```

Meaning:

```text
             Event
               │
        ┌──────┴──────┐
        │             │
      push       pull_request
        │
        ▼
     main?
     /    \
   YES     NO
    │       │
    ▼       ▼
 Deploy    Skip
```

---

# 9. Conditions using environment

Suppose:

```yaml
env:
  ENVIRONMENT: production
```

You can use:

```yaml
if: ${{ env.ENVIRONMENT == 'production' }}
```

Example:

```yaml
- name: Production deployment
  if: ${{ env.ENVIRONMENT == 'production' }}
  run: echo "Deploying production"
```

---

# 10. Conditions using `vars`

Suppose your GitHub repository has a variable:

```text
DEPLOY_ENABLED = true
```

You can use:

```yaml
if: ${{ vars.DEPLOY_ENABLED == 'true' }}
```

Example:

```yaml
- name: Deploy
  if: ${{ vars.DEPLOY_ENABLED == 'true' }}
  run: |
    echo "Deployment enabled"
```

---

# 11. Conditions using workflow inputs

Suppose you have:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Environment"
        required: true
        type: choice
        options:
          - dev
          - qa
          - production
```

You can conditionally deploy:

```yaml
jobs:

  deploy:
    if: ${{ inputs.environment == 'production' }}
    runs-on: ubuntu-latest

    steps:
      - run: echo "Production deployment"
```

Or:

```yaml
jobs:

  deploy-dev:
    if: ${{ inputs.environment == 'dev' }}
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying DEV"


  deploy-qa:
    if: ${{ inputs.environment == 'qa' }}
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying QA"


  deploy-prod:
    if: ${{ inputs.environment == 'production' }}
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying PRODUCTION"
```

This creates environment-specific jobs.

---

# 12. Conditions using matrix

Since we discussed Matrix Strategy, you can use conditions with matrix values.

```yaml
jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

    runs-on: ${{ matrix.os }}

    steps:

      - name: Linux test
        if: ${{ matrix.os == 'ubuntu-latest' }}
        run: |
          echo "Running Linux-specific test"

      - name: Windows test
        if: ${{ matrix.os == 'windows-latest' }}
        run: |
          echo "Running Windows-specific test"

      - name: macOS test
        if: ${{ matrix.os == 'macos-latest' }}
        run: |
          echo "Running macOS-specific test"
```

Conceptually:

```text
             Matrix
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
   Ubuntu    Windows     macOS
      │         │         │
    Linux     Windows    macOS
    step       step       step
```

---

# 13. `success()`

GitHub provides built-in status functions.

One of the most important is:

```yaml
if: ${{ success() }}
```

It means:

> Run only if previous steps/jobs succeeded.

Example:

```yaml
- name: Deploy
  if: ${{ success() }}
  run: echo "Deploying..."
```

Normally, steps already behave this way unless you override their conditions, but explicitly using status functions is useful for more complex workflows.

---

# 14. `failure()`

Run something only when a previous step fails.

```yaml
- name: Run tests
  run: |
    ./run-tests.sh

- name: Send failure notification
  if: ${{ failure() }}
  run: |
    echo "Tests failed!"
```

Execution:

```text
Run Tests
    │
    ├── SUCCESS → Notification skipped
    │
    └── FAILURE → Notification runs
```

This is very useful for:

- Slack notifications
- Email notifications
- Incident creation
- Log collection

---

# 15. `always()`

`always()` runs regardless of whether previous steps succeeded or failed.

```yaml
- name: Collect logs
  if: ${{ always() }}
  run: |
    echo "Collecting logs..."
```

Very useful for cleanup.

Example:

```yaml
steps:

  - name: Run tests
    run: ./tests.sh

  - name: Collect test logs
    if: ${{ always() }}
    run: |
      tar -czf test-logs.tar.gz logs/

  - name: Upload logs
    if: ${{ always() }}
    uses: actions/upload-artifact@v4
    with:
      name: test-logs
      path: test-logs.tar.gz
```

Even if the tests fail, logs can still be collected.

---

# 16. `cancelled()`

You can execute something if the workflow was cancelled:

```yaml
- name: Cleanup
  if: ${{ cancelled() }}
  run: |
    echo "Workflow was cancelled"
    ./cleanup.sh
```

Useful for cleaning temporary resources.

---

# 17. Status functions summary

| Function | Meaning |
|---|---|
| `success()` | Previous execution succeeded |
| `failure()` | Previous execution failed |
| `always()` | Run regardless of status |
| `cancelled()` | Workflow/job was cancelled |

---

# 18. Conditions with `needs`

This is extremely important in multi-job workflows.

Suppose:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: ./build.sh


  test:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - run: ./test.sh


  deploy:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - run: ./deploy.sh
```

Normally:

```text
Build
  │
  ▼
Test
  │
  ▼
Deploy
```

If Build fails:

```text
Build ❌
  │
  ▼
Test skipped
  │
  ▼
Deploy skipped
```

---

# 19. Deploy only if previous job succeeded

You can explicitly check:

```yaml
deploy:
  needs: test
  if: ${{ needs.test.result == 'success' }}
```

Example:

```yaml
deploy:
  needs: test
  if: ${{ needs.test.result == 'success' }}
  runs-on: ubuntu-latest

  steps:
    - run: echo "Deploying..."
```

`needs.test.result` can provide the result of the dependent job.

Common values:

```text
success
failure
cancelled
skipped
```

---

# 20. Deploy if tests fail — special case

Normally a failed dependency prevents the next job from running.

If you intentionally want to run something after failure:

```yaml
deploy:
  needs: test
  if: ${{ always() && needs.test.result == 'failure' }}
  runs-on: ubuntu-latest

  steps:
    - run: echo "Handling test failure"
```

This is more useful for failure handling than actual deployment.

---

# 21. Conditions with multiple jobs

Consider:

```yaml
jobs:

  unit-test:
    runs-on: ubuntu-latest

  integration-test:
    runs-on: ubuntu-latest

  deploy:
    needs:
      - unit-test
      - integration-test

    if: >
      needs.unit-test.result == 'success' &&
      needs.integration-test.result == 'success'

    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying"
```

The flow:

```text
          Unit Test
             │
             │
             ▼
           ┌─────┐
           │     │
           ▼     ▼
      Integration
         Test
           │
           ▼
        Deploy
```

Deployment happens only when both succeed.

---

# 22. Conditions with Matrix + `needs`

Example:

```yaml
jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

    runs-on: ${{ matrix.os }}

    steps:
      - run: echo "Testing ${{ matrix.os }}"


  deploy:
    needs: test
    if: ${{ needs.test.result == 'success' }}

    runs-on: ubuntu-latest

    steps:
      - run: echo "All OS tests passed"
```

Conceptually:

```text
              Test
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
    Ubuntu   Windows    macOS
       │        │        │
       └────────┼────────┘
                │
              SUCCESS
                │
                ▼
             Deploy
```

---

# 23. Using `contains()`

GitHub expressions provide useful functions.

For example:

```yaml
if: ${{ contains(github.ref_name, 'release') }}
```

If the branch is:

```text
release/v1.0
```

then the condition is true.

Another example:

```yaml
if: ${{ contains(github.event_name, 'pull') }}
```

---

# 24. Using `startsWith()`

```yaml
if: ${{ startsWith(github.ref_name, 'release/') }}
```

This can detect release branches:

```text
release/v1.0 → TRUE
release/v2.0 → TRUE
feature/test → FALSE
main         → FALSE
```

---

# 25. Using `endsWith()`

```yaml
if: ${{ endsWith(github.ref_name, '-production') }}
```

For example:

```text
app-production → TRUE
app-qa         → FALSE
```

---

# 26. Real-world CI/CD example

Here's a realistic pipeline:

```yaml
name: CI/CD

on:
  push:
    branches:
      - main
      - develop

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - name: Build
        run: |
          echo "Building application..."


  test:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Test
        run: |
          echo "Running tests..."


  deploy-dev:
    needs: test
    if: ${{ github.ref_name == 'develop' }}
    runs-on: ubuntu-latest

    steps:
      - name: Deploy DEV
        run: |
          echo "Deploying to DEV"


  deploy-prod:
    needs: test
    if: ${{ github.ref_name == 'main' }}
    runs-on: ubuntu-latest

    steps:
      - name: Deploy PROD
        run: |
          echo "Deploying to PRODUCTION"
```

Flow:

```text
                    Build
                      │
                      ▼
                    Test
                      │
              ┌───────┴───────┐
              │               │
          develop            main
              │               │
              ▼               ▼
          Deploy DEV      Deploy PROD
```

---

# 27. Real-world failure handling

A better pipeline could be:

```yaml
jobs:

  test:
    runs-on: ubuntu-latest

    steps:
      - name: Run tests
        run: ./tests.sh

      - name: Collect logs
        if: ${{ always() }}
        run: ./collect-logs.sh

  notify:
    needs: test
    if: ${{ failure() }}
    runs-on: ubuntu-latest

    steps:
      - name: Notify
        run: |
          echo "Pipeline failed!"
```

A common pattern is:

```text
Test
 │
 ├── Success → Continue
 │
 └── Failure
       │
       ├── Collect logs
       │
       └── Notify
```

---

# 28. Conditions + Matrix + OS-specific commands

This is a practical example based on your previous Matrix topic:

```yaml
name: Cross Platform Test

on:
  workflow_dispatch:

jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

    runs-on: ${{ matrix.os }}

    steps:

      - name: Linux command
        if: ${{ matrix.os == 'ubuntu-latest' }}
        run: |
          echo "Running Linux commands"
          uname -a

      - name: Windows command
        if: ${{ matrix.os == 'windows-latest' }}
        shell: pwsh
        run: |
          Write-Host "Running Windows commands"
          Get-ComputerInfo

      - name: macOS command
        if: ${{ matrix.os == 'macos-latest' }}
        run: |
          echo "Running macOS commands"
          sw_vers
```

This demonstrates three concepts together:

```text
Matrix
   │
   ▼
Different OS
   │
   ▼
if condition
   │
   ▼
OS-specific step
```

---

# 29. `if` without `${{ }}`

You'll sometimes see:

```yaml
if: github.ref_name == 'main'
```

instead of:

```yaml
if: ${{ github.ref_name == 'main' }}
```

For `if`, GitHub Actions automatically evaluates the expression, so both forms can work.

I recommend using:

```yaml
if: ${{ github.ref_name == 'main' }}
```

when you're learning because it makes the expression syntax explicit.

---

# 30. Conditions cheat sheet

### Branch

```yaml
if: ${{ github.ref_name == 'main' }}
```

### Event

```yaml
if: ${{ github.event_name == 'push' }}
```

### AND

```yaml
if: ${{ condition1 && condition2 }}
```

### OR

```yaml
if: ${{ condition1 || condition2 }}
```

### NOT

```yaml
if: ${{ condition != value }}
```

### Job success

```yaml
if: ${{ success() }}
```

### Job failure

```yaml
if: ${{ failure() }}
```

### Always execute

```yaml
if: ${{ always() }}
```

### Cancelled

```yaml
if: ${{ cancelled() }}
```

### Matrix

```yaml
if: ${{ matrix.os == 'ubuntu-latest' }}
```

### Previous job result

```yaml
if: ${{ needs.test.result == 'success' }}
```

### Contains

```yaml
if: ${{ contains(github.ref_name, 'release') }}
```

### Starts with

```yaml
if: ${{ startsWith(github.ref_name, 'feature/') }}
```

### Ends with

```yaml
if: ${{ endsWith(github.ref_name, '-prod') }}
```

---

## The big picture

You've now covered several GitHub Actions concepts that fit together:

```text
                    GitHub Actions
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
      Triggers          Contexts          Variables
        │                  │                  │
        ▼                  ▼                  ▼
      push              github              env
      PR                matrix              vars
      manual            needs               secrets
      schedule          steps
                           │
                           ▼
                      Conditions
                           │
              ┌────────────┼────────────┐
              │            │            │
            Branch       Status       Matrix
              │            │            │
              ▼            ▼            ▼
           Deploy       Notify       OS-specific
```

The most important practical pattern to master is:

**Trigger → Context → Condition → `needs` → Matrix → Deploy**

That combination is what you'll use to build sophisticated CI/CD workflows.
