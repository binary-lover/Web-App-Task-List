---
description: List of Medium web Challenge list
icon: list-radio
---

# Medium Challenge List

### Web Server Challenges

#### &#x20;**1. HTTP Request Smuggling** The Classic CL.TE Bypass

* **Attack Type:** HTTP Request Smuggling - CL.TE Vulnerability
* **Name:** Front Door Bypass
* **Level:** Medium
* **Scenario:** A web application is protected by a security gateway that blocks direct access to the admin panel. The gateway and back end server handle HTTP headers differently, creating a request smuggling vulnerability.
* **Where the Flag is Hidden:** The flag is returned when successfully accessing `/admin/internal` endpoint. `RAZZ{http_r3qu3st_5muggl1ng_clt3}`

**How to Solve:**

1. Identify the CL.TE (Content-Length vs Transfer-Encoding) vulnerability between the frontend and backend servers
2. Craft a smuggled request that bypasses the security gateway
3. Use the vulnerability to access the blocked admin endpoint

**Attack Payload:**

```http
POST / HTTP/1.1
Host: vulnerable-server.com
Content-Length: 41
Transfer-Encoding: chunked

0

GET /admin/internal HTTP/1.1
X-Ignore: x
```



#### **2. HTTP Verb Tampering with TRACE**

**Attack Type:** HTTP Method Abuse - TRACE Method\
**Name:** Verb Tampering Attack\
**Level:** Medium\
**Scenario:** The TRACE method is enabled, allowing attackers to bypass security controls and perform cross-site tracing attacks.

**Where the Flag is Hidden:** The flag appears in the TRACE response when specific headers are sent. `RAZZ{tr4c3_m3th0d_3nabl3d}`

**How to Solve:**

1. Discover TRACE method is enabled
2. Send TRACE request with special headers
3. Analyze response for the flag
4. Chain with XSS for cross-site tracing

**Attack Payload:**

```http
TRACE / HTTP/1.1
Host: target.com
X-Flag: RAZZ{tr4c3_m3th0d_3nabl3d}
Custom-Header: test
```

\


### **3. Cache Deception with Parameter Pollution**

**Attack Type:** Web Cache Deception + HTTP Parameter Pollution\
**Name:** Parameter Cache Bypass\
**Level:** Medium\
**Scenario:** The cache key doesn't include all parameters, allowing cache deception through parameter pollution. Sensitive pages can be cached and accessed without authentication.

**Where the Flag is Hidden:** The flag is in cached search results at `/search?q=flag&format=css`. `RAZZ{p4r4m_p0llut3d_c4ch3}`

**How to Solve:**

1. Find parameters ignored by cache key
2. Chain parameters to cache sensitive data
3. Access cached data without proper authorization
4. Extract flag from poisoned cache entry

**Attack Flow:**

```http
# First request caches sensitive data:
GET /search?q=internal_docs&user=admin&format=css HTTP/1.1
Cookie: admin_session

# Attacker accesses cached version:
GET /search?q=internal_docs&user=admin&format=css HTTP/1.1
```
