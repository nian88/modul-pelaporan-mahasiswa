# 🟦 Bab 2 – MVC & Templating  
**Cakupan:** Pertemuan 3–4  

## 📍 Pertemuan 3 – Route & Controller

### Pendahuluan
Route menghubungkan URL dengan controller. Controller berisi logika aplikasi.

### Tujuan
- Mahasiswa memahami route.  
- Mahasiswa membuat controller.  

### Materi Pokok
- Route dasar  
- Controller  
- MVC  

### Langkah Praktikum
1. **Buat Controller**
```bash
php artisan make:controller LaporanController
```
Isi:
```php
public function index() {
    return "Daftar Laporan Mahasiswa";
}
```

2. **Tambah Route**
```php
Route::get('/laporan', [LaporanController::class, 'index']);
```

3. **Uji di browser**
`http://127.0.0.1:8000/laporan`

### Studi Kasus
Buat `MahasiswaController` dengan route `/mahasiswa`.

### Ringkasan
Route → Controller → Response.

---

## 📍 Pertemuan 4 – Blade Templating & Tailwind

### Pendahuluan
Blade adalah template engine Laravel. Tailwind untuk styling.

### Tujuan
- Mahasiswa membuat layout Blade.  
- Mahasiswa integrasi Tailwind.  

### Materi Pokok
- Blade Layout  
- `@yield` & `@extends`  
- Tabel dengan Tailwind  

### Langkah Praktikum
1. **Buat layout master**
**`resources/views/layouts/app.blade.php`**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sistem Pelaporan Masalah</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100">
  <div class="container mx-auto p-6">
    <h1 class="text-2xl font-bold mb-4">Sistem Pelaporan Masalah</h1>
    @yield('content')
  </div>
</body>
</html>
```

2. **Buat view laporan**
**`resources/views/laporan/index.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-3">Daftar Laporan</h2>
<table class="w-full border-collapse border border-gray-300">
  <thead>
    <tr class="bg-gray-200">
      <th class="border px-4 py-2">No</th>
      <th class="border px-4 py-2">Judul</th>
      <th class="border px-4 py-2">Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="border px-4 py-2">1</td>
      <td class="border px-4 py-2">Masalah KRS</td>
      <td class="border px-4 py-2"><span class="bg-red-500 text-white px-2 py-1 rounded">Baru</span></td>
    </tr>
  </tbody>
</table>
@endsection
```

3. **Controller gunakan View**
```php
public function index() {
    return view('laporan.index');
}
```

4. **Uji di browser**
`http://127.0.0.1:8000/laporan` → tampil tabel.  

### Studi Kasus
Buat view daftar mahasiswa dengan Blade dan Tailwind.

### Ringkasan
Blade + Tailwind = tampilan modern dan rapi.

---
