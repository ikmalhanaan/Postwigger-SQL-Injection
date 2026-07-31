# 🎯 PortSwigger Web Security Academy - SQL Injection Writeups & Notes

![Category](https://img.shields.io/badge/Category-Cyber%20Security-blue.svg)
![Topic](https://img.shields.io/badge/Topic-SQL%20Injection-orange.svg)
![Platform](https://img.shields.io/badge/Platform-PortSwigger%20Web%20Security%20Academy-red.svg)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen.svg)

Repositori ini berisi catatan teori komprehensif dan panduan langkah demi langkah (*writeups*) penyelesaian Lab **SQL Injection** dari [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection).

---

## 📁 Structure

```text
.
├── README.md
└── Postwigger Sql Injection/
    ├── 1. What is SQL injection (SQLi).md
    ├── 2. How to detect SQL injection vulnerabilities.md
    ├── 3. Retrieving hidden data.md
    ├── 4. Subverting application logic.md
    ├── 5. SQL injection UNION attacks.md
    ├── 6. Determining the number of columns required.md
    ├── 7. Finding columns with a useful data type.md
    ├── 8. Using a SQL injection UNION attack to retrieve interesting data.md
    ├── Image Asset/
    └── Labs/
        ├── lab-01-SQL injection vulnerability in WHERE clause allowing retrieval of hidden data.md
        ├── Lab-02-SQL injection vulnerability allowing login bypass.md
        ├── Lab-03-SQL injection UNION attack, determining the number of columns returned by the query.md
        ├── Lab-04-SQL injection UNION attack, finding a column containing text.md
        └── Lab-05-SQL injection UNION attack, retrieving data from other tables.md
```

---

## 📚 Theory & Fundamentals

| # | Topic | Description |
|---|---|---|
| 01 | [What is SQL injection (SQLi)](Postwigger%20Sql%20Injection/1.%20What%20is%20SQL%20injection%20%28SQLi%29.md) | Pengenalan dasar SQL Injection dan dampaknya. |
| 02 | [How to detect SQL injection vulnerabilities](Postwigger%20Sql%20Injection/2.%20How%20to%20detect%20SQL%20injection%20vulnerabilities.md) | Cara mengidentifikasi celah SQLi secara manual maupun otomatis. |
| 03 | [Retrieving hidden data](Postwigger%20Sql%20Injection/3.%20Retrieving%20hidden%20data.md) | Mengakses data tersembunyi dengan memanipulasi `WHERE` clause. |
| 04 | [Subverting application logic](Postwigger%20Sql%20Injection/4.%20Subverting%20application%20logic.md) | Melakukan login bypass tanpa kata sandi menggunakan komentar SQL. |
| 05 | [SQL injection UNION attacks](Postwigger%20Sql%20Injection/5.%20SQL%20injection%20UNION%20attacks.md) | Konsep dasar & syarat melakukan serangan `UNION` SQLi. |
| 06 | [Determining the number of columns required](Postwigger%20Sql%20Injection/6.%20Determining%20the%20number%20of%20columns%20required.md) | Mengukur jumlah kolom query menggunakan `ORDER BY` & `UNION SELECT NULL`. |
| 07 | [Finding columns with a useful data type](Postwigger%20Sql%20Injection/7.%20Finding%20columns%20with%20a%20useful%20data%20type.md) | Menentukan kolom yang mendukung tipe data String/Text. |
| 08 | [Using a SQL injection UNION attack to retrieve interesting data](Postwigger%20Sql%20Injection/8.%20Using%20a%20SQL%20injection%20UNION%20attack%20to%20retrieve%20interesting%20data.md) | Mengekstrak kredensial dan data sensitif dari tabel lain. |

---

## 🧪 Hands-On Labs Writeups

| Lab # | Title | Writeup Link |
|---|---|---|
| 🧪 **Lab 01** | SQL injection vulnerability in WHERE clause allowing retrieval of hidden data | [Read Writeup](Postwigger%20Sql%20Injection/Labs/lab-01-SQL%20injection%20vulnerability%20in%20WHERE%20clause%20allowing%20retrieval%20of%20hidden%20data.md) |
| 🧪 **Lab 02** | SQL injection vulnerability allowing login bypass | [Read Writeup](Postwigger%20Sql%20Injection/Labs/Lab-02-SQL%20injection%20vulnerability%20allowing%20login%20bypass.md) |
| 🧪 **Lab 03** | SQL injection UNION attack, determining the number of columns returned by the query | [Read Writeup](Postwigger%20Sql%20Injection/Labs/Lab-03-SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query.md) |
| 🧪 **Lab 04** | SQL injection UNION attack, finding a column containing text | [Read Writeup](Postwigger%20Sql%20Injection/Labs/Lab-04-SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text.md) |
| 🧪 **Lab 05** | SQL injection UNION attack, retrieving data from other tables | [Read Writeup](Postwigger%20Sql%20Injection/Labs/Lab-05-SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables.md) |

---

## 💡 Notes & Tools Used
- **Burp Suite Community / Professional**: Digunakan untuk mengintersepsi & memodifikasi HTTP Request.
- **DBMS Tested**: PostgreSQL / MySQL / Oracle.
