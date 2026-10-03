# Lab 02 — SAST Theory & SonarQube Setup

| | |
|---|---|
| 🕘 **Session** | Day 2 · 10:30 – 11:30 (60 min) |
| 🌐☁️ **Where** | AWS Console → new server `sonarqube-server` (SONAR terminal) → SonarQube in the browser |
| 🎯 **Objective** | Understand **SAST** (and how it differs from DAST and SCA), understand **SonarQube's architecture**, then launch a dedicated server and run **SonarQube in Docker** |
| 🏁 **You will have** | SonarQube running at `http://<SONAR_SERVER_IP>:9000`, a new admin password, and an **analysis token** saved |

---

## 📚 Part A — Concepts

### A1. What is SAST?

**SAST — Static Application Security Testing** analyses an application's **source code** (or compiled code)
**without running it**, looking for insecure patterns: injection, weak cryptography, hard-coded credentials,
null-pointer bugs, etc.

> 🔍 **Analogy:** SAST is a **proof-reader** who checks the *text* of a book for errors before it's printed.
> DAST is a **mystery shopper** who uses the finished shop and tries to break things.

### A2. The application security testing family

| | **SAST** | **DAST** | **SCA** | **Secrets scanning** |
|---|---|---|---|---|
| Full name | Static Application Security Testing | Dynamic Application Security Testing | Software Composition Analysis | — |
| Looks at | **Your source code** | The **running** application (HTTP requests) | **Third-party libraries** | Code + Git history |
| Needs the app running? | ❌ No | ✅ Yes | ❌ No | ❌ No |
| When in the pipeline | Build (very early) | After deploy to a test environment | Build | Commit / Build |
| Finds | SQL injection patterns, weak crypto, bugs | Real exploitable behaviour, server misconfig | Known CVEs in libraries | Keys, passwords, tokens |
| Shows exact code line? | ✅ Yes | ❌ No (shows URL/request) | Shows library + version | ✅ Yes |
| Typical weakness | **False positives**, no runtime context | Late, slower, partial coverage | Only *known* CVEs | Pattern-based misses |
| Examples | **SonarQube**, Semgrep, Checkmarx | OWASP ZAP, Burp Suite | **OWASP Dependency-Check**, Snyk, Trivy | **Gitleaks**, TruffleHog |

> 💡 No single tool finds everything. Mature DevSecOps pipelines **layer** them — that's what you're building today.

### A3. How SAST works (simplified)

```text
Source code (.java) → Parser (syntax tree) → Rules engine → Data-flow analysis ("can null reach here?") → Issues (file + line + fix)
```

### A4. What is SonarQube?

**SonarQube** (by SonarSource) is a platform for **continuous inspection of code quality and security**. The
free **Community Build** supports 20+ languages including Java.

#### SonarQube's issue types

| Software quality | Issue type | Example in our app | Rating |
|---|---|---|---|
| **Security** | **Vulnerability** — code that can be exploited | Using the outdated **TLSv1** protocol | Security Rating A–E |
| **Security** | **Security Hotspot** — security-sensitive code a **human must review** | SQL built by concatenating strings; hard-coded password | Hotspots reviewed % |
| **Reliability** | **Bug** — code that will behave wrongly | Possible `NullPointerException` | Reliability Rating A–E |
| **Maintainability** | **Code smell** — confusing or hard-to-maintain code | Unused variable, too-complex method | Maintainability Rating A–E |

**Ratings:** **A** = best (no issues of that type) … **E** = worst (at least one *blocker*).

#### Quality Profile vs Quality Gate

| | **Quality Profile** | **Quality Gate** |
|---|---|---|
| What | The **set of rules** used to analyse code (per language) | The **pass/fail conditions** checked after analysis |
| Default | *Sonar way* | *Sonar way* |
| Example | "Detect weak TLS protocols" ON | "Security Rating must be A" |
| Analogy | The **exam questions** | The **pass mark** |

> 💡 *Sonar way* checks conditions on **new code** only (code changed recently) — "*clean as you code*".
> In Lab 03 you will create a stricter gate that also checks **overall code**.

### A5. SonarQube architecture

```text
devsecops-main                              sonarqube-server (Docker, port 9000)
┌──────────────────────┐   ① upload report  ┌────────────────────────────────────────────┐
│ SonarScanner (Maven) │ ─────────────────▶ │ Web server ─▶ Compute engine ─▶ Database   │
│ mvn verify sonar:sonar│ ◀───────────────── │                 └──────────▶ Search engine │
└──────────────────────┘ ⑤ Quality Gate     └────────────────────────────────────────────┘
                           PASSED / FAILED            ④ 👩‍💻 you view the dashboard in a browser
```

| Component | Role |
|---|---|
| **Scanner** | Runs on the **build machine**; analyses code locally and uploads a report (code is analysed where it's built) |
| **Web Server** | UI and REST API on port **9000** |
| **Compute Engine** | Processes uploaded reports in the background |
| **Search Engine** | Elasticsearch — makes issues searchable; this is why SonarQube needs kernel tuning and RAM |
| **Database** | Stores everything. The Docker image uses an **embedded** database — fine for learning, **not** for production |

---

## 🛠️ Part B — Launch the SonarQube server 🌐 Browser

SonarQube needs several GB of RAM, so it gets its own server.

**B1.** EC2 → **Launch instance**:

Use the same wizard as Day 1 ([Lab 03, Part A](../Day-1/03-EC2-Setup-and-Linux-Basics.md)) with these values:

| Setting | Value |
|---|---|
| Name | `sonarqube-server` |
| AMI | **Ubuntu Server 24.04 LTS (HVM), SSD Volume Type** · 64-bit (x86) |
| Instance type | **`m7i-flex.large`** (2 vCPU, 8 GB) — if unavailable: `t3.large` |
| Key pair | **`devsecops-key`** (the same one as yesterday — select it, don't create a new one) |

**B2.** **Network settings → Edit**:

| Setting | Value |
|---|---|
| Auto-assign public IP | Enable |
| Firewall | **Create security group** · name `sonarqube-sg` |
| Rule 1 | ssh · 22 · Anywhere |
| Rule 2 (Add rule) | **Custom TCP · 9000** · Anywhere · description `SonarQube` |

**B3.** **Storage:** **20** GiB gp3.

**B4.** **Launch instance** → wait for **Running** + **3/3 checks**.

**B5.** Copy its **Public IPv4** → `workshop-secrets.txt` as `sonarqube-server IP`. This is **`<SONAR_SERVER_IP>`**.

**B6. 💻 Laptop** — open a **second** terminal window (this is the **SONAR** terminal):

```bash
cd ~/devsecops
```

```bash
ssh -i devsecops-key.pem ubuntu@<SONAR_SERVER_IP>
```

✅ Prompt shows `ubuntu@ip-...` — make sure you know which window is MAIN and which is SONAR!

> 💡 Tip: run `echo "SONAR SERVER"` here and `echo "MAIN SERVER"` in the other window so the label stays on screen.

---

## 🛠️ Part C — Prepare the server for SonarQube ☁️ SONAR

SonarQube's embedded **Elasticsearch** refuses to start unless the Linux kernel allows it enough memory-mapped
areas and open files. This is the **#1 reason SonarQube fails to start**.

**C1.** Write the required kernel settings to a config file (so they survive reboots):

```bash
echo "vm.max_map_count=524288" | sudo tee /etc/sysctl.d/99-sonarqube.conf
```

```bash
echo "fs.file-max=131072" | sudo tee -a /etc/sysctl.d/99-sonarqube.conf
```

**C2.** Apply them now:

```bash
sudo sysctl --system | grep -E "max_map_count|file-max"
```

✅ **Expected:**

```text
vm.max_map_count = 524288
fs.file-max = 131072
```

**C3.** Install Docker. On this single-purpose server, Ubuntu's own Docker package is quick and sufficient:

```bash
sudo apt update
```

```bash
sudo apt install -y docker.io
```

```bash
sudo usermod -aG docker $USER
```

```bash
newgrp docker
```

✅ **Check:**

```bash
docker ps
```
→ an empty table (no "permission denied").

---

## 🛠️ Part D — Run SonarQube in Docker ☁️ SONAR

**D1.** Start the container (copy the **whole** block — the `\` at line ends means "command continues on next line"):

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:community
```

```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  --restart unless-stopped \
  --ulimit nofile=131072:131072 \
  --ulimit nproc=8192:8192 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:community
```

| Option | Why |
|---|---|
| `-p 9000:9000` | Web UI port |
| `--restart unless-stopped` | Starts again automatically after a server reboot / stop-start |
| `--ulimit nofile / nproc` | Open-file and process limits Elasticsearch requires |
| `-v sonarqube_data:...` (×3) | **Named volumes** — data survives if the container is recreated |
| `sonarqube:community` | Free Community Build, latest version |

> ⏳ Downloads ~1 GB, then starts. **First start takes 2–4 minutes.**

**D2.** Watch the log until SonarQube is ready:

```bash
docker logs -f sonarqube
```

Wait for this line, then press **Ctrl + C**:

```text
SonarQube is operational
```

**D3.** Ask the API for its status:

```bash
curl -s http://localhost:9000/api/system/status; echo
```

✅ **Expected:** `"status":"UP"` (if you see `STARTING`, wait 30 seconds and retry).

---

## 🛠️ Part E — First login, password, token 🌐 Browser

**E1.** Open `http://<SONAR_SERVER_IP>:9000`.

**E2.** Log in with the default credentials **`admin` / `admin`**.

<img src="https://github.com/user-attachments/assets/06ce799a-54ce-4b06-aa8f-6bd286155f02" alt="SonarQube login page" width="700">

<sub>Source: reference repo vickydevo/DevSecOps-WS (SonarQube-SAST/InstallSonar.md)</sub>


**E3.** You are forced to change the password. Choose a **strong** one (SonarQube requires at least 12
characters with upper-case, lower-case, a digit and a special character) → save it in `workshop-secrets.txt`.

> 🛡️ Default credentials (`admin/admin`) are one of the most common ways attackers get into tools like
> SonarQube, Jenkins and databases. Changing them is the **first** step of every installation.

**E4.** Create an **analysis token** (Maven and Jenkins will use it instead of your password):

1. Top-right **avatar (A)** → **My Account** → **Security** tab.
2. **Generate Tokens:**

   | Field | Value |
   |---|---|
   | Name | `jenkins` |
   | Type | **User Token** |
   | Expires in | **30 days** |

3. **Generate** → **copy the token** (starts with `squ_`) → paste into `workshop-secrets.txt`.
   It is shown **only once**.

✅ **Check:** the token `jenkins` appears in the list on the Security page.

**E5.** Quick tour (2 minutes): open **Rules** (top menu) → filter **Language: Java** → see how many rules exist.
Open **Quality Profiles** → *Sonar way* for Java. Open **Quality Gates** → *Sonar way* conditions.

<img src="https://github.com/user-attachments/assets/a28caa7a-b75c-46dc-9522-33a69fbe6952" alt="SonarQube home after login" width="900">

<sub>Source: reference repo vickydevo/DevSecOps-WS (SonarQube-SAST/InstallSonar.md)</sub>


---

## 🛡️ Security angle

| Done | Why |
|---|---|
| Changed default `admin/admin` password | Default creds are a top attack vector |
| Token with **expiry** instead of password for automation | Revocable, limited, auditable |
| SonarQube on its own server | Separation of duties; a heavy service doesn't starve the build server |
| ⚠️ Port 9000 open to anywhere, embedded DB | Lab only — production uses HTTPS, restricted access and PostgreSQL |

---

## ❌ Common errors

| You see | Why | Fix |
|---|---|---|
| Container stops; log shows `max virtual memory areas vm.max_map_count [65530] is too low` | Part C skipped/failed | Redo C1–C2, then `docker start sonarqube` |
| `docker logs` shows `OutOfMemoryError` / container keeps restarting | Instance too small | Use `m7i-flex.large` / `t3.large` (≥ 8 GB) |
| Browser can't reach `:9000` | Still starting, or security group missing port 9000 | Wait for *operational*; check `sonarqube-sg` inbound rules |
| `docker: Error response ... Conflict. The container name "/sonarqube" is already in use` | Ran `docker run` twice | `docker start sonarqube` (don't create a new one) |
| `permission denied ... docker.sock` | Group not active | `newgrp docker` or re-login |
| `m7i-flex.large` "not supported in this Availability Zone" | AZ limitation | Network settings → **Edit** → choose a different **Subnet**, or use `t3.large` |
| Password rejected | Doesn't meet complexity rules | 12+ chars, upper, lower, digit, special |

---

## 🎤 Interview questions

<details>
<summary><b>1. SAST vs DAST — when would you use each?</b></summary>

SAST early in the pipeline on source code (fast feedback, exact line numbers, no running app needed). DAST later
against a running test deployment to find runtime/configuration issues SAST can't see. Use both.
</details>

<details>
<summary><b>2. What is a Security Hotspot in SonarQube?</b></summary>

Security-sensitive code that is not necessarily vulnerable but must be **reviewed by a human** (e.g. SQL built
from strings, use of a hard-coded password). The reviewer marks it *Safe*, *Fixed* or *Acknowledged*.
</details>

<details>
<summary><b>3. Quality Profile vs Quality Gate?</b></summary>

A Quality Profile is the set of rules used during analysis. A Quality Gate is the set of pass/fail conditions
evaluated on the analysis results (e.g. no new bugs, coverage ≥ 80%).
</details>

<details>
<summary><b>4. Why does SonarQube need <code>vm.max_map_count</code> increased?</b></summary>

It embeds Elasticsearch, which uses memory-mapped files heavily and refuses to start in production mode unless
`vm.max_map_count` is at least the required value.
</details>

<details>
<summary><b>5. What are false positives and how do you handle them in SAST?</b></summary>

Findings that aren't real problems in context. Review them, mark them as false positive/won't fix with a
justification in the tool, and tune rules/profiles — never ignore findings silently.
</details>

---

➡️ **Next lab (after tea):** [03 — SAST Hands-on with SonarQube](03-SAST-Handson-SonarQube-Analysis.md)
