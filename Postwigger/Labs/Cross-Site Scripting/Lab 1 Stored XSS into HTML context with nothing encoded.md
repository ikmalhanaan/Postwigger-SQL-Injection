# Lab 01: Stored XSS into HTML context with nothing encoded

**Goal**: Submit a comment on a blog post that executes the `alert()` JavaScript function whenever the blog post is viewed by users.

This lab contains a stored cross-site scripting (Stored XSS) vulnerability in the blog comment submission functionality. Input submitted in the comment field is stored directly in the database without HTML encoding or sanitization and is rendered raw in the HTML context when viewing the post.

---

## 🎯 Solution

1. Navigate to any blog post on the target site.
2. Scroll down to the comment form.
3. Submit a comment containing the XSS payload:
   ```html
   <script>alert(1)</script>
   ```
4. Fill in the required fields (Name, Email) and post the comment.
5. Reload or view the blog post to trigger the stored JavaScript execution.

![Lab 1 Form Submission](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20184621.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Access Blog Post & Comment Form
Click on any available blog post from the homepage and scroll down to the **Leave a comment** section.

### Step 2: Inject Stored XSS Payload
In the **Comment** text area, enter the standard script payload:

```html
<script>alert(1)</script>
```

Fill out the remaining form fields:
- **Name**: `testme`
- **Email**: `testme@gmail.com`
- **Website**: *(Optional / Leave blank)*

Click **Post Comment**.

### Step 3: Trigger Payload Execution
Once the comment is submitted, click **Back to blog**. The page will reload, render the comment from the database, and execute the JavaScript payload:

![Lab 1 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20184806.png)

> [!SUCCESS]
> **Lab Solved**: The stored payload executed `alert(1)` automatically upon page rendering!

---

## 💡 Key Takeaways: Stored XSS

- **Persistence**: Unlike Reflected XSS, Stored XSS payloads are saved permanently in the backend data store (database, file system, etc.).
- **Impact**: Every victim who visits the affected page will execute the attacker's script without needing to click a malicious link.

---

## 🔒 Prevention

> [!TIP]
> 1. **HTML Entity Encoding**: Encode all user input before rendering inside HTML body elements (`<` to `&lt;`, `>` to `&gt;`, `&` to `&amp;`).
> 2. **Content Security Policy (CSP)**: Implement a strict CSP to disable inline script execution (`script-src 'self'`).
