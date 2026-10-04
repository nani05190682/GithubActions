# Matrix Strategy in GitHub Actions

The **matrix strategy** is one of the most useful features in GitHub Actions for CI/CD.

It allows you to run the **same job multiple times with different combinations of variables**.

Instead of writing:

```text
Job 1 → Node 18
Job 2 → Node 20
Job 3 → Node 22
```

you can define one job and let GitHub Actions create all three automatically.

---

# 1. The basic idea

Suppose you want to test your application on:

- Ubuntu
- Windows
- macOS

Without matrix:

```yaml
jobs:

  ubuntu:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing Ubuntu"

  windows:
    runs-on: windows-latest
    steps:
      - run: echo "Testing Windows"

  macos:
    runs-on: macos-latest
    steps:
      - run: echo "Testing macOS"
```

This becomes repetitive.

With a matrix:

```yaml
jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

    runs-on: ${{ matrix.os }}

    steps:
      - name: Test
        run: |
          echo "Testing on ${{ matrix.os }}"
```

GitHub automatically creates **three job executions**.

```text
                 test job
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       Ubuntu    Windows     macOS
```

---

# 2. Understanding `matrix`

The important section is:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest
```

GitHub creates a matrix variable:

```text
matrix.os
```

You access it with:

```yaml
${{ matrix.os }}
```

And use it here:

```yaml
runs-on: ${{ matrix.os }}
```

---

# 3. Multiple matrix variables

This is where matrix strategy becomes really powerful.

Suppose you want to test:

```text
Operating Systems:
Ubuntu
Windows

Versions:
Node 18
Node 20
Node 22
```

You can write:

```yaml
name: Matrix Example

on:
  workflow_dispatch:

jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest

        node:
          - 18
          - 20
          - 22

    runs-on: ${{ matrix.os }}

    steps:

      - name: Display configuration
        run: |
          echo "OS: ${{ matrix.os }}"
          echo "Node: ${{ matrix.node }}"
```

GitHub creates:

```text
Ubuntu + Node 18
Ubuntu + Node 20
Ubuntu + Node 22

Windows + Node 18
Windows + Node 20
Windows + Node 22
```

Total:

**2 × 3 = 6 jobs**

---

# 4. Matrix combinations

Think of a matrix like this:

| OS | Node |
|---|---|
| Ubuntu | 18 |
| Ubuntu | 20 |
| Ubuntu | 22 |
| Windows | 18 |
| Windows | 20 |
| Windows | 22 |

GitHub automatically generates all combinations.

```text
                  Node
             18     20     22
          ┌──────┬──────┬──────┐
Ubuntu    │ Job1 │ Job2 │ Job3 │
          ├──────┼──────┼──────┤
Windows   │ Job4 │ Job5 │ Job6 │
          └──────┴──────┴──────┘
```

This is called the **Cartesian product** of the matrix values.

---

# 5. Real example — Python versions

Suppose your application supports Python:

```text
3.10
3.11
3.12
3.13
```

Instead of creating four jobs:

```yaml
jobs:

  test-python:
    strategy:
      matrix:
        python-version:
          - '3.10'
          - '3.11'
          - '3.12'
          - '3.13'

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v6
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Run tests
        run: |
          pytest
```

GitHub creates:

```text
Python 3.10 → Test
Python 3.11 → Test
Python 3.12 → Test
Python 3.13 → Test
```

---

# 6. Real example — Node.js

```yaml
name: Node Matrix

on:
  push:

jobs:

  test:
    strategy:
      matrix:
        node-version:
          - 18
          - 20
          - 22

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v6
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

Execution:

```text
             Test
               │
       ┌───────┼────────┐
       │       │        │
       ▼       ▼        ▼
    Node 18  Node 20  Node 22
       │       │        │
       ▼       ▼        ▼
     Tests   Tests    Tests
```

---

# 7. Matrix with operating system + version

This is a common production CI pattern:

```yaml
jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest

        node:
          - 20
          - 22

    runs-on: ${{ matrix.os }}

    steps:

      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v6
        with:
          node-version: ${{ matrix.node }}

      - name: Test
        run: npm test
```

GitHub generates:

```text
Ubuntu + Node 20
Ubuntu + Node 22
Windows + Node 20
Windows + Node 22
```

Total:

**4 jobs**

---

# 8. Matrix with Docker

This is especially useful for DevOps.

Suppose you want to build images for different versions:

```yaml
name: Docker Matrix

on:
  workflow_dispatch:

jobs:

  build:
    strategy:
      matrix:
        version:
          - "1.0"
          - "2.0"
          - "3.0"

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build \
            --build-arg VERSION=${{ matrix.version }} \
            -t myapp:${{ matrix.version }} .

      - name: Display image
        run: |
          docker images
```

You get:

```text
myapp:1.0
myapp:2.0
myapp:3.0
```

---

# 9. Matrix with environment

Suppose you want:

```text
dev
qa
production
```

You can define:

```yaml
jobs:

  deploy:
    strategy:
      matrix:
        environment:
          - dev
          - qa
          - production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          echo "Deploying to ${{ matrix.environment }}"
```

GitHub creates:

```text
Deploy → dev
Deploy → qa
Deploy → production
```

### But be careful

You normally **don't want production deployment automatically running in parallel with dev/QA** just because you used a matrix.

For deployment pipelines, environments and controlled sequencing are usually better.

---

# 10. Matrix with `include`

`include` allows you to add extra information to specific matrix combinations.

Example:

```yaml
jobs:

  deploy:
    strategy:
      matrix:
        environment:
          - dev
          - qa
          - production

        include:
          - environment: dev
            region: ap-south-1

          - environment: qa
            region: ap-south-1

          - environment: production
            region: ap-south-1
```

Then:

```yaml
- name: Deploy
  run: |
    echo "Environment: ${{ matrix.environment }}"
    echo "Region: ${{ matrix.region }}"
```

---

# 11. A better `include` example

Consider:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest

    include:
      - os: ubuntu-latest
        shell: bash

      - os: windows-latest
        shell: pwsh
```

Then:

```yaml
- name: Run script
  shell: ${{ matrix.shell }}
  run: |
    echo "Running on ${{ matrix.os }}"
```

Now the additional `shell` information is associated with each OS.

---

# 12. `exclude`

Sometimes you **don't want every combination**.

Suppose you have:

```yaml
matrix:
  os:
    - ubuntu-latest
    - windows-latest

  node:
    - 18
    - 20
    - 22
```

Normally:

```text
2 × 3 = 6 jobs
```

But suppose Node 18 isn't supported on Windows.

You can exclude that combination:

```yaml
strategy:
  matrix:

    os:
      - ubuntu-latest
      - windows-latest

    node:
      - 18
      - 20
      - 22

    exclude:
      - os: windows-latest
        node: 18
```

Now:

```text
Ubuntu + Node 18
Ubuntu + Node 20
Ubuntu + Node 22

Windows + Node 20
Windows + Node 22
```

Total:

**5 jobs**

---

# 13. `include` vs `exclude`

### `exclude`

Remove a combination:

```yaml
exclude:
  - os: windows-latest
    node: 18
```

### `include`

Add or customize combinations:

```yaml
include:
  - os: ubuntu-latest
    node: 22
    environment: production
```

Think:

```text
matrix
  │
  ├── include → Add/customize
  │
  └── exclude → Remove
```

---

# 14. `fail-fast`

By default, GitHub Actions matrix jobs use:

```yaml
fail-fast: true
```

Suppose you have:

```text
Job 1 → PASS
Job 2 → PASS
Job 3 → FAIL
Job 4 → Running
Job 5 → Running
```

With `fail-fast: true`, GitHub can cancel in-progress matrix jobs when one fails.

You can disable this:

```yaml
strategy:
  fail-fast: false

  matrix:
    node:
      - 18
      - 20
      - 22
```

Now even if Node 18 fails:

```text
Node 18 → FAIL
Node 20 → Continue
Node 22 → Continue
```

This is useful when you want to see **all test results**, rather than stopping early.

---

# 15. `max-parallel`

Matrix jobs normally run concurrently depending on available runners.

You can limit the number running simultaneously:

```yaml
strategy:
  max-parallel: 2

  matrix:
    node:
      - 18
      - 20
      - 22
      - 24
```

Instead of:

```text
Node 18 ──────────┐
Node 20 ──────────┤
Node 22 ──────────┤
Node 24 ──────────┘
       All at once
```

GitHub limits it to two at a time:

```text
Node 18 ──────┐
Node 20 ──────┤
              │
              ▼
          Node 22 ──────┐
          Node 24 ──────┘
```

---

# 16. Matrix + `needs`

Matrix jobs can also be part of a sequential pipeline.

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Building application"


  test:
    needs: build

    strategy:
      matrix:
        node:
          - 18
          - 20
          - 22

    runs-on: ubuntu-latest

    steps:
      - name: Test
        run: |
          echo "Testing Node ${{ matrix.node }}"


  deploy:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - run: echo "Deploying application"
```

Execution:

```text
                 Build
                   │
                   ▼
              ┌─────────┐
              │ Matrix  │
              │ Testing │
              └────┬────┘
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
      Node18     Node20     Node22
        │          │          │
        └──────────┼──────────┘
                   │
                   ▼
                Deploy
```

This is a very useful pattern.

**Build → Test against multiple versions → Deploy only after all matrix jobs succeed.**

---

# 17. Matrix + Docker + Registry

Here's a more realistic DevOps example.

Suppose you want to build multiple application versions:

```yaml
name: Multi-Version Docker Build

on:
  workflow_dispatch:

jobs:

  build:
    strategy:
      matrix:
        version:
          - "1.0"
          - "2.0"
          - "3.0"

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build \
            --build-arg APP_VERSION=${{ matrix.version }} \
            -t myapp:${{ matrix.version }} .

      - name: Show image
        run: |
          docker images myapp
```

The same job definition is executed three times.

---

# 18. Matrix strategy vs multiple jobs

Without matrix:

```yaml
jobs:

  node18:
    ...

  node20:
    ...

  node22:
    ...
```

You maintain three separate jobs.

With matrix:

```yaml
jobs:

  test:
    strategy:
      matrix:
        node:
          - 18
          - 20
          - 22
```

One job definition.

### Matrix is useful when:

The **same steps** need to run with different parameters.

```text
Same process
     │
     ├── Version A
     ├── Version B
     ├── Version C
     └── Version D
```

---

# 19. Matrix is NOT always the right solution

Don't use matrix when the jobs have completely different workflows.

For example:

```text
Build
   ↓
Security Scan
   ↓
Deploy
```

These should normally be separate jobs.

But:

```text
Test
 ├── Node 18
 ├── Node 20
 └── Node 22
```

is a perfect matrix use case.

---

# 20. Complete production-style example

Here's a good example combining several concepts:

```yaml
name: Application CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:

  build:
    name: Build Application
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: |
          echo "Building application..."
          ./build.sh


  test:
    name: Test Node ${{ matrix.node }}
    needs: build

    strategy:
      fail-fast: false
      max-parallel: 2

      matrix:
        node:
          - 18
          - 20
          - 22

    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v6
        with:
          node-version: ${{ matrix.node }}

      - name: Install dependencies
        run: |
          npm install

      - name: Run tests
        run: |
          echo "Testing Node ${{ matrix.node }}"
          npm test


  deploy:
    name: Deploy
    needs: test

    runs-on: ubuntu-latest

    concurrency:
      group: production
      cancel-in-progress: false

    environment:
      name: production

    steps:

      - name: Deploy
        run: |
          echo "All matrix tests passed"
          echo "Deploying application..."
```

The complete flow is:

```text
                         BUILD
                           │
                           ▼
                    ┌─────────────┐
                    │    TEST     │
                    │   MATRIX    │
                    └──────┬──────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Node 18       Node 20       Node 22
             │             │             │
             └─────────────┼─────────────┘
                           │
                    All tests pass
                           │
                           ▼
                       DEPLOY
                           │
                    concurrency:
                     production
                           │
                           ▼
                      Production
```

## The most important matrix keywords

| Keyword | Purpose |
|---|---|
| `matrix` | Defines the variables/combinations |
| `matrix.os` | Access matrix value |
| `matrix.node` | Access another matrix value |
| `include` | Add/customize combinations |
| `exclude` | Remove combinations |
| `fail-fast` | Stop/cancel other matrix jobs after failure |
| `max-parallel` | Limit simultaneous matrix jobs |
| `needs` | Make matrix job depend on another job |

### Easy way to remember

```text
Matrix
  │
  ├── What should vary?
  │       │
  │       ├── OS
  │       ├── Language version
  │       ├── Database
  │       └── Application version
  │
  ├── include → Add/customize
  │
  ├── exclude → Remove
  │
  ├── fail-fast → Stop on failure?
  │
  └── max-parallel → How many simultaneously?
```

**Interview definition:**  
> **A matrix strategy in GitHub Actions allows a single job definition to run multiple times with different combinations of configuration values. It is commonly used for testing applications across multiple operating systems, language versions, runtimes, or dependency versions. `include` and `exclude` customize the combinations, while `max-parallel` controls concurrency and `fail-fast` controls how matrix failures affect other jobs.**
