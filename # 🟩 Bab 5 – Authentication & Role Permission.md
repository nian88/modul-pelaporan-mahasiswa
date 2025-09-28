**Cakupan:** Pertemuan 9–10  
**Proyek:** Sistem Pelaporan Masalah Akademik (Laravel + Tailwind)

---

## 📍 Pertemuan 9 – Authentication (Auth dengan Laravel Breeze)

### Pendahuluan
Agar mahasiswa dan dosen pendamping akademik (DPA) dapat **login** ke sistem, Laravel menyediakan paket **Breeze** sebagai solusi sederhana untuk autentikasi (register, login, logout, reset password).

---

### Tujuan
- Mahasiswa memahami sistem autentikasi di Laravel.  
- Mahasiswa dapat menginstall dan mengkonfigurasi **Laravel Breeze**.  
- Mahasiswa dapat membedakan user yang login dan guest.  

---

### Materi Pokok
- Install Laravel Breeze  
- Migrasi tabel users  
- Register, login, logout  
- Middleware `auth`  

---

### Langkah Praktikum (Step-by-Step)

#### 1) Install Breeze
```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install && npm run dev
php artisan migrate
```

#### 2) Jalankan Server & Uji
```bash
php artisan serve
```
- Buka browser → `http://127.0.0.1:8000/register`  
- Daftarkan akun baru (Mahasiswa).  
- Logout → Login ulang.  

#### 3) Proteksi Route dengan Middleware `auth`
Edit **`routes/web.php`**:
```php
Route::get('/', function () {
    return view('welcome');
});

Route::middleware(['auth'])->group(function(){
    Route::resource('laporan', \App\Http\Controllers\LaporanController::class);
    Route::resource('mahasiswa', \App\Http\Controllers\MahasiswaController::class);
    Route::resource('dosen', \App\Http\Controllers\DosenController::class);
});
```

Sekarang, hanya user yang login yang bisa mengakses `/laporan`, `/mahasiswa`, `/dosen`.

#### 4) Simpan `mahasiswa_id` Otomatis dari User Login
Edit **`app/Http/Controllers/LaporanController.php` → store():**
```php
public function store(Request $request)
{
    $validated = $request->validate([
        'judul' => ['required','string','max:150'],
        'deskripsi' => ['required','string','max:2000'],
    ]);

    $nomorLaporan = $this->generateNomorLaporan();

    $laporan = \App\Models\Laporan::create([
        'judul' => $validated['judul'],
        'deskripsi' => $validated['deskripsi'],
        'nomor_laporan' => $nomorLaporan,
        'status' => 'baru',
        'mahasiswa_id' => auth()->id(), // otomatis ambil user login
    ]);

    return redirect()->route('laporan.show', $laporan)
        ->with('success','Laporan berhasil dibuat dengan nomor tiket: '.$nomorLaporan);
}
```

> **Instruksi klik:**  
> - Login sebagai Mahasiswa.  
> - Buat laporan → pelapor otomatis user login, tidak perlu dropdown lagi.

---

### Studi Kasus
1. Registrasi akun baru dengan email mahasiswa.  
2. Login → buat laporan baru.  
3. Cek di database: `mahasiswa_id` sesuai user login.  
4. Logout → coba akses `/laporan` → otomatis diarahkan ke halaman login.  

---

### Tugas Mahasiswa
- Tambahkan validasi password minimal 8 karakter saat register.  
- Buat link “Dashboard” di layout utama yang hanya muncul saat user login (`@auth` directive).  
- Tambahkan nama user login di pojok kanan atas layout (`{{ Auth::user()->name }}`).  

---

### Ringkasan
- Breeze menyediakan sistem register, login, logout otomatis.  
- Middleware `auth` membatasi akses hanya untuk user login.  
- `auth()->id()` menghubungkan laporan dengan user login.  

---

## 📍 Pertemuan 10 – Role & Permission

### Pendahuluan
Sistem pelaporan memiliki **3 role**:  
1. **Guest** → hanya melihat halaman awal.  
2. **Mahasiswa (pelapor)** → membuat laporan & melihat statusnya.  
3. **DPA (admin)** → mengelola semua laporan & mengubah status.  

Pertemuan ini berfokus pada penambahan kolom role pada tabel `users` dan penerapan **role-based access control**.

---

### Tujuan
- Mahasiswa memahami konsep role & permission.  
- Mahasiswa menambahkan kolom `role` pada tabel users.  
- Mahasiswa membatasi akses berdasarkan role.  

---

### Materi Pokok
- Penambahan kolom role pada `users`.  
- Middleware role.  
- Blade directive `@can` atau custom.  

---

### Langkah Praktikum (Step-by-Step)

#### 1) Tambahkan Kolom Role pada Tabel Users
Buat migration baru:
```bash
php artisan make:migration add_role_to_users_table --table=users
```
Edit file migration:
```php
Schema::table('users', function (Blueprint $table) {
    $table->enum('role',['mahasiswa','dpa'])->default('mahasiswa');
});
```
Jalankan:
```bash
php artisan migrate
```

#### 2) Update Model User
**`app/Models/User.php`**
```php
class User extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = [
        'name', 'email', 'password', 'role'
    ];
}
```

#### 3) Middleware Role
Buat middleware baru:
```bash
php artisan make:middleware RoleMiddleware
```
Edit file **`app/Http/Middleware/RoleMiddleware.php`**:
```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class RoleMiddleware
{
    public function handle(Request $request, Closure $next, $role)
    {
        if (!auth()->check() || auth()->user()->role !== $role) {
            abort(403,'Unauthorized.');
        }
        return $next($request);
    }
}
```

Daftarkan di **`app/Http/Kernel.php`**:
```php
protected $routeMiddleware = [
    // ...
    'role' => \App\Http\Middleware\RoleMiddleware::class,
];
```

#### 4) Terapkan Role pada Route
**`routes/web.php`**
```php
// Hanya mahasiswa yang bisa membuat laporan
Route::middleware(['auth','role:mahasiswa'])->group(function(){
    Route::resource('laporan', \App\Http\Controllers\LaporanController::class);
});

// Hanya DPA (admin) yang bisa mengelola mahasiswa, dosen, dan melihat semua laporan
Route::middleware(['auth','role:dpa'])->group(function(){
    Route::resource('mahasiswa', \App\Http\Controllers\MahasiswaController::class);
    Route::resource('dosen', \App\Http\Controllers\DosenController::class);
    Route::get('/admin/laporan', [\App\Http\Controllers\LaporanController::class,'index'])->name('admin.laporan.index');
});
```

#### 5) Batasi Tombol Aksi di Blade
Contoh: hanya DPA yang boleh ubah status laporan.

```php
@can('isDPA')
  <form action="{{ route('laporan.update', $laporan) }}" method="POST" class="inline-block ml-3">
    @csrf @method('PUT')
    <input type="hidden" name="judul" value="{{ $laporan->judul }}">
    <input type="hidden" name="deskripsi" value="{{ $laporan->deskripsi }}">
    <input type="hidden" name="mahasiswa_id" value="{{ $laporan->mahasiswa_id }}">
    <input type="hidden" name="status" value="{{ $laporan->status === 'baru' ? 'diproses' : 'selesai' }}">
    <button type="submit" class="text-green-600 hover:underline">
      {{ $laporan->status === 'baru' ? 'Proses' : ($laporan->status === 'diproses' ? 'Selesaikan' : '') }}
    </button>
  </form>
@endcan
```

Atau gunakan manual:
```php
@if(Auth::user()->role === 'dpa')
  <!-- tombol aksi -->
@endif
```

---

### Studi Kasus
1. Login sebagai **Mahasiswa** → bisa membuat laporan, tapi tidak bisa mengubah status.  
2. Login sebagai **DPA** → bisa melihat semua laporan dan mengubah status.  
3. Jika mahasiswa mencoba akses `/admin/laporan` → muncul error **403 Unauthorized**.  

---

### Tugas Mahasiswa
- Buat 1 akun role `mahasiswa` dan 1 akun role `dpa`.  
- Uji coba: Mahasiswa hanya bisa membuat laporan, DPA hanya bisa mengubah status.  
- Tambahkan menu navigasi: **Mahasiswa → Buat Laporan**, **DPA → Kelola Mahasiswa, Dosen, Semua Laporan**.  

---

### Ringkasan
- Role-based access control membedakan hak akses user.  
- Middleware custom `role` membatasi akses pada level route.  
- Blade directive memudahkan menampilkan UI sesuai role user.  

---

## ✅ Checklist Hasil Belajar Bab 5
- [ ] Sistem login & register berfungsi dengan Breeze.  
- [ ] Middleware `auth` melindungi route.  
- [ ] User login otomatis menjadi pelapor.  
- [ ] Kolom `role` ditambahkan pada tabel users.  
- [ ] Middleware `role` membatasi akses Mahasiswa & DPA.  
- [ ] UI menyesuaikan role user login.  

---

### Catatan Lanjutan (Menuju Bab 6)
- Pertemuan berikutnya fokus pada **Dashboard khusus Mahasiswa & DPA** (Pertemuan 11) serta **Notifikasi & Tracking** (Pertemuan 12).  
- Mahasiswa akan belajar membedakan UI untuk tiap role dengan Blade & Tailwind.  

---
