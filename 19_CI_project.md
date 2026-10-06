Absolutely. For teaching **GitHub Actions `steps.id` + conditions**, a much better exercise is to use a **real Node.js application** rather than artificial `echo` commands.

# Real-Time Node.js CI/CD Lab

### Scenario

You are working on a Node.js REST API. Every time a developer pushes code, GitHub Actions should:

```text
Developer Push
      ↓
Install Dependencies
      ↓
Lint
      ↓
Unit Tests
      ↓
Build
      ↓
Docker Build
      ↓
Deploy
```

The pipeline should make decisions based on the results of previous steps.

---

## Lab Objective

Students will implement:

- Node.js CI
- `npm install`
- `npm test`
- `npm run lint`
- Step `id`
- Step outputs
- `steps.<id>.outcome`
- `if`
- `continue-on-error`
- Branch-based deployment
- Conditional Docker build

---

# 1. Create Node.js Application

Create a GitHub repository:

```text
nodejs-ci-demo
```

Project structure:

```text
nodejs-ci-demo/
├── package.json
├── server.js
├── test/
│   └── app.test.js
└── .github/
    └── workflows/
        └── nodejs-ci.yml
```

---

# 2. Create `package.json`

```json
{
  "name": "nodejs-ci-demo",
  "version": "1.0.0",
  "description": "GitHub Actions Node.js CI Demo",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "test": "node --test",
    "lint": "echo 'Running ESLint'",
    "build": "echo 'Building Node.js application'"
  },
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

---

# 3. Create Node.js Application

`server.js`

```javascript
const express = require("express");

const app = express();

const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
    res.json({
        message: "Node.js application is running",
        version: "1.0.0"
    });
});

app.get("/health", (req, res) => {
    res.status(200).json({
        status: "UP"
    });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

---

# 4. Create Test

`test/app.test.js`

```javascript
const test = require("node:test");
const assert = require("node:assert");

test("Application health check", () => {

    const response = {
        status: "UP"
    };

    assert.strictEqual(response.status, "UP");
});

test("Application name", () => {

    const application = "nodejs-ci-demo";

    assert.strictEqual(application, "nodejs-ci-demo");
});
```

Run locally:

```bash
npm install
npm test
```

Expected:

```text
✔ Application health check
✔ Application name

2 tests passed
```

---

# 5. Create GitHub Actions Workflow

Create:

```text
.github/workflows/nodejs-ci.yml
```

Start with:

```yaml
name: Node.js CI

on:
  push:
    branches:
      - main
      - develop

  pull_request:
    branches:
      - main
      - develop

jobs:

  ci:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Dependencies
        id: install
        run: npm ci

      - name: Run Tests
        id: tests
        run: npm test

      - name: Run Lint
        id: lint
        continue-on-error: true
        run: npm run lint

      - name: Build Application
        id: build
        run: npm run build

      - name: Display Results
        run: |
          echo "Install: ${{ steps.install.outcome }}"
          echo "Tests: ${{ steps.tests.outcome }}"
          echo "Lint: ${{ steps.lint.outcome }}"
          echo "Build: ${{ steps.build.outcome }}"
```

---

# 6. Important Student Exercise

Ask students to explain:

```yaml
id: install
```

```yaml
id: tests
```

```yaml
id: lint
```

```yaml
id: build
```

Then explain this:

```yaml
${{ steps.tests.outcome }}
```

For example:

```yaml
- name: Display Test Result
  run: |
    echo "Test result = ${{ steps.tests.outcome }}"
```

Possible result:

```text
Test result = success
```

or:

```text
Test result = failure
```

---

# 7. Real CI Condition

Now modify the workflow.

Add:

```yaml
- name: Deploy Application
  if: ${{ steps.tests.outcome == 'success' }}
  run: |
    echo "Deploying Node.js application..."
```

### Expected

If tests pass:

```text
Tests
  ↓
SUCCESS
  ↓
Deploy
```

If tests fail:

```text
Tests
  ↓
FAILURE
  ↓
Deploy SKIPPED
```

---

# 8. Add Branch-Based Deployment

Now make the exercise more realistic.

### DEV

Deploy when:

```text
develop + tests successful
```

### PROD

Deploy when:

```text
main + tests successful
```

Students should create:

```yaml
- name: Deploy to DEV
  if: ${{ github.ref_name == 'develop' && steps.tests.outcome == 'success' }}
  run: |
    echo "Deploying Node.js application to DEV"
```

and:

```yaml
- name: Deploy to PROD
  if: ${{ github.ref_name == 'main' && steps.tests.outcome == 'success' }}
  run: |
    echo "Deploying Node.js application to PRODUCTION"
```

---

# 9. Add Lint Condition

Currently:

```yaml
continue-on-error: true
```

means lint can fail without stopping the workflow.

Now create a warning:

```yaml
- name: Lint Warning
  if: ${{ steps.lint.outcome == 'failure' }}
  run: |
    echo "WARNING: ESLint checks failed"
    echo "Review the lint errors before merging."
```

Now students can see a real use of:

```text
id
 ↓
outcome
 ↓
if
```

---

# 10. Add Build Condition

The Docker build should happen only if:

```text
Tests = SUCCESS
AND
Build = SUCCESS
```

Students should implement:

```yaml
- name: Docker Build
  if: ${{ steps.tests.outcome == 'success' && steps.build.outcome == 'success' }}
  run: |
    docker build -t nodejs-ci-demo:${{ github.run_number }} .
```

---

# 11. Add Dockerfile

Create:

```text
Dockerfile
```

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Now the CI flow becomes:

```text
             Git Push
                │
                ▼
        ┌───────────────┐
        │ npm ci        │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ npm test      │
        └───────┬───────┘
                │
        ┌───────┴────────┐
        │                │
     SUCCESS           FAILURE
        │                │
        ▼                ▼
     npm build        STOP DEPLOY
        │
        ▼
   Docker Build
        │
        ▼
   Branch Condition
      /       \
 develop       main
    │            │
    ▼            ▼
   DEV          PROD
```

---

# 12. Advanced Exercise — Generate Docker Tag

Ask students to create an output.

```yaml
- name: Generate Docker Tag
  id: version
  run: |
    VERSION="1.0.${{ github.run_number }}"
    echo "version=$VERSION" >> "$GITHUB_OUTPUT"
```

Then use:

```yaml
- name: Display Version
  run: |
    echo "Application version: ${{ steps.version.outputs.version }}"
```

Then:

```yaml
- name: Docker Build
  if: ${{ steps.tests.outcome == 'success' && steps.build.outcome == 'success' }}
  run: |
    docker build \
      -t nodejs-ci-demo:${{ steps.version.outputs.version }} .
```

Now students are using both:

```text
steps.<id>.outcome
```

and:

```text
steps.<id>.outputs.<name>
```

---

# 13. Final Production-Style Challenge ⭐

Give students this requirement **without providing the solution**:

> Build a GitHub Actions CI pipeline for a Node.js application.

### Requirements

**Trigger:**

```text
push → main/develop
pull_request → main/develop
```

### Pipeline

```text
Checkout
   ↓
Setup Node 20
   ↓
npm ci
   ↓
npm test
   ↓
npm run lint
   ↓
npm run build
   ↓
Generate Version
   ↓
Docker Build
   ↓
Deployment
```

### Rules

| Condition | Action |
|---|---|
| `npm ci` fails | Stop |
| Tests fail | Stop deployment |
| Lint fails | Continue but show warning |
| Build fails | Stop |
| Tests + Build pass | Docker build |
| `develop` | Deploy DEV |
| `main` | Deploy PROD |
| Tests fail | No deployment |
| Lint fails | Pipeline can continue |

### Students must use

```yaml
id:
```

```yaml
if:
```

```yaml
continue-on-error:
```

```yaml
steps.<id>.outcome
```

```yaml
steps.<id>.outputs.<name>
```

```yaml
github.ref_name
```

```yaml
&&
```

---

## Expected Final Pipeline

```text
                    Git Push / PR
                         │
                         ▼
                    Checkout
                         │
                         ▼
                    Setup Node
                         │
                         ▼
                      npm ci
                         │
                         ▼
                     npm test
                         │
                 ┌───────┴───────┐
                 │               │
              SUCCESS          FAILURE
                 │               │
                 ▼               X
                Lint          STOP
                 │
          continue-on-error
                 │
          ┌──────┴──────┐
          │             │
       SUCCESS        FAILURE
          │             │
          │             ▼
          │          Warning
          │             │
          └──────┬──────┘
                 ▼
              npm build
                 │
                 ▼
          Generate Version
                 │
                 ▼
            Docker Build
                 │
          ┌──────┴──────┐
          │             │
       develop         main
          │             │
          ▼             ▼
       Deploy DEV    Deploy PROD
```

This gives students a **real Node.js CI problem** rather than just practicing `if` statements. It also closely resembles the kind of pipeline they would encounter in an actual DevOps/SRE environment.
