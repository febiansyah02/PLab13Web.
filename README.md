# Laporan Praktikum Pemrograman Web
## Pertemuan 13: Autentikasi REST API & Navigation Guards pada SPA Frontend

---

## 1. Deskripsi Tugas & Tujuan Praktikum
Praktikum ini merupakan tahap final integrasi arsitektur **Single Page Application (SPA)**. Fokus utamanya adalah membangun sistem keamanan sinkron antara sisi client dan server:
1. **Frontend (Client-Side):** Menggunakan **Vue Router Navigation Guards** (`router.beforeEach`) untuk mengunci halaman sensitif sebelum user melakukan login.
2. **Backend (Server-Side):** Mengonfigurasi **RESTful Resource Controller** pada CodeIgniter 4 untuk memproses data payload JSON, mencocokkan kredensial database MySQL, serta menyuntikkan **CORS Header Validation** agar request Axios dari frontend tidak diblokir browser.

---

##  2. Lingkungan Kerja & Endpoint Jaringan
Aplikasi berjalan pada arsitektur terpisah (*Decoupled Architecture*) dengan pemetaan port berikut:
* **Frontend SPA Server (VueJS 3):** `http://localhost/lab8_vuejs/` (Port Apache XAMPP: `80`)
* **Backend REST API Server (CodeIgniter 4):** `http://localhost:8080` (Dijalankan via CLI Spark)
* **Database Server:** MySQL / MariaDB via phpMyAdmin (`http://localhost/phpmyadmin/`)

---

##  3. Struktur Direktori Proyek Aktif
Pastikan seluruh file berada pada posisi direktori aktif ini agar *namespace* penunjuk kelas komponen terbaca 100% tanpa eror:

```text
proyek-spa/
├── lab8_vuejs/                  # DIREKTORI FRONTEND (VUEJS 3)
│   ├── assets/
│   │   ├── css/
│   │   │   └── style.css        # Desain CSS Layout & Form Box Login
│   │   └── js/
│   │       ├── components/
│   │       │   ├── About.js     # Komponen Halaman Profil Pengembang
│   │       │   ├── Artikel.js   # Komponen Tabel CRUD Data Artikel
│   │       │   ├── Home.js      # Komponen Halaman Beranda Publik
│   │       │   └── Login.js     # Komponen Form Login Admin (Mandiri URL)
│   │       └── app.js           # Konfigurasi Vue Router & Navigation Guards
│   └── index.html               # Main Layout Induk SPA Frontend
│
└── lab11_ci/                    # DIREKTORI BACKEND (CODEIGNITER 4)
    └── ci4/
        ├── app/
        │   ├── Config/
        │   │   └── Routes.php   # Pemetaan Endpoint Menggunakan ::class
        │   ├── Controllers/
        │   │   ├── Api/
        │   │   │   └── Auth.php # Controller Login & Injeksi CORS Header Paksa
        │   │   └── Post.php     # Resource Controller CRUD Data Artikel
        │   └── Models/
        │       └── UserModel.php # Model Data Autentikasi User
        └── spark                # CLI Executable File Backend
```

4. Kode Sumber Implementasi Kunci
   
A. Sisi Backend: app/Config/Routes.php

Menggunakan penulisan kelas absolut (::class) modern untuk menghindari peringatan warna kuning di VS Code:

```
PHP
<?php

use CodeIgniter\Router\RouteCollection;
use App\Controllers\Artikel;
use App\Controllers\User;
use App\Controllers\AjaxController;
use App\Controllers\Api\Auth;

/** @var RouteCollection $routes */

$routes->get('/', [Artikel::class, 'index']); 
$routes->get('/artikel', [Artikel::class, 'index']); 
$routes->get('/artikel/(:any)', [Artikel::class, "view/$1"]);

$routes->add('user/login', [User::class, 'login']); 
$routes->get('user/logout', [User::class, 'logout']);

// Endpoint Autentikasi REST API Praktikum 13
$routes->post('api/login', [Auth::class, 'login']);

$routes->get('ajax', [AjaxController::class, 'index']);
$routes->get('ajax/getData', [AjaxController::class, 'getData']);
$routes->delete('ajax/delete/(:num)', [AjaxController::class, "delete/$1"]);

$routes->group('admin', ['filter' => 'auth'], function($routes) {
    $routes->get('artikel', [Artikel::class, 'admin_index']);
    $routes->add('artikel/add', [Artikel::class, 'add']);
    $routes->add('artikel/edit/(:any)', [Artikel::class, 'edit/$1']);
    $routes->get('artikel/delete/(:any)', [Artikel::class, 'delete/$1']);
});

$routes->resource('post');
```

B. Sisi Frontend: Navigation Guards (assets/js/app.js)

Logika inti yang bertugas mencegat dan memeriksa status token pengguna di localStorage:

```
JavaScript
// Konfigurasi Routing Guard Keamanan SPA
router.beforeEach((to, from, next) => {
    const isAuthenticated = localStorage.getItem('isLoggedIn') === 'true';
    
    // Jika halaman membutuhkan otentikasi DAN pengguna belum melakukan login
    if (to.matched.some(record => record.meta.requiresAuth) && !isAuthenticated) {
        alert('Akses Ditolak! Anda harus login terlebih dahulu.');
        next('/login'); // Belokkan paksa rute ke halaman form login
    } else {
        next(); // Izinkan akses rute tujuan
    }
});
```

5. Langkah Menjalankan & Pengujian Aplikasi
   
Langkah 1: Aktivasi Database MySQL

1. Pastikan module Apache dan MySQL pada XAMPP Control Panel sudah berjalan (Start).

2. Masuk ke http://localhost/phpmyadmin/, pastikan terdapat database lab11_ci dan tabel user berisi kolom kredensial (Contoh isi: username admin, password admin123).

Langkah 2: Menjalankan Server API CodeIgniter

1. Buka XAMPP Shell (tombol Shell di panel kanan XAMPP).

2. Ketik perintah pemindahan folder ke direktori proyek aktif:
```
Bash
cd \xampp\htdocs\lab11_ci\ci4
```
3. Nyalakan local server backend:
```
Bash
php spark serve
```

Langkah 3: Eksekusi Pengujian Frontend SPA

1. Buka browser, bersihkan sisa cache lama dengan menekan Ctrl + F5 (Hard Refresh), lalu akses: http://localhost/lab8_vuejs/

2. Uji Coba Proteksi: Klik menu Kelola Artikel atau About Pengembang. Sistem wajib memunculkan pop-up blokir akses dan melempar halaman langsung ke rute #/login.

3. Uji Coba Autentikasi: Masukkan kredensial database (admin / admin123) ke form login putih yang muncul. Klik Masuk Aplikasi. Data session token otomatis tersimpan, tautan menu berubah menjadi Logout, dan halaman tabel CRUD artikel langsung terbuka sempurna.

Dokumentasi:
<img width="959" height="539" alt="Cuplikan layar 2026-06-18 185008" src="https://github.com/user-attachments/assets/b1442b8d-c9db-4943-957d-3de4ceee2212" />
<img width="959" height="539" alt="Cuplikan layar 2026-06-18 185028" src="https://github.com/user-attachments/assets/b9f898fe-f243-43c9-9588-62ca3c0913f4" />
<img width="959" height="539" alt="Cuplikan layar 2026-06-18 185050" src="https://github.com/user-attachments/assets/48dcdfd7-0dec-4397-8537-c3937f904b58" />
<img width="957" height="539" alt="Cuplikan layar 2026-06-18 190241" src="https://github.com/user-attachments/assets/9341e473-cff7-429d-b497-d800918a9112" />
<img width="959" height="539" alt="Cuplikan layar 2026-06-18 190325" src="https://github.com/user-attachments/assets/5b54ff1d-3f96-416d-88dd-cc6ae100ed5c" />
