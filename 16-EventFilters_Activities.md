# GitHub Actions Lab Exercise: Workflow Event Filters & Activity Types

## Lab Objective

Create a GitHub Actions workflow that demonstrates:

- Push events
- Pull request events
- Branch filters
- Path filters
- Pull request activity types
- Issue activity types
- Tags
- Conditions using `github.event`
- Combining filters and activity types

---

## Lab 1 — Basic Push Trigger

### Requirement

Create a workflow that runs whenever code is pushed to the repository.

### Tasks

1. Create:

```text
.github/workflows/push.yml
```

2. Configure the workflow for the `push` event.
3. Add a job called `show-event`.
4. Print:
   - Repository name
   - Branch name
   - Event name
   - Commit SHA

### Expected Result

Push code and verify that the workflow executes.

---

# Lab 2 — Branch Filter

### Requirement

Modify the workflow so that it runs only when code is pushed to `main`.

### Tasks

Test the workflow with:

```text
main
develop
feature/test
```

### Expected Result

| Branch | Workflow |
|---|---|
| `main` | ✅ |
| `develop` | ❌ |
| `feature/test` | ❌ |

---

# Lab 3 — Multiple Branches

### Requirement

Configure the workflow to run for:

```text
main
develop
release/*
```

### Tasks

Test with:

```text
main
develop
release/v1.0
feature/login
```

### Expected Result

Only the first three should trigger the workflow.

---

# Lab 4 — Path Filter

Create the following repository structure:

```text
project/
├── application/
│   └── app.py
├── terraform/
│   └── main.tf
├── docs/
│   └── README.md
└── Dockerfile
```

### Requirement

Trigger the workflow only when files under:

```text
application/**
```

are modified.

### Tasks

Modify:

```text
application/app.py
```

Then modify:

```text
terraform/main.tf
```

### Expected Result

Application changes trigger the workflow; Terraform changes don't.

---

# Lab 5 — Branch + Path Filter

### Requirement

Trigger the workflow only when:

```text
Branch = main
AND
application/** changes
```

### Test Matrix

| Branch | File | Expected |
|---|---|---|
| main | application/app.py | ✅ |
| main | terraform/main.tf | ❌ |
| develop | application/app.py | ❌ |
| develop | README.md | ❌ |

---

# Lab 6 — Pull Request Activity Types

### Requirement

Create a workflow triggered by a pull request targeting `main`.

Trigger only for:

```text
opened
synchronize
reopened
```

### Tasks

1. Create a feature branch.
2. Create a PR to `main`.
3. Add another commit to the PR.
4. Close the PR.
5. Reopen the PR.

### Expected Result

Identify which activities triggered the workflow.

---

# Lab 7 — Pull Request Branch + Path + Activity Type

### Requirement

Create a workflow with:

```text
Target branch: main

Paths:
application/**
Dockerfile

Activity types:
opened
synchronize
reopened
```

### Test

Perform the following:

1. Open PR modifying `application/app.py`.
2. Push another commit.
3. Modify `terraform/main.tf`.
4. Reopen the PR.

Record which actions trigger the workflow.

---

# Lab 8 — Issue Activity Types

### Requirement

Create a workflow that runs when a new GitHub Issue is opened.

### Tasks

1. Create an Issue.
2. Configure the workflow for:
   ```text
   opened
   ```
3. Print:
   - Issue number
   - Issue title
   - Issue author

### Expected Output

```text
New Issue Created
Issue Number: 10
Issue Title: Application is not starting
Issue Author: student
```

---

# Lab 9 — Issue Label Activity

### Requirement

Create a workflow triggered whenever an Issue receives a label.

Use:

```text
labeled
```

### Tasks

1. Create an Issue.
2. Add:
   ```text
   bug
   ```
3. Add:
   ```text
   enhancement
   ```
4. Display the label name using the GitHub event context.

### Expected Result

The workflow should display which label was added.

---

# Lab 10 — Conditional Label Processing

### Requirement

Trigger the workflow whenever an Issue is labeled.

But execute the job only when:

```text
label = bug
```

### Expected Behavior

```text
bug          → ✅ Execute job
enhancement  → ⏭️ Skip job
documentation → ⏭️ Skip job
```

---

# Lab 11 — Git Tag Trigger

### Requirement

Create a workflow that runs when a tag matching:

```text
v*
```

is pushed.

### Test

Create and push:

```text
v1.0.0
v2.0.0
release-1
test
```

### Expected Result

Only:

```text
v1.0.0
v2.0.0
```

should trigger the workflow.

---

# Lab 12 — Release Activity

### Requirement

Create a workflow triggered when a GitHub Release is published.

Use:

```text
release:
  types:
    - published
```

### Tasks

When the release is published, display:

```text
Release Name
Release Tag
Release Author
```

---

# Lab 13 — Event Context Investigation

Create one workflow supporting:

```text
push
pull_request
issues
workflow_dispatch
```

### Requirement

Print:

```text
github.event_name
github.ref_name
github.actor
github.repository
github.sha
```

### Task

Trigger the workflow using each event and compare the values.

Create a table:

| Event | event_name | ref_name | actor |
|---|---|---|---|
| Push | | | |
| Pull Request | | | |
| Issue | | | |
| Manual | | | |

---

# Lab 14 — Event-Based Conditional Jobs

Create a workflow triggered by:

```text
push
pull_request
```

Create three jobs:

```text
build
test
deploy
```

### Requirements

**Build**

Runs for both events.

**Test**

Runs for both events.

**Deploy**

Runs only when:

```text
event = push
AND
branch = main
```

### Expected Flow

```text
Pull Request
    │
    ├── Build ✅
    ├── Test  ✅
    └── Deploy ⏭️

Push → main
    │
    ├── Build ✅
    ├── Test  ✅
    └── Deploy ✅
```

---

# Lab 15 — Final Challenge: Complete CI/CD Pipeline

Create a repository:

```text
github-actions-event-lab/
├── application/
│   └── app.py
├── tests/
│   └── test_app.py
├── Dockerfile
└── .github/
    └── workflows/
```

Create a workflow implementing the following requirements.

### CI Trigger

Run when:

```text
Pull Request → main
```

Activity types:

```text
opened
synchronize
reopened
```

Paths:

```text
application/**
tests/**
Dockerfile
```

### Production Trigger

Run when:

```text
push → main
```

and:

```text
application/**
Dockerfile
```

has changed.

### Jobs

```text
Build
   ↓
Test
   ↓
Deploy
```

### Deployment Rule

Deploy only when:

```text
github.event_name == push
```

and:

```text
github.ref_name == main
```

### Final Expected Architecture

```text
                    GitHub Repository
                           │
              ┌────────────┴────────────┐
              │                         │
        Pull Request                  Push
              │                         │
         Target: main                main
              │                         │
        Path Filter                  Path Filter
              │                         │
        Activity Types                 │
              │                         │
              ▼                         ▼
            Build                     Build
              │                         │
              ▼                         ▼
             Test                     Test
              │                         │
              ▼                         ▼
           No Deploy                Deploy
```

### Deliverables

Students should submit:

```text
1. GitHub repository URL
2. Workflow YAML file
3. Screenshots of successful workflow runs
4. Screenshot showing skipped deployment
5. Screenshot showing event/activity information
6. Short explanation of:
   - Event
   - Filter
   - Activity type
   - Condition
```
