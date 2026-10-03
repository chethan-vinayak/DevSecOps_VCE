# Lab 05 — Dependency Scanning (SCA) using OWASP Dependency-Check

| | |
|---|---|
| 🕘 **Session** | Part A: Day 2 · **12:55** (2 min, before lunch) · Parts B–F: **14:00 – 14:45** (45 min) |
| ☁️ **Where** | **MAIN** terminal (`devsecops-main`) |
| 🎯 **Objective** | Download the vulnerability database, scan the application's **third-party libraries**, understand the **CRITICAL Text4Shell** finding, **upgrade** the libraries you can, **document** the one you can't (a dated suppression) and rescan until the dependency gate passes |
| 🏁 **You will have** | A shared vulnerability database in `/opt/dc-data` (also used by Jenkins in Lab 06), an HTML/JSON report, `commons-text` and Tomcat upgraded, and a documented suppression file, in your fork |

---

## 🛠️ Part A — Start the database download (BEFORE LUNCH) ☁️ MAIN

Dependency-Check needs a local copy of the **NVD** vulnerability database. The first download takes **10–30 minutes**,
so we start it now and let it run during lunch.

**A1.** Create a shared data folder that both **you** (`ubuntu`) and **Jenkins** (`jenkins`) can use:

```bash
sudo mkdir -p /opt/dc-data
```

```bash
sudo chown ubuntu:jenkins /opt/dc-data
```

```bash
sudo chmod 2775 /opt/dc-data
```

> 💡 `2775` = owner and group can read/write/enter (`775`), and the leading **`2`** (*setgid*) makes every new file
> inside inherit the **`jenkins`** group — so Jenkins can use the database you download.

**A2.** Load your **NVD API key** into an environment variable (hidden input, not saved in history):

```bash
read -s -p "Paste your NVD API key: " NVD_API_KEY; echo
```

```bash
export NVD_API_KEY
```

✅ **Check** (shows only the first 8 characters):

```bash
echo "${NVD_API_KEY:0:8}..."
```

> 💡 The project's `pom.xml` is configured with `<nvdApiKeyEnvironmentVariable>NVD_API_KEY</nvdApiKeyEnvironmentVariable>`
> — the plugin **reads the key from this variable**, so it never appears in a file or on the command line.

**A3.** Start the download in the background:

```bash
cd ~/workshop-app
```

```bash
nohup mvn -B dependency-check:update-only > ~/dc-update.log 2>&1 &
```

**A4.** Watch it for ~30 seconds to be sure it started, then press **Ctrl + C** (the download continues):

```bash
tail -f ~/dc-update.log
```

✅ **Expected:** lines like

```text
[INFO] NVD API has NNN,NNN records in this update
[INFO] Downloaded 10,000/NNN,NNN (4%)
```

> ❌ If you see `NVD Returned Status Code: 403` or `404` → your key is wrong or not activated
> (Prerequisite 05). Tell a volunteer.

🍽️ **Go to lunch.** Don't close the terminal window (closing is OK too — `nohup` keeps it running).

---

## 📚 Part B — Concepts (14:00)

### B1. What is SCA?

**SCA — Software Composition Analysis** finds **known vulnerabilities in the open-source libraries** your application
depends on (directly or transitively).

> 📦 Typically **70–90 % of a modern application's code is open-source libraries**. You didn't write it, but you ship it.
> Run `mvn dependency:tree` — `workshop-app` has ~10 lines of `<dependency>` but pulls in dozens of jars.

### B2. Software supply chain risk — real examples

| Incident | Library | Impact |
|---|---|---|
| **Log4Shell** (2021, CVSS 10.0) | Apache Log4j 2 | Remote code execution via a log message; affected a huge share of Java applications |
| **Equifax breach** (2017) | Apache Struts 2 | Unpatched known CVE → data of ~147 million people |
| **Text4Shell** (2022, CVSS 9.8) | **Apache Commons Text** ≤ 1.9 | Code execution via string interpolation — **it's in our app** |

### B3. How OWASP Dependency-Check works

```text
pom.xml + jars → analyzers collect EVIDENCE → identify library (CPE + purl) → match CVEs in local NVD DB (/opt/dc-data) → HTML/JSON report + fail if CVSS ≥ 9
```

| Term | Meaning |
|---|---|
| **NVD** | U.S. National Vulnerability Database — public list of CVEs with CVSS scores |
| **CPE** | *Common Platform Enumeration* — standard name for a product+version, e.g. `cpe:2.3:a:apache:commons_text:1.9` |
| **purl** | *Package URL* — e.g. `pkg:maven/org.apache.commons/commons-text@1.9` |
| **Evidence** | Clues used to guess the CPE. Weak evidence → possible **false positives** |
| **Suppression file** | XML file to mark reviewed false positives so they stop failing the build |

### B4. How it's configured in `workshop-app` (already in `pom.xml`)

```bash
grep -A 25 "<artifactId>dependency-check-maven" ~/workshop-app/pom.xml
```

| Setting | Value | Meaning |
|---|---|---|
| `dataDirectory` | `/opt/dc-data` | Shared DB folder (Part A) |
| `nvdApiKeyEnvironmentVariable` | `NVD_API_KEY` | Read the API key from the environment |
| `failBuildOnCVSS` | **`9`** | **Gate:** fail if any library has a vulnerability with CVSS ≥ 9.0 (**CRITICAL**) |
| `formats` | `HTML`, `JSON` | Human report + machine-readable report |
| `ossIndexAnalyzerEnabled` | `false` | The Sonatype OSS Index service now requires an account — disabled for the lab |
| `retireJs…`, `node…`, `assembly…` | `false` | JavaScript/.NET analyzers not needed for a Java app → faster scans |

> 💡 **Choosing the threshold is a policy decision.** `9` blocks CRITICAL only (reasonable for a first rollout);
> mature teams often use `7` (HIGH and above).

---

## 🛠️ Part C — Check that the download finished ☁️ MAIN

```bash
tail -n 5 ~/dc-update.log
```

✅ **Expected:**

```text
[INFO] BUILD SUCCESS
```

```bash
du -sh /opt/dc-data
```
→ a few hundred MB.

| If you see | Do this |
|---|---|
| Still `Downloaded ... (85%)` lines | Wait a few minutes; run `tail -n 5 ~/dc-update.log` again |
| `BUILD FAILURE` with `NVD ... 503` / timeout | NVD was busy. Re-run A2 (key) + A3; it resumes from where it stopped |
| No log file | Part A was skipped — do it now (A1–A3) and wait |

---

## 🛠️ Part D — Scan the application ☁️ MAIN

```bash
cd ~/workshop-app
```

**D1.** Run the scan using the local database **without** updating it (`-DautoUpdate=false` → fast, no API key needed):

```bash
mvn -B dependency-check:check -DautoUpdate=false
```

> ⏳ 1–3 minutes.

✅ **Expected — the gate fails:**

```text
[WARNING]

One or more dependencies were identified with known vulnerabilities in workshop-app:

commons-text-1.9.jar (pkg:maven/org.apache.commons/commons-text@1.9, cpe:2.3:a:apache:commons_text:1.9:*:*:*:*:*:*:*) : CVE-2022-42889
...
[ERROR] Failed to execute goal org.owasp:dependency-check-maven:12.2.2:check (default-cli) on project workshop-app:
[ERROR]
[ERROR] One or more dependencies were identified with vulnerabilities that have a CVSS score greater than or equal to '9.0':
[ERROR]
[ERROR] commons-text-1.9.jar: CVE-2022-42889(9.8)
...
[INFO] BUILD FAILURE
```

```bash
echo "Exit code: $?"
```
→ `Exit code: 1` 🚦

> 🔎 You may also see other libraries listed with lower scores (e.g. MEDIUM/HIGH findings in transitive dependencies).
> They are **reported** but don't break the build because they're below the `9.0` threshold.

> 🆕 **The list depends on the day you run it.** New CVEs are published daily, so besides `commons-text` you may also
> see CRITICAL findings for `spring-core` and `tomcat-embed-core` (libraries that Spring Boot brings in). Part E
> handles all three cases.

**D2.** Summarise the JSON report with `jq` — library → CVE → severity → score:

```bash
jq -r '.dependencies[] | select(.vulnerabilities) | .fileName as $f | .vulnerabilities[] | "\($f)  \(.name)  \(.severity)  \(.cvssv3.baseScore // "-")"' target/dependency-check-report.json
```

```text
commons-text-1.9.jar  CVE-2022-42889  CRITICAL  9.8
...
```

**D3.** Open the **HTML report** on your laptop. 💻 In your **laptop** terminal (not SSH!):

```bash
cd ~/devsecops
```

```bash
scp -i devsecops-key.pem ubuntu@<MAIN_SERVER_IP>:~/workshop-app/target/dependency-check-report.html .
```

Then open `dependency-check-report.html` from your `devsecops` folder in a browser (double-click it in File
Explorer / Finder).

> 💡 `scp` = **s**ecure **c**o**p**y — copies files over SSH using the same key. Syntax: `scp <from> <to>`.

In the report: scroll to **commons-text-1.9.jar** → read the CVE description, CVSS vector, references and the
**Evidence** section (how it identified the library).

### What is Text4Shell (CVE-2022-42889)?

`commons-text` versions **1.5 – 1.9** perform **variable interpolation** on strings like `${prefix:value}`. Some
prefixes (`script:`, `dns:`, `url:`) are enabled by default — so if an application passes **attacker-controlled
text** to `StringSubstitutor`, the attacker may run code or make the server contact other systems. **Fixed in 1.10.0.**

---

## 🛠️ Part E — Fix: upgrade, override, or document ☁️ MAIN

There are three kinds of CRITICAL finding, and **each needs a different fix**:

| Finding | Why it appears | Fix |
|---|---|---|
| **`commons-text` 1.9** | A library **you** declared in `pom.xml` is old | **E1** — change its version |
| **`tomcat-embed-core`** | Spring Boot pins a Tomcat version; newer CVEs were published after that Boot release | **E2** — override the version with a newer patch release |
| **`spring-core`** (and other Spring jars) | The newest Spring 6.2.x **has no fixed release yet** | **E3** — document a **temporary suppression** (nothing to upgrade to) |

**E1. Upgrade `commons-text`.**

```bash
grep -n -B2 -A1 "<version>1.9</version>" pom.xml
```

```text
NN-            <groupId>org.apache.commons</groupId>
NN-            <artifactId>commons-text</artifactId>
NN:            <version>1.9</version>
```

```bash
sed -i 's#<version>1.9</version>#<version>1.15.0</version>#' pom.xml
```

```bash
grep -n -B2 "<version>1.15.0</version>" pom.xml
```

✅ shows `commons-text` with `1.15.0`.

Upgrades can break code — run the unit tests:

```bash
mvn -B -q test
```

✅ No errors.

**E2. Override the Tomcat version.** First look for the newest 10.1.x release:

```bash
curl -s https://repo1.maven.org/maven2/org/apache/tomcat/embed/tomcat-embed-core/maven-metadata.xml | grep "<version>10.1" | tail -3
```

The last line is the newest. Put it in `pom.xml` as a property (replace `10.1.60` below if yours is newer):

```bash
cp pom.xml pom.xml.bak
```

```bash
sed -i '/<dependency-check.version>/a\        <tomcat.version>10.1.60</tomcat.version>' pom.xml
```

```bash
mvn -B dependency:tree | grep tomcat-embed-core
```

✅ must show the new version (e.g. `10.1.60`), not `10.1.55`.

> 💡 Spring Boot manages library versions through **properties** such as `tomcat.version`. Setting the property
> in your own `pom.xml` **overrides** the Boot default — the fix for a *transitive* dependency.

**E3. Record a temporary suppression for what has no fix.** Check first that no fixed release exists:

```bash
curl -s https://repo1.maven.org/maven2/org/springframework/spring-core/maven-metadata.xml | grep "<version>6.2" | tail -3
```

If the newest version is the one you already use, there is nothing to upgrade to. Do **not** lower the CVSS
threshold and do **not** skip the scan — **document** the exception instead.

Create the suppression file. Copy the **CVE numbers from your own `[ERROR]` line** for `spring-core` (they change
as new CVEs are published; the ones below are examples from 3 Oct 2026):

```bash
cat > dependency-check-suppressions.xml <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <suppress until="2026-11-15Z">
        <notes>No fixed Spring Framework 6.2.x release yet. Re-check on the next Spring Boot patch.</notes>
        <packageUrl regex="true">^pkg:maven/org\.springframework/spring-.*@.*$</packageUrl>
        <cve>CVE-2026-47884</cve>
        <cve>CVE-2026-47892</cve>
        <cve>CVE-2026-47891</cve>
        <cve>CVE-2026-47890</cve>
        <cve>CVE-2026-59313</cve>
        <cve>CVE-2026-59283</cve>
    </suppress>
</suppressions>
EOF
```

Tell the plugin to use it (adds three lines inside `<configuration>`, right after `<failBuildOnCVSS>`):

```bash
sed -i -e '/<failBuildOnCVSS>/a\                    <suppressionFiles>' -e '/<failBuildOnCVSS>/a\                        <suppressionFile>dependency-check-suppressions.xml</suppressionFile>' -e '/<failBuildOnCVSS>/a\                    </suppressionFiles>' pom.xml
```

```bash
diff pom.xml.bak pom.xml
```

✅ shows **four** added lines: `tomcat.version` and the three `suppressionFiles` lines. (Made a mistake? Restore with
`cp pom.xml.bak pom.xml` and redo E2–E3 once.)

| Why this and not the alternatives? | |
|---|---|
| Lower `failBuildOnCVSS` / skip the scan | ❌ hides **every** future critical CVE too |
| Jump to another Spring major version | ❌ Spring Boot 3.5 is built for Spring 6.2 — can break the app |
| **Suppress the specific CVEs with a reason + `until` date** | ✅ the gate stays strict; after the date the build **fails again**, forcing a re-check |

> ⚠️ A suppression is **not a fix**. It records a **temporary, reviewed risk acceptance**.

**E4.** Rescan:

```bash
mvn -B clean verify dependency-check:check -DautoUpdate=false
```

✅ **Expected:**

```text
[INFO] BUILD SUCCESS
```

Lower-severity findings (below 9.0) may still be listed as `[WARNING]` — they appear in the report but **don't block** the build.
Anything still named in an `[ERROR]` line needs the same treatment: **upgrade if a fix exists, otherwise a dated suppression.**

**E5.** Commit and push — include the suppression file (Jenkins checks out the repo from GitHub in Lab 06, so it **must** be pushed):

```bash
git add pom.xml dependency-check-suppressions.xml
```

```bash
git commit -m "Fix dependency CVEs: commons-text 1.15.0, Tomcat override, documented spring-core suppression"
```

```bash
git push
```

> 🔗 Remember **Day 1, Lab 06**: Trivy found the same CVE **inside the Docker image**. Once the pipeline rebuilds the
> jar and image in Lab 06, that CRITICAL disappears from the Trivy report too — **one fix, two gates turn green.**

---

## 🛠️ Part F — Suppressions: false positives and "no fix yet" (read)

You just wrote a suppression for **"no fix yet"**. The other classic reason is a **false positive**: Dependency-Check maps a
library to the wrong CPE and reports CVEs that don't apply. In both cases the professional response is **not** to raise
the threshold, but to:

1. **Verify** (read the CVE + the *Evidence* section of the report).
2. Add a **suppression** with a justification (`<notes>`), the exact CVEs, and — for temporary decisions — an `until` date.
3. **Commit the file** and **review suppressions regularly** (`until` makes the build fail again when it expires).

```xml
<suppress>
   <notes>False positive: CPE for 'foo-client' matches unrelated product 'foo server'. Reviewed by A. Kumar, 2026-10-06.</notes>
   <packageUrl regex="true">^pkg:maven/com\.example/foo-client@.*$</packageUrl>
   <cve>CVE-2099-12345</cve>
</suppress>
```

---

## 🛡️ Security angle — summary

| Practice | Why |
|---|---|
| Scan dependencies on every build | New CVEs are published daily — yesterday's "clean" build may be vulnerable today |
| Keep the vulnerability DB updated (scheduled job) | A stale DB misses new CVEs |
| Gate on a CVSS threshold | Turns policy into an automatic decision |
| Upgrade + **run tests** | Fix the vulnerability without breaking the app |
| Suppress only with justification **and an expiry date** | Audit trail; the build fails again when the date passes, so the risk is re-reviewed |
| Override transitive versions with a property | Fixes libraries that Spring Boot pins, without changing the framework |
| API keys from environment variables | No secrets in `pom.xml` |

---

## ❌ Common errors

| You see | Why | Fix |
|---|---|---|
| `An NVD API Key was not provided - it is highly recommended...` and very slow download | `NVD_API_KEY` not exported in this terminal | Repeat A2 (`read` + `export`) and restart A3 |
| `NVD Returned Status Code: 403/404` | Invalid / unactivated key | Re-activate from the NVD email (Prerequisite 05) |
| `Unable to create the data directory '/opt/dc-data'` / `AccessDeniedException` | Folder missing or wrong permissions | Repeat A1 exactly |
| `Unable to continue dependency-check analysis` / `NoDataException: No documents exist` | Database download didn't finish | Check `~/dc-update.log`; re-run A3 (with the key exported) |
| `Database is locked` / `The database ... is already in use` | Two scans running at once (e.g. background update still running) | Wait for the update to finish (`ps aux \| grep update-only`), then rescan |
| Many `OSS Index` errors | OSS Index analyzer enabled without an account | Make sure you didn't remove `ossIndexAnalyzerEnabled=false` from the pom |
| Build still fails after the `commons-text` upgrade | Other libraries (Tomcat, Spring) also have CRITICAL CVEs | Read the `[ERROR]` list; apply E2 (override) or E3 (suppression) |
| `tomcat-embed-core` still shows the old version | The `tomcat.version` line is missing or outside `<properties>` | `grep -n tomcat.version pom.xml`; `mvn dependency:tree \| grep tomcat-embed-core` |
| Suppression has no effect | File not created, or `<suppressionFiles>` missing from the pom, or CVE list incomplete | `ls dependency-check-suppressions.xml`; `grep -n suppression pom.xml`; copy CVEs from your own `[ERROR]` line |
| Build passes locally but fails in Jenkins | `dependency-check-suppressions.xml` not pushed to GitHub | `git add` + commit + push the file |
| Build fails again after a few weeks | The suppression's `until` date has passed | Check for a fixed release; upgrade, or renew the date with a new note |
| `scp: ... No such file or directory` | Scan didn't produce a report / wrong path | Run D1 first; check `ls ~/workshop-app/target/*.html` |

---

## 🎤 Interview questions

<details>
<summary><b>1. What is the difference between a direct and a transitive dependency vulnerability, and how do you fix the latter?</b></summary>

A direct dependency is declared in your `pom.xml`; a transitive one is pulled in by another library. Fix a transitive
one by upgrading the parent library, or by overriding the version (e.g. `<dependencyManagement>` in Maven) — and
re-testing.
</details>

<details>
<summary><b>2. Why might SCA tools report false positives?</b></summary>

They identify libraries from evidence (names, manifests) and map them to CPEs; similar names or ambiguous metadata
can map to the wrong product or version range.
</details>

<details>
<summary><b>3. What is an SBOM and how does it relate to SCA?</b></summary>

A Software Bill of Materials (e.g. CycloneDX, SPDX) is an inventory of all components in an application. SCA tools
can generate SBOMs and continuously match them against new vulnerability data.
</details>

<details>
<summary><b>4. Dependency-Check vs Trivy for a Java app?</b></summary>

Both can find vulnerable Java libraries. Dependency-Check focuses on application dependencies (deep Java evidence
analysis, NVD-based). Trivy covers the whole container image — OS packages, language libraries, misconfigurations
and secrets. Using both gives defence in depth.
</details>

---

➡️ **Next lab — the final one:** [06 — Securing a Complete Declarative Pipeline](06-Secure-Declarative-Pipeline-Lab.md)
