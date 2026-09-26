# Lab 08: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

**Goal**: Execute an AngularJS expression in the search functionality that calls the `alert()` function when angle brackets and double quotes are HTML-encoded.

This lab contains a DOM-based cross-site scripting vulnerability within an AngularJS expression in the search feature. AngularJS evaluates expressions placed inside double curly braces (`{{ ... }}`) within HTML nodes managed by `ng-app` directives.

Even though the application HTML-encodes angle brackets (`<`, `>`) and double quotes (`"`), AngularJS expressions execute without requiring HTML tags.

---

## 🎯 Solution

1. Test the search input box with a standard AngularJS expression:
   ```html
   {{$on.constructor('alert(1)')()}}
   ```
2. Click **Search**.
3. Observe `alert(1)` execute instantly upon AngularJS rendering.

![Lab 8 AngularJS Expression Payload](../../Image%20Asset/Cross-Site%20Scripting/Screenshot%202026-09-26%20125659.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Search Feature & Detect AngularJS
Enter a test string (e.g. `test123`) in the search box. Inspect the DOM to confirm that the page root or body tag includes an AngularJS directive:

```html
<body ng-app="labApp">
```

Or observe that the search query reflection is rendered inside an HTML element scoped by an AngularJS application:

```html
<h1 class="header">0 search results for 'test123'</h1>
```

### Step 2: Inject AngularJS Expression
Because AngularJS compiles and executes expressions inside double curly braces `{{ ... }}`, we can construct a Sandbox escape payload targeting the Function constructor.

Enter the following payload into the search box:

```html
{{$on.constructor('alert(1)')()}}
```

#### Payload Breakdown:
- `{{ ... }}`: Instructs AngularJS to evaluate the enclosed expression.
- `$on`: Accesses a built-in AngularJS scope method.
- `.constructor`: Accesses the global JavaScript `Function` constructor.
- `('alert(1)')()`: Instantiates and immediately executes the function `alert(1)`.

### Step 3: Verify Payload Execution
Click **Search**. AngularJS processes the template binding and executes `alert(1)`:

![Lab 8 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926125751.png)

> [!SUCCESS]
> **Lab Solved**: Successfully executed `alert(1)` using an AngularJS expression!

---

## 💡 Key Takeaways: Client-Side Template Injection (CSTI / AngularJS)

- **Template Compilers**: Client-side frameworks (AngularJS, Vue, React) parse HTML for template directives. When user input is rendered directly inside a template directive (`ng-app`), attackers can inject template expressions `{{ ... }}`.
- **Bypassing HTML Filters**: Because `{{ ... }}` expressions do not require `<` or `>` characters, traditional HTML entity encoding filters fail to prevent execution.

---

## 🔒 Prevention

> [!TIP]
> 1. **Avoid Server-Side Dynamic Template Interpolation**: Avoid reflecting raw user input directly inside `ng-app` or AngularJS template nodes.
> 2. **Use `ng-non-bindable`**: Apply the `ng-non-bindable` directive to user-generated content regions to prevent AngularJS from compiling expressions.
