# Timeout for Jobs and Tests in GitHub Actions

GitHub Actions provides the `timeout-minutes` property to prevent a **job or a step** from running indefinitely.

This is very important for CI/CD because a hanging build or test can consume runner time without producing any useful result.

---

## 1. Job-level timeout

The simplest example:

```yaml
name: Timeout Example

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: Build application
        run: |
          echo "Build started"
          sleep 5
          echo "Build completed"
```

Here:

```yaml
timeout-minutes: 10
```

means:

> The entire `build` job is allowed to run for a maximum of 10 minutes.

If the job exceeds 10 minutes, GitHub terminates it.

---

# 2. Why do we need job timeouts?

Imagine a test command:

```bash
./run-tests.sh
```

The test might accidentally hang:

```text
Running tests...
Running tests...
Running tests...
Running tests...
...
```

Without an appropriate timeout, the job could continue consuming runner resources.

With:

```yaml
timeout-minutes: 10
```

GitHub stops the job after 10 minutes.

```text
Job starts
    │
    ▼
Build
    │
    ▼
Tests
    │
    │
    │ 10 minutes
    ▼
TIMEOUT
    │
    ▼
Job failed/cancelled
```

---

# 3. Step-level timeout

You can also control the timeout of an individual step.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    steps:

      - name: Build
        run: |
          echo "Building..."
          sleep 5

      - name: Run tests
        timeout-minutes: 5
        run: |
          echo "Running tests..."
          ./run-tests.sh
```

Here:

```yaml
timeout-minutes: 5
```

applies **only to the test step**.

---

# 4. Job timeout vs Step timeout

This distinction is important.

### Job-level

```yaml
jobs:
  test:
    timeout-minutes: 30
```

Maximum duration of the **entire job**.

### Step-level

```yaml
steps:
  - name: Tests
    timeout-minutes: 10
```

Maximum duration of **that particular step**.

Conceptually:

```text
JOB: 30 minutes
│
├── Checkout
│
├── Build
│
├── Test ─────────── 10 minute limit
│
├── Package
│
└── Upload
```

---

# 5. Practical test example

Suppose you have:

```yaml
name: Application Tests

on:
  pull_request:

jobs:

  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          echo "Installing dependencies..."
          ./install.sh

      - name: Run unit tests
        timeout-minutes: 5
        run: |
          echo "Running unit tests..."
          ./run-tests.sh

      - name: Generate report
        run: |
          echo "Generating test report..."
          ./generate-report.sh
```

Here:

```text
Entire job
└── 15 minute maximum

Unit tests
└── 5 minute maximum
```

---

# 6. What happens when a timeout occurs?

Suppose:

```yaml
timeout-minutes: 5
```

and your test takes 8 minutes:

```text
0 min
 │
 ▼
Test starts
 │
 │
 │
5 min
 │
 ▼
TIMEOUT
 │
 ▼
Step terminated
 │
 ▼
Job fails
```

The workflow will not continue normally to subsequent steps.

---

# 7. Default timeout

GitHub Actions jobs have a default timeout of **360 minutes (6 hours)**.

That's why it's generally a good practice to explicitly configure a reasonable timeout for jobs that should finish much sooner.

For example:

```yaml
jobs:
  unit-test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
```

Rather than relying on the very large default.

---

# 8. Different timeouts for different jobs

In a real CI/CD pipeline, different jobs can have different limits:

```yaml
jobs:

  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 5

    steps:
      - run: ./lint.sh


  unit-test:
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - run: ./unit-tests.sh


  integration-test:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - run: ./integration-tests.sh


  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 20

    steps:
      - run: ./deploy.sh
```

This is much better than giving every job an unnecessarily large timeout.

---

# 9. Combining `needs` and timeout

You can combine job sequencing with timeouts:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - run: ./build.sh


  test:
    runs-on: ubuntu-latest
    needs: build
    timeout-minutes: 15

    steps:
      - run: ./test.sh


  deploy:
    runs-on: ubuntu-latest
    needs: test
    timeout-minutes: 20

    steps:
      - run: ./deploy.sh
```

Execution:

```text
Build
 │
 │ max 10 min
 ▼
Test
 │
 │ max 15 min
 ▼
Deploy
 │
 │ max 20 min
 ▼
Complete
```

---

# 10. Timeout + Concurrency

You can also combine the two concepts we just discussed:

```yaml
jobs:

  deploy:
    runs-on: ubuntu-latest

    timeout-minutes: 20

    concurrency:
      group: production
      cancel-in-progress: false

    steps:
      - name: Deploy
        run: |
          ./deploy.sh
```

Now you have two protections:

```text
             Production Deploy
                    │
          ┌─────────┴─────────┐
          │                   │
     Concurrency            Timeout
          │                   │
    One at a time          Max 20 min
```

So:

- **Concurrency** → prevents multiple deployments running simultaneously.
- **Timeout** → prevents one deployment from running forever.

---

# 11. Timeout for tests

For tests, I recommend putting the timeout directly on the test step when you want to isolate the test duration:

```yaml
- name: Run unit tests
  timeout-minutes: 10
  run: |
    pytest -v
```

For Maven:

```yaml
- name: Run Maven tests
  timeout-minutes: 15
  run: |
    mvn test
```

For npm:

```yaml
- name: Run npm tests
  timeout-minutes: 10
  run: |
    npm test
```

For a shell script:

```yaml
- name: Run integration tests
  timeout-minutes: 30
  run: |
    ./integration-tests.sh
```

---

# 12. Timeout vs test framework timeout

There can be **two different timeout mechanisms**.

### GitHub Actions timeout

```yaml
timeout-minutes: 10
```

Controls the GitHub Actions step/job.

### Test framework timeout

For example, a test framework may have its own timeout:

```text
GitHub Actions
     │
     │ 10 minutes
     ▼
Test framework
     │
     │ 30 seconds/test
     ▼
Individual tests
```

These solve different problems.

For example:

```yaml
- name: Run tests
  timeout-minutes: 15
  run: |
    pytest -v --timeout=30
```

Here:

- GitHub Actions → maximum 15 minutes for the test step
- pytest → maximum 30 seconds per test

---

# 13. Recommended CI/CD design

For a typical pipeline:

```yaml
name: CI/CD

on:
  push:
    branches:
      - main

jobs:

  build:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: ./build.sh


  test:
    runs-on: ubuntu-latest
    needs: build
    timeout-minutes: 20

    steps:
      - name: Unit tests
        timeout-minutes: 10
        run: ./unit-tests.sh

      - name: Integration tests
        timeout-minutes: 15
        run: ./integration-tests.sh


  deploy:
    runs-on: ubuntu-latest
    needs: test
    timeout-minutes: 20

    concurrency:
      group: production-deployment
      cancel-in-progress: false

    environment:
      name: production

    steps:
      - name: Deploy
        run: ./deploy.sh
```

The resulting pipeline is:

```text
                    GitHub Actions
                          │
                          ▼
                    ┌───────────┐
                    │   Build   │
                    │ 10 min    │
                    └─────┬─────┘
                          │
                         needs
                          ▼
                    ┌───────────┐
                    │   Test    │
                    │ 20 min    │
                    │           │
                    │ Unit      │
                    │ 10 min    │
                    │           │
                    │ Integration│
                    │ 15 min    │
                    └─────┬─────┘
                          │
                         needs
                          ▼
                  ┌─────────────────┐
                  │     Deploy      │
                  │     20 min      │
                  │                 │
                  │  Concurrency    │
                  │  production     │
                  └─────────────────┘
```

### Key points to remember

| Feature | Purpose |
|---|---|
| `timeout-minutes` | Limits execution time |
| Job-level timeout | Limits entire job |
| Step-level timeout | Limits individual step |
| `needs` | Controls job dependency |
| `concurrency` | Controls simultaneous runs |
| `environment` | Controls deployment environment/protection |

**Interview answer:**  
> `timeout-minutes` in GitHub Actions defines the maximum amount of time a job or individual step can execute. It is useful for preventing hanging builds, tests, or deployments from consuming runner resources indefinitely. It can be configured at the job level or step level, and different jobs can have different timeout values.
