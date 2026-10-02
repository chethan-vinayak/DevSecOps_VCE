# 00 — Prerequisites (complete BEFORE Day 1)

> ⏰ **Deadline:** finish everything on this page **at least one day before Day 1**.
> AWS account verification and the NVD API key can take several hours to arrive by email.

If you arrive on Day 1 without these, you will spend the morning creating accounts instead of learning.

---

## ✅ Checklist

Tick each box only after its **✅ Check** step in the guide succeeds.

| # | Task | Guide | Time needed | Done |
|---|---|---|---|---|
| 1 | AWS account created, card verified, region set to **Mumbai**, budget alert set | [01-AWS-Account-Setup.md](01-AWS-Account-Setup.md) | 20–30 min (+ activation wait) | ☐ |
| 2 | GitHub account created, Git installed on laptop, **Personal Access Token** saved | [02-GitHub-Account-and-Token.md](02-GitHub-Account-and-Token.md) | 15 min | ☐ |
| 3 | DockerHub account created, **Access Token** saved | [03-DockerHub-Account-and-Token.md](03-DockerHub-Account-and-Token.md) | 10 min | ☐ |
| 4 | SSH client works on your laptop (`ssh -V` prints a version) | [04-SSH-Client-Setup.md](04-SSH-Client-Setup.md) | 5–10 min | ☐ |
| 5 | **NVD API key** requested and activated (needed on Day 2) | [05-NVD-API-Key.md](05-NVD-API-Key.md) | 5 min (+ email wait) | ☐ |

---

## 💻 What your laptop needs

| Item | Why | Notes |
|---|---|---|
| Windows 10/11, macOS or Linux | To connect to your cloud server | Any laptop from the last ~6 years is fine |
| A modern browser (Chrome / Edge / Firefox) | AWS Console, GitHub, Jenkins, SonarQube | |
| **Git** (includes **Git Bash** on Windows) | Day 1 Git lab + SSH | Installed in guide 02 |
| **SSH client** | Log in to your EC2 server | Comes with Git Bash / Windows 10+ / macOS / Linux |
| A text editor (VS Code recommended, Notepad is OK) | Small edits, saving tokens | Optional |
| Charger + Wi-Fi access | Labs are online the whole day | |

> 💡 **You do NOT need** to install Java, Maven, Docker or Jenkins on your laptop. All of those run on your
> cloud server (EC2). Your laptop is only a "remote control".

---

## 🔐 Keep a private notes file

You will create several secrets (tokens, keys, passwords). Create a file on your laptop called
`workshop-secrets.txt` and keep it **only on your laptop** — never commit it, never share it in a chat group.

Copy this template into it:

```text
==== DevSecOps Workshop — PRIVATE (do not share) ====

AWS
  Account email        :
  Region               : ap-south-1 (Mumbai)
  Key pair file        : (Day 1) devsecops-key.pem  — saved in folder:

GitHub
  Username             :
  Personal Access Token: ghp_...

DockerHub
  Username             :
  Access Token         : dckr_pat_...

NVD (Day 2)
  API key              :

Servers (fill in during the workshop — IPs change after Stop/Start!)
  devsecops-main IP    :
  sonarqube-server IP  :
  Jenkins admin user   :
  Jenkins password     :
  SonarQube password   :
  SonarQube token      : squ_...
```

> 🛡️ **Security angle:** this file is exactly the kind of thing attackers search for. In Day 2 you will learn how
> tools like **Gitleaks** catch secrets that accidentally end up in Git — and why a token in a chat
> message or a public repo must be treated as **already stolen**.

---

## ❓ FAQ

<details>
<summary><b>I don't have a credit/debit card. Can I still attend?</b></summary>

AWS requires a card that supports international transactions for identity verification. If you cannot get one,
tell your coordinator **before** the workshop — you will be paired with another student and share their server.
You still need your **own** GitHub and DockerHub accounts (they are free and need no card).
</details>

<details>
<summary><b>Will AWS charge me money?</b></summary>

The instance types used in this workshop are Free Tier eligible for new accounts. Costs stay near zero **if you
stop instances every evening and terminate them after Day 2**. Guide 01 shows how to set a **budget alert** so
AWS emails you before anything is charged.
</details>

<details>
<summary><b>Can I use a college/lab computer instead of my laptop?</b></summary>

Yes, as long as it has a browser and Git Bash (or another SSH client), and you can save your `.pem` key file on
it for both days (or carry it on a pen drive). Without the key file you cannot log in to your server.
</details>

<details>
<summary><b>I already have AWS / GitHub / DockerHub accounts.</b></summary>

Great — use them. Still go through each guide's **✅ Check** steps (region, token, budget) to make sure
everything is set up the way the labs expect.
</details>

---

➡️ **Start with:** [01-AWS-Account-Setup.md](01-AWS-Account-Setup.md)
