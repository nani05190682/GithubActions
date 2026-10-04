# GitHub Actions Lab Exercise — Skipping Workflow Runs

## Objective

Learn how to **skip GitHub Actions workflow runs** using:

- Commit messages
- Branch/path filters
- Job-level conditions
- Step-level conditions

---

## Lab 1 — Skip Workflow Using Commit Message

### Requirement

Create a workflow that runs on every push.

```text
.github/workflows/ci.yml
```

The workflow should:

1. Trigger on `push`.
2. Display the commit message.
3. Skip the workflow when the commit message contains:

```text
[skip ci]
```

### Test

Make a normal commit:

```bash
git commit -m "Update application"
```

Then make:

```bash
git commit -m "Update documentation [skip ci]"
```

### Expected Result

```text
Normal commit
     │
     ▼
Workflow runs ✅

[skip ci]
     │
     ▼
Workflow skipped ⏭️
```

---

# Lab 2 — Test Different Skip Keywords

### Requirement

Investigate GitHub's supported commit-message skip keywords.

Test commits containing:

```text
[skip ci]
[ci skip]
[skip actions]
[actions skip]
```

### Task

Record which commit messages prevent the workflow from being triggered.

| Commit Message | Workflow |
|---|---|
| `Update code` | |
| `Update code [skip ci]` | |
| `Update code [ci skip]` | |
| `Update code [skip actions]` | |
| `Update code [actions skip]` | |

---

# Lab 3 — Skip Using Branch Filters

### Requirement

Create a workflow that runs for:

```text
main
develop
```

but does not run for:

```text
feature/*
```

### Test

Push commits to:

```text
main
develop
feature/login
feature/payment
```

### Expected Result

```text
main             → ✅
develop          → ✅
feature/login    → ❌
feature/payment  → ❌
```

---

# Lab 4 — Skip Using Path Filters

Create:

```text
application/
documentation/
terraform/
```

### Requirement

CI should run only when:

```text
application/**
```

changes.

### Test

Modify:

```text
application/app.py
```

Then modify:

```text
documentation/README.md
```

Then modify:

```text
terraform/main.tf
```

### Expected Result

```text
application/app.py       → Workflow ✅
documentation/README.md  → Workflow skipped
terraform/main.tf        → Workflow skipped
```

---

# Lab 5 — Skip Job Using `if`

### Requirement

Create a workflow triggered by every push.

Create two jobs:

```text
build
deploy
```

The `build` job should always run.

The `deploy` job should run only on:

```text
main
```

### Expected

```text
Push → main
   │
   ├── Build   ✅
   └── Deploy  ✅

Push → develop
   │
   ├── Build   ✅
   └── Deploy  ⏭️
```

---

# Lab 6 — Skip a Step

### Requirement

Create one job with three steps:

```text
Checkout
Build
Deploy
```

The Deploy step should run only when the branch is `main`.

### Expected

```text
main

Checkout  ✅
Build     ✅
Deploy    ✅


develop

Checkout  ✅
Build     ✅
Deploy    ⏭️
```

---

# Lab 7 — Skip Based on Commit Message

### Requirement

Create a workflow triggered by `push`.

If the commit message contains:

```text
docs
```

skip the deployment job.

### Example

```text
"Update application"
        ↓
Deploy ✅

"Update docs"
        ↓
Deploy ⏭️
```

---

# Lab 8 — Skip Documentation-Only Changes

### Repository

```text
project/
├── src/
│   └── app.py
├── tests/
│   └── test.py
├── docs/
│   └── README.md
└── Dockerfile
```

### Requirement

Create two workflows:

### Application CI

Runs when:

```text
src/**
tests/**
Dockerfile
```

changes.

### Documentation workflow

Runs when:

```text
docs/**
```

changes.

### Test

Make separate commits modifying:

```text
src/app.py
docs/README.md
tests/test.py
```

Observe which workflow executes.

---

# Lab 9 — Skip Production Deployment

### Requirement

Create:

```text
Build
Test
Deploy
```

The workflow should trigger for both:

```text
main
develop
```

But production deployment should execute only for `main`.

### Expected

```text
                 Push
                   │
             ┌─────┴─────┐
             │           │
           main       develop
             │           │
             ▼           ▼
           Build       Build
             │           │
             ▼           ▼
           Test        Test
             │           │
             ▼           ▼
          Deploy       Skip
```

---

# Lab 10 — Final Challenge

Create a complete workflow with the following rules.

### Trigger

```text
push
pull_request
```

### Branches

```text
main
develop
```

### Requirements

#### Build

Runs for every valid workflow trigger.

#### Test

Runs after Build.

#### Deploy to DEV

Runs only when:

```text
push + develop
```

#### Deploy to Production

Runs only when:

```text
push + main
```

#### Documentation Changes

Do not trigger CI when only:

```text
docs/**
```

changes.

#### Commit Skip

Do not run the workflow when the commit message contains:

```text
[skip ci]
```

---

## Expected Final Flow

```text
                         Git Event
                             │
                 ┌───────────┴───────────┐
                 │                       │
              Push                  Pull Request
                 │                       │
                 ▼                       ▼
          Branch / Path Filter     Branch / Path Filter
                 │                       │
                 └───────────┬───────────┘
                             │
                             ▼
                           Build
                             │
                             ▼
                           Test
                             │
                    ┌────────┴────────┐
                    │                 │
              develop/main        Pull Request
                    │
             ┌──────┴──────┐
             │             │
          develop         main
             │             │
             ▼             ▼
         DEV Deploy    PROD Deploy
```

### Student Deliverables

1. Workflow YAML
2. Repository URL
3. Screenshot of a successful Build/Test
4. Screenshot showing a skipped job
5. Screenshot showing a skipped workflow
6. Table documenting each skip mechanism tested.
