 **multiple GitHub Actions jobs sequentially**, use the `needs` keyword.

### Simple example

```yaml
name: Sequential Jobs

on:
  workflow_dispatch:

jobs:

  job1:
    name: Job 1 - Build
    runs-on: ubuntu-latest

    steps:
      - name: Build
        run: |
          echo "Job 1 started"
          sleep 5
          echo "Job 1 completed"


  job2:
    name: Job 2 - Test
    runs-on: ubuntu-latest
    needs: job1

    steps:
      - name: Test
        run: |
          echo "Job 2 started"
          sleep 5
          echo "Job 2 completed"


  job3:
    name: Job 3 - Deploy
    runs-on: ubuntu-latest
    needs: job2

    steps:
      - name: Deploy
        run: |
          echo "Job 3 started"
          sleep 5
          echo "Job 3 completed"
```

### Execution order

```text
┌──────────────┐
│    Job 1     │
│    Build     │
└──────┬───────┘
       │
       │ needs: job1
       ▼
┌──────────────┐
│    Job 2     │
│     Test     │
└──────┬───────┘
       │
       │ needs: job2
       ▼
┌──────────────┐
│    Job 3     │
│    Deploy    │
└──────────────┘
```

The important lines are:

```yaml
job2:
  needs: job1
```

and:

```yaml
job3:
  needs: job2
```

This creates:

**Job 1 → Job 2 → Job 3**

### Four jobs in sequence

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building application"


  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "Testing application"


  package:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - run: echo "Packaging application"


  deploy:
    runs-on: ubuntu-latest
    needs: package
    steps:
      - run: echo "Deploying application"
```

So the pipeline becomes:

```text
Build
  ↓
Test
  ↓
Package
  ↓
Deploy
```

**Important:** If `build` fails, `test`, `package`, and `deploy` will normally be skipped because their dependencies weren't successful.

For a real DevOps pipeline, this is the common pattern:

```text
Code Checkout
      ↓
Build
      ↓
Unit Test
      ↓
Docker Build
      ↓
Push Image
      ↓
Deploy to Kubernetes
```
