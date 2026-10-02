# DevSecOps Workshop — Hands-on Lab Guide

> **Build it. Break it. Secure it.**
> Over two days you will take one real Java application from source code to a running container,
> and then protect it with an automated security pipeline — the same way engineering teams in industry do it.

---

## 📅 Schedule at a glance

### Day 1 — DevOps to DevSecOps Fundamentals

| Time | Session | Lab file |
|---|---|---|
| 09:30 – 10:15 | Inauguration, DevOps Culture & Introduction to DevSecOps | [Day-1/01](Day-1/01-DevOps-to-DevSecOps-Fundamentals.md) |
| 10:15 – 11:15 | Git Basics & Version Control (hands-on) | [Day-1/02](Day-1/02-Git-Basics-and-Version-Control.md) |
| 11:15 – 11:30 | ☕ Tea break | |
| 11:30 – 12:30 | EC2 Setup & Linux Basics | [Day-1/03](Day-1/03-EC2-Setup-and-Linux-Basics.md) |
| 12:30 – 13:30 | Application Deployment on EC2 (+ Maven essentials) | [Day-1/04](Day-1/04-Application-Deployment-on-EC2.md) |
| 13:30 – 14:15 | 🍽️ Lunch | |
| 14:15 – 15:15 | Docker Image Building, Containerization & DockerHub | [Day-1/05](Day-1/05-Docker-Containerization-and-DockerHub.md) |
| 15:15 – 16:30 | Container Security Scanning using Trivy | [Day-1/06](Day-1/06-Container-Security-Scanning-Trivy.md) |

### Day 2 — Pipeline Security & Code Analysis

| Time | Session | Lab file |
|---|---|---|
| 09:30 – 10:30 | Secure CI/CD Pipeline Concepts + Jenkins Setup | [Day-2/01](Day-2/01-Secure-CICD-Pipeline-Concepts.md) |
| 10:30 – 11:30 | SAST Theory & SonarQube Setup | [Day-2/02](Day-2/02-SAST-Theory-and-SonarQube-Setup.md) |
| 11:30 – 11:45 | ☕ Tea break | |
| 11:45 – 12:30 | SAST Hands-on with SonarQube | [Day-2/03](Day-2/03-SAST-Handson-SonarQube-Analysis.md) |
| 12:30 – 13:00 | Secrets Detection & Remediation using Gitleaks | [Day-2/04](Day-2/04-Secrets-Detection-Gitleaks.md) |
| 13:00 – 14:00 | 🍽️ Lunch *(start the vulnerability-database download before you leave — see Day-2/05 Part A)* | |
| 14:00 – 14:45 | Dependency Scanning (SCA) using OWASP Dependency-Check | [Day-2/05](Day-2/05-OWASP-Dependency-Check.md) |
| 14:45 – 16:30 | Final Lab: Securing a Complete Declarative Pipeline | [Day-2/06](Day-2/06-Secure-Declarative-Pipeline-Lab.md) |

---

## 🧭 The learning journey

Every lab builds on the previous one. You work on **one application** the whole time — the
**Course Registration Portal** (`workshop-app`) — and make it more secure step by step.

```text
Day 1:  Git & GitHub → Build with Maven → Run on EC2 → Package in Docker → Scan image (Trivy)
                                                                                 │
Day 2:  Jenkins pipeline → SAST (SonarQube) → Secrets (Gitleaks) → SCA (Dependency-Check) → 🛡️ Secure pipeline with security gates
```

**What you will have at the end of Day 2:** a Jenkins pipeline that automatically

```
GitHub → Gitleaks → Maven Build & Test → SonarQube → Dependency-Check → Docker Build → Trivy → Deploy
```

…and **refuses to deploy** if any security check fails.

---

## ✅ Before Day 1 — complete the prerequisites

You **cannot** do the labs without these accounts. Some take time to activate, so do them at least
**one day before** the workshop.

👉 **[00-Prerequisites/README.md](00-Prerequisites/README.md)** — checklist with step-by-step guides.

---

## 📖 How to read a lab file

Every lab follows the same structure, so you always know where to look:

| Section | What it gives you |
|---|---|
| 🎯 **Objective** | What you will be able to do at the end |
| 📋 **Before you start** | What must already be working (with a command to check it) |
| 📚 **Concept** | Plain-language explanation, analogy and diagram |
| 🧾 **Syntax reference** | Table of every command used in the lab |
| 🛠️ **Lab steps** | Numbered steps — one command at a time, with expected output |
| 🛡️ **Security angle** | Why this matters for DevSecOps |
| ❌ **Common errors** | Exact error message → why it happens → how to fix it |
| 🏋️ **Practice tasks** | Try-it-yourself exercises |
| 🎤 **Interview questions** | With answers (click to expand) |

### Symbols used in every lab

| Symbol | Meaning |
|---|---|
| 💻 **Laptop** | Run this on **your own computer** (Git Bash / PowerShell / Terminal) |
| ☁️ **EC2** | Run this **inside your EC2 server** (after you SSH in) |
| 🌐 **Browser** | Do this in a **web browser** (AWS Console, GitHub, Jenkins…) |
| ✅ **Check** | Verify the step worked **before** moving on |
| ❌ **If you see…** | A known error and its fix |
| 💡 **Beginner note** | Extra background — skip if you already know it |
| ⚠️ **Warning** | Read carefully — mistakes here cost time or money |

### Placeholders — always replace these

Anything written in `<ANGLE_BRACKETS_AND_CAPITALS>` is a placeholder. **Replace the whole thing,
including the `< >`**, with your own value.

| Placeholder | Replace with | Example |
|---|---|---|
| `<YOUR_GITHUB_USERNAME>` | Your GitHub username | `ravi-kumar-21` |
| `<YOUR_DOCKERHUB_USERNAME>` | Your DockerHub username | `ravikumar21` |
| `<MAIN_SERVER_IP>` | Public IPv4 of your **devsecops-main** EC2 | `13.233.45.67` |
| `<SONAR_SERVER_IP>` | Public IPv4 of your **sonarqube-server** EC2 (Day 2) | `65.0.112.9` |
| `<TRAINER_GITHUB>` | GitHub account that hosts the workshop app (your trainer will tell you) | — |

> ✏️ **Wrong:** `ssh -i devsecops-key.pem ubuntu@<13.233.45.67>`
> ✅ **Right:** `ssh -i devsecops-key.pem ubuntu@13.233.45.67`

### Copy-paste rules

1. Copy **one code block at a time**. Each block does one job.
2. Lines starting with `#` inside a code block are **comments** — they explain, they don't run.
3. **Never** copy the `$` or `>` prompt symbol if you see one in an output example.
4. After each block, look at the ✅ **Check** before continuing. Fixing an error immediately takes 1 minute; finding it 5 steps later takes 20.

---

## 🖥️ Your lab environment

| | Day 1 | Day 2 |
|---|---|---|
| **devsecops-main** | `t3.micro` (1 GB RAM) + 2 GB swap. Runs: Git, Java 21, Maven, Docker, Trivy, the app | Resized to **`c7i-flex.large`** (2 vCPU, 4 GB). Adds: Jenkins, Gitleaks, Dependency-Check |
| **sonarqube-server** | — | New **`m7i-flex.large`** (2 vCPU, 8 GB). Runs SonarQube in Docker |
| **Region** | Asia Pacific (Mumbai) `ap-south-1` | Same |
| **OS** | Ubuntu Server 24.04 LTS | Same |

> 💡 All three instance types (`t3.micro`, `c7i-flex.large`, `m7i-flex.large`) are **Free Tier eligible** for AWS accounts
> created on or after 15 July 2025. If your account is older, they are billed at normal hourly rates (small amounts if you stop them each evening).

```text
💻 Your laptop (browser + Git Bash/Terminal)
     │  SSH :22 · HTTP :8080 / :8081 / :9000
     ▼
☁️ AWS ap-south-1 (Mumbai)
     ├── devsecops-main     Day 1: t3.micro · Day 2: c7i-flex.large   (app :8081, Jenkins :8080)
     └── sonarqube-server   Day 2: m7i-flex.large                     (SonarQube :9000)
     ↕ GitHub (code)   ↕ DockerHub (images)
```

### 💰 Cost control (read this!)

- **Stop** your instances at the end of each day: EC2 → Instances → select → **Instance state → Stop instance**.
  A stopped instance does not charge for compute (only a small amount for its disk).
- **Terminate** all instances after the workshop: **Instance state → Terminate instance**.
- ⚠️ After *Stop → Start*, your **public IP changes**. Always copy the new IP from the EC2 console.

---

## 🗂️ Repository map

```
DevSecOps_VCE/
├── README.md                     ← you are here
├── 00-Prerequisites/             ← complete BEFORE Day 1
│   ├── README.md                 ← checklist
│   ├── 01-AWS-Account-Setup.md
│   ├── 02-GitHub-Account-and-Token.md
│   ├── 03-DockerHub-Account-and-Token.md
│   ├── 04-SSH-Client-Setup.md
│   └── 05-NVD-API-Key.md
├── Day-1/
│   ├── README.md
│   ├── 01-DevOps-to-DevSecOps-Fundamentals.md
│   ├── 02-Git-Basics-and-Version-Control.md
│   ├── 03-EC2-Setup-and-Linux-Basics.md
│   ├── 04-Application-Deployment-on-EC2.md
│   ├── 05-Docker-Containerization-and-DockerHub.md
│   ├── 06-Container-Security-Scanning-Trivy.md
│   └── Day-1-Completion-Checklist.md
└── Day-2/
    ├── README.md
    ├── 01-Secure-CICD-Pipeline-Concepts.md
    ├── 02-SAST-Theory-and-SonarQube-Setup.md
    ├── 03-SAST-Handson-SonarQube-Analysis.md
    ├── 04-Secrets-Detection-Gitleaks.md
    ├── 05-OWASP-Dependency-Check.md
    ├── 06-Secure-Declarative-Pipeline-Lab.md
    └── Day-2-Completion-Checklist.md
```

**Related repository:** `workshop-app` — the Spring Boot application used in every lab
(`https://github.com/<TRAINER_GITHUB>/workshop-app`). You will **fork** it in Day 1, Lab 04.

---

## 🧰 Tool versions used in this guide

| Tool | Version | Where it runs |
|---|---|---|
| Ubuntu Server | 24.04 LTS | EC2 |
| Java (OpenJDK) | 21 | EC2 |
| Maven | 3.8+ (Ubuntu package) | EC2 |
| Spring Boot | 3.5.x | inside the app |
| Docker Engine | latest stable (official Docker repository) | EC2 |
| Trivy | latest (official Aqua repository) | EC2 |
| Jenkins | LTS (official Jenkins repository) | EC2 |
| SonarQube | Community Build (`sonarqube:community` image) | EC2 (Docker) |
| Gitleaks | 8.30.1 | EC2 |
| OWASP Dependency-Check | 12.2.2 (Maven plugin) | EC2 |

---

## 🆘 Getting help during the workshop

1. Read the **❌ Common errors** section at the bottom of the lab — most problems are listed there.
2. Re-run the last ✅ **Check** command and compare its output with the expected output.
3. Still stuck? Raise your hand and show the volunteer **the exact command you ran and the full error message**.
   A screenshot of only the last line is usually not enough.

---

## 👩‍🏫 Note for trainers

- Replace every `<TRAINER_GITHUB>` in this repository with the GitHub account hosting `workshop-app`
  before sharing (one find-and-replace).
- A **dry run of both days on a fresh AWS account** is required before delivery.
- The trainer solution kit (fixed code, answer keys, timing notes) is distributed **separately** and is
  not part of this repository.
