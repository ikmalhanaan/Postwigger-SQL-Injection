# Lab 16: Reflected XSS with some SVG markup allowed

**Goal**: Bypass a WAF blocking common HTML tags by using allowed SVG tags and event handlers to execute the `alert()` function.

This lab contains a reflected cross-site scripting vulnerability. The application WAF blocks common HTML tags (`<script>`, `<img>`, `<body>`, etc.) but fails to block certain SVG tags and SVG-specific animation event handlers.

---

## 🎯 Solution

1. Intercept a search query in Burp Suite and send the request to **Burp Intruder**.
2. Brute-force HTML tags using the PortSwigger XSS Cheat Sheet list to discover allowed tags:
   - Allowed tags: `<svg>`, `<animatetransform>`, `<title>`, `<image>`
3. Brute-force event handlers for `<svg><animatetransform>` to find allowed event attributes:
   - Allowed event: `onbegin`
4. Construct the SVG payload:
   ```html
   "><svg><animatetransform onbegin=alert(1)>
   ```
5. Submit the payload in the search parameter to trigger `alert(1)` automatically upon page load.

![Lab 16 SVG Payload Execution](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20171423.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Brute-Force Allowed Tags
Perform a search query in the browser. In Burp Proxy **HTTP history**, right-click the search request and click **Send to Intruder**.

```http
GET /?search=<§§> HTTP/2
Host: your-lab-id.web-security-academy.net
```

1. Open the [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) and click **Copy tags to clipboard**.
2. In Burp Intruder **Payloads** tab, paste the tags list and click **Start attack**.

#### Intruder Tag Brute-Force Results:
Sort by **Status**. Most tags return `400 Bad Request`, but four SVG tags return `200 OK`:
- `<svg>`
- `<animatetransform>`
- `<title>`
- `<image>`

![Lab 14 Tag Brute Force Results](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20170706.png)

### Step 2: Brute-Force Allowed Event Handlers
Update the position in Intruder to test event attributes on `<animatetransform>`:

```http
GET /?search=<svg><animatetransform%20§§=1> HTTP/2
```

1. Return to the XSS Cheat Sheet and click **Copy events to clipboard**.
2. Clear previous payloads, paste the events list into Intruder, and click **Start attack**.

#### Intruder Event Brute-Force Results:
The `onbegin` event handler returns `200 OK` (all other events return `400 Bad Request`):

![Lab 16 Event Brute Force Results](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20170853.png)

### Step 3: Execute Payload
Combine the allowed SVG tags and event handler into the URL search parameter:

```http
/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```

When the page renders, the SVG animation starts immediately (`onbegin`), executing `alert(1)` without requiring user interaction:

![Lab 16 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20171423.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` using `<svg><animatetransform onbegin=alert(1)>`!

---

## 💡 Key Takeaways: SVG Markup Exploitation

- **Vector Diversity**: Modern browsers support Inline SVG elements embedded directly in HTML documents. SVG elements introduce alternative event handlers (`onbegin`, `onend`, `onrepeat`) that traditional WAF filters frequently miss.
- **Zero-Interaction Execution**: Animation events like `onbegin` execute automatically upon element rendering, requiring no user clicks or mouse movement.

---

## 🔒 Prevention

> [!TIP]
> 1. **Context-Aware Encoding**: HTML entity-encode all user input before reflecting it inside HTML body context (`<` to `&lt;`, `>` to `&gt;`).
> 2. **Comprehensive Tag Filtering**: Ensure WAF filters cover SVG, MathML, and modern HTML5 tags alongside standard markup elements.
