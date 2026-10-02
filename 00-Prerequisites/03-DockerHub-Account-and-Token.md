# 03 — DockerHub Account & Access Token

| | |
|---|---|
| ⏱️ **Time** | 10 minutes |
| 🎯 **Outcome** | DockerHub account ✔ · a **Personal Access Token** with *Read & Write* permission saved in `workshop-secrets.txt` ✔ |

---

## 📚 Concept

| Term | What it is | Analogy |
|---|---|---|
| **Docker image** | A packaged application + everything it needs to run | A sealed lunch box |
| **DockerHub** | An online "store" (registry) where images are uploaded and downloaded | A Play Store for Docker images |
| **Repository (on DockerHub)** | One image name, e.g. `ravikumar21/workshop-app`, holding many versions (**tags**) | One app on the Play Store with many versions |
| **Access Token** | A password-like secret used by the `docker login` command | A key-card you can cancel anytime |

In Day 1, Lab 05 you will **push** (upload) your own image of the workshop application to DockerHub.

> 💡 Like GitHub, DockerHub recommends **access tokens instead of your account password** for the command line.
> If a token leaks, you delete just that token — your account password stays safe.

---

## 🛠️ Steps

### Step 1 — Create a DockerHub account 🌐 Browser

1. Go to **https://hub.docker.com** → **Sign up**.
2. Enter email, **username** and password (or sign up with your GitHub account).
   💡 Your DockerHub username will be part of every image name: `<YOUR_DOCKERHUB_USERNAME>/workshop-app`.
   Use **lowercase letters and numbers only** to avoid surprises.
3. Verify your email address using the link DockerHub sends.
4. If asked to choose a plan, choose **Personal (free)**.

✅ **Check:** you can sign in at https://hub.docker.com and see your (empty) repositories page.

Write your DockerHub username in `workshop-secrets.txt`.

---

### Step 2 — Create an Access Token 🌐 Browser

1. Click your **avatar** (top-right) → **Account settings**.
2. In the left menu → **Personal access tokens** → **Generate new token**.
3. Fill in:

   | Field | Value |
   |---|---|
   | **Access token description** | `devsecops-workshop` |
   | **Expiration date** | **30 days** |
   | **Access permissions** | **Read & Write** |

4. Click **Generate**.
5. **Copy the token** (starts with `dckr_pat_`) — it is shown **only once** — and paste it into
   `workshop-secrets.txt`.
6. Click **Back to access tokens**.

✅ **Check:** the **Personal access tokens** list shows `devsecops-workshop` with scope **Read & Write**.

> We will test this token on Day 1 with `docker login`. You don't need Docker on your laptop.

---

## 🛡️ Security angle

- **Read & Write** is enough to push images. We do **not** choose *Read, Write & Delete* — a leaked token
  should not be able to delete your images. This is the **principle of least privilege**.
- When you run `docker login` on a server, Docker saves your credentials in `~/.docker/config.json` (only
  base64-encoded, **not encrypted**). That's why, at the end of Day 1, we run `docker logout` on shared machines.

---

## ❌ Common errors

| You see | Why | Fix |
|---|---|---|
| Email never arrives | Spam folder / typo in email | Check spam; resend verification from account settings |
| Lost the token | Shown only once | Delete it and generate a new one |
| Username has capitals | Docker image names must be lowercase | Use the lowercase form in all commands |

---

➡️ **Next:** [04-SSH-Client-Setup.md](04-SSH-Client-Setup.md)
