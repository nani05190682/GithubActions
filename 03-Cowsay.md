 Here is a simple GitHub Actions workflow that installs `cowsay`, generates the **dragon** output, redirects it to `dragon.txt`, and then displays the file in the Actions log.

```yaml
name: Cowsay Dragon

on:
  workflow_dispatch:

jobs:
  dragon:
    runs-on: ubuntu-latest

    steps:
      - name: Install cowsay
        run: |
          sudo apt-get update
          sudo apt-get install -y cowsay

      - name: Create dragon.txt
        run: |
          cowsay -f dragon "Hello from GitHub Actions!" > dragon.txt

      - name: View dragon.txt
        run: |
          cat dragon.txt
```

### What happens

The important command is:

```bash
cowsay -f dragon "Hello from GitHub Actions!" > dragon.txt
```

Here:

* `cowsay` → runs the cowsay program
* `-f dragon` → selects the **dragon** character
* `"Hello from GitHub Actions!"` → message
* `>` → redirects the output
* `dragon.txt` → file that receives the output

The next step:

```bash
cat dragon.txt
```

displays the contents in the **GitHub Actions job log**.

### Example output

You'll see something similar to:

```text
 ____________________________
< Hello from GitHub Actions! >
 ----------------------------
                       \                    ^    /^
                        \                  / \  // \
                         \   |\___/|      /   \//  .\
                          \  /O  O  \__  /    //  |
                            /     /  \/_/    //   |
                            @___@`   \/_   //    |
                           0/0/|       \/_//     |
                       0/0/0/0/        \/_/      |
```

If you also want to **download `dragon.txt` from the GitHub Actions run**, add an artifact upload step:

```yaml
      - name: Upload dragon.txt
        uses: actions/upload-artifact@v4
        with:
          name: dragon-output
          path: dragon.txt
```

Then GitHub will make `dragon.txt` available as a downloadable **workflow artifact**.
