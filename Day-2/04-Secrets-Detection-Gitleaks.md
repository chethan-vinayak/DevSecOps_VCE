# Lab 04 — Secrets Detection & Remediation using Gitleaks

| | |
|---|---|
| 🕘 **Session** | Day 2 · 12:30 – 13:00 (30 min) |
| ☁️ **Where** | **MAIN** terminal (`devsecops-main`) |
| 🎯 **Objective** | Find hard-coded secrets in the **files** and the **Git history** of `workshop-app` with **Gitleaks**, remediate them the right way, document accepted history findings, and block future leaks with a **pre-commit hook** |
| 🏁 **You will have** | `gitleaks git .` → **no leaks found**, secrets replaced by environment variables, fixes pushed to GitHub |
| ⚠️ **Before lunch** | At 12:55 do [Lab 05 → Part A](05-OWASP-Dependency-Check.md) (2 minutes) so a download runs during lunch |

---

## 📚 Concept

### What counts as a "secret"?

Anything that grants access: passwords, API keys, cloud access keys (AWS `AKIA...`), tokens (GitHub `ghp_...`,
DockerHub `dckr_pat_...`, SonarQube `squ_...`), private keys (`-----BEGIN RSA PRIVATE KEY-----`), database
connection strings with passwords.

### How secrets leak — and why it's urgent

```text
Developer hard-codes an AWS key → git commit → git push (public repo) → 🤖 bots scan GitHub → key abused within minutes
                                     ▲
                       🛡️ Gitleaks pre-commit hook / CI gate stops it HERE
```

Remember **Day 1, Lab 02 Part F**: deleting the file later **does not** remove it from Git history — every clone
and fork still has it.

### How Gitleaks works

| Step | What Gitleaks does |
|---|---|
| 1 | Reads content: files in a folder (`gitleaks dir`) **or** every commit's changes in history (`gitleaks git`, using `git log -p`) |
| 2 | Applies **~200 rules** — regular expressions for known formats (AWS, GitHub, Slack, Stripe, private keys…) plus a generic rule |
| 3 | Uses **keywords** (fast pre-filter) and **entropy** (how random a string looks) to cut false positives |
| 4 | Reports each finding with file, line, commit, author, rule and a unique **Fingerprint** |
| 5 | Exits with code **1** if leaks were found (→ fails a pipeline), **0** if clean |

### The correct order of remediation

| # | Step | Why this order |
|---|---|---|
| 1 | **Revoke / rotate** the secret at its provider (AWS, GitHub…) | Once pushed, assume it's **stolen** — removing it from code doesn't stop an attacker who already copied it |
| 2 | **Remove** it from the code | Stop shipping it |
| 3 | **Store it properly** — environment variables, Jenkins Credentials, a vault/secret manager | The app still needs the value at runtime |
| 4 | **Prevent** recurrence — pre-commit hook + CI gate | Catch it before it's pushed next time |
| 5 | *(Optional)* **Rewrite history** (`git filter-repo`, BFG) | Cleans up, but can't recall copies already cloned — never a substitute for step 1 |

---

## 🧾 Syntax reference (Gitleaks v8.30)

| Command / flag | What it does |
|---|---|
| `gitleaks dir <path>` | Scan **current files** in a folder (no Git needed) |
| `gitleaks git <repo-path>` | Scan **every commit** in the Git history |
| `gitleaks git --pre-commit --staged` | Scan only what you're about to commit (used in hooks) |
| `-v` / `--verbose` | Print each finding in detail |
| `--redact` | Hide the secret values in the output (**always** use this in shared logs/CI) |
| `--report-format json --report-path <file>` | Save findings to a report file (also `csv`, `sarif`, `junit`) |
| `--exit-code <n>` | Exit code when leaks are found (default **1**) |
| `.gitleaksignore` | File of **fingerprints** to ignore (accepted/remediated findings) |

> ℹ️ Older tutorials use `gitleaks detect` / `gitleaks protect`. Those commands are **deprecated** since v8.19 —
> use `gitleaks git` / `gitleaks dir`.

---

## 🛠️ Lab

### Part A — Install Gitleaks and jq ☁️ MAIN

**A1.** Download the official release (Linux x64) into `/tmp`:

```bash
cd /tmp
```

```bash
wget -q https://github.com/gitleaks/gitleaks/releases/download/v8.30.1/gitleaks_8.30.1_linux_x64.tar.gz
```

**A2.** Extract just the `gitleaks` program and install it system-wide (so the `jenkins` user can use it too):

```bash
tar -xzf gitleaks_8.30.1_linux_x64.tar.gz gitleaks
```

```bash
sudo install -m 755 gitleaks /usr/local/bin/gitleaks
```

**A3.** Install `jq` (a JSON command-line tool we'll use to read the report):

```bash
sudo apt install -y jq
```

✅ **Check:**

```bash
gitleaks version
```

```text
8.30.1
```

```bash
cd ~/workshop-app
```

---

### Part B — Scan the current files ☁️ MAIN

**B1.** Remove build output first — `target/` contains *copies* of your config files and would show every finding twice:

```bash
mvn -q clean
```

**B2.** Scan the folder:

```bash
gitleaks dir -v --redact .
```

✅ **Expected (abridged):**

```text
Finding:     app.certificates.aws-access-key-id=REDACTED
Secret:      REDACTED
RuleID:      aws-access-token
Entropy:     4.12
File:        src/main/resources/application.properties
Line:        NN
Fingerprint: src/main/resources/application.properties:aws-access-token:NN

Finding:     ENV AWS_ACCESS_KEY_ID=REDACTED
RuleID:      aws-access-token
File:        Dockerfile-bad
...
WRN leaks found: N
```

```bash
echo "Exit code: $?"
```
→ `Exit code: 1`

| Field | Meaning |
|---|---|
| **RuleID** | Which rule matched (`aws-access-token`, `generic-api-key`, …) |
| **Entropy** | Randomness score — real keys are highly random |
| **File / Line** | Exactly where it is |
| **Fingerprint** | Unique ID of this finding (used to ignore it later) |

---

### Part C — Scan the Git history ☁️ MAIN

```bash
gitleaks git -v --redact .
```

✅ **Expected:** the same secrets — but now with **Commit**, **Author** and **Date**: *who* introduced the
secret and *when*. The fingerprint now has the form `commit:file:rule:line`.

> 🔎 That's what an attacker's bot sees in a public repo — and what your security team needs to decide **which keys to rotate**.

---

### Part D — Remediate ☁️ MAIN

**Step 1 — Rotate.** 🔐 In real life you would **now deactivate the AWS key** in the AWS IAM console and create a new one.
(The keys in this app are **fake** — there's nothing to rotate in the workshop, but never skip this step at work.)

**Step 2 — Remove the secrets from the code and read them from environment variables instead.**

Look at the current lines:

```bash
grep -n "aws-" src/main/resources/application.properties
```

```text
22:app.certificates.aws-access-key-id=AKIA_EXAMPLE_NOT_A_REAL_KEY
23:app.certificates.aws-secret-access-key=EXAMPLE_SECRET_NOT_A_REAL_KEY
```

Replace the **values** with Spring Boot placeholders. `${AWS_ACCESS_KEY_ID:}` means *"read environment variable
`AWS_ACCESS_KEY_ID`; if it's missing, use an empty value"*:

```bash
sed -i 's/^app.certificates.aws-access-key-id=.*/app.certificates.aws-access-key-id=${AWS_ACCESS_KEY_ID:}/' src/main/resources/application.properties
```

```bash
sed -i 's/^app.certificates.aws-secret-access-key=.*/app.certificates.aws-secret-access-key=${AWS_SECRET_ACCESS_KEY:}/' src/main/resources/application.properties
```

✅ **Check:**

```bash
grep -n "aws-" src/main/resources/application.properties
```

```text
22:app.certificates.aws-access-key-id=${AWS_ACCESS_KEY_ID:}
23:app.certificates.aws-secret-access-key=${AWS_SECRET_ACCESS_KEY:}
```

> 💡 At runtime the real values are injected by the platform — e.g. `docker run -e AWS_ACCESS_KEY_ID=...`, a Kubernetes
> Secret, AWS Secrets Manager, or Jenkins Credentials (Lab 06). **The code never contains them.**

**Step 3 — `Dockerfile-bad` has done its teaching job (Day 1) and contains the same keys. Delete it from the repository:**

```bash
git rm Dockerfile-bad
```

**Step 4 — Rescan the files:**

```bash
gitleaks dir -v --redact .
```

✅ **Expected:**

```text
INF no leaks found
```

**Step 5 — Make sure the app still builds and its tests pass:**

```bash
mvn -B -q test
```

✅ No errors (silence = success with `-q`).

**Step 6 — Commit:**

```bash
git add src/main/resources/application.properties
```

```bash
git commit -m "Remove hard-coded AWS credentials; read from environment; delete Dockerfile-bad"
```

---

### Part E — The history is still dirty ☁️ MAIN

```bash
gitleaks git --redact .
```

```text
WRN leaks found: N
```

🤯 The files are clean, but **old commits still contain the keys**. Options:

| Option | When |
|---|---|
| **A. Rotate the key and record the historical finding as accepted** (`.gitleaksignore`) | ✅ Standard practice once the key is **revoked** — the old value is useless |
| B. Rewrite history (`git filter-repo`) + force-push | When the value must disappear (e.g. personal data) — disruptive for everyone who cloned |

We use **Option A**, because Step 1 (rotation) made the old keys worthless.

**E1.** Save the history findings as a JSON report:

```bash
gitleaks git --redact --report-format json --report-path gitleaks-report.json .
```

**E2.** Look at the findings — one line each (rule, file, commit):

```bash
jq -r '.[] | "\(.RuleID)  \(.File)  \(.Commit[0:7])"' gitleaks-report.json
```

**E3.** Write their fingerprints into `.gitleaksignore`:

```bash
jq -r '.[].Fingerprint' gitleaks-report.json > .gitleaksignore
```

```bash
cat .gitleaksignore
```

```text
<commit-id>:src/main/resources/application.properties:aws-access-token:NN
<commit-id>:Dockerfile-bad:aws-access-token:NN
...
```

**E4.** Rescan:

```bash
gitleaks git -v --redact .
```

✅ **Expected:**

```text
INF no leaks found
```

```bash
echo "Exit code: $?"
```
→ `Exit code: 0`

> ⚠️ **`.gitleaksignore` is not a "make the scanner quiet" button.** Each line is a decision: *"this exact secret,
> in this exact commit, has been rotated."* A **new** secret gets a **new** fingerprint and is still caught. In
> companies, changes to this file are reviewed like code.

**E5.** Commit the ignore file (the JSON report is **not** committed — it's a build output):

```bash
git add .gitleaksignore
```

```bash
git commit -m "Accept rotated historical secrets in .gitleaksignore"
```

```bash
git push
```

---

### Part F — Prevention: a pre-commit hook ☁️ MAIN

A **Git hook** is a script Git runs automatically at certain moments. A **pre-commit** hook runs **before** each
commit — if it fails, the commit is **refused**. The secret never even reaches your local history.

**F1.** Create the hook:

```bash
printf '#!/bin/sh\ngitleaks git --pre-commit --staged --redact -v\n' > .git/hooks/pre-commit
```

```bash
chmod +x .git/hooks/pre-commit
```

```bash
cat .git/hooks/pre-commit
```

```text
#!/bin/sh
gitleaks git --pre-commit --staged --redact -v
```

**F2.** Test it with a **fake** GitHub token (generated randomly so it's unique):

```bash
echo "token=ghp_$(head -c 400 /dev/urandom | tr -dc 'A-Za-z0-9' | head -c 36)" > leak-test.txt
```

```bash
git add leak-test.txt
```

```bash
git commit -m "test leak"
```

✅ **Expected — the commit is BLOCKED:**

```text
Finding:     token=REDACTED
RuleID:      github-pat
File:        leak-test.txt
...
WRN leaks found: 1
```

```bash
git log --oneline -1
```
→ the last commit is still your `.gitleaksignore` commit — **"test leak" was never created**.

**F3.** Clean up the test file:

```bash
git restore --staged leak-test.txt
```

```bash
rm leak-test.txt
```

```bash
git status
```
→ `nothing to commit, working tree clean`

> 💡 Hooks live in `.git/hooks`, which is **not** pushed to GitHub — each developer must install them (teams use
> tools like the `pre-commit` framework to share them). That's why we **also** run Gitleaks in the CI pipeline (Lab 06):
> the server-side gate catches what a missing hook lets through.

---

## 🛡️ Security angle — summary

| Layer | Control | Catches |
|---|---|---|
| Developer laptop | Pre-commit hook | Secrets **before** they enter history |
| CI pipeline | Gitleaks stage with exit code 1 | Anything that bypassed hooks |
| Platform | GitHub secret scanning / push protection | Known token formats on push |
| Response | Rotate → remove → store properly → prevent | Limits the damage of a leak |

---

## ❌ Common errors

| You see | Why | Fix |
|---|---|---|
| `gitleaks: command not found` | Install step skipped / failed | Repeat Part A; `ls -l /usr/local/bin/gitleaks` |
| `tar: gitleaks: Not found in archive` | Download incomplete | `rm -f /tmp/gitleaks_*.tar.gz` and repeat A1 |
| Same finding listed twice in `dir` scan | `target/` contains copies | `mvn -q clean` and scan again |
| `jq: error ... Cannot iterate over null` | Report empty or missing (no leaks) | `cat gitleaks-report.json` — `[]` means nothing was found |
| `gitleaks git` still finds leaks after `.gitleaksignore` | A **new** leak (different fingerprint), or the file isn't in the repo root | Read the finding; ensure `.gitleaksignore` is in `~/workshop-app` |
| `fatal: pathspec 'Dockerfile-bad' did not match` | Already deleted | Continue |
| Hook doesn't block the test commit | Hook not executable / wrong path | `chmod +x .git/hooks/pre-commit`; run it manually: `.git/hooks/pre-commit` |

---

## 🏋️ Practice tasks

1. Run `gitleaks git --redact --report-format sarif --report-path gitleaks.sarif .` — SARIF is the format GitHub's
   *Security* tab understands.
2. Open `.gitignore` — `gitleaks-report.json` is already listed. Add `*.sarif` so SARIF reports are never committed either.

---

## 🎤 Interview questions

<details>
<summary><b>1. A developer pushed an AWS key 10 minutes ago and deleted it 2 minutes later. What do you do first?</b></summary>

Revoke/rotate the key immediately at AWS and check CloudTrail/billing for misuse. Then remove it from code, move
it to a secret store, add prevention (hooks/CI scanning) and decide whether to rewrite history.
</details>

<details>
<summary><b>2. <code>gitleaks dir</code> vs <code>gitleaks git</code>?</b></summary>

`dir` scans the current files on disk; `git` scans the patches of every commit in the history (secrets that were
added and later removed are still found).
</details>

<details>
<summary><b>3. What is entropy in secret scanning?</b></summary>

A measure of randomness of a string. Real keys/tokens have high entropy; requiring a minimum entropy reduces
false positives like `password=changeme`.
</details>

<details>
<summary><b>4. Why use both pre-commit hooks and CI scanning?</b></summary>

Hooks give the fastest feedback and keep secrets out of local history, but they're optional per developer and can
be bypassed (`--no-verify`). CI scanning is centrally enforced.
</details>

---

⚠️ **12:55 — Before lunch:** go to [Lab 05 → Part A](05-OWASP-Dependency-Check.md) and start the database download.
