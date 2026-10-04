# Job Concurrency in GitHub Actions

**Concurrency** controls how many workflow runs or jobs are allowed to execute at the same time.

This is particularly important for **deployment pipelines**. For example, you don't want two deployments to production running simultaneously.

---

## 1. Basic concurrency

```yaml
name: Deployment

on:
  push:
    branches:
      - main

concurrency:
  group: production-deployment
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy
        run: |
          echo "Deploying to production..."
          sleep 30
```

Here:

```yaml
concurrency:
  group: production-deployment
```

means all workflow runs belong to the same concurrency group.

Only **one run** in that group can execute at a time.

---

## 2. What happens with multiple deployments?

Suppose three commits are pushed quickly:

```text
Commit A
   │
   ▼
Deployment A ────────────────┐
                             │
Commit B                     │ waiting
   │                         ▼
   ▼                    Deployment B
Deployment B                 │
                             │ waiting
Commit C                     ▼
   │                    Deployment C
   ▼
Deployment C
```

With:

```yaml
cancel-in-progress: false
```

the running deployment isn't cancelled.

New runs wait for the concurrency slot.

---

# 3. Cancel the previous deployment

For CI/CD, you might instead want the latest deployment to win.

```yaml
concurrency:
  group: production-deployment
  cancel-in-progress: true
```

Now:

```text
Deployment A
     │
     │ running
     ▼
Deployment B starts
     │
     ├── Cancel A
     │
     ▼
Deployment B
```

If Deployment C arrives:

```text
A ──► Cancelled
B ──► Cancelled
C ──► Running
```

This is useful when older deployments are no longer relevant.

---

# 4. Concurrency based on branch

You can create a different concurrency group for each branch:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

For example:

```text
main
  │
  └── CI-main

develop
  │
  └── CI-develop

feature/login
  │
  └── CI-feature-login
```

A deployment on `main` won't block a deployment on `develop`.

---

# 5. Job-level concurrency

Concurrency can also be applied to an individual job.

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Building..."


  deploy:
    runs-on: ubuntu-latest

    concurrency:
      group: production
      cancel-in-progress: false

    steps:
      - name: Deploy
        run: |
          echo "Deploying..."
```

Here only the **deploy job** participates in the concurrency restriction.

The build jobs can continue running normally.

---

# 6. Workflow-level vs Job-level

### Workflow-level

```yaml
concurrency:
  group: production
```

Controls the **entire workflow run**.

```text
Workflow A
 ├── Build
 ├── Test
 └── Deploy
       │
       │ concurrency
       ▼
Workflow B waits
```

### Job-level

```yaml
jobs:
  deploy:
    concurrency:
      group: production
```

Only the deployment job is restricted:

```text
Workflow A                  Workflow B
    │                           │
  Build                       Build
    │                           │
  Test                        Test
    │                           │
 Deploy ◄── concurrency ──► Deploy
```

This is often more useful for larger pipelines.

---

# 7. Environment-specific concurrency

A very practical pattern is:

```yaml
jobs:

  deploy:
    runs-on: ubuntu-latest

    environment: production

    concurrency:
      group: deploy-production
      cancel-in-progress: false

    steps:
      - name: Deploy
        run: |
          echo "Deploying to production"
```

You can have:

```text
                    Deployment
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
        DEV           QA            PROD
          │             │             │
       Group:        Group:        Group:
       dev           qa            prod
```

Therefore a DEV deployment doesn't block PROD.

---

# 8. Dynamic concurrency group

You can dynamically create the group:

```yaml
concurrency:
  group: deploy-${{ github.ref_name }}
  cancel-in-progress: true
```

For example:

```text
main       → deploy-main
develop    → deploy-develop
release    → deploy-release
```

---

# 9. Real-world CI/CD example

Consider this pipeline:

```yaml
name: Application CI/CD

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
          echo "Building application"


  test:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Test
        run: |
          echo "Running tests"


  deploy:
    runs-on: ubuntu-latest
    needs: test

    environment: production

    concurrency:
      group: production-deployment
      cancel-in-progress: false

    steps:
      - name: Deploy
        run: |
          echo "Deploying application to production"
```

Execution:

```text
        Build
          │
          ▼
        Test
          │
          ▼
    ┌──────────────┐
    │ Production   │
    │ Deployment   │
    │              │
    │ Concurrency  │
    └──────────────┘
```

If another production deployment arrives while one is running, GitHub prevents the two deployments from running simultaneously.

---

## `cancel-in-progress` — remember this

```yaml
cancel-in-progress: false
```

**Don't cancel the current run.**

```text
A running
B waiting
C waiting
```

Whereas:

```yaml
cancel-in-progress: true
```

**Cancel the current run when a newer run enters the same concurrency group.**

```text
A running
   ↓
B arrives → A cancelled
   ↓
C arrives → B cancelled
   ↓
C running
```

### Practical rule

For **production deployment**, I would generally start with:

```yaml
concurrency:
  group: production-deployment
  cancel-in-progress: false
```

For **build/test pipelines**, where only the latest commit matters:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

This prevents overlapping CI/CD activity while avoiding accidental simultaneous production deployments.
