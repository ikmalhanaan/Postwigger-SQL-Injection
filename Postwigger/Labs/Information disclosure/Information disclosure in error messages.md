# Lab: Information disclosure in error messages

**Category**: Information Disclosure / Verbose Error Messages  
**Goal**: Mengidentifikasi versi framework pihak ketiga (*third-party framework*) yang rentan melalui pesan error verbose (*verbose error message / stack trace*).  
*Identify and submit the vulnerable version number of the third-party framework revealed in verbose error messages.*

---

## 📌 Overview & Prerequisite Knowledge / Gambaran Umum & Pengetahuan Dasar

- **Verbose Error Messages (Pesan Error Detail)**: 
  Pesan kesalahan yang menampilkan informasi internal secara mendalam (*stack trace*, nama class, direktori server, dan versi perangkat lunak).
  *Error messages that reveal internal system details such as stack traces, class names, server paths, and exact software versions.*
- **Information Leakage via Unhandled Exceptions**: 
  Kegagalan penanganan pengecualian (*unhandled exception*) yang menyebabkan aplikasi membocorkan versi *framework* kepada pengguna yang tidak berwenang.
  *Failure in exception handling that causes the application to leak framework version info to unauthorized users.*

---

## 🎯 Solution Summary / Ringkasan Solusi

1. **ID**: Buka halaman salah satu produk pada aplikasi web.  
   **EN**: Open one of the product pages on the web application.
2. **ID**: Picu kesalahan tipe data (*type error / unhandled exception*) dengan mengubah parameter `productId` menjadi data non-integer (misalnya: `productId=not_a_number` atau string).  
   **EN**: Trigger a type error by supplying a non-integer data type to the `productId` parameter (e.g., `productId=not_a_number`).
3. **ID**: Analisis respon *Internal Server Error (500)* dan *stack trace* untuk menemukan versi framework `Apache Struts 2 2.3.31`.  
   **EN**: Analyze the *500 Internal Server Error* response and stack trace to locate the framework version `Apache Struts 2 2.3.31`.
4. **ID**: Submit jawaban `2 2.3.31` pada tombol **Submit solution** untuk menyelesaikan lab.  
   **EN**: Submit `2 2.3.31` via the **Submit solution** button to solve the lab.

![Solution Overview](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805164519.png)

---

## 🛠️ Step-by-Step Guide / Panduan Langkah demi Langkah

### Step 1: Identifikasi Endpoint & Parameter Target / Identify Target Endpoint & Parameter

**ID**: Buka salah satu halaman produk pada situs web target, lalu amati struktur URL-nya:  
**EN**: Open any product page on the target application and observe the URL structure:

```http
GET /product?productId=1 HTTP/1.1
Host: 0ac5002304c31dc0809c12c700010003.web-security-academy.net
```

![Product Detail Page](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805164543.png)

---

### Step 2: Trigger Type Error Exception / Memicu Pengecualian Error

**ID**: Kirim request HTTP dengan nilai non-numeric pada parameter `productId` (misalnya `productId=not_a_number`) menggunakan `curl` atau Burp Suite Repeater untuk memicu error `NumberFormatException`:  
**EN**: Send an HTTP request with a non-numeric string value for `productId` (e.g., `productId=not_a_number`) using `curl` or Burp Suite Repeater to trigger a `NumberFormatException`:

```bash
curl -s "https://0ac5002304c31dc0809c12c700010003.web-security-academy.net/product?productId=not_a_number"
```

---

### Step 3: Inspect Full Stack Trace & Software Version / Analisis Stack Trace & Versi Software

**ID**: Respon server mengembalikan *full Java stack trace* yang membocorkan versi framework di bagian akhir output:  
**EN**: The server response returns a full Java stack trace disclosing the framework version at the bottom of the output:

```text
Internal Server Error: java.lang.NumberFormatException: For input string: "not_a_number"
        at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
        at java.base/java.lang.Integer.parseInt(Integer.java:661)
        at java.base/java.lang.Integer.parseInt(Integer.java:777)
        ...
        at java.base/java.lang.Thread.run(Thread.java:1583)

Apache Struts 2 2.3.31
```

> [!NOTE]
> **Leaked Information / Informasi Terbocor**:
> Framework & Version: **Apache Struts 2 2.3.31**

---

### Step 4: Submit Solution / Submit Jawaban

**ID**: Kembali ke halaman lab, klik **Submit solution**, lalu masukkan string versi: `2 2.3.31`.  
**EN**: Return to the lab banner, click **Submit solution**, and enter the version string: `2 2.3.31`.

![Lab Solved Confirmation](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805164924.png)

> [!SUCCESS]
> **Lab Solved**: Versi Apache Struts 2 berhasil diidentifikasi dan disubmit!  
> *Apache Struts 2 version successfully identified and submitted!*