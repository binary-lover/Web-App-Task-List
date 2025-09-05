---
description: Some Web List of Easy Challenges Labs for Web expllot
icon: list-radio
---

# Easy Challenges List

### SQLi Lab

#### **Lab 1: The Classic Login Bypass**&#x20;

* **Attack Type:** Union-Based or Authentication Bypass
* **Name:** Always True
* **Level:** Easy
* **Scenario:** A simple login page for a "Company Internal Portal". The user is told they are a new intern and need to get into the portal, but they don't have a password yet. The hint is: "What if you could trick the system into thinking you're someone you're not?"
* **Where the Flag is Hidden:** The flag is returned on a successful login. `RAZZ{l0g1n_6yp455}`
* **How to Solve:**
  1. Enter a classic payload in the username field: `' OR '1'='1' --`
  2. This comments out the password check and makes the query always true.
  3. The `UNION SELECT` technique can also be used to retrieve the flag directly if the number of columns is known.

#### Lab 2:  The Product Filter Leak

* **Name:** Filter Frenzy
* **Scenario:** You're browsing an e-commerce site, "GadgetHub." You can filter products by category (e.g., `category=phones`). Your goal is to extract the hidden flag from the database.
* **Vulnerability:** Union-Based SQLi in a `GET` parameter.
* **How to Solve:**
  1. Test the `category` parameter: `category=phones'--` (see if it breaks the query).
  2. Find the number of columns: `category=phones' ORDER BY 3--` (increment until error).
  3. Once number of columns is known (e.g., 3), use UNION SELECT to retrieve the flag:\
     `category=phones' UNION SELECT 1,@@version,3--`\
     `category=phones' UNION SELECT 1,table_name,3 FROM information_schema.tables--`\
     `category=phones' UNION SELECT 1,flag,3 FROM secret_table--`
* **Flag:** `RAZZ{un10n_5ucc3ss}`



### XSS

#### **Lab 3: DOM-Based XSS in Dynamic Theme Switcher**&#x20;

* **Attack Type:** DOM-Based XSS
* **Name:** Themed Deception
* **Scenario:** A modern web app has a client-side theme switcher. The selected theme is read from the URL hash (e.g., `#theme=dark`) and written directly into the DOM using `innerHTML` without sanitization. This is a pure client-side vulnerability; the server is not involved.
* **Where the Flag is Hidden:** The flag is in the browser's Local Storage (`localStorage.getItem('flag')`).
* **How to Solve:**
  1. The attacker crafts a URL where the `theme` parameter is not a color, but a script.
  2. **Payload:** `http://vulnerable-site.com/theme#theme=red;</style><script>alert(localStorage.getItem('flag'))</script>`
  3. This payload first closes the `<style>` tag and then injects a new `<script>` tag. When the victim visits this URL, the malicious script executes, accessing Local Storage and exfiltrating the flag. The server never sees the payload; the attack happens entirely in the victim's browser.



### Path traversal

#### **Lab 4: Forced Download + Path Traversal**&#x20;

* **Vulnerabilities:** Unrestricted File Download (Forced Download) + Path Traversal
* **Name:** The Backup Heist
* **Scenario:** A simple view allows users to download their own uploaded files by filename. The code checks if the file exists in the upload directory but does not validate the filename for path traversal.
* **Where the Flag is Hidden:** The flag is at `/var/backups/app.db`.
* **How to Solve:**
  1. The attacker uses path traversal to download any file on the system.
  2. **Payload:** `http://vulnerable-site.com/download?file=../../../../var/backups/app.db`
  3. The `os.path.join` creates the path `/opt/app/uploads/../../../../var/backups/app.db`, which normalizes to `/var/backups/app.db`.
  4. The weak `startswith` check passes because the constructed path _does_ start with `/opt/app/uploads/` before normalization.
  5. The file is found and downloaded by the browser. This lab highlights the importance of normalizing the path _before_ performing security checks.

### File Upload

#### **Lab 5: Bypassing Blacklist Filters**&#x20;

* **Attack Type:** File upload with blacklist bypass
* **Name:**  The Forbidden Upload
* **Scenario:** A document sharing service for a company intranet. The developer has implemented a blacklist of dangerous extensions (e.g., `.php`, `.py`, `.exe`). The goal is to bypass this list.
* **Where the Flag is Hidden:** The flag is in the database, accessible via a Django command: `python manage.py get_flag`.
* **How to Solve:**
  1. The attacker must find an alternative extension that the server will still execute. For Apache servers, `.php5`, `.phtml`, `.phps` are common backups.
  2. **Payload:** Rename the `shell.php` file to `shell.phtml` and upload it.
  3. If the server is configured to execute `.phtml` as PHP, the web shell will work. The attacker can then use the shell to run the Django command: `python manage.py get_flag`.

#### **Lab 6: Week Authentication**

* **Attack Type:** Weak Authentication (Default Credentials)
* **Level:** Easy
* **Name:** The Lazy Admin
* **Scenario:** You are a security auditor for "SecureTech Inc." Your first task is to perform a basic penetration test on their new employee portal. The development team was rushed to meet a deadline and may have cut corners on initial setup. The login portal is at `/admin`.
* **Where the Flag is Hidden:** The flag is displayed on the admin dashboard after successful login.
* **How to Solve:**
  1. The attacker attempts to log in with common default credentials.
  2. Payload 1 (Username/Password): `admin / admin`
  3. Payload 2 (Username/Password): `admin / password`
  4. Payload 3 (Username/Password): `administrator / SecureTech123` (a weak, company-specific password)
  5. One of these combinations will successfully log the user into the admin dashboard, where the flag is displayed in a welcome message.
* **Flag:** `RAZZ{d3f4ul7_5up3r_53cr3t}` &#x20;



#### **Lab 7: The Robot's Secret**

* **Attack Type:** Information Disclosure → Weak Hash Cracking
* **Level:** Easy
* **Name:**  The Robot's Confession
* **Scenario:** You're checking out a new site, "TechTronix." You remember that a `robots.txt` file can sometimes list interesting directories. Maybe you can find something useful there.
* **Where the Flag is Hidden:** The flag is displayed after successfully logging into the admin panel.
* **How to Solve:**
  1. **Step 1: Find the clue.** The attacker visits `https://techtronix.com/robots.txt`.
  2.  **The File's Contents:**

      text

      ```
      User-agent: *
      Disallow: /admin
      Disallow: /backup
      Disallow: /secret_note.txt
      ```
  3. **Step 2: Discover the hash.** The attacker visits `https://techtronix.com/secret_note.txt`. The file contains a note: `// TODO: Remove this before production. Admin hash for now: e6b061c9677b4e6e7d6925d6c269ac47`
  4. **Step 3: Crack the hash.** The attacker recognizes the 32-character string as an MD5 hash. Using an online cracker (like CrackStation) or a tool like `hashcat`, they crack the hash. The plaintext is `secret123`.
  5. **Step 4: Login.** The attacker visits `https://techtronix.com/admin` and logs in with the credentials `admin : secret123`.
  6. The admin dashboard loads, displaying the flag.
* **Flag:** `RAZZ{r0b0ts_t0_riches}`

