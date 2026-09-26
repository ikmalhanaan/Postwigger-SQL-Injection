# Lab 20: Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

**Goal**: Submit a comment containing a malicious Website URL payload that executes `alert(1)` when the comment author's name is clicked in an `onclick` event attribute.

This lab contains a stored cross-site scripting vulnerability in the comment author link functionality. The input is reflected inside an inline `onclick` event handler (`onclick="trackBlogs('...')"`). Angle brackets (`<`, `>`) and double quotes (`"`) are HTML-encoded, while single quotes (`'`) and backslashes (`\`) are backslash-escaped by the server.

Because `onclick` is an HTML attribute, the browser performs **HTML entity decoding** before passing the string to the JavaScript engine!

---

## 🎯 Solution

1. Intercept a comment submission in Burp Suite Proxy and send the request to **Repeater**.
2. Inspect the post rendering response in Repeater to locate the `onclick` attribute:
   ```html
   <a id="author" href="http://example.com" onclick="trackBlogs('http://example.com')">attacker</a>
   ```
3. Use HTML entity representation (`&apos;`) for single quotes inside the Website field payload:
   ```http
   http://foo?&apos;-alert(1)-&apos;
   ```
4. Submit the comment.
5. Click the comment author's name to execute `alert(1)`.

![Lab 20 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20184718.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Submit Comment & Analyze Reflection
Post a comment with test input in the **Website** field (e.g. `http://testme.com`). Intercept the request in Burp Suite and view the rendered blog post HTML:

```html
<a id="author" href="http://testme.com" onclick="trackBlogs('http://testme.com')">attacker</a>
```

If we try submitting single quotes (`'`), the server escapes them as `\'`:
```html
onclick="trackBlogs('http://testme.com\'payload')"
```

Because double quotes `"` are encoded as `&quot;` and `\'` keeps the quote inside the JavaScript string, standard quote breakouts fail.

### Step 2: Leverage HTML Entity Decoding in Attribute Context
Remember the browser execution order for HTML attributes:

```text
HTML Parser -> HTML Entity Decoding -> JS Engine Execution
```

When the browser parses an HTML attribute (like `onclick="..."`), it converts HTML entities (such as `&apos;` or `&#39;`) into their corresponding characters (`'`) **BEFORE** handing the string over to JavaScript!

### Step 3: Construct Payload
Using `&apos;` in the Website field:

```http
http://foo?&apos;-alert(1)-&apos;
```

#### Parsing Breakdown:
1. Server receives `http://foo?&apos;-alert(1)-&apos;`. The server does NOT escape `&apos;` because it doesn't contain raw quotes (`'`).
2. Server renders HTML:
   ```html
   <a id="author" href="..." onclick="trackBlogs('http://foo?&apos;-alert(1)-&apos;')">attacker</a>
   ```
3. When clicked, the browser decodes `&apos;` to `'`, passing the following valid JavaScript to `eval`:
   ```javascript
   trackBlogs('http://foo?'-alert(1)-'')
   ```
4. JavaScript evaluates `'http://foo?' - alert(1) - ''`, executing `alert(1)`.

### Step 4: Verify Payload Execution
Submit the payload in the comment form Website field. Click **attacker** (the comment author link) on the blog post page to trigger `alert(1)`:

![Lab 20 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20184718.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` by using HTML entity `&apos;` inside an `onclick` attribute!

---

## 💡 Key Takeaways: HTML Entity Decoding in Attribute Sinks

- **Double Parsing Layer**: Event handler attributes (`onclick`, `onmouseover`) are parsed first as HTML, then executed as JavaScript. HTML entity encoding (`&apos;`, `&quot;`) is automatically decoded by the browser before JavaScript execution.
- **Server Sanitization Flaw**: Server-side sanitizers that check for raw quotes (`'`) fail when users supply valid HTML entities (`&apos;`).

---

## 🔒 Prevention

> [!TIP]
> 1. **Avoid Inline Event Handlers**: Do not embed dynamic user variables inside inline event handler attributes (`onclick="..."`). Use `addEventListener` in external scripts instead.
> 2. **Context-Aware JavaScript Escaping**: If user input must be placed inside event handlers, perform JavaScript Unicode escaping (`\x27`) before rendering into the attribute.
