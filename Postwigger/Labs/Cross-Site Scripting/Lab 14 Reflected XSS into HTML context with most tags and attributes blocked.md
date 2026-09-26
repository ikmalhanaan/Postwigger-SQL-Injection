# Lab 14: Reflected XSS into HTML context with most tags and attributes blocked

**Goal**: Bypass a Web Application Firewall (WAF) blocking most HTML tags and attributes, and deliver an exploit payload to the victim that calls `print()`.

This lab contains a reflected XSS vulnerability in the search functionality protected by a Web Application Firewall (WAF). Most standard HTML tags and event handlers are blocked (returning `HTTP 400 Bad Request`). We must use **Burp Intruder** and the **PortSwigger XSS Cheat Sheet** to brute-force allowed tags and attributes, then deliver an exploit via the Exploit Server.

---

## 🎯 Solution

1. Intercept a search request in Burp Suite and send it to **Burp Intruder**.
2. Brute-force HTML tags using the PortSwigger XSS Cheat Sheet list to find allowed tags:
   - Allowed tag: `<body>` (Returns `200 OK`)
3. Brute-force event handlers for `<body>` to find allowed attributes:
   - Allowed attribute: `onresize` (Returns `200 OK`)
4. Construct an `<iframe>` exploit on the Exploit Server that triggers `onresize` automatically:
   ```html
   <iframe src="https://<YOUR-LAB-ID>.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload="this.style.width='100px'"></iframe>
   ```
5. Click **Deliver exploit to victim** to solve the lab.

![Lab 14 Exploit Server Delivery](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20163420.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Intercept & Send Request to Burp Intruder
Perform a search query in the browser. In Burp Proxy **HTTP history**, right-click the search request and select **Send to Intruder**.

```http
GET /?search=test123 HTTP/2
Host: your-lab-id.web-security-academy.net
```

### Step 2: Brute-Force Allowed HTML Tags
In Burp Intruder **Positions** tab, set the payload position around the search parameter tag value:

```http
GET /?search=<§§> HTTP/2
```

1. Open the [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) in your browser and click **Copy tags to clipboard**.
2. In Burp Intruder **Payloads** tab, paste the tags list.
3. Click **Start attack**.

#### Intruder Results Analysis:
Sort results by the **Status** column. Almost all tags return `400 Bad Request` except the `<body>` tag, which returns `200 OK`:

![Lab 14 Intruder Tag Brute Force](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20162443.png)

This confirms that the WAF allows `<body>`.

### Step 3: Brute-Force Allowed Event Handlers
Change the position in Intruder to test event attributes on the `<body>` tag:

```http
GET /?search=<body%20§§=1> HTTP/2
```

1. Return to the XSS Cheat Sheet and click **Copy events to clipboard**.
2. Clear previous payloads in Intruder, paste the event handlers list, and click **Start attack**.

#### Intruder Results Analysis:
Sorting by status reveals that the `onresize` event handler returns `200 OK` (all others return `400 Bad Request`):

![Lab 14 Intruder Event Brute Force](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20163248.png)

### Step 4: Construct Automated Exploit Payload
Since the `onresize` event fires when the window is resized (which victim users will not trigger manually), we force an automated resize event using an `<iframe>` on the Exploit Server.

In the **Exploit Server** Body field, enter the following script:

```html
<iframe src="https://<YOUR-LAB-ID>.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload="this.style.width='100px'"></iframe>
```

#### Payload Mechanism:
1. The `<iframe>` loads the target lab search URL containing `"><body onresize=print()>`.
2. When the `<iframe>` finishes loading, its `onload` handler changes the iframe width (`this.style.width='100px'`).
3. Resizing the iframe element fires the `onresize` event inside the iframe's `<body>`, executing `print()`.

### Step 5: Test and Deliver
1. Click **Store**, then click **View exploit** to test locally (verifying the browser Print dialog appears).
2. Click **Deliver exploit to victim**.

![Lab 14 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20163451.png)

> [!SUCCESS]
> **Lab Solved**: Successfully bypassed the WAF and delivered the automated `onresize` exploit to the victim!

---

## 💡 Key Takeaways: WAF Bypass Strategy

- **Automated Enumeration**: Web Application Firewalls often rely on blacklists of known tags (`<script>`, `<img>`, `svg`). Systematic fuzzing using Burp Intruder isolates non-blacklisted elements and attributes.
- **Forced Event Triggering**: Event handlers that require user interaction (`onresize`, `onscroll`) can be forcefully triggered cross-origin via CSS modifications inside an `<iframe>`.

---

## 🔒 Prevention

> [!TIP]
> 1. **Do Not Rely on WAF Blacklists**: Blacklist-based WAF filtering is easily bypassed by obscure tags and event handlers.
> 2. **Context-Aware Escaping**: HTML entity-encode all user input rendered in HTML body context (`<` to `&lt;`, `>` to `&gt;`).
