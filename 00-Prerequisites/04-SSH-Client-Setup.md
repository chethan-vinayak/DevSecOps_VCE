# 04 — SSH Client Setup (Windows / macOS / Linux)

| | |
|---|---|
| ⏱️ **Time** | 5–10 minutes |
| 🎯 **Outcome** | `ssh -V` works on your laptop and you have a `devsecops` folder ready for your key file |

---

## 📚 Concept — what is SSH?

**SSH (Secure Shell)** lets you open a terminal on a **remote** computer over the internet, securely.
Everything you type is encrypted.

```text
💻 Your laptop                                         ☁️ EC2 server
   ssh client  ─────── encrypted connection, port 22 ──────▶  sshd
   🔑 devsecops-key.pem (PRIVATE key)                         🔒 public key (placed by AWS)
```

AWS servers don't use passwords for SSH. They use a **key pair**:

| Part | Where it lives | Analogy |
|---|---|---|
| **Public key** | Placed on the server by AWS | The **lock** on the door |
| **Private key** (`.pem` file) | Downloaded **once** to your laptop | The only **key** that opens that lock |

> ⚠️ AWS lets you download the private key **only once**, when you create it (Day 1, Lab 03).
> If you lose the `.pem` file, you cannot log in to that server anymore.

---

## 🛠️ Steps

### Step 1 — Check that SSH is available 💻 Laptop

<details open>
<summary><b>🪟 Windows — use Git Bash (recommended)</b></summary>

Open **Git Bash** (installed in guide 02) and run:

```bash
ssh -V
```

✅ **Expected output** (versions may differ):

```text
OpenSSH_9.x, OpenSSL 3.x.x
```

> 💡 Windows 10/11 also has SSH in **PowerShell**. We recommend **Git Bash** because every command in this
> workshop (Linux style) works there exactly as written. If you must use PowerShell, see Step 3.
</details>

<details>
<summary><b>🍎 macOS / 🐧 Linux</b></summary>

Open **Terminal** and run:

```bash
ssh -V
```

✅ **Expected output:** `OpenSSH_9.x ...` (any version is fine).
</details>

❌ **If you see** `ssh: command not found` on Windows → you are in *Command Prompt*. Open **Git Bash** instead.

---

### Step 2 — Create a workshop folder 💻 Laptop

This is where you will keep your `.pem` key file and notes. Run in Git Bash / Terminal:

```bash
mkdir -p ~/devsecops
```

```bash
cd ~/devsecops
```

```bash
pwd
```

✅ **Expected output:**

| OS | Output | The same folder in File Explorer / Finder |
|---|---|---|
| Windows (Git Bash) | `/c/Users/<you>/devsecops` | `C:\Users\<you>\devsecops` |
| macOS | `/Users/<you>/devsecops` | Home folder → `devsecops` |
| Linux | `/home/<you>/devsecops` | Home → `devsecops` |

> 💡 `~` (tilde) is a shortcut for **your home folder**. `mkdir -p` creates a folder (and does nothing if it already exists).

Move your `workshop-secrets.txt` into this folder too.

---

### Step 3 — (Only if you use PowerShell instead of Git Bash) 💻 Laptop

PowerShell's SSH refuses to use a key file that other Windows users can read, and shows
`WARNING: UNPROTECTED PRIVATE KEY FILE!`. You will fix that on Day 1 with these commands (run them **after**
you have downloaded the key, inside the folder that contains it):

```powershell
icacls.exe .\devsecops-key.pem /reset
```

```powershell
icacls.exe .\devsecops-key.pem /grant:r "$($env:USERNAME):(R)"
```

```powershell
icacls.exe .\devsecops-key.pem /inheritance:r
```

They mean: *reset permissions → give only me read access → remove inherited permissions*.
On Git Bash/macOS/Linux, the equivalent is simply `chmod 400 devsecops-key.pem` (covered in Lab 03).

---

### Step 4 — Know your basic terminal moves 💻 Laptop

| Action | Git Bash / Terminal |
|---|---|
| Paste | **Shift + Insert** or **right-click → Paste** (Git Bash) · **Cmd + V** (macOS) · **Ctrl + Shift + V** (Linux) |
| Copy | Select text with the mouse (Git Bash copies on select) · **Cmd + C** (macOS) · **Ctrl + Shift + C** (Linux) |
| Stop a running command | **Ctrl + C** |
| Previous command | **↑** arrow key |
| Auto-complete a file/folder name | **Tab** |
| Clear the screen | `clear` or **Ctrl + L** |

> ⚠️ In a terminal, **Ctrl + C does not copy** — it **stops** the running program. This surprises everyone once.

---

## ✅ Ready?

- [ ] `ssh -V` prints a version
- [ ] `~/devsecops` folder exists
- [ ] You know how to paste into your terminal

---

➡️ **Next:** [05-NVD-API-Key.md](05-NVD-API-Key.md)
