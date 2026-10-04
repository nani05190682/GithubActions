Absolutely. `include` and `exclude` become particularly useful when you have **multiple operating systems and different configurations** in a GitHub Actions matrix.

Let's build this from simple to realistic examples.

---

# 1. Basic Matrix with Multiple OS

Suppose you want to run tests on:

- Ubuntu
- Windows
- macOS

```yaml
name: OS Matrix

on:
  workflow_dispatch:

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
      - name: Display OS
        run: |
          echo "Running on ${{ matrix.os }}"
```

GitHub creates three jobs:

```text
              Test
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
    Ubuntu   Windows    macOS
```

---

# 2. Adding another matrix variable

Now suppose you want to test two Node.js versions:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest

    node:
      - 20
      - 22
```

GitHub creates:

```text
Ubuntu  + Node 20
Ubuntu  + Node 22

Windows + Node 20
Windows + Node 22

macOS   + Node 20
macOS   + Node 22
```

That's:

**3 OS × 2 Node versions = 6 jobs**

---

# 3. Why do we need `exclude`?

Suppose your application doesn't support:

```text
Windows + Node 20
```

You don't want GitHub to run that combination.

Use:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest

    node:
      - 20
      - 22

    exclude:
      - os: windows-latest
        node: 20
```

Now GitHub creates:

```text
Ubuntu  + Node 20
Ubuntu  + Node 22

Windows + Node 22

macOS   + Node 20
macOS   + Node 22
```

So:

```text
Before:
3 × 2 = 6 jobs

After exclude:
6 - 1 = 5 jobs
```

---

# 4. Multiple `exclude` entries

You can exclude multiple combinations.

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest

    node:
      - 18
      - 20
      - 22

    exclude:

      - os: windows-latest
        node: 18

      - os: macos-latest
        node: 18

      - os: windows-latest
        node: 20
```

The resulting combinations are:

```text
Ubuntu:
  Node 18
  Node 20
  Node 22

Windows:
  Node 22

macOS:
  Node 20
  Node 22
```

---

# 5. What exactly does `exclude` do?

Think of the matrix as a table:

| OS | Node |
|---|---:|
| Ubuntu | 18 |
| Ubuntu | 20 |
| Ubuntu | 22 |
| Windows | 18 |
| Windows | 20 |
| Windows | 22 |
| macOS | 18 |
| macOS | 20 |
| macOS | 22 |

If you say:

```yaml
exclude:
  - os: windows-latest
    node: 18
```

GitHub removes:

```text
Windows + Node 18
```

Result:

| OS | Node |
|---|---:|
| Ubuntu | 18 |
| Ubuntu | 20 |
| Ubuntu | 22 |
| Windows | 20 |
| Windows | 22 |
| macOS | 18 |
| macOS | 20 |
| macOS | 22 |

---

# 6. Now let's understand `include`

`include` is different.

It allows you to **add additional information or additional combinations** to your matrix.

Example:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest

    include:
      - os: ubuntu-latest
        shell: bash

      - os: windows-latest
        shell: pwsh

      - os: macos-latest
        shell: bash
```

Now each OS gets an associated shell:

```text
Ubuntu
 └── bash

Windows
 └── pwsh

macOS
 └── bash
```

Then:

```yaml
steps:
  - name: Run script
    shell: ${{ matrix.shell }}
    run: |
      echo "Running on ${{ matrix.os }}"
```

This is a very practical use of `include`.

---

# 7. `include` can add a completely new combination

Consider:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest

    node:
      - 20
      - 22

    include:
      - os: macos-latest
        node: 22
```

The original matrix gives:

```text
Ubuntu + Node 20
Ubuntu + Node 22
Windows + Node 20
Windows + Node 22
```

`include` adds:

```text
macOS + Node 22
```

Final result:

```text
Ubuntu  + Node 20
Ubuntu  + Node 22

Windows + Node 20
Windows + Node 22

macOS   + Node 22
```

---

# 8. `include` can add custom variables

This is one of the most useful features.

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest

    include:
      - os: ubuntu-latest
        package-manager: apt

      - os: windows-latest
        package-manager: choco

      - os: macos-latest
        package-manager: brew
```

Then:

```yaml
- name: Display package manager
  run: |
    echo "OS: ${{ matrix.os }}"
    echo "Package Manager: ${{ matrix.package-manager }}"
```

You can build OS-specific logic without creating three separate jobs.

---

# 9. Practical example — OS-specific installation

Suppose you want to install a package.

Linux:

```bash
sudo apt-get install cowsay
```

macOS:

```bash
brew install cowsay
```

Windows:

```powershell
choco install cowsay
```

You can use `include`:

```yaml
name: OS Package Test

on:
  workflow_dispatch:

jobs:

  test:
    strategy:
      matrix:
        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

        include:
          - os: ubuntu-latest
            package-manager: apt

          - os: windows-latest
            package-manager: choco

          - os: macos-latest
            package-manager: brew

    runs-on: ${{ matrix.os }}

    steps:

      - name: Show configuration
        run: |
          echo "OS: ${{ matrix.os }}"
          echo "Package Manager: ${{ matrix.package-manager }}"
```

Now you can use:

```yaml
${{ matrix.package-manager }}
```

inside your workflow.

---

# 10. Important difference: `include` vs `exclude`

### `exclude`

Means:

> **Don't run this combination.**

Example:

```yaml
exclude:
  - os: windows-latest
    node: 20
```

Result:

```text
Windows + Node 20 → ❌
```

---

### `include`

Means:

> **Add information to a combination or add a new combination.**

Example:

```yaml
include:
  - os: windows-latest
    node: 22
    shell: pwsh
```

Result:

```text
Windows + Node 22
        │
        └── shell = pwsh
```

---

# 11. Combining `include` and `exclude`

You can use both.

```yaml
name: Matrix Include Exclude

on:
  workflow_dispatch:

jobs:

  test:

    strategy:
      matrix:

        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

        node:
          - 20
          - 22

        exclude:

          - os: windows-latest
            node: 20

        include:

          - os: ubuntu-latest
            shell: bash

          - os: windows-latest
            shell: pwsh

          - os: macos-latest
            shell: bash

    runs-on: ${{ matrix.os }}

    steps:

      - name: Display configuration
        shell: ${{ matrix.shell }}
        run: |
          echo "OS: ${{ matrix.os }}"
          echo "Node: ${{ matrix.node }}"
```

Conceptually:

```text
                 Matrix
                    │
        ┌───────────┴───────────┐
        │                       │
     exclude                  include
        │                       │
   Remove invalid          Add/customize
   combinations            configurations
```

---

# 12. A more realistic OS matrix

Let's create a realistic cross-platform test:

```yaml
name: Cross Platform Testing

on:
  push:
    branches:
      - main

jobs:

  test:

    strategy:
      fail-fast: false

      matrix:

        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

        node:
          - 20
          - 22

        exclude:

          # Node 20 not supported on Windows
          - os: windows-latest
            node: 20

        include:

          # Custom shell for Windows
          - os: windows-latest
            node: 22
            shell: pwsh

          # Linux shell
          - os: ubuntu-latest
            node: 20
            shell: bash

          - os: ubuntu-latest
            node: 22
            shell: bash

          # macOS shell
          - os: macos-latest
            node: 20
            shell: bash

          - os: macos-latest
            node: 22
            shell: bash

    runs-on: ${{ matrix.os }}

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v6
        with:
          node-version: ${{ matrix.node }}

      - name: Install dependencies
        shell: ${{ matrix.shell }}
        run: npm install

      - name: Run tests
        shell: ${{ matrix.shell }}
        run: npm test
```

---

# 13. A cleaner way to use `include`

For many OS-specific configurations, you can make `include` define the entire test matrix.

For example:

```yaml
strategy:
  matrix:
    include:

      - os: ubuntu-latest
        node: 20
        shell: bash

      - os: ubuntu-latest
        node: 22
        shell: bash

      - os: windows-latest
        node: 22
        shell: pwsh

      - os: macos-latest
        node: 20
        shell: bash

      - os: macos-latest
        node: 22
        shell: bash
```

Then:

```yaml
runs-on: ${{ matrix.os }}
```

and:

```yaml
node-version: ${{ matrix.node }}
```

This produces exactly the combinations you want:

```text
┌─────────┬──────┬────────┐
│ OS      │ Node │ Shell  │
├─────────┼──────┼────────┤
│ Ubuntu  │ 20   │ bash   │
│ Ubuntu  │ 22   │ bash   │
│ Windows │ 22   │ pwsh   │
│ macOS   │ 20   │ bash   │
│ macOS   │ 22   │ bash   │
└─────────┴──────┴────────┘
```

This approach is often **easier to maintain when every OS has a different configuration**.

---

# 14. `include` vs separate jobs

Without matrix:

```yaml
jobs:

  ubuntu:
    runs-on: ubuntu-latest
    ...

  windows:
    runs-on: windows-latest
    ...

  macos:
    runs-on: macos-latest
    ...
```

With matrix:

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
```

The matrix approach is much cleaner when the **overall workflow is the same**.

---

# 15. When should you use `exclude`?

Use `exclude` when:

- One OS doesn't support a version
- A particular OS has a known failing combination
- A dependency doesn't work on one platform
- You don't want to waste runner time
- A specific combination isn't required

Example:

```yaml
exclude:
  - os: windows-latest
    python: '3.10'

  - os: macos-latest
    python: '3.10'
```

---

# 16. When should you use `include`?

Use `include` when:

- You need OS-specific variables
- You need different shells
- You need different package managers
- You need different environment variables
- You want to add a special combination
- You want to customize an existing combination

Example:

```yaml
include:
  - os: windows-latest
    shell: pwsh
    package_manager: choco

  - os: ubuntu-latest
    shell: bash
    package_manager: apt
```

---

# 17. Easy mental model

Think about a matrix like an Excel table.

Start with:

```text
OS              Version
------------------------
Ubuntu          20
Ubuntu          22
Windows         20
Windows         22
macOS           20
macOS           22
```

Then:

### `exclude`

**Delete rows:**

```text
Windows + 20  ❌
```

### `include`

**Add/customize rows:**

```text
Windows + 22 → shell=pwsh
Ubuntu + 20  → shell=bash
```

So remember:

> **Exclude removes combinations; Include adds or customizes combinations.**

---

## Complete example for teaching/demo

This is a good lab exercise:

```yaml
name: OS Matrix Demo

on:
  workflow_dispatch:

jobs:

  test:

    name: ${{ matrix.os }} - Node ${{ matrix.node }}

    strategy:
      fail-fast: false
      max-parallel: 2

      matrix:

        os:
          - ubuntu-latest
          - windows-latest
          - macos-latest

        node:
          - 20
          - 22

        exclude:

          # Don't test Node 20 on Windows
          - os: windows-latest
            node: 20

        include:

          - os: windows-latest
            node: 22
            shell: pwsh

          - os: ubuntu-latest
            node: 20
            shell: bash

          - os: ubuntu-latest
            node: 22
            shell: bash

          - os: macos-latest
            node: 20
            shell: bash

          - os: macos-latest
            node: 22
            shell: bash

    runs-on: ${{ matrix.os }}

    steps:

      - name: Display OS
        shell: ${{ matrix.shell }}
        run: |
          echo "Operating System: ${{ matrix.os }}"
          echo "Node Version: ${{ matrix.node }}"
          echo "Shell: ${{ matrix.shell }}"

      - name: Test
        shell: ${{ matrix.shell }}
        run: |
          echo "Running tests..."
          echo "Test completed!"
```

The resulting execution is:

```text
                    Matrix
                       │
             ┌─────────┴─────────┐
             │                   │
          EXCLUDE              INCLUDE
             │                   │
      Windows + Node 20    OS-specific settings
             │                   │
             ▼                   ▼
          Removed          shell / config
             │
             └─────────┬─────────┘
                       ▼
                  Final Jobs
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Ubuntu         Windows         macOS
     Node 20        Node 22         Node 20
     Node 22                        Node 22
```

This is the key concept to remember for interviews and real-world workflows:

**Matrix → generate combinations → `exclude` removes unwanted combinations → `include` adds/customizes configurations → GitHub executes the resulting jobs in parallel (subject to `max-parallel`).**
