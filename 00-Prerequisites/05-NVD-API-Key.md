# 05 — NVD API Key (needed on Day 2)

| | |
|---|---|
| ⏱️ **Time** | 5 minutes (+ waiting for an email) |
| 🎯 **Outcome** | An activated **NVD API key** saved in `workshop-secrets.txt` |
| 📅 **Used in** | Day 2, Lab 05 — OWASP Dependency-Check |

---

## 📚 Concept — why do I need this?

On Day 2 you will scan the application's **libraries** (dependencies) for known vulnerabilities using
**OWASP Dependency-Check**. To know which libraries are dangerous, the tool downloads the list of all
publicly known vulnerabilities (**CVEs**) from the **NVD — National Vulnerability Database**, run by the
U.S. National Institute of Standards and Technology (NIST).

That is a big download — hundreds of thousands of CVE records. NVD limits how fast anyone can download:

| | Requests allowed (approx.) | First download of the full database |
|---|---|---|
| **Without** an API key | ~5 requests per 30 seconds | Very slow — can take **hours** |
| **With** a free API key | ~50 requests per 30 seconds | Typically **10–30 minutes** |

> 🏫 With 80+ students downloading at the same time, **you will not finish the lab without a key.**

---

## 🛠️ Steps

### Step 1 — Request the key 🌐 Browser

1. Open **https://nvd.nist.gov/developers/request-an-api-key**
2. Fill in the form:

   | Field | What to enter |
   |---|---|
   | **Organization Name** | Your college name |
   | **Email Address** | An email you can open right now |
   | **Organization Type** | Choose **Education** (or the closest option) |

3. Accept the terms of use → **Submit**.

---

### Step 2 — Activate it from your email 🌐 Browser

1. Open the email from NVD (check **Spam/Promotions** too). It can take from a few minutes to a few hours.
2. Click the **activation link** in the email.
   ⚠️ The link is single-use and expires after some days — **activate it as soon as it arrives**.
3. The page that opens shows your **API key** — a value that looks like
   `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.
4. Copy it into `workshop-secrets.txt` under **NVD**.

✅ **Check:** your notes file contains a key in the format `8-4-4-4-12` characters.

---

### Step 3 — (Optional) Test the key 💻 Laptop

Read the key silently (it won't appear on screen or in history):

```bash
read -s -p "Paste your NVD API key and press Enter: " NVD_API_KEY; echo
```

Ask NVD for one CVE (Log4Shell) using the key:

```bash
curl -s -o /dev/null -w "HTTP status: %{http_code}\n" -H "apiKey: $NVD_API_KEY" "https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2021-44228"
```

✅ **Expected output:**

```text
HTTP status: 200
```

```bash
unset NVD_API_KEY
```

❌ **If you see** `HTTP status: 404` or `403` → the key is not activated yet or was copied incorrectly.
Open the activation email again.

> 💡 NVD's servers are sometimes slow or briefly unavailable. If you get `503`, wait a few minutes and retry.

---

## 🛡️ Security angle

The NVD API key is not as dangerous as a GitHub token (it can't change any data), but it is **yours** —
if others use it, **your** rate limit gets used up. On Day 2 you will pass it to the tool through an
**environment variable**, never by typing it into a file that gets committed to Git.

---

✅ **All five prerequisites done?** Go back to the [checklist](README.md) and tick them off.

➡️ **On Day 1, start with:** [Day-1/README.md](../Day-1/README.md)
