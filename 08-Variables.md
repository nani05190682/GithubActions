## Working with Variables in GitHub Actions

GitHub Actions provides several ways to define and use variables. For learning DevOps, it's useful to understand the difference between **workflow variables, environment variables, job/step variables, secrets, and outputs**.

### 1. Workflow-level variable

You can define variables using `env:` at the workflow level:

```yaml
name: Variables Demo

on:
  workflow_dispatch:

env:
  APP_NAME: myapp
  ENVIRONMENT: production

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Display variables
        run: |
          echo "Application: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
```

The variables are available to all jobs and steps.

---

## 2. Job-level variables

You can define variables only for a particular job:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    env:
      APP_NAME: myapp
      VERSION: 1.0

    steps:
      - name: Build
        run: |
          echo "Application: $APP_NAME"
          echo "Version: $VERSION"
```

These variables are available throughout the `build` job.

---

## 3. Step-level variables

You can define a variable for only one step:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:

      - name: Build
        env:
          APP_NAME: myapp
          VERSION: 1.0
        run: |
          echo "Building $APP_NAME"
          echo "Version $VERSION"

      - name: Test
        run: |
          echo "Application: $APP_NAME"
```

The second step **will not have access** to `APP_NAME` because it was defined only for the first step.

---

## 4. GitHub predefined variables

GitHub provides many built-in variables through the `github` context.

For example:

```yaml
- name: GitHub information
  run: |
    echo "Repository: ${{ github.repository }}"
    echo "Branch: ${{ github.ref_name }}"
    echo "Commit: ${{ github.sha }}"
    echo "Actor: ${{ github.actor }}"
```

Typical output:

```text
Repository: username/myrepo
Branch: main
Commit: 8f72a9...
Actor: username
```

Some useful GitHub contexts:

```text
github.repository
github.ref
github.ref_name
github.sha
github.actor
github.workflow
github.run_id
github.run_number
```

---

## 5. Using variables inside shell commands

For Linux runners:

```yaml
- name: Application information
  env:
    APP_NAME: myapp
    VERSION: 2.5

  run: |
    echo "Application = $APP_NAME"
    echo "Version = $VERSION"

    mkdir -p $APP_NAME
    echo "$VERSION" > $APP_NAME/version.txt
```

Notice:

```bash
$APP_NAME
$VERSION
```

These are **shell environment variables**.

---

# 6. GitHub Expressions

GitHub Actions also has its own expression syntax:

```yaml
${{ ... }}
```

For example:

```yaml
- name: Display GitHub variables
  run: |
    echo "Repository: ${{ github.repository }}"
    echo "Branch: ${{ github.ref_name }}"
```

So there are two syntaxes you'll frequently see:

```bash
$APP_NAME
```

for shell environment variables, and:

```yaml
${{ github.ref_name }}
```

for GitHub Actions expressions.

---

# 7. Creating a variable during a workflow

Suppose you calculate a version during one step and want to use it in the next step.

Use `$GITHUB_ENV`.

```yaml
name: Dynamic Variables

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:

      - name: Set version
        run: |
          VERSION="1.0.$GITHUB_RUN_NUMBER"
          echo "VERSION=$VERSION" >> "$GITHUB_ENV"

      - name: Display version
        run: |
          echo "Build version is $VERSION"
```

Output:

```text
Build version is 1.0.25
```

The important command is:

```bash
echo "VERSION=$VERSION" >> "$GITHUB_ENV"
```

This makes the variable available to **subsequent steps in the same job**.

---

# 8. Passing variables between jobs

`$GITHUB_ENV` does **not** automatically pass variables between jobs.

For that, use **job outputs**.

```yaml
name: Job Outputs

on:
  workflow_dispatch:

jobs:

  build:
    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:

      - name: Generate version
        id: version
        run: |
          VERSION="1.0.$GITHUB_RUN_NUMBER"
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"


  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:

      - name: Deploy
        run: |
          echo "Deploying version ${{ needs.build.outputs.version }}"
```

The flow is:

```text
             Build Job
                 │
                 │ version
                 ▼
          Job Output
                 │
                 ▼
            Deploy Job
```

---

# 9. GitHub Secrets

Don't put passwords directly into YAML.

Bad:

```yaml
env:
  PASSWORD: MyPassword123
```

Instead, create a GitHub repository secret and use:

```yaml
env:
  PASSWORD: ${{ secrets.PASSWORD }}
```

Example:

```yaml
- name: Login to Docker
  env:
    DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
    DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}

  run: |
    echo "$DOCKER_PASSWORD" | docker login \
      -u "$DOCKER_USERNAME" \
      --password-stdin
```

Secrets are appropriate for:

```text
Passwords
API tokens
Access keys
Private credentials
Certificates
```

---

# 10. Variables vs Secrets

GitHub also has **Actions Variables** for non-sensitive configuration.

For example:

```text
APP_NAME = myapp
REGISTRY = docker.io
ENVIRONMENT = production
```

You can reference them as:

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

A useful distinction:

| Type            | Example    | Syntax                               |
| --------------- | ---------- | ------------------------------------ |
| Shell variable  | `APP_NAME` | `$APP_NAME`                          |
| Workflow `env`  | `APP_NAME` | `$APP_NAME`                          |
| GitHub variable | `APP_NAME` | `${{ vars.APP_NAME }}`               |
| GitHub secret   | `PASSWORD` | `${{ secrets.PASSWORD }}`            |
| GitHub context  | Branch     | `${{ github.ref_name }}`             |
| Job output      | Version    | `${{ needs.build.outputs.version }}` |

---

# Complete Practical Example

Here's a small workflow combining several concepts:

```yaml
name: Variables Demo

on:
  workflow_dispatch:

env:
  APP_NAME: myapp
  ENVIRONMENT: production

jobs:

  build:
    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.version.outputs.image_tag }}

    steps:

      - name: Generate image tag
        id: version
        run: |
          IMAGE_TAG="v1.0.${GITHUB_RUN_NUMBER}"

          echo "IMAGE_TAG=$IMAGE_TAG" >> "$GITHUB_ENV"
          echo "image_tag=$IMAGE_TAG" >> "$GITHUB_OUTPUT"

      - name: Build
        run: |
          echo "Application: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "Image tag: $IMAGE_TAG"
          echo "Branch: ${{ github.ref_name }}"

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:

      - name: Deploy
        run: |
          echo "Deploying ${{ needs.build.outputs.image_tag }}"
          echo "Environment: $ENVIRONMENT"
```

The overall concept is:

```text
                     GitHub Actions
                           │
             ┌─────────────┼─────────────┐
             │             │             │
          env vars      GitHub vars    Secrets
             │             │             │
             ▼             ▼             ▼
          $APP_NAME    ${{ vars.X }}  ${{ secrets.X }}
                           │
                           ▼
                       Job Output
                           │
                           ▼
                    Another Job
```

