# Lab 07: DOM XSS in jQuery selector sink using a hashchange event

**Goal**: Deliver an exploit payload to the victim that triggers the browser's `print()` function via a DOM-based XSS vulnerability in a jQuery selector sink.

This lab contains a DOM-based cross-site scripting vulnerability on the home page. The site uses jQuery's `$()` selector function to auto-scroll to a blog post whose title is passed via the `location.hash` property whenever a `hashchange` event fires.

---

## 🎯 Solution

1. Inspect the home page client-side JavaScript to identify the vulnerable `hashchange` listener and jQuery `$()` selector sink.
2. Craft an `<iframe>` payload on the Exploit Server targeting the lab URL with an invalid `<img>` payload appended to the URL hash:
   ```html
   <iframe src="https://<YOUR-LAB-ID>.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
   ```
3. Test the exploit using **View exploit** to verify `print()` is executed.
4. Click **Deliver exploit to victim** to solve the lab.

![Lab 7 Exploit Server Delivery](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20124322.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze Client-Side Source and Sink
Open Developer Tools (F12) or view the source code of the homepage to locate the inline script:

```javascript
$(window).on('hashchange', function(){
    var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
    if (post) post.get(0).scrollIntoView();
});
```

- **Source**: `location.hash` (`window.location.hash.slice(1)`)
- **Sink**: jQuery `$()` selector function (`$('section.blog-list h2:contains(' + ...)`)

> [!WARNING]
> In older versions of jQuery, passing HTML tags (such as `<img ...>`) into the `$()` selector causes jQuery to create and instantiate new DOM elements rather than selecting existing elements, executing any inline event handlers immediately.

### Step 2: Test Exploit on Exploit Server
Navigate to the **Exploit Server** and enter the following payload in the **Body** text area:

```html
<iframe src="https://<YOUR-LAB-ID>.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

*Replace `<YOUR-LAB-ID>` with your lab instance hostname.*

#### Payload Breakdown:
1. The `<iframe>` initially loads the homepage with an empty hash (`#`).
2. When the `<iframe>` finishes loading, the `onload` handler appends `<img src=x onerror=print()>` to the `src` URL hash.
3. Updating the hash triggers the `hashchange` event listener on the main window.
4. The vulnerable jQuery selector `$()` receives the `<img ...>` tag from `window.location.hash`, parses it as HTML, and executes `onerror=print()`.

### Step 3: Test and Deliver to Victim
1. Click **Store** to save the exploit script.
2. Click **View exploit** to test it locally. The browser will display the document **Print** dialog.
3. Click **Deliver exploit to victim**.

![Lab 7 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926125105.png)

> [!SUCCESS]
> **Lab Solved**: Delivered the Exploit Server payload to the victim, executing `print()` automatically!

---

## 💡 Key Takeaways: jQuery Selector Sinks

- **Selector Element Creation**: Older versions of jQuery automatically instantiate elements if input passed to `$()` starts with a `<` character.
- **Hashchange Exploitation**: Since `location.hash` can be modified cross-origin without reloading the target page, embedding the target inside an `<iframe>` and dynamically changing its `#` hash delivers seamless client-side exploitation.

---

## 🔒 Prevention

> [!TIP]
> 1. **Upgrade jQuery**: Use jQuery 3.0.0 or higher where element creation in `$()` is disallowed unless explicitly specified.
> 2. **Avoid Dynamic Selectors**: Do not pass raw, unescaped user input or URL hashes directly into `$()`. Use `$.find()` or escape CSS selectors properly.
