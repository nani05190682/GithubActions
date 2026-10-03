In GitHub Actions, **artifacts** are used to store files generated during a workflow so that they can be downloaded later or passed from one job to another.

A good example using your **cowsay/dragon** workflow is:

```text
Job 1: Generate file
        ↓
    dragon.txt
        ↓
   Upload Artifact
        ↓
Job 2: Download Artifact
        ↓
    View dragon.txt
```

### 1. Generate and upload an artifact

```yaml
name: Store Workflow Data

on:
  workflow_dispatch:

jobs:

  generate:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install cowsay
        run: |
          sudo apt-get update
          sudo apt-get install -y cowsay

      - name: Create dragon.txt
        run: |
          cowsay -f dragon "Hello from GitHub Actions!" > dragon.txt

      - name: Display file
        run: |
          cat dragon.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: dragon-output
          path: dragon.txt
```

The important part is:

```yaml
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: dragon-output
    path: dragon.txt
```

Here:

| Parameter | Meaning                  |
| --------- | ------------------------ |
| `name`    | Name of the artifact     |
| `path`    | File/directory to upload |

---

## 2. Download artifact in another job

Now let's have a second job download the file.

```yaml
name: Artifact Example

on:
  workflow_dispatch:

jobs:

  generate:
    runs-on: ubuntu-latest

    steps:

      - name: Install cowsay
        run: |
          sudo apt-get update
          sudo apt-get install -y cowsay

      - name: Create dragon.txt
        run: |
          cowsay -f dragon "Hello from Job 1!" > dragon.txt

      - name: Upload dragon.txt
        uses: actions/upload-artifact@v4
        with:
          name: dragon-output
          path: dragon.txt


  display:
    runs-on: ubuntu-latest
    needs: generate

    steps:

      - name: Download dragon artifact
        uses: actions/download-artifact@v5
        with:
          name: dragon-output

      - name: View dragon.txt
        run: |
          echo "===== Dragon ====="
          cat dragon.txt
          echo "=================="
```

### Execution

```text
             Job 1
        ┌───────────────┐
        │ Create file   │
        │ dragon.txt    │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │ Upload        │
        │ Artifact      │
        └───────┬───────┘
                │
                ▼
             Job 2
        ┌───────────────┐
        │ Download      │
        │ Artifact      │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │ cat           │
        │ dragon.txt    │
        └───────────────┘
```

### 3. Upload multiple files

You can upload an entire directory:

```yaml
- name: Upload build files
  uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: |
      build/
      logs/
      config/
```

Or specific files:

```yaml
- name: Upload files
  uses: actions/upload-artifact@v4
  with:
    name: application-data
    path: |
      app.tar.gz
      version.txt
      deployment.yaml
```

### 4. Download to a specific directory

```yaml
- name: Download artifact
  uses: actions/download-artifact@v5
  with:
    name: dragon-output
    path: ./output
```

Then:

```bash
cat ./output/dragon.txt
```

### Artifact vs Environment Variable

This distinction is important:

| Requirement                      | Use                |
| -------------------------------- | ------------------ |
| Pass a small value between steps | `$GITHUB_ENV`      |
| Pass a value between jobs        | Job outputs        |
| Pass files between jobs          | **Artifacts**      |
| Store build packages             | **Artifacts**      |
| Store logs/reports               | **Artifacts**      |
| Share Docker image               | Container registry |
| Store passwords/tokens           | **GitHub Secrets** |

For example:

```text
Job 1
 │
 ├── version.txt ────────► Artifact
 │
 ├── report.html ────────► Artifact
 │
 └── app.tar.gz ─────────► Artifact
                              │
                              ▼
                           Job 2
                              │
                         Download
                              │
                              ▼
                         Deploy
```

**One important point:** artifacts are primarily for **files**, whereas `$GITHUB_OUTPUT` is better when you need to pass a small value such as an image tag, version number, or deployment environment between jobs.
