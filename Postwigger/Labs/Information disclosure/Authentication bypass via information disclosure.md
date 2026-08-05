# Lab: Authentication bypass via information disclosure

**Category**: Information Disclosure / HTTP Verb Tampering / Header Disclosure  
**Goal**: Menggunakan HTTP `TRACE` method untuk membocorkan custom HTTP header (`X-Custom-IP-Authorization`), memanipulasi header tersebut menjadi `127.0.0.1` (localhost), melakukan *authentication bypass*, dan menghapus user `carlos`.  
*Use the HTTP `TRACE` method to reveal custom front-end headers (`X-Custom-IP-Authorization`), spoof localhost IP (`127.0.0.1`), bypass authentication, and delete user `carlos`.*  
**Credentials**: `wiener:peter`

---

## 📌 Overview & Prerequisite Knowledge / Gambaran Umum & Pengetahuan Dasar

- **HTTP TRACE Method Exploitation**: 
  HTTP `TRACE` method mengembalikan persis request yang diterima oleh server (*diagnostic loop-back*). Jika aplikasi menggunakan reverse proxy atau load balancer yang menambahkan custom header (misal `X-Custom-IP-Authorization`), respon `TRACE` akan membocorkan nama header internal tersebut.
  *The HTTP `TRACE` method echoes back the exact request received by the server, revealing internal custom headers appended by reverse proxies.*
- **IP Spoofing via Custom Header / Authentication Bypass**: 
  Jika aplikasi mempercayai custom header internal untuk memverifikasi lokasi klien (apakah dari `localhost / 127.0.0.1`), attacker dapat menyuntikkan header tersebut untuk melakukan bypass autentikasi pada panel admin.
  *Injecting proxy headers to impersonate localhost (`127.0.0.1`) and bypass administrative access controls.*

---

## 🎯 Solution Summary / Ringkasan Solusi

1. **ID**: Tangkap request `GET /admin` menggunakan Burp Suite, lalu kirim ke **Repeater**.  
   **EN**: Capture the `GET /admin` request in Burp Suite and send it to **Repeater**.
2. **ID**: Ubah HTTP Method dari `GET` menjadi `TRACE` (`TRACE /admin`), lalu kirim request.  
   **EN**: Change the HTTP Method from `GET` to `TRACE` (`TRACE /admin`) and send the request.
3. **ID**: Amati respon `TRACE` yang membocorkan header internal `X-Custom-IP-Authorization: <IP_CLIENT>`.  
   **EN**: Study the `TRACE` response to find the internal header `X-Custom-IP-Authorization: <CLIENT_IP>`.
4. **ID**: Konfigurasi **Burp Proxy > Match and Replace** untuk secara otomatis menambahkan header `X-Custom-IP-Authorization: 127.0.0.1` pada setiap request outbound.  
   **EN**: Configure **Burp Proxy > Match and Replace** rules to automatically append `X-Custom-IP-Authorization: 127.0.0.1` to all outbound requests.
5. **ID**: Akses kembali `/admin` di browser, buka panel admin, lalu hapus pengguna `carlos` untuk menyelesaikan lab.  
   **EN**: Access `/admin` in your browser, enter the admin panel, and delete user `carlos` to complete the lab.

![Solution Overview](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805184046.png)

---

## 🛠️ Step-by-Step Guide / Panduan Langkah demi Langkah

### Step 1: Intercept & Capture `/admin` Request / Tangkap Request `/admin`

**ID**: Buka Burp Suite, pastikan Intercept `ON`, lalu akses URL `/admin`. Kirim request tersebut ke Burp Repeater (`Ctrl + R`).  
**EN**: Open Burp Suite, turn Intercept `ON`, and browse to `/admin`. Send the request to Burp Repeater (`Ctrl + R`).

```http
GET /admin HTTP/2
Host: 0aea00bc040d3652804eb25100e50015.web-security-academy.net
Cookie: session=e89rfKenNy8ZMKXrf7ZpNzTojA5QYXUV
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
```

![GET /admin Request](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805185114.png)

---

### Step 2: Leverage HTTP TRACE Method to Discover Internal Headers / Menggunakan HTTP TRACE

**ID**: Di Burp Repeater, ubah HTTP Method dari `GET /admin` menjadi `TRACE /admin`:  
**EN**: In Burp Repeater, change the HTTP Method from `GET /admin` to `TRACE /admin`:

```http
TRACE /admin HTTP/2
Host: 0aea00bc040d3652804eb25100e50015.web-security-academy.net
Cookie: session=e89rfKenNy8ZMKXrf7ZpNzTojA5QYXUV
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
```

Kirim request tersebut. Respon server mengembalikan isi request yang diproses oleh reverse proxy:
*Send the request. The server response echoes back the request as received by the backend:*

```http
HTTP/2 200 OK
Content-Type: message/http
X-Frame-Options: SAMEORIGIN

TRACE /admin HTTP/1.1
Host: 0aea00bc040d3652804eb25100e50015.web-security-academy.net
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Cookie: session=e89rfKenNy8ZMKXrf7ZpNzTojA5QYXUV
X-Custom-IP-Authorization: 182.10.131.220
```

> [!NOTE]
> **Disclosed Custom Header**:  
> Reverse proxy menyuntikkan header `X-Custom-IP-Authorization` untuk memverifikasi alamat IP klien.

---

### Step 3: Configure Burp Proxy Match and Replace Rule / Konfigurasi Match and Replace

**ID**: Buka Burp Suite **Proxy > Match and Replace**, lalu tambahkan aturan (*rule*) baru:  
**EN**: Open Burp Suite **Proxy > Match and Replace**, and add a new rule:

- **Type**: `Request header`
- **Match**: *(Kosongkan / Leave empty)*
- **Replace**: `X-Custom-IP-Authorization: 127.0.0.1`
- **Comment**: `IP Authorization Bypass`

![Match and Replace Rule](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805190025.png)

Aturan ini akan menyuntikkan `X-Custom-IP-Authorization: 127.0.0.1` secara otomatis pada setiap request browser.

---

### Step 4: Access Admin Panel & Delete Carlos / Akses Admin Panel & Hapus User Carlos

**ID**: Buka kembali halaman `/admin` di browser. Akses panel admin berhasil didapatkan karena server menganggap request berasal dari `localhost (127.0.0.1)`.  
**EN**: Refresh `/admin` in your browser. Access to the administrative interface is granted as the server trusts the spoofed `127.0.0.1` IP header.

![Admin Panel Access](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805184918.png)

Klik **Delete** pada baris pengguna `carlos`.

![Lab Solved Confirmation](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805190213.png)

> [!SUCCESS]
> **Lab Solved**: User `carlos` berhasil dihapus dan lab diselesaikan!  
> *User `carlos` successfully deleted and lab solved!*