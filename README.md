---
icon: users-rectangle
---

# CacheMe

## Description

You're an intern integrating "CyberCorp" the new in-house caching library. The API is flawless, but the developer portal hides strange config snippets, references to shadow endpoints and dormant feature flags the senior devs dismiss as "internal tools."

Your code works, but your instincts scream. This library isn't just storing data; it's hiding something. While the team celebrates the performance boost, you're digging, convinced that the very thing making the system faster is also its most dangerous blind spot. You need to find the flaw they either don't see or don't want you to see.

Creds:&#x20;

* [eve.intern@cybercorp.com](mailto:eve.intern@cybercorp.com)
* EveIntern2024!



### Cache **Deception**&#x20;

#### **What is Cache Deception?**

Cache Deception is a web vulnerability where an attacker tricks a caching server (like Varnish, CDNs, or reverse proxies) into storing and serving a sensitive, user-specific page as a public cached resource. Unlike Cache Poisoning (where malicious content is injected), here the attacker steals cached private data from other users.

#### **How Does it Work?**

1. **The Trick**: An attacker requests a non-sensitive page with a deceptive URL, like `example.com/account/profile.css`.
2. **Server Misunderstanding**: The server might ignore the `.css` extension and serve the private `/account/profile` page.
3. **Caching Error**: The caching system sees `.css` (a "cacheable" static extension) and stores the private page.
4. **Data Leak**: When another user visits `example.com/account/profile.css`, they receive the attacker’s cached private page.

#### **Real-World Impact**

* **Session hijacking**: Cached pages may contain CSRF tokens, authentication cookies, or personal data.
* **Information disclosure**: User-specific dashboards, API keys, or financial data can be exposed.
* **Widespread effect**: One request can leak data to many users across a CDN.

#### **The Root Cause**

The vulnerability arises from a mismatch:

* **Cache key**: Often based on URL extensions (treating `.css` as static).
* **Backend behavior**: Dynamic routes that ignore extensions, serving private content.

#### **How to Protect Your Application**

1. **Cache Intelligently**: Avoid caching based solely on file extensions. Use explicit cache-control headers (`private`, `no-store`) for authenticated pages.
2. **User-Specific Caching**: Configure caches to include session identifiers or authentication state in cache keys.
3. **Path Validation**: Ensure static file handlers validate paths before serving content.
4. **Security Headers**: Implement `Vary: Cookie` headers to differentiate cached responses by user.

#### **The Bottom Line**

Cache Deception turns a performance feature into a privacy threat. By understanding caching logic and applying security-aware configurations, developers can prevent this subtle but dangerous vulnerability.

## CacheMe Overview and flow of Solution

1. Login with given credentials

<figure><img src="../.gitbook/assets/unknown.png" alt=""><figcaption></figcaption></figure>

<br>

2. Head over to doc you will see many files here trying to open each file one by one but you see most of the files can’t be accessed directly as an intern user, for internal walkthrough our file target is\_\_\_\_\_\_\_\_\_\_. It stores our files.

<figure><img src="../.gitbook/assets/unknown (1).png" alt=""><figcaption></figcaption></figure>

<br>

3. Use dirsearch to get some directory listing you will find a logs directory

<figure><img src="../.gitbook/assets/unknown (2).png" alt=""><figcaption></figcaption></figure>

4. Head over to http://\<url>\_32/logs, you will notice 2 things: it's a log file of this application.&#x20;

<figure><img src="../.gitbook/assets/unknown (3).png" alt=""><figcaption></figcaption></figure>

And a request Header is also leaked X-INTERNAL\_AUTH=internal\_trusted\_service\_2025

<figure><img src="../.gitbook/assets/unknown (4).png" alt=""><figcaption></figcaption></figure>

5. Turn on the proxy of burp suit and capture the request of opening the file shown in picture and send it to repeater

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<br>

Change the Method to POST and change the path to /cdn/push to cash this file and also include the header that we found in logs

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

We got response&#x20;

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Means the file is cached as as now stop the proxy and click again on the document&#x20;



6. Click on the file and get the content

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>



<br>
