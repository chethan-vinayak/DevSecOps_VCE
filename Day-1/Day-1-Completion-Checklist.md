# ✅ Day 1 — Completion Checklist

Run these checks at **16:30**. Every item must be ✅ before you leave — **Day 2 starts from exactly this state**.

---

## 1. Verify your work ☁️ EC2

Copy this whole block — it prints a short status report:

```bash
cd ~/workshop-app
echo "== Java ==";        java -version 2>&1 | head -1
echo "== Maven ==";       mvn -v 2>/dev/null | head -1
echo "== Docker ==";      docker --version
echo "== Trivy ==";       trivy --version | head -1
echo "== Jar ==";         ls -lh target/workshop-app.jar
echo "== Images ==";      docker images workshop-app --format '{{.Repository}}:{{.Tag}}  {{.Size}}'
echo "== Container ==";   docker ps --filter name=workshop-app --format '{{.Names}}  {{.Status}}'
echo "== Runs as ==";     docker exec workshop-app whoami
echo "== Git remote ==";  git remote get-url origin
echo "== Last commit =="; git log --oneline -1
echo "== Pushed? ==";     git status -sb | head -1
```

✅ **Expected:**

| Line | Should show |
|---|---|
| Java | `openjdk version "21...` |
| Maven | `Apache Maven 3.x` |
| Docker / Trivy | a version number |
| Jar | `target/workshop-app.jar` |
| Images | `workshop-app:2.0` and `workshop-app:1.0` (and `bad` if you kept it) |
| Container | `workshop-app  Up ... (healthy)` |
| Runs as | `app` |
| Git remote | `https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app.git` — **your** username |
| Last commit | `Add hardened Dockerfile ...` |
| Pushed? | `## main...origin/main` with **no** `[ahead 1]` |

---

## 2. Verify online 🌐 Browser

- [ ] `http://<MAIN_SERVER_IP>:8081` shows the Course Registration Portal
- [ ] `https://github.com/<YOUR_GITHUB_USERNAME>/workshop-app` contains your hardened **`Dockerfile`**
- [ ] `https://github.com/<YOUR_GITHUB_USERNAME>/git-practice` shows 2 branches (`main`, `feature/login`) and a `.gitignore`
- [ ] `https://hub.docker.com/r/<YOUR_DOCKERHUB_USERNAME>/workshop-app/tags` shows tags **1.0** and **2.0**

---

## 3. Can you explain…? (say it to your neighbour)

- [ ] The difference between `git add`, `git commit` and `git push`
- [ ] Why a deleted secret is still dangerous in Git history
- [ ] What a branch is, and why `login.txt` disappeared when you switched back to `main`
- [ ] What `chmod 400` and `chmod 777` mean — and which one is a security finding
- [ ] Maven lifecycle: `compile → test → package`
- [ ] Image vs container · `EXPOSE` vs `-p`
- [ ] What a CVE and a CVSS score are
- [ ] Why `workshop-app:2.0` still has a CRITICAL finding

---

## 4. Clean up and save money ☁️ EC2 → 🌐

**4.1** Log out of DockerHub on the server (removes the stored token):

```bash
docker logout
```

**4.2** Free some disk space (keeps your `1.0` and `2.0` images):

```bash
docker rmi workshop-app:bad 2>/dev/null; docker image prune -f
```

**4.3** Leave the server:

```bash
exit
```

**4.4 🌐 STOP the instance** (do **not** terminate!):
EC2 → Instances → select **devsecops-main** → **Instance state → Stop instance** → **Stop**.

✅ **Check:** Instance state = **Stopped**.

> ⚠️ Tomorrow morning you'll get a **new public IP** after starting it. Update `workshop-secrets.txt`.

---

## 📌 Tomorrow (Day 2) preview

| You'll add | To answer |
|---|---|
| **Jenkins** | "Can the whole build → scan → deploy run automatically on every change?" |
| **SonarQube** (SAST) | "Is the code we wrote secure?" — e.g. the SQL injection from Lab 04 |
| **Gitleaks** | "Did we commit secrets?" — the AWS key in `application.properties` |
| **OWASP Dependency-Check** (SCA) | "Are our libraries vulnerable?" — `commons-text 1.9` |
| **Security gates** | "Can insecure code reach production?" → **No.** |

**Before Day 2:** make sure your **NVD API key** is activated ([Prerequisite 05](../00-Prerequisites/05-NVD-API-Key.md)).

➡️ **Day 2:** [Day-2/README.md](../Day-2/README.md)
