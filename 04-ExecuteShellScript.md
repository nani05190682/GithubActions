**GitHub Actions calling an external shell script**.

### 1. Create `install-cowsay.sh`

```bash id="p3h2k7"
#!/bin/bash

echo "Installing cowsay..."

sudo apt-get update
sudo apt-get install -y cowsay

echo "Cowsay installation completed."
```

Make it executable:

```bash
chmod +x install-cowsay.sh
```

### 2. Create `dragon.sh`

```bash id="k7m4x1"
#!/bin/bash

echo "Generating dragon..."

cowsay -f dragon "Hello from GitHub Actions!" > dragon.txt

echo "Dragon file created."
cat dragon.txt
```

Make it executable:

```bash
chmod +x dragon.sh
```

### 3. GitHub Actions workflow

Create:

```text
.github/
└── workflows/
    └── dragon.yml
```

with:

```yaml id="w9n2qa"
name: Cowsay Dragon

on:
  workflow_dispatch:

jobs:
  dragon:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install cowsay
        run: |
          chmod +x install-cowsay.sh
          ./install-cowsay.sh

      - name: Generate Dragon
        run: |
          chmod +x dragon.sh
          ./dragon.sh

      - name: Upload dragon.txt
        uses: actions/upload-artifact@v4
        with:
          name: dragon-output
          path: dragon.txt
```

### Repository structure

Your GitHub repository will look like:

```text
cowsay-demo/
│
├── install-cowsay.sh
├── dragon.sh
│
└── .github/
    └── workflows/
        └── dragon.yml
```

### Execution flow

```text
GitHub Actions
      │
      ▼
checkout repository
      │
      ▼
install-cowsay.sh
      │
      ├── apt-get update
      └── apt-get install cowsay
      │
      ▼
dragon.sh
      │
      ├── cowsay -f dragon
      └── > dragon.txt
      │
      ▼
cat dragon.txt
      │
      ▼
Upload dragon.txt
      │
      ▼
GitHub Actions Artifact
```

