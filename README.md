# SQL INJECTION

---

# BAGIAN 1

## Apa Itu SQL Injection (Konsep Paling Dasar)

### 1.1 Definisi Sederhana

**SQL Injection (SQLi)** adalah kondisi ketika **input pengguna masuk langsung ke query SQL tanpa pengamanan**, sehingga pengguna dapat **mengubah struktur query database** dan membuat database menjalankan perintah yang tidak dimaksudkan oleh developer.

Intinya:

> Data yang seharusnya hanya menjadi nilai, berubah menjadi bagian dari perintah SQL.

---

### 1.2 Mengapa SQL Injection Sangat Berbahaya

Database adalah pusat sistem:

* Menyimpan akun dan password
* Menyimpan transaksi
* Menyimpan data pribadi
* Menentukan hak akses

Jika database bisa dimanipulasi:

* Login bisa dibypass
* Data bisa dicuri massal
* Data bisa diubah atau dihapus
* Sistem bisa dikendalikan sepenuhnya

Karena itu SQL Injection dikategorikan sebagai **vulnerability kritis**.

---

### 1.3 SQL Injection Bukan Bug Database

SQL Injection:

* Bukan bug MySQL
* Bukan bug PostgreSQL
* Bukan bug Oracle
* Bukan bug SQL Server

SQL Injection adalah **bug aplikasi**, khususnya:

* Cara menyusun query
* Cara memperlakukan input user
* Cara memisahkan data dan perintah

---

# BAGIAN 2

## Cara Website dan Database Bekerja (Fondasi Wajib)

Sebelum memahami SQL Injection, kita harus memahami **alur kerja aplikasi web**.

---

## 2.1 Alur Normal Aplikasi Web

```
User → Browser → Server → Backend → Database → Backend → Server → Browser
```

Penjelasan:

1. User mengetik URL atau mengisi form
2. Browser mengirim HTTP request ke server
3. Server meneruskan request ke backend logic
4. Backend menyusun query SQL
5. Database menjalankan query
6. Hasil dikembalikan ke backend
7. Backend membentuk response HTML/JSON
8. Response dikirim ke browser

---

## 2.2 Contoh Sistem Login Normal

User membuka halaman login:

```
https://example.com/login
```

Form HTML:

```html
<form method="POST" action="/login">
  <input name="username">
  <input name="password" type="password">
  <button>Login</button>
</form>
```

User mengisi:

```
username = andi
password = 12345
```

Browser mengirim:

```
POST /login
username=andi&password=12345
```

Backend memproses dan membuat query:

```sql
SELECT * FROM users WHERE username='andi' AND password='12345'
```

Jika cocok → login sukses
Jika tidak cocok → login gagal

Ini adalah **alur normal tanpa SQL Injection**.

---

# BAGIAN 3

## Akar Masalah SQL Injection (Root Cause Sebenarnya)

SQL Injection tidak terjadi karena hacker jenius, tetapi karena **aplikasi salah memperlakukan input**.

---

## 3.1 Pola Kesalahan Paling Umum

Backend code yang rentan (contoh PHP):

```php
$query = "SELECT * FROM users WHERE username = '" . $_POST['username'] . "' AND password = '" . $_POST['password'] . "'";
```

Masalahnya:

* `$_POST['username']` berasal dari user
* `$_POST['password']` berasal dari user
* Digabung langsung ke string SQL
* Database tidak tahu mana data, mana perintah

---

## 3.2 Kenapa Ini Sangat Berbahaya

Database membaca query sebagai **bahasa pemrograman**, bukan sebagai data.

Jika user memasukkan:

```
andi' OR '1'='1
```

Maka query berubah struktur, bukan hanya nilainya.

---

## 3.3 Prinsip Inti SQL Injection

SQL Injection terjadi ketika:

> Input user ikut membentuk struktur grammar SQL.

Jika:

* Input hanya menjadi parameter nilai → aman
* Input bisa mengubah logika query → rentan

---

# BAGIAN 4

## Bagaimana Database Membaca Query (Agar Paham Injection Secara Teknis)

Database memproses query melalui tahap:

1. **Tokenizing**
   Query dipecah menjadi kata-kata SQL

2. **Parsing**
   Database membangun struktur logika query

3. **Semantic validation**
   Mengecek tabel, kolom, dan izin akses

4. **Execution**
   Query dijalankan

Jika input user sudah masuk sebelum parsing, maka database akan memperlakukan input itu sebagai bagian dari struktur SQL.

---

# BAGIAN 5

## SQL Injection dari Nol: Contoh Login Bypass Nyata

Bagian ini menjelaskan **secara konkret**, dari input form sampai query di database.

---

## 5.1 Query Login Rentan

Backend:

```sql
SELECT * FROM users WHERE username='$u' AND password='$p'
```

---

## 5.2 User Normal Login

User mengisi:

```
Username: andi
Password: 12345
```

Query:

```sql
SELECT * FROM users WHERE username='andi' AND password='12345'
```

Jika cocok → login sukses.

---

## 5.3 User Menyisipkan Input Berbahaya

User mengisi:

```
Username: andi' --
Password: bebas
```

Browser mengirim:

```
username=andi' --&password=bebas
```

Backend menyusun query:

```sql
SELECT * FROM users WHERE username='andi' -- ' AND password='bebas'
```

Penjelasan:

* `'` menutup string
* `--` membuat komentar SQL
* Semua setelah `--` diabaikan

Query efektif menjadi:

```sql
SELECT * FROM users WHERE username='andi'
```

Password tidak diperiksa sama sekali.
Login berhasil tanpa password.

---

## 5.4 Mengapa Ini Bisa Terjadi?

Karena:

* Backend menggabungkan string SQL
* Tidak ada parameter binding
* Tidak ada validasi struktur input
* Database tidak membedakan data vs perintah

---

# BAGIAN 6

## Di Mana Saja SQL Injection Bisa Terjadi (Semua Lokasi Input Nyata)

SQL Injection tidak hanya di login.

Jika input masuk ke SQL tanpa parameterization, maka injection mungkin terjadi.

---

## 6.1 URL Parameter (GET Request)

### Contoh URL:

```
https://example.com/product.php?id=10
```

Backend:

```php
$id = $_GET['id'];
$query = "SELECT * FROM products WHERE id = '$id'";
```

User normal:

```
id=10
```

Query:

```sql
SELECT * FROM products WHERE id='10'
```

---

### Input Disusupi

User mengakses:

```
https://example.com/product.php?id=10' OR '1'='1
```

Server menerima:

```
id=10' OR '1'='1
```

Query menjadi:

```sql
SELECT * FROM products WHERE id='10' OR '1'='1'
```

Karena `'1'='1'` selalu benar, database mengembalikan semua produk.

---

## 6.2 Form Search

### Form:

```html
<form method="GET" action="/search">
  <input name="q">
</form>
```

User normal:

```
q=laptop
```

Backend:

```sql
SELECT * FROM products WHERE name LIKE '%laptop%'
```

---

### Input Disusupi

User memasukkan:

```
q=laptop' OR '1'='1
```

Query menjadi:

```sql
SELECT * FROM products WHERE name LIKE '%laptop' OR '1'='1%'
```

Logika query berubah dan semua data bisa muncul.

---

## 6.3 Form Login (POST Request)

```
POST /login
username=admin&password=1234
```

Backend:

```sql
SELECT * FROM users WHERE username='$username' AND password='$password'
```

Jika `username` atau `password` dimodifikasi → injection.

---

## 6.4 REST API (JSON Body)

Frontend mobile app mengirim:

```
POST /api/user/profile
Content-Type: application/json

{
  "userId": "10"
}
```

Backend Node.js:

```js
db.query("SELECT * FROM users WHERE id = " + req.body.userId);
```

User normal:

```
"userId": "10"
```

Query:

```sql
SELECT * FROM users WHERE id = 10
```

---

### Input Disusupi

```
"userId": "10 OR 1=1"
```

Query:

```sql
SELECT * FROM users WHERE id = 10 OR 1=1
```

Database mengembalikan semua user.

---

## 6.5 Cookie-Based SQL Injection

Browser mengirim cookie:

```
Cookie: user_id=10
```

Backend:

```php
$user_id = $_COOKIE['user_id'];
$query = "SELECT * FROM users WHERE id = '$user_id'";
```

Jika attacker memodifikasi cookie:

```
user_id=10' OR '1'='1
```

Query berubah.

---

## 6.6 HTTP Header-Based SQL Injection

Server logging:

```sql
INSERT INTO logs (user_agent) VALUES ('$ua')
```

Jika `User-Agent` berisi SQL fragment dan nanti dipakai di query lain → injection bisa terjadi (sering second-order).

---

## 6.7 File Upload Metadata

Contoh:

* Filename disimpan ke DB
* EXIF metadata disimpan
* Description disimpan

Jika nanti dipakai dalam query lain → injection bisa terjadi.

---

# BAGIAN 7

## Bentuk URL yang Paling Sering Rentan (Real Pattern)

| Pola URL     | Contoh                      |
| ------------ | --------------------------- |
| ID parameter | `/product?id=10`            |
| Search query | `/search?q=laptop`          |
| Filter       | `/products?category=mouse`  |
| Sort         | `/products?sort=price`      |
| Pagination   | `/products?page=2&limit=10` |
| REST path    | `/api/user/10`              |

Semua pola ini bisa jadi injection point jika backend menyusun SQL dengan string concatenation.

---

# BAGIAN 8

## SQL Injection pada Login System (Studi Kasus Lengkap dari Input sampai Query)

---

## 8.1 Struktur Login Nyata

Form:

```html
<form method="POST" action="/login">
  <input name="username">
  <input name="password" type="password">
</form>
```

Backend PHP:

```php
$username = $_POST['username'];
$password = $_POST['password'];
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysqli_query($conn, $query);
```

---

## 8.2 Login Normal

User:

```
username=andi
password=12345
```

Query:

```sql
SELECT * FROM users WHERE username='andi' AND password='12345'
```

---

## 8.3 Login Bypass Injection

User input:

```
username=admin' --
password=apa_saja
```

Query:

```sql
SELECT * FROM users WHERE username='admin' -- ' AND password='apa_saja'
```

Password tidak diperiksa. Login sebagai admin berhasil.

---

## 8.4 Analisis Teknis Kenapa Ini Berhasil

1. `'` menutup string
2. `--` mengomentari sisa query
3. Struktur query berubah
4. Kondisi password dihilangkan
5. Database menjalankan query baru tanpa error

---

# BAGIAN 9

## SQL Injection di Search, Filter, Pagination, dan Sort (Contoh Nyata)

---

## 9.1 Search Box

URL:

```
/search?q=mouse
```

Backend:

```sql
SELECT * FROM products WHERE name LIKE '%$q%'
```

Jika `q` memuat SQL grammar → injection.

---

## 9.2 Filter

URL:

```
/products?category=keyboard
```

Backend:

```sql
SELECT * FROM products WHERE category = '$category'
```

---

## 9.3 Pagination

URL:

```
/products?page=2&limit=10
```

Backend:

```sql
SELECT * FROM products LIMIT $limit OFFSET $offset
```

Jika `limit` atau `offset` tidak divalidasi sebagai integer → injection.

---

## 9.4 Sort (Kasus Paling Sering Terlewat)

URL:

```
/products?sort=price
```

Backend:

```sql
SELECT * FROM products ORDER BY $sort
```

Parameter `ORDER BY` tidak bisa diparameterisasi dengan placeholder, sehingga harus whitelist. Jika tidak → injection sangat sering terjadi.

---

# BAGIAN 10

## SQL Injection di API Modern (SPA, Mobile App, JSON)

Walaupun frontend modern, backend tetap membangun query SQL.

---

## Contoh Nyata

Mobile app mengirim:

```
POST /api/orders
{
  "userId": "12",
  "status": "paid"
}
```

Backend:

```js
db.query("SELECT * FROM orders WHERE user_id = " + body.userId + " AND status = '" + body.status + "'");
```

Jika `userId` atau `status` bisa dimanipulasi → SQL Injection terjadi.

---

# BAGIAN 11

## SQL Injection di Cookie dan Header (Second-Order Umum)

---

## Contoh Cookie

Cookie:

```
role=user
```

Backend:

```sql
SELECT * FROM permissions WHERE role = '$role'
```

Jika attacker mengubah cookie → injection.

---

## Contoh Header

Backend logging:

```sql
INSERT INTO logs (user_agent) VALUES ('$ua')
```

Jika `User-Agent` berisi SQL fragment dan nanti digunakan dalam query admin panel → second-order SQL Injection.

---

# BAGIAN 12

## SQL Injection di File Upload dan Metadata

Misalnya:

* Filename disimpan
* EXIF metadata disimpan
* Description disimpan

Jika data tersebut digunakan dalam query lain tanpa parameter binding → injection terjadi.

---

# BAGIAN 13

## SQL Injection dalam Stored Procedure

Stored procedure **tidak otomatis aman**.

Contoh rentan:

```sql
CREATE PROCEDURE getUser(IN name VARCHAR(100))
BEGIN
  SET @q = CONCAT('SELECT * FROM users WHERE name = ''', name, '''');
  PREPARE stmt FROM @q;
  EXECUTE stmt;
END;
```

Jika `name` berisi SQL grammar → injection tetap terjadi karena query dibangun sebagai string.

---

# BAGIAN 14

## SQL Injection dalam ORM dan Framework

ORM hanya aman jika:

* Tidak menggunakan raw SQL
* Tidak membangun query string manual
* Tidak mengizinkan input masuk ke bagian struktural query

---

## Contoh Rentan

Python:

```python
query = f"SELECT * FROM users ORDER BY {sort}"
db.execute(query)
```

Jika `sort` berasal dari user → injection.

Harus:

* Whitelist kolom
* Validasi nilai

---

# BAGIAN 15

## Second-Order SQL Injection (Injection Tertunda)

Second-order SQL Injection terjadi ketika:

1. Payload dimasukkan
2. Disimpan ke database
3. Tidak langsung dieksekusi
4. Digunakan kembali dalam query lain
5. Baru terjadi injection

---

## Contoh Nyata

User mendaftar:

```
username = andi' --
```

Disimpan di database sebagai string biasa.

Admin membuka halaman:

```sql
SELECT * FROM users WHERE username = '$username'
```

Query menjadi:

```sql
SELECT * FROM users WHERE username = 'andi' -- '
```

Struktur query rusak atau dimanipulasi. Injection terjadi **bukan saat input**, tapi saat data digunakan kembali.

---

# BAGIAN 16

## Klasifikasi SQL Injection Berdasarkan Cara Data Keluar

---

## 16.1 In-Band SQL Injection

Data hasil injection langsung terlihat di response HTML/JSON.

---

## 16.2 Blind SQL Injection

Tidak ada data langsung, tapi:

* Halaman berbeda
* Status code berbeda
* Waktu respon berbeda

Attacker menyimpulkan informasi berdasarkan perbedaan tersebut.

---

## 16.3 Out-of-Band SQL Injection

Database mengirim data lewat:

* DNS request
* HTTP request

Ke server luar.

Digunakan jika:

* Tidak ada output
* Tidak ada error
* Tidak ada perbedaan waktu

---

# BAGIAN 17

## Jenis SQL Injection Berdasarkan Teknik

| Teknik              | Penjelasan                    |
| ------------------- | ----------------------------- |
| Error-based         | Menggunakan pesan error DB    |
| Union-based         | Menggabungkan hasil query     |
| Boolean-based blind | Mengamati true/false response |
| Time-based blind    | Mengukur delay respon         |
| Stacked queries     | Menjalankan multi statement   |
| Out-of-band         | Mengirim data via DNS/HTTP    |
| Second-order        | Injection tertunda            |

---

# BAGIAN 18

## Error-Based SQL Injection (Penjelasan Teknis)

Jika aplikasi menampilkan error database:

* Attacker bisa melihat:

  * Nama tabel
  * Nama kolom
  * Struktur query
  * DBMS type

Masalah utamanya:

* Error tidak disembunyikan
* Error menjadi sumber informasi

---

# BAGIAN 19

## Union-Based SQL Injection (Penjelasan Teknis)

Jika query awal:

```sql
SELECT name, price FROM products WHERE id = 10
```

Attacker mencoba menggabungkan query lain:

```sql
SELECT name, price FROM products WHERE id = 10
UNION SELECT username, password FROM users
```

Syarat:

* Jumlah kolom sama
* Tipe data kompatibel
* Output ditampilkan ke user

---

# BAGIAN 20

## Blind SQL Injection (Boolean-Based)

Aplikasi tidak menampilkan data database, tapi:

* Jika kondisi benar → halaman normal
* Jika kondisi salah → halaman error / kosong

Dengan mengubah kondisi logika, attacker bisa menyimpulkan isi database.

---

# BAGIAN 21

## Blind SQL Injection (Time-Based)

Jika respon selalu sama, attacker:

* Menyisipkan perintah delay
* Mengukur waktu respon
* Menentukan true/false kondisi

Teknik ini lambat tapi tetap efektif.

---

# BAGIAN 22

## Out-of-Band SQL Injection

Jika database bisa:

* DNS lookup
* HTTP request

Maka data bisa dikirim keluar melalui channel tersebut.

Digunakan jika semua channel lain tertutup.

---

# BAGIAN 23

## Stacked Queries Injection

Jika database mengizinkan:

```sql
SELECT * FROM users; DROP TABLE users;
```

Maka attacker bisa menjalankan beberapa perintah dalam satu request.

---

# BAGIAN 24

## SQL Injection di Berbagai Database Engine

---

## MySQL / MariaDB

* Komentar: `--`, `#`
* Delay: `SLEEP()`
* File access: `LOAD_FILE()`

---

## PostgreSQL

* Delay: `pg_sleep()`
* Bisa akses OS via extension tertentu

---

## MSSQL

* Delay: `WAITFOR DELAY`
* OS command via `xp_cmdshell`

---

## Oracle

* Network access via `UTL_HTTP`, `UTL_INADDR`

Setiap DBMS punya karakteristik yang memengaruhi teknik eksploit.

---

# BAGIAN 25

## SQL Injection di Sistem Modern

---

## SPA (React/Vue/Angular)

Frontend modern tetap mengirim data ke backend API. Jika backend menyusun SQL dengan string concatenation → injection tetap terjadi.

---

## GraphQL

Resolver GraphQL membangun query SQL dari argument → injection terjadi jika tidak diparameterisasi.

---

## Microservices

Satu service vulnerable bisa membuka akses ke banyak service lain melalui trust boundary internal.

---

# BAGIAN 26

## Dampak SQL Injection (Teknis & Bisnis)

| Dampak Teknis          | Dampak Bisnis     |
| ---------------------- | ----------------- |
| Login bypass           | Account takeover  |
| Data dump              | Kebocoran data    |
| Data manipulation      | Fraud             |
| Data deletion          | Kerusakan sistem  |
| Remote code execution  | Server compromise |
| Backdoor persistence   | Re-intrusion      |
| Pivot internal network | Lateral movement  |
| Compliance violation   | Sanksi hukum      |

---

# BAGIAN 27

## Siklus Serangan SQL Injection (Kill Chain)

1. Recon: Cari parameter input
2. Testing: Validasi injection point
3. Fingerprinting: Tentukan DBMS
4. Enumeration: Struktur database
5. Extraction: Dump data
6. Privilege escalation
7. Persistence
8. Lateral movement

---

# BAGIAN 28

## Bagaimana SQL Injection Ditemukan (Metodologi Defensive)

---

## Manual Testing

* Input karakter aneh
* Bandingkan response
* Perhatikan error
* Perhatikan waktu respon

---

## Automated Testing

* Scanner digunakan di lab/test environment

---

# BAGIAN 29

## Kesalahan Developer yang Paling Sering Menyebabkan SQL Injection

1. Menggabungkan string SQL
2. Menganggap input numeric aman
3. Menggunakan blacklist filter
4. Percaya validasi frontend
5. Menampilkan error SQL ke user
6. Menggunakan akun DB dengan hak root
7. Tidak hashing password
8. Tidak logging query abnormal
9. Tidak memisahkan data dan perintah

---

# BAGIAN 30

## Arsitektur Sistem yang Aman terhadap SQL Injection

```
Client
  ↓
Controller (validasi input)
  ↓
Service Layer
  ↓
Repository / DAO (prepared statements)
  ↓
Database
```

Prinsip:

* Parameter binding
* Least privilege
* Whitelist struktural
* Error masking
* Monitoring

---

# BAGIAN 31

## Cara Pencegahan SQL Injection (Lengkap dan Praktis)

---

## Level Kode

* Prepared statements
* ORM query builder
* Type casting
* Whitelist kolom ORDER BY

---

## Level Framework

* Input validation middleware
* Centralized error handler

---

## Level Database

* Least privilege
* Disable dangerous functions
* Disable stacked queries

---

## Level Infrastruktur

* WAF
* IDS/IPS
* Rate limiting
* Query anomaly logging

---

# BAGIAN 32

## SQL Injection vs Vulnerability Lain

| Vulnerability     | Target          |
| ----------------- | --------------- |
| SQL Injection     | Database        |
| XSS               | Browser         |
| Command Injection | OS shell        |
| LDAP Injection    | Directory       |
| SSTI              | Template engine |
| NoSQL Injection   | JSON query      |

SQL Injection khusus menyerang SQL database.

---

# BAGIAN 33

## Studi Kasus Lengkap dari Input sampai Database (Contoh Nyata)

---

## Sistem: Halaman Detail Produk

URL:

```
/product.php?id=100
```

Backend PHP:

```php
$id = $_GET['id'];
$query = "SELECT * FROM products WHERE id = '$id'";
$result = mysqli_query($conn, $query);
```

---

### User Normal

```
/product.php?id=100
```

Query:

```sql
SELECT * FROM products WHERE id='100'
```

---

### User Menyisipkan Input

```
/product.php?id=100' OR '1'='1
```

Server menerima:

```
id=100' OR '1'='1
```

Query:

```sql
SELECT * FROM products WHERE id='100' OR '1'='1'
```

Karena `'1'='1'` selalu benar, semua produk dikembalikan.

---

## Analisis Teknis

1. Input masuk ke SQL string
2. `'` menutup literal string
3. `OR '1'='1'` mengubah logika WHERE
4. Struktur query berubah
5. Database menjalankan query baru

---

# BAGIAN 34

## Inti Filosofis SQL Injection

SQL Injection bukan soal payload.

SQL Injection adalah soal:

> Apakah data bisa berubah menjadi perintah.

Jika:

* Data tidak bisa mengubah struktur query → aman
* Data bisa mengubah struktur query → rentan

---

# BAGIAN 35

## Kesimpulan Akhir

SQL Injection terjadi karena:

1. Input user dimasukkan langsung ke query SQL
2. Query disusun sebagai string
3. Database tidak membedakan data vs perintah
4. Struktur query bisa diubah
5. Database mengeksekusi query yang berubah

Solusi utama:

* Parameterized query
* ORM safe query
* Whitelist input struktural
* Least privilege database user
* Error masking
* Monitoring
