# Lab 06 — Final Lab: Securing a Complete Declarative Pipeline

| | |
|---|---|
| 🕘 **Session** | Day 2 · 14:45 – 16:30 (105 min) |
| 🌐☁️ **Where** | Jenkins (browser) + **MAIN** terminal |
| 🎯 **Objective** | Build — stage by stage — a Jenkins pipeline that checks out your code and runs **every security check from the workshop as a gate**, deploys only if all pass, then store it as a **`Jenkinsfile`** (pipeline as code) and **prove the gates work** by breaking them on purpose |
| 🏁 **You will have** | A green 8-stage pipeline deploying `workshop-app` to `http://<MAIN_SERVER_IP>:8081`, stored in your repo as `Jenkinsfile` |

```text
① Checkout → ② Gitleaks → ③ Build & Unit Tests → ④ SonarQube + Quality Gate
   → ⑤ OWASP Dependency-Check → ⑥ Docker Build → ⑦ Trivy Scan → ⑧ Deploy + Health Check
```

---

## 📋 Before you start

| Requirement | Check | Expected |
|---|---|---|
| Labs 03, 04, 05 fixes are pushed | 🌐 your fork → **Commits** | 3 new commits (Sonar fixes, secrets, commons-text) |
| Jenkins running | 🌐 `http://<MAIN_SERVER_IP>:8080` | Dashboard |
| SonarQube UP | 🌐 `http://<SONAR_SERVER_IP>:9000` | Login page / dashboard |
| Dependency-Check DB ready | ☁️ `tail -n 3 ~/dc-update.log` | `BUILD SUCCESS` |
| Hardened `Dockerfile` in repo | ☁️ `cd ~/workshop-app && git pull && head -5 Dockerfile` | `FROM eclipse-temurin:21-jre-alpine` |

---

## 🛠️ Part A — Prepare Jenkins (10 min)

### A1. Make sure the `jenkins` user can use every tool ☁️ MAIN

Pipelines run as **`jenkins`**, not `ubuntu`. Test each tool **as Jenkins** (`sudo -u jenkins <command>`):

```bash
sudo -u jenkins docker ps > /dev/null && echo "docker OK"
```

```bash
sudo -u jenkins gitleaks version
```

```bash
sudo -u jenkins trivy --version | head -1
```

```bash
sudo -u jenkins mvn -v | head -1
```

Give the Jenkins group write access to the Dependency-Check database (safety net for file permissions):

```bash
sudo chmod -R g+rwX /opt/dc-data
```

```bash
sudo -u jenkins ls /opt/dc-data
```

✅ **Expected:** `docker OK`, a Gitleaks version, a Trivy version, `Apache Maven 3.x`, and a file list (no "Permission denied").

> ❌ `docker: permission denied` → `sudo usermod -aG docker jenkins && sudo systemctl restart jenkins`.

### A2. Store the SonarQube token in Jenkins Credentials 🌐 Browser

Secrets must **never** be written inside a pipeline script. Jenkins has an encrypted **Credentials** store; pipelines
reference a secret by its **ID**, and Jenkins **masks** it (`****`) in build logs.

1. **Manage Jenkins → Credentials**.
2. Click **System** → **Global credentials (unrestricted)** → **Add Credentials**.
3. Fill in:

   | Field | Value |
   |---|---|
   | Kind | **Secret text** |
   | Scope | Global |
   | Secret | your SonarQube token (`squ_...`) |
   | ID | **`sonar-token`** (exactly this — the pipeline uses it) |
   | Description | `SonarQube analysis token` |

4. **Create**.

✅ **Check:** the list shows `sonar-token` — the secret itself is **not** displayed anywhere.

---

## 🛠️ Part B — Iteration 1: Checkout → Gitleaks → Build & Test (20 min) 🌐 Browser

### B1. Create the job

Dashboard → **New Item** → name **`workshop-app-pipeline`** → **Pipeline** → **OK**.

### B2. Paste the first version

Scroll to **Pipeline** → Definition **Pipeline script** → paste the script below.
✏️ **Replace `<YOUR_GITHUB_USERNAME>`** (and later `<SONAR_SERVER_IP>`) before saving.

```groovy
pipeline {
    agent any

    environment {
        APP_NAME     = 'workshop-app'
        GIT_REPO_URL = 'https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app.git'
    }

    options {
        timeout(time: 45, unit: 'MINUTES')            // kill a stuck build
        disableConcurrentBuilds()                     // one build at a time (shared ports & databases)
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('1. Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO_URL}"
            }
        }

        stage('2. Secrets Scan - Gitleaks') {
            steps {
                // Scans the full Git history. Exit code 1 (= leaks found) fails the build.
                sh 'gitleaks git --redact -v --report-format json --report-path gitleaks-report.json .'
            }
        }

        stage('3. Build & Unit Tests') {
            steps {
                sh 'mvn -B clean test'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
        }
    }
}
```

→ **Save** → **Build Now**.

> ⏳ The **first** Maven build as the `jenkins` user downloads all libraries again (its own `~/.m2`) — 3–5 minutes.

✅ **Expected:** build **#1** green. In **Console Output**:

```text
INF no leaks found
...
[INFO] Tests run: X, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
...
Finished: SUCCESS
```

> 💡 **Gitleaks is green** because of the `.gitleaksignore` you committed in Lab 04. Without it, this stage would fail
> on the historical AWS keys — exactly what you want from a secrets gate.

| Line | Explanation |
|---|---|
| `git branch: 'main', url: ...` | Clones your fork (full history — needed by Gitleaks) into the job **workspace** `/var/lib/jenkins/workspace/workshop-app-pipeline` |
| `sh '...'` (single quotes) | Passed to the Linux shell as-is |
| `archiveArtifacts` in `post { always }` | Keeps the report with the build **even when the build fails** — that's when you need it most |

---

## 🛠️ Part C — Iteration 2: add SonarQube SAST + Quality Gate (15 min) 🌐 Browser

**C1.** **Configure** the job → replace the whole script with this version (✏️ replace **both** placeholders):

```groovy
pipeline {
    agent any

    environment {
        APP_NAME       = 'workshop-app'
        GIT_REPO_URL   = 'https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app.git'
        SONAR_HOST_URL = 'http://<SONAR_SERVER_IP>:9000'
    }

    options {
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('1. Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO_URL}"
            }
        }

        stage('2. Secrets Scan - Gitleaks') {
            steps {
                sh 'gitleaks git --redact -v --report-format json --report-path gitleaks-report.json .'
            }
        }

        stage('3. Build & Unit Tests') {
            steps {
                sh 'mvn -B clean test'
            }
        }

        stage('4. SAST - SonarQube + Quality Gate') {
            steps {
                // Load the secret from the Jenkins credential store into the variable SONAR_TOKEN
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn -B verify sonar:sonar -Dsonar.projectKey=workshop-app -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_TOKEN -Dsonar.qualitygate.wait=true'
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
        }
    }
}
```

→ **Save** → **Build Now**.

✅ **Expected:** green. Console Output contains:

```text
+ mvn -B verify sonar:sonar ... -Dsonar.token=**** -Dsonar.qualitygate.wait=true
...
[INFO] QUALITY GATE STATUS: PASSED - View details on http://<SONAR_SERVER_IP>:9000/dashboard?id=workshop-app
```

> 🔐 **Look at `-Dsonar.token=****`** — Jenkins **masked** the secret in the log.

### ⚠️ Why single quotes around `sh '...'` matter (credential hygiene)

| ❌ Insecure | ✅ Secure |
|---|---|
| `sh "mvn ... -Dsonar.token=${SONAR_TOKEN}"` | `sh 'mvn ... -Dsonar.token=$SONAR_TOKEN'` |
| **Double quotes**: Groovy pastes the secret's **value** into the command text *before* the shell runs it — Jenkins warns *"A secret was passed to "sh" using Groovy String interpolation, which is insecure"* | **Single quotes**: the text `$SONAR_TOKEN` goes to the shell, which reads the **environment variable** at run time — the value never becomes part of the script text |

---

## 🛠️ Part D — Iteration 3: the complete secure pipeline (25 min) 🌐 Browser

**D1.** **Configure** → replace the script with the **final** version (✏️ both placeholders again):

```groovy
pipeline {
    agent any

    environment {
        APP_NAME       = 'workshop-app'
        IMAGE_TAG      = "${BUILD_NUMBER}"            // every build gets a unique image tag
        GIT_REPO_URL   = 'https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app.git'
        SONAR_HOST_URL = 'http://<SONAR_SERVER_IP>:9000'
    }

    options {
        timeout(time: 45, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('1. Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO_URL}"
            }
        }

        stage('2. Secrets Scan - Gitleaks') {
            steps {
                sh 'gitleaks git --redact -v --report-format json --report-path gitleaks-report.json .'
            }
        }

        stage('3. Build & Unit Tests') {
            steps {
                sh 'mvn -B clean test'
            }
        }

        stage('4. SAST - SonarQube + Quality Gate') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn -B verify sonar:sonar -Dsonar.projectKey=workshop-app -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_TOKEN -Dsonar.qualitygate.wait=true'
                }
            }
        }

        stage('5. SCA - OWASP Dependency-Check') {
            steps {
                // Uses the shared database in /opt/dc-data (Lab 05). Fails if any CVSS >= 9.
                sh 'mvn -B dependency-check:check -DautoUpdate=false'
            }
        }

        stage('6. Docker Build') {
            steps {
                sh 'docker build -t $APP_NAME:$IMAGE_TAG .'
            }
        }

        stage('7. Image Scan - Trivy') {
            steps {
                // Report (HIGH + CRITICAL) saved for the archive - never fails
                sh 'trivy image -q --severity HIGH,CRITICAL --exit-code 0 -o trivy-report.txt $APP_NAME:$IMAGE_TAG'
                // The GATE: fail on any fixable CRITICAL vulnerability
                sh 'trivy image -q --severity CRITICAL --ignore-unfixed --exit-code 1 $APP_NAME:$IMAGE_TAG'
            }
        }

        stage('8. Deploy + Health Check') {
            steps {
                sh 'docker rm -f $APP_NAME || true'
                sh 'docker run -d --name $APP_NAME --restart unless-stopped -p 8081:8081 $APP_NAME:$IMAGE_TAG'
                // Wait up to ~60 s for the app to report UP, otherwise fail the stage
                sh 'for i in $(seq 1 20); do curl -fs http://localhost:8081/actuator/health && exit 0; sleep 3; done; exit 1'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json, target/dependency-check-report.html, trivy-report.txt', allowEmptyArchive: true
        }
        success {
            echo "✅ All security gates passed - ${APP_NAME}:${IMAGE_TAG} deployed on port 8081"
        }
        failure {
            echo '❌ A security gate failed - deployment BLOCKED. Open the red stage to see why.'
        }
    }
}
```

→ **Save** → **Build Now**. ⏳ 4–8 minutes.

✅ **Expected:** all **8 stages green**. Console ends with:

```text
{"status":"UP"}
...
✅ All security gates passed - workshop-app:N deployed on port 8081
Finished: SUCCESS
```

### D2. Verify the deployment 🌐 + ☁️

| Check | How | Expected |
|---|---|---|
| App is up | 🌐 `http://<MAIN_SERVER_IP>:8081` | Course Registration Portal |
| Image tag = build number | ☁️ `docker ps --format '{{.Image}}  {{.Status}}'` | `workshop-app:N   Up … (healthy)` |
| Non-root | ☁️ `docker exec workshop-app whoami` | `app` |
| **SQL injection fixed** | 🌐 `http://<MAIN_SERVER_IP>:8081/api/students/search?name=' OR '1'='1` | `[]` — empty! (Day 1 it leaked every student) |
| Normal search still works | 🌐 `.../api/students/search?name=Asha` | one student |
| Text4Shell gone from image | Stage 7 log / `trivy-report.txt` | no `CVE-2022-42889` |

### D3. Read the evidence 🌐

- Job page → **Stage View**: one row per build, one column per stage, with timings.

<img src="../images/reference/jenkins-stage-view.png" alt="Jenkins Stage View with a passing and a failing build" width="900">

<sub>Source: reference repo vickydevo/DevSecOps-WS</sub>

- Build → **Artifacts** (or the build page's *Build Artifacts* list): download `dependency-check-report.html`,
  `trivy-report.txt`, `gitleaks-report.json` — the **audit evidence** a security team asks for.

---

## 🛠️ Part E — Pipeline as Code: move the script into a `Jenkinsfile` (15 min)

Right now the pipeline lives only inside Jenkins. In industry the pipeline is stored **in the repository** as a
`Jenkinsfile`: it's **versioned**, **reviewed in pull requests**, and can be restored if Jenkins is rebuilt.

**E1. 🌐** Job → **Configure** → click inside the script box → **Ctrl + A**, **Ctrl + C** (copy your working, final script).

**E2. ☁️ MAIN** — create the file in the project root:

```bash
cd ~/workshop-app
```

```bash
git pull
```

```bash
nano Jenkinsfile
```

Paste (**Shift + Insert** or right-click in Git Bash) → **Ctrl + O**, Enter → **Ctrl + X**.

✅ **Check:**

```bash
head -3 Jenkinsfile
```

```text
pipeline {
    agent any

```

```bash
grep -c "stage('" Jenkinsfile
```
→ `8`

**E3.** Commit and push:

```bash
git add Jenkinsfile
```

```bash
git commit -m "Add secure CI/CD pipeline as code (Jenkinsfile)"
```

```bash
git push
```

**E4. 🌐** Job → **Configure** → **Pipeline** section:

| Field | Value |
|---|---|
| Definition | **Pipeline script from SCM** |
| SCM | **Git** |
| Repository URL | `https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app.git` |
| Credentials | *- none -* (public repository) |
| Branch Specifier | `*/main` |
| Script Path | `Jenkinsfile` |

→ **Save** → **Build Now**.

✅ **Expected:** green build. The Console Output now starts with *"Obtained Jenkinsfile from git …"* — Jenkins
reads the pipeline **from your repository**.

> 💡 In a `Jenkinsfile`, many teams replace the `git branch: ..., url: ...` step with **`checkout scm`**, which
> reuses the repository configured in the job. Both work.

---

## 🛠️ Part F — Prove the gates work: break it on purpose (15 min)

A gate you've never seen fail is a gate you can't trust. Let's sneak a vulnerable library back in.

**F1. ☁️ MAIN** — downgrade `commons-text` to the vulnerable version and push:

```bash
cd ~/workshop-app
```

```bash
sed -i 's#<version>1.15.0</version>#<version>1.9</version>#' pom.xml
```

```bash
git commit -am "TEST: reintroduce vulnerable commons-text 1.9"
```

```bash
git push
```

**F2. 🌐** **Build Now**.

✅ **Expected — RED:**

| Stage | Result |
|---|---|
| 1 – 4 | ✅ green (code, secrets and tests are fine) |
| **5. OWASP Dependency-Check** | ❌ **FAILED** — `commons-text-1.9.jar: CVE-2022-42889(9.8)` |
| 6 – 8 | ⏭️ **skipped** — the vulnerable build was **never packaged, scanned or deployed** |

☁️ Prove that production wasn't touched — the running container is still the previous **safe** build:

```bash
docker ps --format '{{.Image}}  {{.Status}}'
```

**F3. ☁️ MAIN** — undo the bad commit the Git way (`git revert` creates a new commit that reverses it — history stays honest):

```bash
git revert --no-edit HEAD
```

```bash
git push
```

**F4. 🌐** **Build Now** → ✅ green again, new build number deployed.

### 🏅 Bonus (if time permits): bypass the hook, get caught by CI

```bash
echo "token=ghp_$(head -c 400 /dev/urandom | tr -dc 'A-Za-z0-9' | head -c 36)" > config-backup.txt
```

```bash
git add config-backup.txt
```

```bash
git commit --no-verify -m "TEST: commit a token, skipping the pre-commit hook"
```

```bash
git push
```

🌐 **Build Now** → ❌ fails at **stage 2 (Gitleaks)** — in seconds, before any build or scan time is wasted.
**Lesson:** developers can skip local hooks (`--no-verify`); the **server-side gate** can't be skipped.

Clean up (remove the file, then — as in Lab 04 — record the rotated secret's fingerprint):

```bash
git rm config-backup.txt
```

```bash
git commit -m "Remove leaked token (rotated)"
```

```bash
gitleaks git --redact --report-format json --report-path /tmp/new-leaks.json . ; jq -r '.[].Fingerprint' /tmp/new-leaks.json >> .gitleaksignore
```

```bash
git add .gitleaksignore && git commit -m "Accept rotated token fingerprint" && git push
```

🌐 **Build Now** → ✅ green.

---

## 🛡️ Security angle — what you built

| Stage | Practice | Gate condition | Evidence archived |
|---|---|---|---|
| ② Gitleaks | Secret detection | Any non-accepted secret in history | `gitleaks-report.json` |
| ③ Maven test | Unit testing | Any failing test | console |
| ④ SonarQube | **SAST** | Workshop Gate: Security & Reliability rating A | SonarQube dashboard |
| ⑤ Dependency-Check | **SCA** | Any library CVE with CVSS ≥ 9.0 | `dependency-check-report.html` |
| ⑦ Trivy | **Container scanning** | Any fixable CRITICAL in the image | `trivy-report.txt` |
| ⑧ Health check | Deployment verification | App not `UP` within ~60 s | console |
| Pipeline itself | Secure CI/CD | Token in Credentials store (masked), single-quoted `sh`, pipeline-as-code reviewed in Git | Git history |

### Where would you go next? (industry roadmap)

| Next step | Tool examples |
|---|---|
| Trigger builds automatically on every push / pull request | GitHub webhooks, multibranch pipelines |
| Sign images and verify signatures before deploy | Cosign / Sigstore |
| Generate an SBOM for every build | Syft, CycloneDX Maven plugin, `trivy image --format cyclonedx` |
| DAST against the deployed app | OWASP ZAP baseline scan |
| Infrastructure-as-Code scanning | `trivy config` on Terraform, Checkov |
| Kubernetes security | Network policies, Pod Security Standards, kube-bench |
| Secrets manager instead of env vars | HashiCorp Vault, AWS Secrets Manager |

---

## ❌ Common errors

| Stage / symptom | Why | Fix |
|---|---|---|
| ① `Couldn't find any revision to build` / `repository not found` | Wrong URL or placeholder not replaced | Check `GIT_REPO_URL` contains your username |
| ② `gitleaks: not found` | Not installed system-wide | Lab 04 Part A (`/usr/local/bin/gitleaks`) |
| ② `leaks found` | `.gitleaksignore` not pushed, or a new secret | Open `gitleaks-report.json` (Artifacts) / console; push `.gitleaksignore` |
| ③ `mvn: not found` | Maven missing for Jenkins | `sudo apt install -y maven` |
| ④ `Not authorized` / `401` | Wrong credential ID or token | Credential ID must be exactly `sonar-token`; re-create it |
| ④ `Connection refused` / timeout to :9000 | SonarQube down or IP changed | 🌐 check SonarQube; update `SONAR_HOST_URL` |
| ④ `QUALITY GATE STATUS: FAILED` | Lab 03 fixes not pushed | `git log --oneline` on MAIN; `git push` |
| ⑤ `Unable to ... /opt/dc-data` / `AccessDenied` | Permissions | `sudo chmod -R g+rwX /opt/dc-data`; check `ls -ld /opt/dc-data` shows group `jenkins` |
| ⑤ `NoDataException` | DB never finished downloading | Lab 05 Part A/C |
| ⑤ fails with commons-text | Lab 05 fix not pushed | `git push` the pom change |
| ⑥ `permission denied ... docker.sock` | `jenkins` not in docker group / not restarted | `sudo usermod -aG docker jenkins && sudo systemctl restart jenkins` |
| ⑥ `COPY failed: ... target/workshop-app.jar` | Jar missing — stage 4 didn't package | Make sure stage 4 uses `verify` (it builds the jar) |
| ⑦ Trivy fails with a CRITICAL **you didn't expect** | A new CVE was published in a base image or library | Read the finding; update the base image tag / library version — that's the gate doing its job |
| ⑧ `port is already allocated` | A container started by hand (Day 1) holds 8081 | `docker ps -a` → `docker rm -f <name>` |
| ⑧ health check fails | App crashed | `docker logs workshop-app` |
| Build stuck "Waiting for next available executor" | Another build running (`disableConcurrentBuilds`) | Wait, or abort the old build |
| `A secret was passed to "sh" using Groovy String interpolation` warning | Double quotes used with a secret | Use **single** quotes in that `sh` step |

---

## 🎤 Interview questions

<details>
<summary><b>1. Walk me through your DevSecOps pipeline.</b></summary>

Checkout → secret scan (Gitleaks) → build & unit tests → SAST with SonarQube and a Quality Gate → SCA with OWASP
Dependency-Check (fail on CVSS ≥ 9) → Docker build → image scan with Trivy (fail on fixable CRITICAL) → deploy with a
health check. Each security step is a gate; reports are archived; secrets come from the Jenkins credential store; the
pipeline is versioned as a Jenkinsfile.
</details>

<details>
<summary><b>2. Why order the stages like this?</b></summary>

Fail fast and cheap: secrets scanning takes seconds, so it runs first; tests before analysis; code and dependency
analysis before spending time building an image; image scanning before anything is deployed.
</details>

<details>
<summary><b>3. How do you handle secrets in Jenkins pipelines?</b></summary>

Store them in the Credentials store, bind with `withCredentials` only in the steps that need them, use single-quoted
shell strings so values aren't interpolated into the script, rely on log masking, and use short-lived, least-privilege
tokens.
</details>

<details>
<summary><b>4. A new CRITICAL CVE appears in the base image and blocks a production hotfix. What do you do?</b></summary>

Check whether a patched base image exists and rebuild; if not, assess exploitability, apply mitigations, and use a
documented, time-limited risk acceptance (e.g. `.trivyignore` with a justification and expiry) approved by security —
rather than disabling the gate.
</details>

<details>
<summary><b>5. Why store the pipeline as a Jenkinsfile?</b></summary>

Version control, peer review, auditability, reproducibility (rebuild Jenkins and restore pipelines from Git), and the
pipeline evolves together with the code it builds.
</details>

---

✅ **Finished?** Go to the [Day 2 Completion Checklist](Day-2-Completion-Checklist.md). 🎉
