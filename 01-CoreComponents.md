# GitHub Actions – Core Components

GitHub Actions is GitHub's **CI/CD and automation platform**. The easiest way to understand it is:

```text
Repository
    │
    ├── Event
    │     ↓
    ├── Workflow
    │     ↓
    ├── Job
    │     ↓
    ├── Runner
    │     ↓
    └── Steps
          ↓
       Actions / Commands
```

![Image](https://images.openai.com/static-rsc-4/iC18IuJCkKNWb-qkuV1Nh_deXnNBa05yOby-EFO3OAthLmgxSdn2hBclK9h96CDGFIoYBjT6Hp6Gd30uc6Yhp216E1t-VlFczo-iShYhCmECTjrr_QWsfEEXR7oSvEprSns9ezpTW5qxBkAJkMvsc7uhWEgdBsHTNMdCYsxSOWltS5Qg39KlCiz8sntRIE27?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/owFpFOhBVoEoIJnWy1J7Xn_nu3DTMNt25-sNyEp4C_uQSTenF1RnvcbMM3bCJakvGjvAPmjYnmdCZIYcKRFoPJ_dBaBBeTDXjVLERKkkPoP-TzrpoEjaJghzk3bOrtPOdL86PBnCH1Yzpmht-wHc68MuCOa8YNKprp42b55n1JsWs2Bu666qfmq_6br_zPdc?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/XOCW4b1IJN8eL0CLnPWsi-AQu9UvS1S8W7rEwjb7JGB1uGZhJBw6xV9msgSzfGPRqBoDB107Z_iGvGXJzGadyFFOPRIRX8BnqhQWTJW74r5qlqjwn9Amx1XMFM6zTlqxVLlVRTR0Yk_Jn4D5liLrjFxYfOA9Jki5ooWkgFKqiguuoTHUjjN9I_jlEvcO8hf0?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/uhJZggRvql-aEXVRlLieqbGDHuGDAH_XKf2WImCvD6fjiXZpqPyjPbtRnHVU9_JSxRq-Np8WDhX95ZVrv6qSbLLq5_ibeErCwgXmq4ZBqmltf0aqBO81hutZy_iQraEbxbXqJSHJ3L7e9YyTS7txIGzoUOk7IB5R2WHJ5jHkd0iWOJqQlM9ZHuS4Pq5Rcpw6?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/iqDT53Pn3lsJn7Z73IMkb5xRNLtR3vnyy6ZH3KwVAm0-YhbQWVmC14UMCkmxLHXZzVymS5JI5zeFYY_lqpv_6DBkf9O_VgkSSvybm0sCkk1vWdIx9VLHUCBL9sA8gtfBaMKniKfLMuGirIQgo_aHULoBWMrnO1g5sAz56_--ggd1xvl5rONzpCo9LwFqDS0E?purpose=fullsize)

## 1. Workflow

A **workflow** is the complete automation process.

Workflow files are YAML files stored under:

```text
.github/workflows/
```

Example:

```yaml
name: Application CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Build application
        run: mvn clean package
```

Think of a workflow as:

> **"What automation should GitHub perform?"**

---

# 2. Events / Triggers

An **event** determines **when a workflow should run**.

Example:

```yaml
on:
  push:
    branches:
      - main
```

This means:

```text
Developer
    │
    │ git push
    ▼
main branch
    │
    ▼
GitHub detects PUSH event
    │
    ▼
Workflow starts
```

Common events:

```yaml
on:
  push:
  pull_request:
  workflow_dispatch:
  schedule:
  release:
```

### Manual trigger

```yaml
on:
  workflow_dispatch:
```

### Scheduled trigger

```yaml
on:
  schedule:
    - cron: "0 2 * * *"
```

### Multiple triggers

```yaml
on:
  push:
    branches:
      - main

  pull_request:

  workflow_dispatch:
```

---

# 3. Jobs

A **job** is a group of steps that execute together on a runner.

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build
        run: mvn clean package

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Test
        run: mvn test
```

Here we have two jobs:

```text
Workflow
   │
   ├── build
   │
   └── test
```

By default, jobs can execute **in parallel**.

---

# 4. Runner

A **runner** is the machine where a job actually executes.

Example:

```yaml
runs-on: ubuntu-latest
```

GitHub provides hosted runners such as:

```text
ubuntu-latest
windows-latest
macos-latest
```

Architecture:

```text
GitHub
   │
   │ starts job
   ▼
Runner
   │
   ├── Checkout code
   ├── Install dependencies
   ├── Build
   ├── Test
   └── Deploy
```

You can also use **self-hosted runners**.

Example:

```yaml
runs-on: self-hosted
```

This is particularly useful in enterprise environments where the runner needs access to internal infrastructure.

---

# 5. Steps

A **step** is an individual operation within a job.

Example:

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Install dependencies
    run: npm install

  - name: Run tests
    run: npm test

  - name: Build
    run: npm run build
```

So:

```text
Job
 │
 ├── Step 1 → Checkout
 ├── Step 2 → Install
 ├── Step 3 → Test
 └── Step 4 → Build
```

Steps execute **sequentially** within a job.

---

# 6. Actions

An **Action** is a reusable unit of automation.

Example:

```yaml
- uses: actions/checkout@v4
```

This action checks out your Git repository onto the runner.

Other examples:

```yaml
uses: actions/setup-python@v5
```

```yaml
uses: actions/setup-node@v4
```

```yaml
uses: docker/login-action@v3
```

Think of an Action as:

> **A reusable piece of automation.**

---

# 7. `run`

Instead of using a predefined Action, you can execute shell commands.

Example:

```yaml
- name: Build Docker image
  run: docker build -t myapp:1.0 .
```

Multiple commands:

```yaml
- name: Build
  run: |
    docker build -t myapp:1.0 .
    docker images
    docker push myapp:1.0
```

So there are two common types of steps:

```text
Step
 ├── uses → Reusable Action
 │
 └── run  → Shell command
```

---

# 8. Dependencies – `needs`

Jobs can depend on other jobs.

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

Execution:

```text
build
  │
  ▼
test
  │
  ▼
deploy
```

Without `needs`, independent jobs can run concurrently.

---

# 9. Variables

Variables allow you to avoid hard-coding values.

```yaml
env:
  APP_NAME: myapp

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo $APP_NAME
```

You can define variables at:

```text
Workflow level
       ↓
Job level
       ↓
Step level
```

---

# 10. Secrets

Secrets are used for sensitive information.

Examples:

```text
AWS_ACCESS_KEY
AWS_SECRET_KEY
DOCKER_PASSWORD
AZURE_CREDENTIALS
SONAR_TOKEN
```

Example:

```yaml
- name: Login to Docker
  run: docker login -u ${{ secrets.DOCKER_USERNAME }} \
       -p ${{ secrets.DOCKER_PASSWORD }}
```

Never put passwords directly into YAML.

---

# 11. Artifacts

Artifacts allow files generated by one job to be stored and downloaded later.

Example:

```text
Build Job
   │
   ├── application.jar
   └── test-results.xml
             │
             ▼
          Artifact
             │
             ▼
       Download later
```

Typical uses:

* Build packages
* Test reports
* Logs
* Coverage reports
* Deployment packages

---

# 12. Matrix Strategy

Matrix allows the same job to run against multiple configurations.

Example:

```yaml
strategy:
  matrix:
    python-version:
      - "3.10"
      - "3.11"
      - "3.12"
```

GitHub creates:

```text
Test Job
 ├── Python 3.10
 ├── Python 3.11
 └── Python 3.12
```

Very useful for compatibility testing.

---

# 13. Conditions – `if`

You can conditionally execute jobs or steps.

Example:

```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

Meaning:

```text
main branch?
     │
   YES ──→ Deploy
     │
    NO ──→ Skip
```

---

# 14. Environments

Environments represent deployment targets.

Example:

```text
Development
     ↓
QA
     ↓
Staging
     ↓
Production
```

Example:

```yaml
jobs:
  deploy:
    environment: production
```

Production environments can have:

* Approval requirements
* Environment-specific secrets
* Protection rules

---

# 15. Services

GitHub Actions can start supporting services for testing.

For example, your application might need MySQL:

```yaml
services:
  mysql:
    image: mysql:8
    env:
      MYSQL_ROOT_PASSWORD: password
```

Architecture:

```text
Runner
 ├── Application
 │
 └── MySQL Container
```

Useful for integration testing.

---

# Complete Example

Here's a simple **Docker CI/CD workflow** bringing the major components together:

```yaml
name: Docker CI/CD

on:
  push:
    branches:
      - main

  pull_request:

  workflow_dispatch:

env:
  IMAGE_NAME: myapp

jobs:

  build:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build \
            -t $IMAGE_NAME:${{ github.sha }} .

      - name: Run tests
        run: |
          echo "Running tests..."

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production

    steps:

      - name: Deploy application
        run: |
          echo "Deploying application..."
```

The complete flow is:

```text
                     GitHub Repository
                            │
                            ▼
                         EVENT
                     push / PR / manual
                            │
                            ▼
                        WORKFLOW
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
               BUILD                 TEST
              JOB                     JOB
                 │                     │
                 ▼                     ▼
              RUNNER                 RUNNER
                 │
              STEPS
                 │
        ┌────────┴─────────┐
        ▼                  ▼
      ACTIONS             RUN
   checkout@v4       docker build
        │                  │
        └────────┬─────────┘
                 ▼
              ARTIFACT
                 │
                 ▼
             DEPLOY JOB
                 │
                 ▼
            PRODUCTION
```

## The 10 components you should remember

| Component     | Purpose                     |
| ------------- | --------------------------- |
| **Workflow**  | Defines the automation      |
| **Event**     | Determines when it runs     |
| **Job**       | Group of operations         |
| **Runner**    | Machine executing the job   |
| **Step**      | Individual operation        |
| **Action**    | Reusable automation         |
| **Run**       | Execute shell commands      |
| **Variables** | Configuration values        |
| **Secrets**   | Sensitive values            |
| **Artifacts** | Files produced by workflows |

### Most important concept

If you're learning GitHub Actions for **DevOps/CI-CD**, remember this hierarchy:

```text
Repository
    ↓
Workflow (.github/workflows/*.yml)
    ↓
Event / Trigger
    ↓
Jobs
    ↓
Runner
    ↓
Steps
    ├── uses → Actions
    └── run  → Commands
```

That hierarchy is the **core mental model** for GitHub Actions.
