# Day 1 — DevOps to DevSecOps Fundamentals

> **Theme of the day:** take an application from **source code → running on a cloud server → packaged as a
> container → scanned for vulnerabilities.**

## 📅 Timetable

| # | Time | Session | Type | Lab file |
|---|---|---|---|---|
| 1 | 09:30 – 10:15 | Inauguration, DevOps Culture & Introduction to DevSecOps | Theory + discussion | [01-DevOps-to-DevSecOps-Fundamentals.md](01-DevOps-to-DevSecOps-Fundamentals.md) |
| 2 | 10:15 – 11:15 | Git Basics & Version Control | Hands-on 💻 | [02-Git-Basics-and-Version-Control.md](02-Git-Basics-and-Version-Control.md) |
| | 11:15 – 11:30 | ☕ Tea break | | |
| 3 | 11:30 – 12:30 | EC2 Setup & Linux Basics | Hands-on 🌐☁️ | [03-EC2-Setup-and-Linux-Basics.md](03-EC2-Setup-and-Linux-Basics.md) |
| 4 | 12:30 – 13:30 | Application Deployment on EC2 (+ Maven essentials) | Hands-on ☁️ | [04-Application-Deployment-on-EC2.md](04-Application-Deployment-on-EC2.md) |
| | 13:30 – 14:15 | 🍽️ Lunch | | |
| 5 | 14:15 – 15:15 | Docker Image Building, Containerization & DockerHub | Hands-on ☁️ | [05-Docker-Containerization-and-DockerHub.md](05-Docker-Containerization-and-DockerHub.md) |
| 6 | 15:15 – 16:30 | Container Security Scanning using Trivy | Hands-on ☁️ | [06-Container-Security-Scanning-Trivy.md](06-Container-Security-Scanning-Trivy.md) |
| | 16:30 | ✅ [Day 1 Completion Checklist](Day-1-Completion-Checklist.md) — then **Stop your instance** | | |

## 🧭 How today's labs connect

```text
Lab 02 Git & GitHub → Lab 03 EC2 + Linux → Lab 04 Build & deploy (:8081) → Lab 05 Docker + DockerHub → Lab 06 Trivy (hardened image)
```

## 🏁 Where you should be at 16:30

| Item | How to check |
|---|---|
| EC2 instance **devsecops-main** (t3.micro) running in **Mumbai** | EC2 console → Instances |
| Your fork `https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app` contains **your hardened `Dockerfile`** | Open it on GitHub |
| Image `<YOUR_DOCKERHUB_USERNAME>/workshop-app` visible on DockerHub | hub.docker.com → Repositories |
| You can explain: CVE, image vs container, why "running as root" is bad | Ask your neighbour 🙂 |

> 🔁 **Tomorrow (Day 2)** builds directly on today's server and today's fork. **Do not terminate** your
> instance tonight — just **Stop** it.

## 📝 Placeholders you will use today

| Placeholder | Where you get it |
|---|---|
| `<YOUR_GITHUB_USERNAME>` | Prerequisite 02 |
| `<YOUR_DOCKERHUB_USERNAME>` | Prerequisite 03 |
| `<MAIN_SERVER_IP>` | EC2 console → your instance → **Public IPv4 address** (Lab 03) |
| `<TRAINER_GITHUB>` | Announced by the trainer (the account that hosts `workshop-app`) |
