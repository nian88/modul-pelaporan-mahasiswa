
# 🟩 Bab 1 – Persiapan Awal Laravel  
**Cakupan:** Pertemuan 1–2  
**Proyek:** Sistem Pelaporan Masalah Akademik (Laravel + Tailwind)

---

## 📍 Pertemuan 1 – Instalasi & Pengenalan Laravel

### Pendahuluan
Laravel adalah framework PHP populer untuk membangun aplikasi web modern. Pertemuan ini berfokus pada instalasi Laravel, pengenalan struktur folder, dan konfigurasi awal `.env`.

### Tujuan
- Mahasiswa mampu menginstal Laravel.  
- Mahasiswa mengenali struktur folder.  
- Mahasiswa dapat mengatur file `.env`.  

### Materi Pokok
- Composer & Laravel  
- Artisan Serve  
- `.env`  
- Struktur Folder  

### Langkah Praktikum (Step-by-Step)
1. **Instal Composer**  
   - Download dari [https://getcomposer.org/download](https://getcomposer.org/download).  
   - Instalasi → cek dengan:  
     ```bash
     composer -V
     ```

2. **Buat project Laravel baru**  
   ```bash
   composer create-project laravel/laravel laporan-masalah
   cd laporan-masalah
   ```

3. **Jalankan server Laravel**  
   ```bash
   php artisan serve
   ```  
   - Akses `http://127.0.0.1:8000` → muncul halaman Laravel Welcome.

4. **Ubah konfigurasi `.env`**  
   ```
   APP_NAME="Sistem Pelaporan Masalah"
   ```

### Studi Kasus
Tambahkan route di `routes/web.php`:
```php
Route::get('/', function () {
    return "Sistem Pelaporan Masalah - Teknik Informatika";
});
```

### Tugas Mahasiswa
- Buat route `/halo` dengan output **“Halo, Laravel pertama saya”**.

### Ringkasan
Laravel berhasil diinstal, route pertama berhasil ditampilkan.

---

## 📍 Pertemuan 2 – Database & Migration

### Pendahuluan
Mahasiswa akan membuat database dan tabel menggunakan migration.

### Tujuan
- Mahasiswa membuat database di MySQL.  
- Mahasiswa membuat migration tabel mahasiswa, dosen, laporan.  
- Mahasiswa memahami relasi antar tabel.  

### Materi Pokok
- MySQL Database  
- Artisan Migration  
- Relasi Tabel  

### Langkah Praktikum (Step-by-Step)
1. **Buat database baru di MySQL**
```sql
CREATE DATABASE pelaporan;
```

2. **Atur koneksi database di `.env`**
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=pelaporan
DB_USERNAME=root
DB_PASSWORD=
```

3. **Buat migration mahasiswa**
```bash
php artisan make:migration create_mahasiswas_table
```
Isi:
```php
Schema::create('mahasiswas', function (Blueprint $table) {
    $table->id();
    $table->string('nama');
    $table->string('nim')->unique();
    $table->string('email')->unique();
    $table->timestamps();
});
```

4. **Buat migration dosen**
```bash
php artisan make:migration create_dosens_table
```
Isi:
```php
Schema::create('dosens', function (Blueprint $table) {
    $table->id();
    $table->string('nama');
    $table->string('nidn')->unique();
    $table->string('email')->unique();
    $table->timestamps();
});
```

5. **Buat migration laporan**
```bash
php artisan make:migration create_laporans_table
```
Isi:
```php
Schema::create('laporans', function (Blueprint $table) {
    $table->id();
    $table->string('judul');
    $table->text('deskripsi');
    $table->string('nomor_laporan')->unique();
    $table->enum('status',['baru','diproses','selesai'])->default('baru');
    $table->foreignId('mahasiswa_id')->constrained();
    $table->timestamps();
});
```

6. **Jalankan migration**
```bash
php artisan migrate
```

### Studi Kasus
Tambahkan relasi mahasiswa–laporan.

**`app/Models/Mahasiswa.php`**
```php
public function laporans() {
    return $this->hasMany(Laporan::class);
}
```

**`app/Models/Laporan.php`**
```php
public function mahasiswa() {
    return $this->belongsTo(Mahasiswa::class);
}
```

### Tugas Mahasiswa
- Buat migration `kategori` dan hubungkan ke laporan.  

### Ringkasan
Migration mempermudah pembuatan tabel dan relasi antar entitas.

---

