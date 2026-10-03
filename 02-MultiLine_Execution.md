In **GitHub Actions**, multiline execution usually means running multiple shell commands inside a single `run:` step.

### 1. Basic multiline `run`

```yaml
name: Multiline Example

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Run multiple commands
        run: |
          echo "Starting build"
          pwd
          ls -la
          echo "Installing dependencies"
          npm install
          echo "Running tests"
          npm test
```

The `|` is the important part. It allows multiple lines to be treated as one shell script.

---

### 2. Multiline with variables

```yaml
steps:
  - name: Build application
    run: |
      APP_NAME="myapp"
      VERSION="1.0.0"

      echo "Application: $APP_NAME"
      echo "Version: $VERSION"

      mkdir -p build
      echo "Build completed"
```

---

### 3. Multiline Docker commands

For example, if you're building a Docker image:

```yaml
- name: Build Docker image
  run: |
    docker build \
      --build-arg APP_ENV=production \
      --build-arg VERSION=1.0 \
      -t myapp:1.0 .
```

Here `\` continues the command onto the next line.

You can also use:

```yaml
- name: Docker operations
  run: |
    docker login -u "$DOCKER_USER" -p "$DOCKER_PASSWORD"
    docker build -t myapp:latest .
    docker push myapp:latest
```

---

### 4. Multiline shell script with conditions

```yaml
- name: Deploy
  run: |
    echo "Starting deployment"

    if [ "$ENVIRONMENT" = "production" ]; then
      echo "Deploying to production"
      ./deploy.sh production
    else
      echo "Deploying to development"
      ./deploy.sh development
    fi
```

You can also use GitHub Actions expressions:

```yaml
- name: Deployment
  run: |
    echo "Branch: ${{ github.ref_name }}"

    if [ "${{ github.ref_name }}" = "main" ]; then
      echo "Production deployment"
    fi
```

---

### 5. Multiline environment variable

GitHub Actions also supports multiline environment variables:

```yaml
- name: Set multiline variable
  run: |
    MESSAGE="Hello
    This is
    a multiline
    message"

    echo "$MESSAGE"
```

For **GitHub Actions output/environment files**, use the special `<<EOF` syntax:

```yaml
- name: Set multiline output
  id: example
  run: |
    {
      echo 'message<<EOF'
      echo 'Hello'
      echo 'This is a'
      echo 'multiline output'
      echo 'EOF'
    } >> "$GITHUB_OUTPUT"
```

Then:

```yaml
- name: Use output
  run: |
    echo "${{ steps.example.outputs.message }}"
```

---

### 6. Different shell

By default, Linux GitHub-hosted runners use Bash. You can explicitly specify:

```yaml
- name: Run Bash commands
  shell: bash
  run: |
    echo "Hello"
    uname -a
    whoami
```

PowerShell:

```yaml
- name: Run PowerShell
  shell: pwsh
  run: |
    Write-Host "Starting"
    Get-Location
    Get-ChildItem
```

Windows CMD:

```yaml
- name: Run CMD
  shell: cmd
  run: |
    echo Starting
    dir
    echo Finished
```

### Key point

There are **two different multiline concepts** you'll commonly see:

```yaml
run: |
  command1
  command2
  command3
```

means **execute several commands**.

Whereas:

```bash
docker build \
  --build-arg VERSION=1.0 \
  --build-arg ENV=prod \
  -t myapp .
```

means **continue one command across multiple lines**.

