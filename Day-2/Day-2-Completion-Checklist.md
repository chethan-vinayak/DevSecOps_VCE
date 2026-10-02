# ✅ Day 2 — Completion Checklist

🎉 **You built a DevSecOps pipeline.** Use this list to confirm everything works and to clean up safely.

---

## 1. Verify your pipeline 🌐 Browser

- [ ] Jenkins job **`workshop-app-pipeline`** — latest build is **green**, all **8 stages** passed
- [ ] The job uses **Pipeline script from SCM** → `Jenkinsfile` in your fork
- [ ] At least one build is **red at stage 5 (Dependency-Check)** — proof the gate works (Lab 06 Part F)
- [ ] Build **Artifacts** contain `gitleaks-report.json`, `dependency-check-report.html`, `trivy-report.txt`
- [ ] SonarQube project **workshop-app** → Quality Gate **Passed** (Workshop Gate), Security **A**, Reliability **A**
- [ ] `http://<MAIN_SERVER_IP>:8081` shows the portal; `/api/students/search?name=' OR '1'='1` returns `[]`

## 2. Verify your repository 🌐 `https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app`

| File | Expected content |
|---|---|
| `Dockerfile` | `FROM eclipse-temurin:21-jre-alpine` · `USER app` · `HEALTHCHECK` |
| `Jenkinsfile` | 8 stages, `withCredentials([string(credentialsId: 'sonar-token' ...` |
| `pom.xml` | `commons-text` **1.15.0** |
| `src/main/resources/application.properties` | `${AWS_ACCESS_KEY_ID:}` — **no** real-looking keys |
| `.gitleaksignore` | fingerprints of rotated historical secrets |
| `Dockerfile-bad` | **deleted** |

## 3. Can you explain…? (interview-ready)

- [ ] SAST vs SCA vs DAST vs secret scanning — one sentence each
- [ ] What a **security gate** is and how Jenkins knows it failed (exit codes)
- [ ] Why the default *Sonar way* gate passed while a vulnerability existed
- [ ] The remediation order for a leaked secret (**rotate first!**)
- [ ] What Text4Shell is and why upgrading one line in `pom.xml` fixed both Dependency-Check **and** Trivy
- [ ] Why `sh 'mvn ... $SONAR_TOKEN'` uses **single** quotes
- [ ] Why the pipeline is stored as a `Jenkinsfile`

---

## 4. Clean up 🧹

### 4.1 Revoke the tokens you created for the workshop 🌐

| Service | Where | Action |
|---|---|---|
| SonarQube | My Account → Security | **Revoke** token `jenkins` |
| GitHub | Settings → Developer settings → Personal access tokens (classic) | **Delete** `devsecops-workshop` *(after the hackathon, if you have one)* |
| DockerHub | Account settings → Personal access tokens | **Delete** `devsecops-workshop` *(after the hackathon)* |

> 🛡️ Revoking credentials you no longer need is real-world hygiene — **unused credentials are pure risk**.

### 4.2 Stop or terminate the servers 🌐 EC2 → Instances

| Situation | `devsecops-main` | `sonarqube-server` |
|---|---|---|
| Hackathon / practice in the next few days | **Stop** | **Stop** |
| Workshop fully finished | **Terminate** | **Terminate** |

Select instance → **Instance state** → **Stop instance** / **Terminate (delete) instance**.

✅ **Check (if terminating):** also open **EC2 → Volumes** — no leftover `available` volumes; and **EC2 → Security Groups**
— you may delete `devsecops-main-sg` and `sonarqube-sg` once no instance uses them.

### 4.3 Watch your bill 🌐

**Billing and Cost Management → Bills** — check over the next days that charges stay at ₹0 / within credits. Your
**Zero-spend budget** (Prerequisite 01) will email you otherwise.

---

## 🧭 What you can put on your CV / LinkedIn

> *Built a DevSecOps CI/CD pipeline in Jenkins for a Java Spring Boot application with automated security gates:
> secret detection (Gitleaks), SAST with quality gates (SonarQube), SCA (OWASP Dependency-Check), container image
> scanning (Trivy), and hardened non-root Docker images on AWS EC2; remediated SQL injection, weak TLS, hard-coded
> credentials and a critical library vulnerability (CVE-2022-42889).*

Link your fork `https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app` — the `Jenkinsfile` and commit history are your proof of work.

---

⬅️ Back to the [workshop overview](../README.md)
