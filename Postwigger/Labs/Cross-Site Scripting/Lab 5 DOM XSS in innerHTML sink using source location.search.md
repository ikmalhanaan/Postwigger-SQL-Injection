# Lab 05: DOM XSS in innerHTML sink using source location.search

**Goal**: Perform a DOM-based cross-site scripting (DOM XSS) attack using an `innerHTML` sink that calls the `alert()` function.

This lab contains a DOM-based cross-site scripting vulnerability in the blog search feature. The application reads user input from `location.search` (source) and assigns it directly to a `div` element's `innerHTML` property (sink) without sanitization.

---

## 🎯 Solution

1. Enter an XSS payload using an `<img>` tag with an `onerror` event handler into the search box:
   ```html
   <img src=1 onerror=alert(1)>
   ```
2. Alternatively, append the query parameter to the URL:
   ```http
   /?search=<img src=1 onerror=alert(1)>
   ```
3. The client-side JavaScript reads the query string and assigns it to `innerHTML`.
4. The browser parses the HTML, fails to load image source `1`, and triggers `onerror=alert(1)`.

![Lab 5 Search Box Input](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20210756.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze Client-Side Source and Sink
Open the developer tools (F12) and inspect the client-side JavaScript handling search results:

```javascript
function doSearchQuery(query) {
    document.getElementById('searchMessage').innerHTML = query;
}
var query = (new URLSearchParams(window.location.search)).get('search');
if (query) {
    doSearchQuery(query);
}
```

- **Source**: `location.search` (`search` query parameter in the URL)
- **Sink**: `innerHTML` property assignment on `element#searchMessage`

> [!NOTE]
> HTML5 specifies that `<script>` tags inserted via `innerHTML` **will not execute**. Therefore, standard `<script>alert(1)</script>` payloads will fail in `innerHTML` sinks. We must use HTML element event handlers like `<img src=1 onerror=alert(1)>` or `<svg onload=alert(1)>`.

### Step 2: Inject Image Error Event Payload
In the blog search box, type the following payload:

```html
<img src=1 onerror=alert(1)>
```

![Lab 5 Injecting img payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20210942.png)

Click **Search**.

### Step 3: Trigger Payload Execution
The client-side JavaScript executes `innerHTML = '<img src=1 onerror=alert(1)>'`. The browser parses the DOM element, attempts to fetch image resource `1`, fails, and instantly triggers the `onerror` event handler:

![Lab 5 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20211148.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via DOM XSS in an `innerHTML` sink!

---

## 💡 Key Takeaways: `innerHTML` Sink Characteristics

- **Script Blocked**: Modern browsers do not execute `<script>` elements inserted via `innerHTML`.
- **Event Handlers**: Inline event handlers (`onerror`, `onload`, `onmouseover`) on elements like `<img>`, `<svg>`, `<iframe>`, or `<autofocus>` bypass this limitation and execute JavaScript.

---

## 🔒 Prevention

> [!TIP]
> 1. **Use `textContent` or `innerText`**: Replace `element.innerHTML` with `element.textContent` when assigning user-supplied text.
> 2. **Sanitize HTML**: If HTML rendering is required, sanitize user input using robust libraries like DOMPurify.
