---
description: Some Web List of Easy Challenges Labs for Web exploit
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

#### **Lab 3: DOM-Based XSS in Dynamic Theme Switcher (client side)**

* **Attack Type:** DOM-Based XSS
* **Name:** Themed Deception
* **Scenario:** A modern web app has a client-side theme switcher. The selected theme is read from the URL hash (e.g., `#theme=dark`) and written directly into the DOM using `innerHTML` without sanitization. This is a pure client-side vulnerability; the server is not involved.
* **Where the Flag is Hidden:** The flag is in the browser's Local Storage (`localStorage.getItem('flag')`).
* **How to Solve:**
  1. The attacker crafts a URL where the `theme` parameter is not a color, but a script.
  2. **Payload:** `http://vulnerable-site.com/theme#theme=red;</style><script>alert(localStorage.getItem('flag'))</script>`
  3. This payload first closes the `<style>` tag and then injects a new `<script>` tag. When the victim visits this URL, the malicious script executes, accessing Local Storage and exfiltrating the flag. The server never sees the payload; the attack happens entirely in the victim's browser.

#### **Lab 4:** Search Poisoning (client side)

* **Attack Type:** Reflected XSS (Cross-Site Scripting)
* **Level:** Easy
* **Name:** Search Poisoning
* **Scenario:** "BloggerNet" is a popular blogging platform. It has a search feature that displays your query back on the page if no results are found. The developer forgot to sanitize this input. Your goal is to pop an alert box proving the vulnerability.
* **Where the Flag is Hidden:** The flag is in the HTML source code of the search results page, inside a comment. You need an XSS payload to read it.
* **How to Solve:**
  1. **Find the Input:** The attacker goes to the search bar on the blog's homepage.
  2. **Test for XSS:** They enter a simple test payload: `<script>alert('XSS')</script>`
  3. **Observe the Result:** After hitting search, an alert box pops up. This confirms the site is vulnerable to Reflected XSS because it reflects the user input directly into the HTML without sanitization.
  4. **Find the Flag:** The page with the search results contains the flag hidden in a comment: `<!-- Flag: RAZZ{xss_1s_3verywh3r3} -->`
  5. **Steal the Flag (The Challenge):** Simply popping an alert is the proof. To actually _steal_ the flag, a more advanced payload is needed. For this easy lab, the goal is just to trigger the alert, proving the vulnerability exists. The flag in the comment is a bonus for those who view the page source.
* **Flag:** `RAZZ{xss_1s_3verywh3r3}`



### Path traversal

#### **Lab 5: Forced Download + Path Traversal**&#x20;

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

#### **Lab 6: Bypassing Blacklist Filters**&#x20;

* **Attack Type:** File upload with blacklist bypass
* **Name:**  The Forbidden Upload
* **Scenario:** A document sharing service for a company intranet. The developer has implemented a blacklist of dangerous extensions (e.g., `.php`, `.py`, `.exe`). The goal is to bypass this list.
* **Where the Flag is Hidden:** The flag is in the database, accessible via a Django command: `python manage.py get_flag`.
*   **How to Solve:**

    1. The attacker must find an alternative extension that the server will still execute. For Apache servers, `.php5`, `.phtml`, `.phps` are common backups.
    2. **Payload:** Rename the `shell.php` file to `shell.phtml` and upload it.
    3. If the server is configured to execute `.phtml` as PHP, the web shell will work. The attacker can then use the shell to run the Django command: `python manage.py get_flag`.



### Week Authentication

#### **Lab 7: Week Authentication**

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



#### **Lab 8: The Robot's Secret**

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



### Race Condition

#### **Lab 9: The Speedy Shopper**

* **Attack Type:** Race Condition
* **Level:** Easy
* **Name:** The Speedy Shopper
* **Scenario:** "FlashSale," an e-commerce site, has a special limited-time offer: the first 100 users to click a button get a discount coupon for 100% off. The coupon can only be claimed once per user. The site checks your account balance _before_ applying the coupon, but the process of claiming it isn't instant. Can you beat the system?
* **Where the Flag is Hidden:** The flag is the discount coupon code itself, which you receive upon successfully claiming it more than once. The coupon code is the flag: `RAZZ{sp33d_d3m0n_win5}`.
* **How to Solve:**
  1. **Understand the Flow:** The user clicks "Claim Coupon." The application does this:
     * `Step 1:` Check if the user has already claimed a coupon (database read).
     * `Step 2:` If not, add the coupon to the user's account (database write).\
       This `read-then-write` sequence creates a tiny window for a race condition.
  2. **Exploit with Parallel Requests:** The attacker uses a tool like `Burp Suite Turbo Intruder` or writes a simple script to send **10-20 simultaneous POST requests** to the "Claim Coupon" endpoint (`POST /claim-coupon`).
  3. **The Race:** Due to the slight delay between the read (check) and write (apply), multiple requests might all pass the "has this user claimed?" check before any of them finish writing to the database. This results in the coupon being applied multiple times to the same user's account.
  4. **Success:** The page refreshes, showing the user has claimed the coupon 2, 3, or more times. The coupon code (which is the flag) is displayed each time.
* **Flag:** `RAZZ{sp33d_d3m0n_win5}`



### LFI

#### **Lab 10:  Document Viewer**

* **Attack Type:** Local File Inclusion (LFI)
* **Level:** Easy
* **Name:**  Directory Diver
* **Scenario:** "CompanyPortal" has a feature that lets employees view their own documents. The URL uses a parameter like `?file=quarterly_report.pdf`. A new intern accidentally deployed a version of the code that doesn't properly restrict which files can be loaded.
* **Where the Flag is Hidden:** The flag is on the server at `/etc/flag.txt`.
* **How to Solve:**
  1. **Find the Vulnerable Parameter:** The URL looks like:\
     `http://companyportal.com/view?file=doc1.pdf`
  2. **Test for LFI:** The attacker tries to break out of the intended directory using Path Traversal sequences:\
     **Payload:** `http://companyportal.com/view?file=../../../../etc/passwd`
  3. **Success:** If the server responds with the contents of the `/etc/passwd` file, LFI is confirmed.
  4. **Read the Flag:** Now the attacker simply changes the path to the flag file:\
     **Final Payload:** `http://companyportal.com/view?file=../../../../etc/flag.txt`
  5. The server includes the local file `/etc/flag.txt` and displays its contents on the page, revealing the flag.
* **Flag:** `RAZZ{lfi_br0adens_y0ur_h0riz0ns}`

### IDOR

#### **Lab 11: The Profile Peeker**

* **Attack Type:** Insecure Direct Object Reference (IDOR)
* **Level:** Easy
* **Name:** Number Swap
* **Scenario:** "Connectify" is a new social media platform. Users have profiles accessible at URLs like `/profile?user_id=5`. You have a standard account with `user_id=101`. Your goal is to view the private profile of the site administrator.
* **Where the Flag is Hidden:** The flag is displayed on the admin's private profile page (`user_id=1`).
* **How to Solve:**
  1. **Log In:** The user logs into their own account at `user_id=101`. Their profile URL is `http://connectify.com/profile?user_id=101`.
  2. **Change the Parameter:** The attacker simply changes the `user_id` parameter in the URL to `1` (or another number likely to be the admin).
  3. **Payload:** `http://connectify.com/profile?user_id=1`
  4. **Success:** The page loads the admin's profile, which contains sensitive information and the flag, because the backend failed to check if the logged-in user had permission to view that specific profile.
* **Flag:** `RAZZ{1d0r_4cc3ss_gr4nt3d}`

### Expose Directory

#### **Lab 12: The Exposed Directory**

* **Attack Type:** Security Misconfiguration (Directory Listing)
* **Level:** Easy
* **Name:** The Index Finger
* **Scenario:** "DevStart," a startup's website, has a section for downloading brochures at `/downloads/`. The developer forgot to disable a common server feature before pushing the site to production.
* **Where the Flag is Hidden:** The flag is inside a file named `flag.txt` that is sitting in the `/downloads/` directory.
* **How to Solve:**
  1. **Find the Directory:** The attacker visits `http://devstart.com/downloads/`.
  2. **Discover the Misconfiguration:** Instead of a custom webpage, the browser displays a raw list of files (e.g., `brochure.pdf`, `pricelist.xlsx`, `flag.txt`). This is called "directory listing" and is enabled by default on many web servers.
  3. **Retrieve the Flag:** The attacker simply clicks on the `flag.txt` link in the list. The browser displays the contents of the file, revealing the flag.
* **Flag:** `RAZZ{l1st3d_4nd_exposed}`

