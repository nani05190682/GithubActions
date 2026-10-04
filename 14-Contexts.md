# Contexts in GitHub Actions

**Contexts** are one of the most important concepts in GitHub Actions. They provide information about the **workflow, repository, event, jobs, steps, runner, variables, secrets, and inputs**.

You access a context using the expression syntax:

```yaml
${{ context.property }}
```

For example:

```yaml
${{ github.repository }}
```

might return:

```text
vaitheeswaran/my-project
```

---

# 1. Why do we need contexts?

Imagine GitHub starts your workflow because someone pushed code.

Your workflow may need to know:

- Which repository triggered it?
- Which branch?
- Who pushed the code?
- What commit was pushed?
- Which workflow is running?
- What operating system is the runner using?
- What secrets are available?
- What environment was selected?
- What output did the previous job produce?

Instead of hard-coding these values, GitHub exposes them through **contexts**.

```text
GitHub Event
     │
     ▼
GitHub Actions
     │
     ├── Repository information
     ├── Branch information
     ├── Commit information
     ├── Actor information
     ├── Runner information
     ├── Variables
     ├── Secrets
     ├── Job information
     └── Step information
```

---

# 2. Basic syntax

Contexts use:

```yaml
${{ ... }}
```

Example:

```yaml
- name: Display repository
  run: |
    echo "Repository: ${{ github.repository }}"
```

The expression:

```yaml
${{ github.repository }}
```

is evaluated by GitHub Actions before the command is executed.

---

# 3. Main GitHub Actions contexts

The most important contexts are:

| Context | Purpose |
|---|---|
| `github` | Information about GitHub repository/event |
| `env` | Environment variables |
| `vars` | GitHub configuration variables |
| `secrets` | Secrets |
| `job` | Current job |
| `jobs` | Job outputs in reusable workflows |
| `steps` | Information/output from previous steps |
| `runner` | Runner information |
| `strategy` | Matrix strategy information |
| `matrix` | Current matrix values |
| `needs` | Outputs/results from dependent jobs |
| `inputs` | Workflow/action inputs |

Let's go through them.

---

# 4. `github` context

The `github` context is probably the most commonly used.

Example:

```yaml
name: GitHub Context Demo

on:
  push:

jobs:

  info:
    runs-on: ubuntu-latest

    steps:

      - name: GitHub information
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          echo "Workflow: ${{ github.workflow }}"
          echo "Run ID: ${{ github.run_id }}"
          echo "Run Number: ${{ github.run_number }}"
```

Important properties:

```text
github.repository
github.repository_owner
github.ref
github.ref_name
github.sha
github.actor
github.workflow
github.run_id
github.run_number
github.event_name
github.workspace
```

---

# 5. `github.ref_name`

This gives you the branch or tag name.

If you push to:

```text
main
```

then:

```yaml
${{ github.ref_name }}
```

returns:

```text
main
```

Very useful for deployment logic:

```yaml
- name: Display branch
  run: |
    echo "Branch = ${{ github.ref_name }}"
```

---

# 6. `github.event_name`

This tells you **what triggered the workflow**.

For example:

```yaml
- name: Show trigger
  run: |
    echo "Event: ${{ github.event_name }}"
```

Possible values:

```text
push
pull_request
workflow_dispatch
schedule
workflow_call
workflow_run
```

You can use it in conditions:

```yaml
if: ${{ github.event_name == 'push' }}
```

---

# 7. `github.actor`

This tells you who triggered the workflow.

```yaml
- name: Display user
  run: |
    echo "Workflow triggered by: ${{ github.actor }}"
```

Example:

```text
Workflow triggered by: vaitheeswaran
```

---

# 8. `github.sha`

This contains the commit SHA that triggered the workflow.

```yaml
- name: Display commit
  run: |
    echo "Commit: ${{ github.sha }}"
```

You might use it to tag a Docker image:

```yaml
- name: Build Docker image
  run: |
    docker build -t myapp:${{ github.sha }} .
```

Then you get:

```text
myapp:a8f72c...
```

This is very useful for traceability.

---

# 9. `env` context

The `env` context represents environment variables.

For example:

```yaml
env:
  APP_NAME: myapp
  ENVIRONMENT: production
```

You can access them using:

```yaml
${{ env.APP_NAME }}
```

Example:

```yaml
name: Environment Demo

on:
  workflow_dispatch:

env:
  APP_NAME: myapp
  ENVIRONMENT: production

jobs:

  build:
    runs-on: ubuntu-latest

    steps:

      - name: Display environment
        run: |
          echo "Application: ${{ env.APP_NAME }}"
          echo "Environment: ${{ env.ENVIRONMENT }}"
```

Inside the shell, you can also use:

```bash
$APP_NAME
$ENVIRONMENT
```

---

# 10. `vars` context

GitHub allows you to define **configuration variables**.

For example, repository variables:

```text
APP_NAME = myapp
REGISTRY = ghcr.io
ENVIRONMENT = production
```

You access them with:

```yaml
${{ vars.APP_NAME }}
```

Example:

```yaml
- name: Display configuration
  run: |
    echo "Application: ${{ vars.APP_NAME }}"
    echo "Registry: ${{ vars.REGISTRY }}"
```

### `vars` vs `env`

A useful distinction:

```text
vars
 │
 └── Configuration managed in GitHub

env
 │
 └── Environment variables defined for workflow/job/step
```

---

# 11. `secrets` context

Secrets contain sensitive information.

For example:

```yaml
${{ secrets.DOCKER_USERNAME }}
```

and:

```yaml
${{ secrets.DOCKER_PASSWORD }}
```

Example:

```yaml
- name: Docker login
  env:
    USERNAME: ${{ secrets.DOCKER_USERNAME }}
    PASSWORD: ${{ secrets.DOCKER_PASSWORD }}

  run: |
    echo "$PASSWORD" | docker login \
      -u "$USERNAME" \
      --password-stdin
```

Don't do this:

```yaml
run: echo "${{ secrets.DOCKER_PASSWORD }}"
```

Avoid printing secrets to logs.

---

# 12. `runner` context

The `runner` context provides information about the machine executing the job.

Example:

```yaml
- name: Runner information
  run: |
    echo "OS: ${{ runner.os }}"
    echo "Architecture: ${{ runner.arch }}"
    echo "Runner name: ${{ runner.name }}"
    echo "Workspace: ${{ runner.workspace }}"
    echo "Temp: ${{ runner.temp }}"
```

You may see:

```text
OS: Linux
Architecture: X64
Runner name: GitHub Actions 123
```

Useful properties include:

```text
runner.os
runner.arch
runner.name
runner.workspace
runner.temp
runner.tool_cache
```

---

# 13. `job` context

The `job` context contains information about the currently running job.

Example:

```yaml
- name: Job information
  run: |
    echo "Job status: ${{ job.status }}"
```

Possible status values include:

```text
success
failure
cancelled
```

A common use:

```yaml
if: ${{ job.status == 'success' }}
```

---

# 14. `steps` context

This is extremely important when passing data from one step to another.

Suppose:

```yaml
- name: Generate version
  id: version
  run: |
    echo "version=1.0.25" >> "$GITHUB_OUTPUT"
```

The step has:

```yaml
id: version
```

You can access its output later:

```yaml
- name: Display version
  run: |
    echo "Version: ${{ steps.version.outputs.version }}"
```

The flow is:

```text
Step 1
  │
  │ GITHUB_OUTPUT
  ▼
steps.version.outputs.version
  │
  ▼
Step 2
```

---

# 15. `needs` context

`needs` is used to access information from a **dependent job**.

For example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:

      - name: Generate version
        id: version
        run: |
          echo "version=1.0.25" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          echo "Deploying version ${{ needs.build.outputs.version }}"
```

Here:

```yaml
${{ needs.build.outputs.version }}
```

means:

> Get the `version` output from the `build` job.

---

# 16. `matrix` context

Since we just discussed Matrix Strategy, this is very important.

Example:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest
```

You can access:

```yaml
${{ matrix.os }}
```

Example:

```yaml
runs-on: ${{ matrix.os }}
```

And:

```yaml
- name: Display OS
  run: |
    echo "Testing on ${{ matrix.os }}"
```

If GitHub is executing the Windows matrix job:

```text
matrix.os = windows-latest
```

---

# 17. `strategy` context

The `strategy` context provides information about the matrix strategy.

For example:

```yaml
- name: Matrix information
  run: |
    echo "Job index: ${{ strategy.job-index }}"
    echo "Total jobs: ${{ strategy.job-total }}"
```

This can help when you have many matrix combinations.

Conceptually:

```text
Matrix
 ├── Job 1 / 6
 ├── Job 2 / 6
 ├── Job 3 / 6
 ├── Job 4 / 6
 ├── Job 5 / 6
 └── Job 6 / 6
```

---

# 18. `inputs` context

Inputs are useful with `workflow_dispatch`.

Example:

```yaml
name: Deployment

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

Then:

```yaml
jobs:

  deploy:
    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          echo "Environment: ${{ inputs.environment }}"
```

If the user selects:

```text
production
```

then:

```yaml
${{ inputs.environment }}
```

returns:

```text
production
```

---

# 19. Contexts working together

This is where contexts become really powerful.

Consider:

```yaml
jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest

    runs-on: ${{ matrix.os }}

    steps:

      - name: Information
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "OS: ${{ matrix.os }}"
          echo "Runner: ${{ runner.os }}"
          echo "Actor: ${{ github.actor }}"
```

Now several contexts are being used:

```text
github
   │
   ├── repository
   ├── ref_name
   └── actor

matrix
   │
   └── os

runner
   │
   └── os
```

---

# 20. Contexts + Conditions

Contexts become particularly useful with `if`.

Example:

```yaml
- name: Production deployment
  if: ${{ github.ref_name == 'main' }}
  run: |
    echo "Deploying to production"
```

This means:

> Execute this step only if the workflow was triggered from `main`.

---

# 21. Branch + event condition

You can combine expressions:

```yaml
- name: Deploy
  if: >
    github.ref_name == 'main' &&
    github.event_name == 'push'

  run: |
    echo "Production deployment"
```

This means:

```text
Branch = main
       AND
Event = push
       │
       ▼
    Deploy
```

---

# 22. Contexts + Matrix

You can also conditionally exclude an OS:

```yaml
- name: Windows-specific step
  if: ${{ matrix.os == 'windows-latest' }}
  run: |
    echo "Running Windows-specific command"
```

And Linux:

```yaml
- name: Linux-specific step
  if: ${{ matrix.os == 'ubuntu-latest' }}
  run: |
    echo "Running Linux-specific command"
```

---

# 23. Contexts + `needs`

A realistic pipeline:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.image.outputs.tag }}

    steps:

      - name: Generate image tag
        id: image
        run: |
          echo "tag=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"


  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          echo "Deploying image:"
          echo "${{ needs.build.outputs.image_tag }}"
```

Now:

```text
github.sha
     │
     ▼
Build Job
     │
     │ steps.image.outputs.tag
     ▼
needs.build.outputs.image_tag
     │
     ▼
Deploy Job
```

---

# 24. The most important contexts

For practical DevOps work, I'd prioritize these:

### 1. `github`

```yaml
${{ github.repository }}
${{ github.ref_name }}
${{ github.sha }}
${{ github.actor }}
${{ github.event_name }}
```

### 2. `env`

```yaml
${{ env.APP_NAME }}
```

### 3. `vars`

```yaml
${{ vars.REGISTRY }}
```

### 4. `secrets`

```yaml
${{ secrets.DOCKER_PASSWORD }}
```

### 5. `matrix`

```yaml
${{ matrix.os }}
${{ matrix.version }}
```

### 6. `steps`

```yaml
${{ steps.build.outputs.image }}
```

### 7. `needs`

```yaml
${{ needs.build.outputs.version }}
```

### 8. `runner`

```yaml
${{ runner.os }}
```

### 9. `inputs`

```yaml
${{ inputs.environment }}
```

---

# 25. One complete example

Let's combine everything we've covered:

```yaml
name: Complete Context Demo

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Deployment environment"
        required: true
        type: choice
        options:
          - dev
          - qa
          - production

env:
  APP_NAME: myapp

jobs:

  build:

    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest

    runs-on: ${{ matrix.os }}

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:

      - name: Generate version
        id: version
        shell: bash
        run: |
          VERSION="1.0.${GITHUB_RUN_NUMBER}"
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"

      - name: Display contexts
        shell: bash
        run: |
          echo "Application: ${{ env.APP_NAME }}"
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          echo "Event: ${{ github.event_name }}"
          echo "OS: ${{ matrix.os }}"
          echo "Runner OS: ${{ runner.os }}"
          echo "Version: ${{ steps.version.outputs.version }}"
          echo "Environment: ${{ inputs.environment }}"


  deploy:

    needs: build

    runs-on: ubuntu-latest

    if: ${{ inputs.environment == 'production' }}

    steps:

      - name: Deploy
        run: |
          echo "Application: ${{ env.APP_NAME }}"
          echo "Version: ${{ needs.build.outputs.version }}"
          echo "Environment: ${{ inputs.environment }}"
          echo "Deploying..."
```

Notice how many contexts are working together:

```text
                         Workflow
                            │
       ┌────────────────────┼─────────────────────┐
       │                    │                     │
     github                inputs                env
       │                    │                     │
 repository             environment            APP_NAME
 branch
 commit
 actor
 event
       │
       ▼
     Build
       │
       ├── matrix.os
       ├── runner.os
       └── steps.version.outputs
                    │
                    ▼
                 needs
                    │
                    ▼
                 Deploy
```

---

# 26. Context vs Environment Variable

This is a common source of confusion.

You might see:

```yaml
${{ github.ref_name }}
```

and:

```bash
$GITHUB_REF_NAME
```

They can provide similar information, but they are **not the same mechanism**.

GitHub Actions expression:

```yaml
${{ github.ref_name }}
```

is evaluated by the Actions expression engine.

Environment variable:

```bash
$GITHUB_REF_NAME
```

is available to the shell.

For example:

```yaml
- name: Example
  run: |
    echo "Expression: ${{ github.ref_name }}"
    echo "Environment: $GITHUB_REF_NAME"
```

Both can give:

```text
main
```

but they are being resolved differently.

---

# 27. Contexts cheat sheet

| Context | Example | Main purpose |
|---|---|---|
| `github` | `${{ github.sha }}` | GitHub/event information |
| `env` | `${{ env.APP_NAME }}` | Environment variables |
| `vars` | `${{ vars.REGISTRY }}` | Configuration variables |
| `secrets` | `${{ secrets.API_KEY }}` | Sensitive data |
| `runner` | `${{ runner.os }}` | Runner information |
| `job` | `${{ job.status }}` | Current job |
| `steps` | `${{ steps.build.outputs.tag }}` | Step outputs |
| `needs` | `${{ needs.build.outputs.tag }}` | Previous job outputs |
| `matrix` | `${{ matrix.os }}` | Matrix values |
| `strategy` | `${{ strategy.job-total }}` | Matrix strategy |
| `inputs` | `${{ inputs.environment }}` | Workflow inputs |

---

## The easiest way to remember contexts

Think of a GitHub Actions workflow as a program:

```text
                 GitHub Actions
                       │
       ┌───────────────┼────────────────┐
       │               │                │
    github           runner            inputs
       │               │                │
   repository        OS/CPU          user input
   branch
   commit
       │
       ▼
      Job
       │
   ┌───┼────────┐
   │   │        │
 env vars     matrix
   │              │
   ▼              ▼
  APP_NAME      OS/version
       │
       ▼
     steps
       │
       ▼
    outputs
       │
       ▼
     needs
       │
       ▼
   next job
```

### The key distinction

**Contexts tell GitHub Actions what is happening around the workflow.**

**Expressions** (`${{ ... }}`) are how you access those contexts.

Once you understand `github`, `env`, `vars`, `secrets`, `matrix`, `steps`, `needs`, `runner`, and `inputs`, you have the foundation for writing sophisticated GitHub Actions pipelines.
