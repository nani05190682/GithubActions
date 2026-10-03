In GitHub Actions, a workflow can contain **multiple jobs**, and each job runs independently unless you explicitly define a dependency using `needs`.

For your **cowsay example**, we can create 3 jobs:

1. **Install** → install cowsay
2. **Generate** → create `dragon.txt`
3. **Display** → display the dragon

However, there is an important concept: **files created in one job are not automatically available in another job**, because each job normally gets a fresh runner. We can use **artifacts** to pass files between jobs.

### Example: Multiple Jobs

```yaml
name: Cowsay Multi Job

on:
  workflow_dispatch:

jobs:

  # Job 1
  install:
    name: Install Cowsay
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install cowsay
        run: |
          chmod +x install-cowsay.sh
          ./install-cowsay.sh


  # Job 2
  generate:
    name: Generate Dragon
    runs-on: ubuntu-latest
    needs: install

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install cowsay
        run: |
          chmod +x install-cowsay.sh
          ./install-cowsay.sh

      - name: Generate dragon
        run: |
          chmod +x dragon.sh
          ./dragon.sh

      - name: Upload dragon.txt
        uses: actions/upload-artifact@v4
        with:
          name: dragon-file
          path: dragon.txt


  # Job 3
  display:
    name: Display Dragon
    runs-on: ubuntu-latest
    needs: generate

    steps:
      - name: Download dragon.txt
        uses: actions/download-artifact@v5
        with:
          name: dragon-file

      - name: Display dragon
        run: |
          echo "===== DRAGON ====="
          cat dragon.txt
          echo "=================="
```

### Job dependency

The important part is:

```yaml
needs: install
```

and:

```yaml
needs: generate
```

So the execution becomes:

```text
        ┌─────────────┐
        │   install   │
        └──────┬──────┘
               │
               ▼
        ┌─────────────┐
        │   generate  │
        └──────┬──────┘
               │
               ▼
        ┌─────────────┐
        │   display   │
        └─────────────┘
```

### Parallel jobs

If you don't specify `needs`, jobs can run **in parallel**:

```yaml
jobs:

  job1:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job 1"

  job2:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job 2"

  job3:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job 3"
```

The execution is approximately:

```text
              ┌── Job 1 ──┐
              │            │
Workflow ─────┼── Job 2 ──┼──► Complete
              │            │
              └── Job 3 ──┘
```

Whereas `needs` creates a dependency:

```text
Job 1
  │
  ▼
Job 2
  │
  ▼
Job 3
```

**Key GitHub Actions concepts to learn next:** `jobs`, `steps`, `needs`, `runs-on`, `if`, **artifacts**, **job outputs**, and **matrix strategy**.
