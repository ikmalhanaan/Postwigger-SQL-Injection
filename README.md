# 🎯 PortSwigger Web Security Academy - Notes & Writeups

![Category](https://img.shields.io/badge/Category-Cyber%20Security-blue.svg)
![Topic](https://img.shields.io/badge/Topics-SQLi%20%7C%20XSS%20%7C%20API%20Testing%20%7C%20Info%20Disclosure-orange.svg)
![Platform](https://img.shields.io/badge/Platform-PortSwigger%20Web%20Security%20Academy-red.svg)
![Status](https://img.shields.io/badge/Status-Active-brightgreen.svg)

Repositori ini berisi catatan teori komprehensif (*bilingual technical notes*) dan panduan langkah demi langkah (*writeups*) penyelesaian laboratorium dari [PortSwigger Web Security Academy](https://portswigger.net/web-security).

---

## 📁 Repository Structure

```text
.
├── README.md
└── Postwigger/
    ├── 1. What is SQL injection (SQLi).md
    ├── 2. How to detect SQL injection vulnerabilities.md
    ├── 3. Retrieving hidden data.md
    ├── 4. Subverting application logic.md
    ├── 5. SQL injection UNION attacks.md
    ├── 6. Determining the number of columns required.md
    ├── 7. Finding columns with a useful data type.md
    ├── 8. Using a SQL injection UNION attack to retrieve interesting data.md
    ├── Image Asset/
    │   ├── API Testing/
    │   ├── Cross-Site Scripting/
    │   ├── Lab-01- .../
    │   ├── Lab-02- .../
    │   ├── Lab-03- .../
    │   ├── Lab-04- .../
    │   ├── Lab-05- .../
    │   ├── Lab-06- .../
    │   ├── Lab-07- .../
    │   ├── Lab-08- .../
    │   ├── Lab-09- .../
    │   ├── Lab-10- .../
    │   ├── Lab-11- .../
    │   └── Source code disclosure/
    └── Labs/
        ├── Alur SQL Injection.md
        ├── lab-01-SQL injection vulnerability in WHERE clause allowing retrieval of hidden data.md
        ├── Lab-02-SQL injection vulnerability allowing login bypass.md
        ├── Lab-03-SQL injection UNION attack, determining the number of columns returned by the query.md
        ├── Lab-04-SQL injection UNION attack, finding a column containing text.md
        ├── Lab-05-SQL injection UNION attack, retrieving data from other tables.md
        ├── Lab-06-SQL injection attack, querying the database type and version on MySQL and Microsoft.md
        ├── Lab-07-SQL injection attack, listing the database contents on non-Oracle databases.md
        ├── Lab-08-Visible error-based SQL injection.md
        ├── Lab-09-Blind SQL Injection With Time Delays and Information Retrieval.md
        ├── Lab-10-SQL Injection UNION Attack, retrieving multiple values in a single column.md
        ├── Lab-11-Blind SQL Injection With Conditional Response.md
        ├── Cross-Site Scripting/
        │   ├── Lab 1 Stored XSS into HTML context with nothing encoded.md
        │   ├── Lab 2 Reflected XSS into HTML context with nothing encoded.md
        │   ├── Lab 3 DOM XSS in document.write sink using source location.search.md
        │   ├── Lab 4 DOM XSS in document.write sink using source location.search inside a select element.md
        │   ├── Lab 5 DOM XSS in innerHTML sink using source location.search.md
        │   └── Lab 6 DOM XSS in jQuery anchor href attribute sink using location.search source.md
        ├── Exploiting an API endpoint using documentation.md
        └── Information disclosure/
            ├── Authentication bypass via information disclosure.md
            ├── Information disclosure in error messages.md
            ├── Information disclosure in version control history.md
            ├── Information disclosure on debug page.md
            └── Source code disclosure via backup files.md
```

---

## 📚 Theory & Fundamentals (SQL Injection)

| # | Topic | Description | Link |
|---|---|---|---|
| 01 | **What is SQL injection (SQLi)** | Pengenalan dasar kerentanan SQLi & dampaknya. | [Read Note](Postwigger/1.%20What%20is%20SQL%20injection%20%28SQLi%29.md) |
| 02 | **How to detect SQL injection vulnerabilities** | Teknik deteksi SQLi secara manual maupun otomatis (Burp Scanner). | [Read Note](Postwigger/2.%20How%20to%20detect%20SQL%20injection%20vulnerabilities.md) |
| 03 | **Retrieving hidden data** | Memanipulasi `WHERE` clause untuk mengakses data tersembunyi. | [Read Note](Postwigger/3.%20Retrieving%20hidden%20data.md) |
| 04 | **Subverting application logic** | Bypass autentikasi/login tanpa password menggunakan komentar SQL. | [Read Note](Postwigger/4.%20Subverting%20application%20logic.md) |
| 05 | **SQL injection UNION attacks** | Prasyarat dan konsep dasar serangan `UNION` SQLi. | [Read Note](Postwigger/5.%20SQL%20injection%20UNION%20attacks.md) |
| 06 | **Determining column count** | Mengukur jumlah kolom query dengan `ORDER BY` & `UNION SELECT NULL`. | [Read Note](Postwigger/6.%20Determining%20the%20number%20of%20columns%20required.md) |
| 07 | **Finding text columns** | Menentukan kolom query yang kompatibel dengan tipe data String. | [Read Note](Postwigger/7.%20Finding%20columns%20with%20a%20useful%20data%20type.md) |
| 08 | **Retrieving interesting data** | Mengekstrak data sensitif (username & password) dari tabel lain. | [Read Note](Postwigger/8.%20Using%20a%20SQL%20injection%20UNION%20attack%20to%20retrieve%20interesting%20data.md) |
| 09 | **SQL Injection Methodology & Workflow** | Panduan alur lengkap tahap demi tahap eksploitasi SQLi di dunia nyata & lab. | [Read Note](Postwigger/Labs/Alur%20SQL%20Injection.md) |

---

## 🧪 Hands-On Labs Writeups

### 1. SQL Injection

| Lab # | Title | Description | Writeup Link |
|---|---|---|---|
| 🧪 **Lab 01** | SQL injection vulnerability in WHERE clause allowing retrieval of hidden data | Bypass category filter `released = 1` dengan payload `' OR 1=1--`. | [Read Writeup](Postwigger/Labs/lab-01-SQL%20injection%20vulnerability%20in%20WHERE%20clause%20allowing%20retrieval%20of%20hidden%20data.md) |
| 🧪 **Lab 02** | SQL injection vulnerability allowing login bypass | Bypass form login administrator dengan payload `administrator'--`. | [Read Writeup](Postwigger/Labs/Lab-02-SQL%20injection%20vulnerability%20allowing%20login%20bypass.md) |
| 🧪 **Lab 03** | SQL injection UNION attack, determining the number of columns returned | Menemukan jumlah kolom query menggunakan `ORDER BY` & `UNION SELECT NULL`. | [Read Writeup](Postwigger/Labs/Lab-03-SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query.md) |
| 🧪 **Lab 04** | SQL injection UNION attack, finding a column containing text | Identifikasi posisi kolom yang mendukung tipe data Text/String. | [Read Writeup](Postwigger/Labs/Lab-04-SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text.md) |
| 🧪 **Lab 05** | SQL injection UNION attack, retrieving data from other tables | Exfiltrate tabel `users` (username & password) dan login sebagai `administrator`. | [Read Writeup](Postwigger/Labs/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables.md) |
| 🧪 **Lab 06** | SQL injection attack, querying the database type and version on MySQL and Microsoft | Ekstraksi versi database (MySQL/Microsoft) menggunakan `@@version` & `UNION SELECT`. | [Read Writeup](Postwigger/Labs/Lab-06-SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft.md) |
| 🧪 **Lab 07** | SQL injection attack, listing the database contents on non-Oracle databases | Enumerasi `information_schema` untuk menemukan nama tabel & kolom, lalu ekstrak credentials dan login sebagai `administrator`. | [Read Writeup](Postwigger/Labs/Lab-07-SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases.md) |
| 🧪 **Lab 08** | Visible error-based SQL injection | Memicu type conversion error (`CAST`) pada cookie `TrackingId` untuk mengekstrak password `administrator` dalam 1 request. | [Read Writeup](Postwigger/Labs/Lab-08-Visible%20error-based%20SQL%20injection.md) |
| 🧪 **Lab 09** | Blind SQL injection with time delays and information retrieval | Eksploitasi blind SQLi berbasis time delay (`pg_sleep`) + Python script untuk mengekstrak password `administrator` karakter per karakter. | [Read Writeup](Postwigger/Labs/Lab-09-Blind%20SQL%20Injection%20With%20Time%20Delays%20and%20Information%20Retrieval.md) |
| 🧪 **Lab 10** | SQL injection UNION attack, retrieving multiple values in a single column | Menggabungkan username & password (`username||'~'||password`) ke dalam 1 kolom text yang tersedia menggunakan operator konkat PostgreSQL (`||`). | [Read Writeup](Postwigger/Labs/Lab-10-SQL%20Injection%20UNION%20Attack,%20retrieving%20multiple%20values%20in%20a%20single%20column.md) |
| 🧪 **Lab 11** | Blind SQL injection with conditional responses | Eksploitasi boolean-based blind SQLi via cookie `TrackingId` untuk mengekstrak password `administrator` berbasis pesan "Welcome back!". | [Read Writeup](Postwigger/Labs/Lab-11-Blind%20SQL%20Injection%20With%20Conditional%20Response.md) |

---

### 2. Cross-Site Scripting (XSS)

| Lab # | Title | Description | Writeup Link |
|---|---|---|---|
| ⚡ **Lab 01** | Stored XSS into HTML context with nothing encoded | Injeksi payload `<script>alert(1)</script>` pada kolom komentar blog tanpa sanitasi. | [Read Writeup](Postwigger/Labs/Cross-Site%20Scripting/Lab%201%20Stored%20XSS%20into%20HTML%20context%20with%20nothing%20encoded.md) |
| ⚡ **Lab 02** | Reflected XSS into HTML context with nothing encoded | Eksekusi `<script>alert(1)</script>` yang direfleksikan langsung melalui parameter pencarian blog. | [Read Writeup](Postwigger/Labs/Cross-Site%20Scripting/Lab%202%20Reflected%20XSS%20into%20HTML%20context%20with%20nothing%20encoded.md) |
| ⚡ **Lab 03** | DOM XSS in document.write sink using source location.search | Break-out dari atribut `<img src>` menggunakan payload `"><svg onload=alert(1)>` via `document.write`. | [Read Writeup](Postwigger/Labs/Cross-Site%20Scripting/Lab%203%20DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search.md) |
| ⚡ **Lab 04** | DOM XSS in document.write sink inside a select element | Break-out dari elemen `<select>` stock checker menggunakan `"></select><img src=1 onerror=alert(1)>`. | [Read Writeup](Postwigger/Labs/Cross-Site%20Scripting/Lab%204%20DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element.md) |
| ⚡ **Lab 05** | DOM XSS in innerHTML sink using source location.search | Bypass pembatasan `<script>` pada sink `innerHTML` menggunakan event handler `<img src=1 onerror=alert(1)>`. | [Read Writeup](Postwigger/Labs/Cross-Site%20Scripting/Lab%205%20DOM%20XSS%20in%20innerHTML%20sink%20using%20source%20location.search.md) |
| ⚡ **Lab 06** | DOM XSS in jQuery anchor href attribute sink | Injeksi pseudo-protocol `javascript:alert(document.cookie)` pada atribut `href` tombol Back via jQuery `attr()`. | [Read Writeup](Postwigger/Labs/Cross-Site%20Scripting/Lab%206%20DOM%20XSS%20in%20jQuery%20anchor%20href%20attribute%20sink%20using%20location.search%20source.md) |

---

### 3. API Testing

| Lab Title | Description | Writeup Link |
|---|---|---|
| 🌐 **Exploiting an API endpoint using documentation** | Menemukan *exposed API documentation* via path traversal `/api` & menghapus user `carlos` via `DELETE` endpoint. | [Read Writeup](Postwigger/Labs/Exploiting%20an%20API%20endpoint%20using%20documentation.md) |

---

### 4. Information Disclosure

| Lab Title | Description | Writeup Link |
|---|---|---|
| 🔍 **Information disclosure in error messages** | Ekstraksi versi framework `Apache Struts 2 2.3.31` melalui *verbose error message / stack trace*. | [Read Writeup](Postwigger/Labs/Information%20disclosure/Information%20disclosure%20in%20error%20messages.md) |
| 🔍 **Information disclosure on debug page** | Pemindaian `/cgi-bin/` untuk menemukan debug page `phpinfo.php` dan ekstraksi `SECRET_KEY`. | [Read Writeup](Postwigger/Labs/Information%20disclosure/Information%20disclosure%20on%20debug%20page.md) |
| 🔍 **Source code disclosure via backup files** | Penelusuran `/robots.txt` & direktori `/backup` untuk mengunduh `ProductTemplate.java.bak` dan ekstraksi password PostgreSQL. | [Read Writeup](Postwigger/Labs/Information%20disclosure/Source%20code%20disclosure%20via%20backup%20files.md) |
| 🔍 **Authentication bypass via information disclosure** | Penggunaan HTTP `TRACE` method untuk membocorkan header `X-Custom-IP-Authorization`, IP spoofing `127.0.0.1`, dan menghapus user `carlos`. | [Read Writeup](Postwigger/Labs/Information%20disclosure/Authentication%20bypass%20via%20information%20disclosure.md) |
| 🔍 **Information disclosure in version control history** | Ekstraksi repositori `.git` publik (`git-dumper`), analisis `git log` & `git show`, pemulihan password lama administrator, dan hapus `carlos`. | [Read Writeup](Postwigger/Labs/Information%20disclosure/Information%20disclosure%20in%20version%20control%20history.md) |

---

## 🛠️ Tools Used
- **Burp Suite Professional / Community Edition** (Proxy, Intercept, Repeater, Match and Replace)
- **cURL / Terminal CLI**
- **Ffuf / Gobuster** (Directory Fuzzing)
- **Git / Git-Dumper** (Version Control Analysis & Recovery)
