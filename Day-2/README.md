# Day 2 — Pipeline Security & Code Analysis

> **Theme of the day:** automate everything you did by hand yesterday — and add three more security checks — so
> that **insecure code can never reach production without being stopped.**

## 📅 Timetable

| # | Time | Session | Type | Lab file |
|---|---|---|---|---|
| 1 | 09:30 – 10:30 | Secure CI/CD Pipeline Concepts · Jenkins setup · Declarative pipeline basics | Theory + hands-on | [01-Secure-CICD-Pipeline-Concepts.md](01-Secure-CICD-Pipeline-Concepts.md) |
| 2 | 10:30 – 11:30 | SAST Theory · SonarQube architecture · Launch & set up SonarQube | Theory + hands-on | [02-SAST-Theory-and-SonarQube-Setup.md](02-SAST-Theory-and-SonarQube-Setup.md) |
| | 11:30 – 11:45 | ☕ Tea break | | |
| 3 | 11:45 – 12:30 | SAST hands-on: analyse the app, Quality Gate, fix & rescan | Hands-on | [03-SAST-Handson-SonarQube-Analysis.md](03-SAST-Handson-SonarQube-Analysis.md) |
| 4 | 12:30 – 13:00 | Secrets Detection & Remediation using Gitleaks | Hands-on | [04-Secrets-Detection-Gitleaks.md](04-Secrets-Detection-Gitleaks.md) |
| | 12:55 | ⚠️ **Before lunch:** start the vulnerability-database download — [Lab 05 → Part A](05-OWASP-Dependency-Check.md) | 2 min | |
| | 13:00 – 14:00 | 🍽️ Lunch | | |
| 5 | 14:00 – 14:45 | Dependency Scanning (SCA) using OWASP Dependency-Check | Hands-on | [05-OWASP-Dependency-Check.md](05-OWASP-Dependency-Check.md) |
| 6 | 14:45 – 16:30 | **Final lab:** Securing a Complete Declarative Pipeline | Hands-on | [06-Secure-Declarative-Pipeline-Lab.md](06-Secure-Declarative-Pipeline-Lab.md) |
| | 16:30 | ✅ [Day 2 Completion Checklist](Day-2-Completion-Checklist.md) — then **Stop/Terminate** instances | | |

## 🖥️ Today's lab environment

```text
GitHub (your fork) ──① checkout──▶ devsecops-main  [ Jenkins :8080 · Maven · Docker · Trivy · Gitleaks · Dependency-Check · app :8081 ]
                                        │  ② code analysis                ▲ ③ Quality Gate result
                                        ▼                                 │
                                   sonarqube-server  [ SonarQube :9000 (Docker) ]
```

| Server | Instance type | New today | Ports |
|---|---|---|---|
| `devsecops-main` | **c7i-flex.large** (2 vCPU, 4 GB) — resized in Lab 01 | Jenkins, Gitleaks, Dependency-Check | 22, 8080, 8081 |
| `sonarqube-server` | **m7i-flex.large** (2 vCPU, 8 GB) — launched in Lab 02 | SonarQube (Docker) | 22, 9000 |

## 🏁 The pipeline you will build (Lab 06)

```
① Checkout → ② Gitleaks → ③ Build & Unit Test → ④ SonarQube SAST + Quality Gate
   → ⑤ OWASP Dependency-Check → ⑥ Docker Build → ⑦ Trivy Image Scan → ⑧ Deploy
```

Every security stage is a **gate**: if it fails, the stages after it **do not run** — nothing insecure gets deployed.

## 🔁 How each tool is learned today

Every tool follows the same loop you used for Trivy yesterday:

```text
Run by hand → read the findings → understand them → FIX → rescan ✅ → automate it in the pipeline (Lab 06)
```

| Finding (planted in `workshop-app`) | Found by | Fixed in |
|---|---|---|
| Weak TLS protocol · NullPointerException bug · SQL injection | SonarQube (SAST) | Lab 03 |
| AWS keys in `application.properties` | Gitleaks | Lab 04 |
| `commons-text 1.9` (CVE-2022-42889, CRITICAL) | OWASP Dependency-Check (SCA) — and Trivy | Lab 05 |
| Root user, old base image | Trivy | Day 1, Lab 06 ✔ |

## 📝 Placeholders you will use today

| Placeholder | Where you get it |
|---|---|
| `<MAIN_SERVER_IP>` | **New** public IP of `devsecops-main` after you start it (Lab 01) |
| `<SONAR_SERVER_IP>` | Public IP of `sonarqube-server` (Lab 02) |
| `<YOUR_GITHUB_USERNAME>` | Prerequisite 02 |

> 💡 Open **two** terminal windows today: one SSH'd into `devsecops-main`, one into `sonarqube-server`.
> Label them in your head: **MAIN** and **SONAR**. Each lab step says which one to use.
