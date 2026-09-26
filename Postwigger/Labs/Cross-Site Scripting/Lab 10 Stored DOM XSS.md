# Lab 10: Stored DOM XSS

**Goal**: Exploit a stored DOM-based cross-site scripting vulnerability in the blog comment feature to trigger `alert(1)`.

This lab demonstrates a Stored DOM XSS vulnerability. Stored DOM XSS occurs when a server receives user input, stores it in a database, and later serves it to users where client-side JavaScript reads the stored data and handles it unsafely in a DOM sink.

---

## 🎯 Solution

1. Open any blog post on the target site.
2. Scroll to the comment section.
3. Submit a comment containing the XSS vector:
   ```html
   <><img src=1 onerror=alert(1)>
   ```
4. Fill in the required fields (Name, Email) and post the comment.
5. Return to the blog post page. The client-side JavaScript reads the comment from the server and passes it to an unsafe DOM sink, executing `alert(1)`.

![Lab 10 Comment Vector Submission](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926132616.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze Client-Side Source and Sink
Inspect `loadComments.js` in Developer Tools to understand how comments are fetched and rendered client-side:

```javascript
function renderComments(comments) {
    for (var i = 0; i < comments.length; i++) {
        var comment = comments[i];
        var commentElement = document.createElement('section');
        commentElement.innerHTML = escapeHTML(comment.author) + comment.content;
        document.getElementById('comments').appendChild(commentElement);
    }
}
```

Notice that the server-side or client-side sanitizer attempts to strip or escape standard HTML tags, but custom or malformed tags such as `<>` bypass the regex filter while allowing `<img src=1 onerror=alert(1)>` to be parsed by the browser when assigned to `innerHTML`.

### Step 2: Submit Stored Payload
In the blog post comment form, enter the following vector into the **Comment** box:

```html
<><img src=1 onerror=alert(1)>
```

Fill in:
- **Name**: `attacker`
- **Email**: `attacker@gmail.com`

Click **Post Comment**.

![Lab 10 Comment Posted](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20132652.png)

### Step 3: Trigger Payload Execution
Click **Back to blog** to view the post. Client-side JavaScript fetches the stored comments from the backend via AJAX and renders them using `innerHTML`:

![Lab 10 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926132819.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via Stored DOM XSS!

---

## 💡 Key Takeaways: Stored DOM XSS

- **Hybrid Vulnerability**: Combines Stored XSS (data persistence in database) with DOM XSS (unsafe client-side JavaScript rendering).
- **Sanitizer Flaws**: Custom regex replacement routines often leave edge cases (such as opening `<>` brackets) unhandled.

---

## 🔒 Prevention

> [!TIP]
> 1. **Use Robust HTML Sanitizer**: Do not use custom regex filters. Use established sanitization libraries like DOMPurify before assigning HTML to `innerHTML`.
> 2. **Use Safe Rendering APIs**: Use `element.textContent` instead of `element.innerHTML` for user comments.
