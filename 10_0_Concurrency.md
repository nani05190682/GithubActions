# GitHub Actions Job Concurrency — Detailed Explanation

**Job concurrency** in GitHub Actions is used to control **how many jobs or workflow runs are allowed to execute at the same time when they belong to the same concurrency group**.

It is especially important in CI/CD when you have situations like:

- Multiple developers pushing code at the same time
- Multiple deployments to the same environment
- Production deployments that must not overlap
- Preventing outdated deployments
- Cancelling older CI runs when a newer commit arrives

---

# 1. Why do we need concurrency?

Imagine you have this workflow:

```text
Developer A pushes
        │
        ▼
    Build A
        │
        ▼
    Test A
        │
        ▼
 Deploy A ───────────────► Production


Developer B pushes
        │
        ▼
    Build B
        │
        ▼
    Test B
        │
        ▼
 Deploy B ───────────────► Production
```

If both deployments happen at the same time:

```text
              Production
                  ▲
             ┌────┴────┐
             │         │
          Deploy A  Deploy B
```

This can cause problems.

For example:

```text
Deploy A → version 1.0
Deploy B → version 1.1
```

If they overlap, you could potentially end up with an inconsistent deployment.

Concurrency lets us say:

> **Only one deployment to production can run at a time.**

---

# 2. Basic syntax

GitHub Actions provides:

```yaml
concurrency:
  group: <group-name>
  cancel-in-progress: <true|false>
```

Example:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

There are two important properties:

### `group`

Defines **which executions belong to the same concurrency group**.

### `cancel-in-progress`

Defines what happens when another execution enters the same group while one is already running.

---

# 3. Understanding `group`

Consider:

```yaml
concurrency:
  group: production
```

Every workflow using:

```text
production
```

belongs to the same concurrency group.

Imagine:

```text
Run 1 → group = production
Run 2 → group = production
Run 3 → group = production
```

GitHub knows:

```text
Run 1
  │
  ├── production group
  │
Run 2
  │
  ├── production group
  │
Run 3
  │
  └── production group
```

Therefore, they compete for the same concurrency slot.

---

# 4. How many can run?

For a concurrency group, GitHub allows:

**One running + one pending**

for that group.

Conceptually:

```text
Concurrency Group: production

             ┌─────────────┐
             │   RUNNING   │
             │     A       │
             └─────────────┘

             ┌─────────────┐
             │   PENDING   │
             │     B       │
             └─────────────┘
```

If another run arrives:

```text
A → Running
B → Pending
C → New
```

GitHub can replace the pending run with the newer one.

```text
A → Running
B → Cancelled
C → Pending
```

This behavior is particularly important when designing CI pipelines.

---

# 5. `cancel-in-progress: false`

Example:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

This means:

> If another run enters the same concurrency group, don't cancel the currently running run.

Suppose:

```text
10:00 → Deployment A starts
10:02 → Deployment B starts/waits
10:03 → Deployment C arrives
```

Conceptually:

```text
Deployment A
     │
     │ RUNNING
     │
     ├───────────────────┐
                         │
Deployment B             │
     │                   │
     │ PENDING           │
                         │
Deployment C arrives     │
     │
     ▼
B may be replaced by C
```

The important point is that **A is not cancelled**.

This is useful for production.

---

# 6. `cancel-in-progress: true`

Now:

```yaml
concurrency:
  group: production
  cancel-in-progress: true
```

means:

> When a newer run enters the same group, cancel the currently running run.

Example:

```text
Deployment A
     │
     │ RUNNING
     │
     ▼
Deployment B arrives
     │
     ▼
Cancel A
     │
     ▼
Run B
```

If C arrives:

```text
A → Cancelled
B → Cancelled
C → Running
```

This is useful when **only the latest version matters**.

---

# 7. CI example

Suppose you have:

```text
Commit 101
Commit 102
Commit 103
Commit 104
```

Developers push these commits quickly.

Without concurrency:

```text
Commit 101 → Build
Commit 102 → Build
Commit 103 → Build
Commit 104 → Build
```

All four could run simultaneously.

This wastes:

- Runner minutes
- CPU
- Memory
- Time

If you configure:

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

you can effectively make the latest run the important one.

```text
101 → Running
102 → Cancel 101
103 → Cancel 102
104 → Cancel 103
104 → Running
```

This is often useful for CI on active development branches.

---

# 8. Why use `${{ github.ref }}`?

Instead of:

```yaml
group: ci
```

you can use:

```yaml
group: ci-${{ github.ref }}
```

`github.ref` identifies the Git reference that triggered the workflow.

For example:

```text
refs/heads/main
refs/heads/develop
refs/heads/feature/login
```

Therefore:

```text
ci-refs/heads/main
ci-refs/heads/develop
ci-refs/heads/feature/login
```

become different concurrency groups.

---

# 9. Branch-specific concurrency

A common pattern is:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Suppose you have:

```text
main
develop
feature/login
feature/payment
```

You effectively get:

```text
CI-main
CI-develop
CI-feature/login
CI-feature/payment
```

So:

```text
main
 └── Run A

develop
 └── Run B

feature/login
 └── Run C
```

They don't all block one another because they have different groups.

---

# 10. Why use `${{ github.workflow }}`?

Consider two workflows:

```text
CI Pipeline
Deployment Pipeline
```

If both simply use:

```yaml
group: ${{ github.ref }}
```

they could potentially end up in the same group for the same branch.

Instead:

```yaml
group: ${{ github.workflow }}-${{ github.ref }}
```

creates something conceptually like:

```text
CI Pipeline-main
Deployment Pipeline-main
```

This keeps the concurrency groups separated by workflow.

---

# 11. Workflow-level concurrency

You can define concurrency at the top level:

```yaml
name: CI Pipeline

on:
  push:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Building..."

  test:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Testing..."
```

Here the concurrency applies to the **workflow run**.

Think:

```text
Workflow Run A
 ├── Build
 └── Test

Workflow Run B
 ├── Build
 └── Test
```

Concurrency controls the workflow runs.

---

# 12. Job-level concurrency

You can also put concurrency inside a job:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Build"

  deploy:
    runs-on: ubuntu-latest

    concurrency:
      group: production
      cancel-in-progress: false

    steps:
      - run: echo "Deploy"
```

Now only the **deploy job** is protected by the concurrency group.

This is extremely useful for CI/CD.

For example:

```text
Workflow A
 ├── Build
 ├── Test
 └── Deploy ─────┐
                 │
Workflow B       │
 ├── Build       │
 ├── Test        │
 └── Deploy ─────┘
                  │
             concurrency
```

Build and test can happen independently.

Only deployment is serialized.

---

# 13. Workflow concurrency vs Job concurrency

This is an important interview question.

### Workflow-level

```yaml
concurrency:
  group: production
```

controls the **workflow run**.

### Job-level

```yaml
jobs:
  deploy:
    concurrency:
      group: production
```

controls the **specific job**.

### Example

```text
Workflow A                 Workflow B

Build                      Build
 │                          │
 ▼                          ▼
Test                       Test
 │                          │
 ▼                          ▼
Deploy A ────┐       Deploy B ────┐
             │                    │
             └── concurrency ─────┘
```

This allows Build/Test to happen in parallel but prevents Deploy from overlapping.

---

# 14. Production deployment example

This is a very realistic pattern:

```yaml
name: Production Deployment

on:
  push:
    branches:
      - main

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - name: Build
        run: |
          echo "Building application..."
          sleep 10


  test:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Test
        run: |
          echo "Running tests..."
          sleep 10


  deploy:
    runs-on: ubuntu-latest
    needs: test

    environment:
      name: production

    concurrency:
      group: production-deployment
      cancel-in-progress: false

    steps:
      - name: Deploy
        run: |
          echo "Deploying to production..."
          sleep 30
          echo "Deployment completed"
```

The pipeline becomes:

```text
             Build
               │
               ▼
             Test
               │
               ▼
      ┌─────────────────┐
      │    Production   │
      │    Deployment   │
      │                 │
      │  concurrency:   │
      │    production   │
      └─────────────────┘
```

If another workflow reaches `deploy` while one is already deploying:

```text
Deployment A
     │
     │ RUNNING
     ▼
Production
     ▲
     │
Deployment B
     │
     │ WAITING
```

They don't deploy simultaneously.

---

# 15. Different concurrency for DEV, QA and PROD

You can make the concurrency group dynamic.

```yaml
jobs:

  deploy:
    runs-on: ubuntu-latest

    environment: ${{ inputs.environment }}

    concurrency:
      group: deploy-${{ inputs.environment }}
      cancel-in-progress: false

    steps:
      - name: Deploy
        run: |
          echo "Deploying to ${{ inputs.environment }}"
```

Now:

```text
DEV
 │
 └── deploy-dev

QA
 │
 └── deploy-qa

PROD
 │
 └── deploy-prod
```

A DEV deployment doesn't block PROD.

---

# 16. Concurrency with manual deployment

You can combine it with `workflow_dispatch`:

```yaml
name: Deploy

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

jobs:

  deploy:
    runs-on: ubuntu-latest

    concurrency:
      group: deploy-${{ inputs.environment }}
      cancel-in-progress: false

    steps:

      - name: Deploy
        run: |
          echo "Deploying to ${{ inputs.environment }}"
```

Now imagine:

```text
User starts:
Environment = production
```

The concurrency group becomes:

```text
deploy-production
```

Another user starts another production deployment:

```text
deploy-production
```

Both belong to the same concurrency group.

Therefore they won't deploy simultaneously.

But:

```text
deploy-dev
```

is a different group.

---

# 17. Concurrency and `needs` are different

This is another important concept.

### `needs`

Controls **dependency/order**.

```yaml
test:
  needs: build
```

means:

```text
Build
  ↓
Test
```

### `concurrency`

Controls **simultaneous execution**.

```yaml
concurrency:
  group: production
```

means:

```text
Only one member of this group can actively execute at a time.
```

So:

```text
needs
 ↓
Controls dependency

concurrency
 ↓
Controls overlapping executions
```

They solve different problems.

---

# 18. Combining `needs` and concurrency

This is a very common production design:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

  test:
    runs-on: ubuntu-latest
    needs: build

  deploy:
    runs-on: ubuntu-latest
    needs: test

    concurrency:
      group: production
      cancel-in-progress: false
```

Execution:

```text
         BUILD
           │
        needs
           ▼
          TEST
           │
        needs
           ▼
        DEPLOY
           │
      concurrency
           │
           ▼
      PRODUCTION
```

So:

- `needs` → Build must finish before Test
- `needs` → Test must finish before Deploy
- `concurrency` → Deployments cannot overlap

---

# 19. Concurrency and environments

For production systems, you will often see both:

```yaml
environment:
  name: production

concurrency:
  group: production
```

These solve different problems.

### Environment

Provides things such as:

- Environment-specific secrets
- Deployment protection rules
- Required reviewers
- Deployment history

### Concurrency

Controls:

- Simultaneous execution
- Queueing/cancellation of runs

So:

```text
              Deployment
                   │
          ┌────────┴────────┐
          │                 │
     Environment        Concurrency
          │                 │
     production          production
          │                 │
     Approvals          One at a time
     Secrets
```

---

# 20. `cancel-in-progress` — when to use what?

### For CI

Usually:

```yaml
cancel-in-progress: true
```

Example:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Why?

Suppose a developer makes five commits:

```text
Commit 1
Commit 2
Commit 3
Commit 4
Commit 5
```

You don't necessarily need to finish testing commits 1–4 if commit 5 contains all their changes.

---

### For production

Usually:

```yaml
cancel-in-progress: false
```

Why?

Imagine:

```text
Production deployment A
        │
        │ halfway complete
        ▼
Production deployment B arrives
```

You generally don't want GitHub blindly cancelling A in the middle of a production deployment.

Instead:

```text
A → Finish
B → Wait
```

This is a safer default.

---

# 21. Real-world DevOps pipeline

For the type of CI/CD pipeline you are learning, a good architecture is:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
┌─────────────┐
│    Build    │
└──────┬──────┘
       │ needs
       ▼
┌─────────────┐
│    Test     │
└──────┬──────┘
       │ needs
       ▼
┌─────────────┐
│ Docker Build│
└──────┬──────┘
       │ needs
       ▼
┌─────────────┐
│ Push Image  │
└──────┬──────┘
       │ needs
       ▼
┌─────────────────────┐
│ Production Deploy   │
│                     │
│ concurrency:        │
│ production          │
└─────────────────────┘
```

This gives you:

**Dependency control:**

```yaml
needs:
```

**Concurrency control:**

```yaml
concurrency:
```

**Environment control:**

```yaml
environment:
```

---

# 22. Interview-ready definition

If you're asked **"What is concurrency in GitHub Actions?"**, a good answer is:

> **Concurrency in GitHub Actions allows us to control how many workflow runs or jobs belonging to the same concurrency group can execute simultaneously. We can use it to prevent conflicting deployments, serialize production deployments, or cancel outdated CI runs when newer commits arrive. The `group` defines the concurrency boundary, while `cancel-in-progress` determines whether an existing run should be cancelled when a newer run starts.**

### The three concepts to remember

```text
                GitHub Actions
                      │
          ┌───────────┼───────────┐
          │           │           │
        needs     concurrency   environment
          │           │           │
       Order      Parallelism   Deployment
       /dependency  control      protection
```

For your next GitHub Actions topic, **Contexts & Expressions** is the natural next step because it explains how `${{ github.* }}`, `${{ env.* }}`, `${{ vars.* }}`, `${{ secrets.* }}`, `${{ needs.* }}`, and `${{ steps.* }}` work together.
