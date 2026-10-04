# GitHub Actions — Triggering Workflows

A GitHub Actions workflow starts when an **event** occurs. The events are defined under the `on:` section of the workflow YAML.

The basic structure is:

```yaml
name: My Workflow

on:
  <event>:

jobs:
  ...
```

## 1. Manual trigger — `workflow_dispatch`

This is the simplest trigger for learning:

```yaml
name: Manual Workflow

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Run test
        run: |
          echo "Workflow started manually"
          echo "Hello GitHub Actions"
```

After pushing this workflow to GitHub, you can go to:

**Actions → Manual Workflow → Run workflow**

---

## 2. Trigger on `push`

Run the workflow whenever code is pushed:

```yaml
name: Push Workflow

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Code was pushed"
```

For example:

```text
Developer
    │
    │ git push
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ▼
Workflow
```

---

## 3. Trigger only on a specific branch

For example, only when code is pushed to `main`:

```yaml
name: Main Branch Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Code pushed to main"
```

You can specify multiple branches:

```yaml
on:
  push:
    branches:
      - main
      - develop
      - feature/*
```

---

## 4. Trigger on Pull Request

```yaml
name: Pull Request Workflow

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Run tests
        run: |
          echo "Pull request detected"
          echo "Running tests..."
```

You can restrict it to specific branches:

```yaml
on:
  pull_request:
    branches:
      - main
```

This means:

```text
feature branch
      │
      │ Pull Request
      ▼
    main
      │
      ▼
GitHub Actions
      │
      ▼
Run tests
```

---

## 5. Trigger on push AND pull request

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main
```

The workflow runs for either event.

---

# 6. Trigger when specific files change

This is very useful in real projects.

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'Dockerfile'
      - '.github/workflows/**'
```

Now the workflow runs only if these files change.

For example:

```text
src/app.py       → Workflow runs
Dockerfile       → Workflow runs
README.md        → Workflow doesn't run
docs/test.md     → Workflow doesn't run
```

---

# 7. Trigger based on file extensions

You can use patterns:

```yaml
on:
  push:
    paths:
      - '**.py'
      - '**.yaml'
      - '**.yml'
```

This is useful when you have different pipelines for different parts of a repository.

---

# 8. Scheduled workflow

You can run a workflow automatically using cron.

For example, every day at 9 AM UTC:

```yaml
name: Daily Job

on:
  schedule:
    - cron: '0 9 * * *'

jobs:
  daily:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Daily job running"
```

Cron format:

```text
┌──────── minute
│ ┌────── hour
│ │ ┌──── day
│ │ │ ┌── month
│ │ │ │ ┌ day of week
│ │ │ │ │
* * * * *
```

Example:

```text
0 9 * * *
```

means **09:00 UTC every day**.

---

# 9. Trigger another workflow

GitHub Actions can trigger workflows based on another workflow completing.

```yaml
on:
  workflow_run:
    workflows: ["Build"]
    types:
      - completed
```

For example:

```text
Build Workflow
      │
      │ completed
      ▼
Deploy Workflow
```

You can also check whether the previous workflow succeeded:

```yaml
jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying..."
```

---

# 10. Trigger from another workflow using `workflow_call`

This is useful when creating **reusable workflows**.

Workflow 1:

```yaml
name: Build

on:
  workflow_call:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Reusable build workflow"
```

Another workflow can call it:

```yaml
jobs:
  call-build:
    uses: ./.github/workflows/build.yml
```

This is very useful for organizations where multiple repositories need the same CI/CD logic.

---

# 11. Trigger using another repository or API

You can also trigger workflows using `repository_dispatch`.

Workflow:

```yaml
on:
  repository_dispatch:
    types:
      - deploy
```

An external system can then send an event to GitHub.

This is useful for:

```text
External system
      │
      │ API
      ▼
GitHub Repository
      │
      ▼
repository_dispatch
      │
      ▼
GitHub Actions
```

---

# 12. Multiple triggers in one workflow

A real workflow can have multiple triggers:

```yaml
name: CI Pipeline

on:

  # Run when code is pushed
  push:
    branches:
      - main

  # Run for pull requests
  pull_request:
    branches:
      - main

  # Allow manual execution
  workflow_dispatch:

  # Run every night
  schedule:
    - cron: '0 18 * * *'
```

So the workflow can be started by:

```text
             ┌── push ─────────────┐
             │                     │
             ├── pull_request ─────┤
             │                     │
             ├── workflow_dispatch ┤
             │                     │
             └── schedule ─────────┘
                       │
                       ▼
                GitHub Actions
```

---

# Important GitHub Actions Triggers

| Trigger | Purpose |
|---|---|
| `push` | Code pushed |
| `pull_request` | PR created/updated |
| `workflow_dispatch` | Manual execution |
| `schedule` | Cron-based execution |
| `workflow_run` | After another workflow |
| `workflow_call` | Reusable workflow |
| `repository_dispatch` | External/API event |
| `release` | Release created/published |
| `issues` | Issue events |
| `workflow_dispatch` | Run with manual inputs |

### A very useful CI/CD example

For your DevOps pipeline, a typical setup would be:

```yaml
name: CI/CD

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:
```

Then:

```text
Developer
    │
    ├── Push ───────────────┐
    │                       │
    └── Pull Request ──────►│
                            ▼
                      GitHub Actions
                            │
                            ▼
                         Build
                            │
                            ▼
                          Test
                            │
                            ▼
                       Docker Build
                            │
                            ▼
                     Docker Registry
                            │
                            ▼
                       Deployment
```

**Next concept to learn:** `workflow_dispatch` with **inputs**. That lets you manually start a workflow and provide values such as `environment=dev`, `version=1.2.0`, or `region=ap-south-1`, which is very useful for deployment pipelines.
