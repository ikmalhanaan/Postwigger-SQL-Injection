# Lab 12: Stored XSS into anchor href attribute with double quotes HTML-encoded

**Goal**: Submit a comment with a malicious Website URL payload that executes `alert(1)` when a user clicks on the comment author's name.

This lab contains a stored cross-site scripting vulnerability in the comment submission feature. Double quotes are HTML-encoded, preventing breakout from quoted attributes. However, the **Website** field input is placed inside an anchor tag's `href` attribute (`<a href="...">`), allowing execution via the `javascript:` pseudo-protocol.

---

## 🎯 Solution

1. Open a blog post and scroll to the comment form.
2. Intercept the comment post submission in Burp Suite Proxy and send it to **Repeater**.
3. Inspect how the `website` input parameter is rendered in the HTML context:
   ```html
   <a href="https://example.com">Author Name</a>
   ```
4. Set the `website` parameter value to a `javascript:` URL scheme:
   ```http
   website=javascript:alert(1)
   ```
5. Submit the comment.
6. Click the comment author's name on the blog post page to trigger `alert(1)`.

![Lab 12 Intercepting Comment Post](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20144151.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Submit Comment & Intercept Request
Fill out the comment form fields with test data:
- **Comment**: `Great post!`
- **Name**: `attacker`
- **Email**: `attacker@gmail.com`
- **Website**: `https://example.com`

Turn on **Intercept** in Burp Suite, submit the comment, and send the POST request to **Repeater**.

### Step 2: Analyze Reflection Context
Inspect the comment rendering in the HTTP response:

```html
<a id="author" href="https://example.com">attacker</a>
```

Double quotes are encoded (`"` to `&quot;`), preventing attribute breakouts like `" onmouseover="...`. However, because the input controls the entire URL inside `href="..."`, pseudo-protocols are valid!

### Step 3: Inject `javascript:` Payload
In Burp Repeater, update the POST body parameter:

```http
name=attacker&email=attacker%40gmail.com&website=javascript%3Aalert%281%29&comment=Great+post%21
```

Click **Send**.

![Lab 12 Submitting Payload in Burp Repeater](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20144655.png)

### Step 4: Verify Payload Execution
Return to the blog post page. The comment author's name is rendered as a hyperlink:

```html
<a id="author" href="javascript:alert(1)">attacker</a>
```

Click on **attacker** (the comment author name). The browser invokes the `javascript:` scheme and executes `alert(1)`:

![Lab 12 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20144736.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via `javascript:` URI scheme inside `href`!

---

## 💡 Key Takeaways: `href` Pseudo-Protocol Vulnerabilities

- **Attribute Escaping Limitations**: Escaping quotes (`"`) prevents attribute breakout attacks, but does **NOT** prevent execution if the target attribute is an `href` or `src` expecting a URL.
- **Pseudo-Protocols**: Browsers interpret `javascript:`, `data:`, and `vbscript:` URLs inside `href` attributes as executable code upon navigation.

---

## 🔒 Prevention

> [!TIP]
> 1. **URL Protocol Validation**: Enforce strict URL schemes (`http://` or `https://` only) for user-supplied link inputs. Reject any URL beginning with `javascript:`.
> 2. **URL Parsing**: Parse and validate user URLs using standard URL parsing libraries before saving them to the database.
