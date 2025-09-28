**Cakupan:** Pertemuan 5–6  
**Proyek:** Sistem Pelaporan Masalah Akademik (Laravel + Tailwind)

---

## 📍 Pertemuan 5 – CRUD Master Data (Mahasiswa & Dosen)

### Pendahuluan
Kita membangun **CRUD** (Create, Read, Update, Delete) untuk **Mahasiswa** dan **Dosen** sebagai **master data**. Data ini akan dipakai pada proses pelaporan di pertemuan berikutnya.

### Tujuan
- Mahasiswa mampu membangun CRUD lengkap menggunakan Eloquent & Blade.  
- Mahasiswa menerapkan validasi, mass assignment (fillable), dan flash message.  
- Mahasiswa membangun UI sederhana dan elegan menggunakan Tailwind.

### Materi Pokok
- Resource Controller & Route Resource  
- Eloquent Model (fillable)  
- Blade View (index, create, edit, show)  
- Validasi & Flash Message  
- Tailwind untuk UI

> **Catatan:** Di Bab 1, kita sudah membuat migration untuk `mahasiswas` dan `dosens`. Di sini kita **tidak** membuat migration lagi (menghindari duplikasi).

---

### Langkah Praktikum (Step-by-Step)

#### 1) Pastikan Struktur Model (Mahasiswa & Dosen)
- **Cek/Perbarui** file model jika belum ada atau belum berisi `fillable`.

**`app/Models/Mahasiswa.php`**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Mahasiswa extends Model
{
    use HasFactory;

    protected $fillable = ['nama','nim','email'];

    public function laporans()
    {
        return $this->hasMany(Laporan::class);
    }
}
```

**`app/Models/Dosen.php`**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Dosen extends Model
{
    use HasFactory;

    protected $fillable = ['nama','nidn','email'];
}
```

#### 2) Buat Resource Controller
```bash
php artisan make:controller MahasiswaController --resource --model=Mahasiswa
php artisan make:controller DosenController --resource --model=Dosen
```

Perintah di atas akan membuat:  
`app/Http/Controllers/MahasiswaController.php` dan `DosenController.php` berisi method: `index, create, store, show, edit, update, destroy`.

#### 3) Daftarkan Route Resource
Buka `routes/web.php` dan tambahkan:
```php
use App\Http\Controllers\MahasiswaController;
use App\Http\Controllers\DosenController;

Route::resource('mahasiswa', MahasiswaController::class);
Route::resource('dosen', DosenController::class);
```

> **Instruksi klik:** Jalankan server (`php artisan serve`) → buka browser `http://127.0.0.1:8000/mahasiswa` dan `http://127.0.0.1:8000/dosen`. Karena view belum dibuat, halaman mungkin kosong atau error **view not found**. Kita buat view-nya.

#### 4) Siapkan Folder View & Komponen Flash
Buat folder:
```
resources/views/mahasiswa
resources/views/dosen
resources/views/components
```

**`resources/views/components/alert-success.blade.php`**
```php
@if(session('success'))
  <div class="mb-4 rounded border border-green-200 bg-green-50 p-3 text-green-700">
    {{ session('success') }}
  </div>
@endif
```

#### 5) View & Controller — Mahasiswa
**a. Index (list + tombol tambah + aksi edit/hapus/show)**

**`resources/views/mahasiswa/index.blade.php`**
```php
@extends('layouts.app')

@section('content')
<div class="flex items-center justify-between mb-4">
  <h2 class="text-xl font-semibold">Daftar Mahasiswa</h2>
  <a href="{{ route('mahasiswa.create') }}" class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700">+ Tambah Mahasiswa</a>
</div>

<x-alert-success />

<table class="w-full border-collapse bg-white rounded shadow">
  <thead>
    <tr class="bg-gray-100 text-left">
      <th class="px-4 py-2 border">#</th>
      <th class="px-4 py-2 border">Nama</th>
      <th class="px-4 py-2 border">NIM</th>
      <th class="px-4 py-2 border">Email</th>
      <th class="px-4 py-2 border">Aksi</th>
    </tr>
  </thead>
  <tbody>
    @forelse ($mahasiswas as $index => $mahasiswa)
      <tr>
        <td class="px-4 py-2 border">{{ $mahasiswas->firstItem() + $index }}</td>
        <td class="px-4 py-2 border">{{ $mahasiswa->nama }}</td>
        <td class="px-4 py-2 border">{{ $mahasiswa->nim }}</td>
        <td class="px-4 py-2 border">{{ $mahasiswa->email }}</td>
        <td class="px-4 py-2 border">
          <a href="{{ route('mahasiswa.show', $mahasiswa) }}" class="text-blue-700 hover:underline">Lihat</a>
          <a href="{{ route('mahasiswa.edit', $mahasiswa) }}" class="ml-3 text-yellow-600 hover:underline">Edit</a>
          <form action="{{ route('mahasiswa.destroy', $mahasiswa) }}" method="POST" class="inline-block ml-3" onsubmit="return confirm('Yakin hapus data?')">
            @csrf @method('DELETE')
            <button type="submit" class="text-red-600 hover:underline">Hapus</button>
          </form>
        </td>
      </tr>
    @empty
      <tr><td colspan="5" class="px-4 py-6 border text-center text-gray-500">Belum ada data</td></tr>
    @endforelse
  </tbody>
</table>

<div class="mt-4">
  {{ $mahasiswas->links() }}
</div>
@endsection
```

**Controller `index()`**  
**`app/Http/Controllers/MahasiswaController.php`**
```php
public function index()
{
    $mahasiswas = \App\Models\Mahasiswa::orderBy('nama')->paginate(10);
    return view('mahasiswa.index', compact('mahasiswas'));
}
```

**b. Create (form tambah)**

**`resources/views/mahasiswa/create.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-4">Tambah Mahasiswa</h2>

@if ($errors->any())
  <div class="mb-4 rounded border border-red-200 bg-red-50 p-3 text-red-700">
    <ul class="list-disc pl-5">
      @foreach ($errors->all() as $error)
        <li>{{ $error }}</li>
      @endforeach
    </ul>
  </div>
@endif

<form action="{{ route('mahasiswa.store') }}" method="POST" class="bg-white p-4 rounded shadow max-w-xl">
  @csrf
  <div class="mb-3">
    <label class="block mb-1">Nama</label>
    <input type="text" name="nama" value="{{ old('nama') }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="mb-3">
    <label class="block mb-1">NIM</label>
    <input type="text" name="nim" value="{{ old('nim') }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="mb-4">
    <label class="block mb-1">Email</label>
    <input type="email" name="email" value="{{ old('email') }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="flex items-center gap-2">
    <button type="submit" class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700">Simpan</button>
    <a href="{{ route('mahasiswa.index') }}" class="px-4 py-2 rounded border hover:bg-gray-50">Batal</a>
  </div>
</form>
@endsection
```

**Controller `create()` dan `store()`**
```php
public function create()
{
    return view('mahasiswa.create');
}

public function store(\Illuminate\Http\Request $request)
{
    $validated = $request->validate([
        'nama' => ['required','string','max:100'],
        'nim'  => ['required','string','max:20','unique:mahasiswas,nim'],
        'email'=> ['required','email','max:100','unique:mahasiswas,email'],
    ]);

    \App\Models\Mahasiswa::create($validated);

    return redirect()->route('mahasiswa.index')->with('success','Data mahasiswa berhasil ditambahkan.');
}
```

**c. Edit & Update**

**`resources/views/mahasiswa/edit.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-4">Edit Mahasiswa</h2>

@if ($errors->any())
  <div class="mb-4 rounded border border-red-200 bg-red-50 p-3 text-red-700">
    <ul class="list-disc pl-5">
      @foreach ($errors->all() as $error)
        <li>{{ $error }}</li>
      @endforeach
    </ul>
  </div>
@endif

<form action="{{ route('mahasiswa.update', $mahasiswa) }}" method="POST" class="bg-white p-4 rounded shadow max-w-xl">
  @csrf @method('PUT')
  <div class="mb-3">
    <label class="block mb-1">Nama</label>
    <input type="text" name="nama" value="{{ old('nama', $mahasiswa->nama) }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="mb-3">
    <label class="block mb-1">NIM</label>
    <input type="text" name="nim" value="{{ old('nim', $mahasiswa->nim) }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="mb-4">
    <label class="block mb-1">Email</label>
    <input type="email" name="email" value="{{ old('email', $mahasiswa->email) }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="flex items-center gap-2">
    <button type="submit" class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700">Update</button>
    <a href="{{ route('mahasiswa.index') }}" class="px-4 py-2 rounded border hover:bg-gray-50">Batal</a>
  </div>
</form>
@endsection
```

**Controller `edit()` dan `update()`**
```php
public function edit(\App\Models\Mahasiswa $mahasiswa)
{
    return view('mahasiswa.edit', compact('mahasiswa'));
}

public function update(\Illuminate\Http\Request $request, \App\Models\Mahasiswa $mahasiswa)
{
    $validated = $request->validate([
        'nama' => ['required','string','max:100'],
        'nim'  => ['required','string','max:20','unique:mahasiswas,nim,'.$mahasiswa->id],
        'email'=> ['required','email','max:100','unique:mahasiswas,email,'.$mahasiswa->id],
    ]);

    $mahasiswa->update($validated);

    return redirect()->route('mahasiswa.index')->with('success','Data mahasiswa berhasil diperbarui.');
}
```

**d. Show (detail)**

**`resources/views/mahasiswa/show.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-4">Detail Mahasiswa</h2>

<div class="bg-white rounded shadow p-4 max-w-xl">
  <div class="mb-2"><span class="font-semibold">Nama:</span> {{ $mahasiswa->nama }}</div>
  <div class="mb-2"><span class="font-semibold">NIM:</span> {{ $mahasiswa->nim }}</div>
  <div class="mb-4"><span class="font-semibold">Email:</span> {{ $mahasiswa->email }}</div>

  <a href="{{ route('mahasiswa.index') }}" class="px-4 py-2 rounded border hover:bg-gray-50">Kembali</a>
</div>
@endsection
```

**Controller `show()`**
```php
public function show(\App\Models\Mahasiswa $mahasiswa)
{
    return view('mahasiswa.show', compact('mahasiswa'));
}
```

**e. Destroy (hapus)**
```php
public function destroy(\App\Models\Mahasiswa $mahasiswa)
{
    $mahasiswa->delete();
    return redirect()->route('mahasiswa.index')->with('success','Data mahasiswa berhasil dihapus.');
}
```

> **Instruksi klik untuk Mahasiswa:**  
> - Buka `http://127.0.0.1:8000/mahasiswa` → klik **+ Tambah Mahasiswa** → isi form → **Simpan**.  
> - Di tabel, klik **Lihat**/**Edit**/**Hapus** untuk menguji fungsi.  

#### 6) View & Controller — Dosen (ringkas, pola sama)

**`resources/views/dosen/index.blade.php`**
```php
@extends('layouts.app')

@section('content')
<div class="flex items-center justify-between mb-4">
  <h2 class="text-xl font-semibold">Daftar Dosen</h2>
  <a href="{{ route('dosen.create') }}" class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700">+ Tambah Dosen</a>
</div>

<x-alert-success />

<table class="w-full border-collapse bg-white rounded shadow">
  <thead>
    <tr class="bg-gray-100 text-left">
      <th class="px-4 py-2 border">#</th>
      <th class="px-4 py-2 border">Nama</th>
      <th class="px-4 py-2 border">NIDN</th>
      <th class="px-4 py-2 border">Email</th>
      <th class="px-4 py-2 border">Aksi</th>
    </tr>
  </thead>
  <tbody>
    @forelse ($dosens as $index => $dosen)
      <tr>
        <td class="px-4 py-2 border">{{ $dosens->firstItem() + $index }}</td>
        <td class="px-4 py-2 border">{{ $dosen->nama }}</td>
        <td class="px-4 py-2 border">{{ $dosen->nidn }}</td>
        <td class="px-4 py-2 border">{{ $dosen->email }}</td>
        <td class="px-4 py-2 border">
          <a href="{{ route('dosen.show', $dosen) }}" class="text-blue-700 hover:underline">Lihat</a>
          <a href="{{ route('dosen.edit', $dosen) }}" class="ml-3 text-yellow-600 hover:underline">Edit</a>
          <form action="{{ route('dosen.destroy', $dosen) }}" method="POST" class="inline-block ml-3" onsubmit="return confirm('Yakin hapus data?')">
            @csrf @method('DELETE')
            <button type="submit" class="text-red-600 hover:underline">Hapus</button>
          </form>
        </td>
      </tr>
    @empty
      <tr><td colspan="5" class="px-4 py-6 border text-center text-gray-500">Belum ada data</td></tr>
    @endforelse
  </tbody>
</table>

<div class="mt-4">
  {{ $dosens->links() }}
</div>
@endsection
```

**`app/Http/Controllers/DosenController.php` (inti method)**
```php
public function index()
{
    $dosens = \App\Models\Dosen::orderBy('nama')->paginate(10);
    return view('dosen.index', compact('dosens'));
}

public function create()
{
    return view('dosen.create');
}

public function store(\Illuminate\Http\Request $request)
{
    $validated = $request->validate([
        'nama' => ['required','string','max:100'],
        'nidn' => ['required','string','max:20','unique:dosens,nidn'],
        'email'=> ['required','email','max:100','unique:dosens,email'],
    ]);
    \App\Models\Dosen::create($validated);
    return redirect()->route('dosen.index')->with('success','Data dosen berhasil ditambahkan.');
}

public function edit(\App\Models\Dosen $dosen)
{
    return view('dosen.edit', compact('dosen'));
}

public function update(\Illuminate\Http\Request $request, \App\Models\Dosen $dosen)
{
    $validated = $request->validate([
        'nama' => ['required','string','max:100'],
        'nidn' => ['required','string','max:20','unique:dosens,nidn,'.$dosen->id],
        'email'=> ['required','email','max:100','unique:dosens,email,'.$dosen->id],
    ]);
    $dosen->update($validated);
    return redirect()->route('dosen.index')->with('success','Data dosen berhasil diperbarui.');
}

public function show(\App\Models\Dosen $dosen)
{
    return view('dosen.show', compact('dosen'));
}

public function destroy(\App\Models\Dosen $dosen)
{
    $dosen->delete();
    return redirect()->route('dosen.index')->with('success','Data dosen berhasil dihapus.');
}
```

> **Instruksi klik untuk Dosen:**  
> - Akses `http://127.0.0.1:8000/dosen` → **+ Tambah Dosen** → isi form → **Simpan**.  
> - Coba fitur **Edit**, **Lihat**, **Hapus**.

---

### Studi Kasus (Pertemuan 5)
1. **Masukkan 3 data Mahasiswa** dan **2 data Dosen** melalui form.  
2. **Uji pencatatan**: Data tampil di tabel, dapat diubah dan dihapus.  
3. **Validasi**: Coba masukkan NIM/NIDN/Email duplikat → sistem menolak dengan pesan error.

---

### Tugas Mahasiswa
- Tambahkan **pencarian sederhana** pada halaman index Mahasiswa (by nama atau NIM).  
- Tambahkan **pagination** (sudah contoh pada kode, pastikan berfungsi).  
- Rapikan UI (padding, spacing, warna) agar tampilan lebih elegan.

---

### Ringkasan
- Resource Controller mempercepat CRUD.  
- Validasi, fillable, dan flash message penting untuk integritas data.  
- Tailwind mempercantik tampilan tanpa CSS kompleks.

---

## 📍 Pertemuan 6 – Form Laporan & Nomor Tiket

### Pendahuluan
Kita membangun **Form Pelaporan** yang menghasilkan **Nomor Laporan/Tiket** otomatis dan menyimpan **status awal** laporan. Karena **Auth** baru dibahas di Pertemuan 9, sementara **mahasiswa** dipilih dari **dropdown** (simulasi pengguna login).

### Tujuan
- Mahasiswa mampu membuat form pelaporan yang tervalidasi.  
- Sistem menghasilkan **nomor tiket unik** otomatis.  
- Laporan tersimpan dengan **status awal** `baru`.

### Materi Pokok
- Model & Controller `Laporan`  
- Generate nomor tiket unik  
- Form pelaporan (judul, deskripsi, mahasiswa)  
- Index laporan (menampilkan nomor, judul, status)

---

### Langkah Praktikum (Step-by-Step)

#### 1) Pastikan Model Laporan
**`app/Models/Laporan.php`**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Laporan extends Model
{
    use HasFactory;

    protected $fillable = [
        'judul', 'deskripsi', 'nomor_laporan', 'status', 'mahasiswa_id'
    ];

    public function mahasiswa()
    {
        return $this->belongsTo(\App\Models\Mahasiswa::class);
    }
}
```

#### 2) Buat Resource Controller Laporan
```bash
php artisan make:controller LaporanController --resource --model=Laporan
```

#### 3) Tambahkan Route Resource
**`routes/web.php`**
```php
use App\Http\Controllers\LaporanController;

Route::resource('laporan', LaporanController::class)->parameters([
    'laporan' => 'laporan'
]);
```

> **Catatan:** Kita sudah punya route `/laporan` dari pertemuan sebelumnya. Sekarang rutenya lengkap (index, create, store, dst).

#### 4) Implementasi Controller — Nomor Tiket Otomatis
**`app/Http/Controllers/LaporanController.php`**
```php
<?php

namespace App\Http\Controllers;

use App\Models\Laporan;
use App\Models\Mahasiswa;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

class LaporanController extends Controller
{
    public function index()
    {
        $laporans = Laporan::with('mahasiswa')->latest()->paginate(10);
        return view('laporan.index', compact('laporans'));
    }

    public function create()
    {
        // Sementara pilih mahasiswa dari dropdown (sebelum Auth)
        $mahasiswas = Mahasiswa::orderBy('nama')->get();
        return view('laporan.create', compact('mahasiswas'));
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'judul' => ['required','string','max:150'],
            'deskripsi' => ['required','string','max:2000'],
            'mahasiswa_id' => ['required','exists:mahasiswas,id'],
        ]);

        $nomorLaporan = $this->generateNomorLaporan();

        $laporan = Laporan::create([
            'judul' => $validated['judul'],
            'deskripsi' => $validated['deskripsi'],
            'nomor_laporan' => $nomorLaporan,
            'status' => 'baru',
            'mahasiswa_id' => $validated['mahasiswa_id'],
        ]);

        return redirect()->route('laporan.show', $laporan)
            ->with('success','Laporan berhasil dibuat dengan nomor tiket: '.$nomorLaporan);
    }

    private function generateNomorLaporan(): string
    {
        // Format: LAP-YYYYMMDD-HHMMSS-AB12
        return 'LAP-'.now()->format('Ymd-His').'-'.Str::upper(Str::random(4));
    }

    public function show(Laporan $laporan)
    {
        $laporan->load('mahasiswa');
        return view('laporan.show', compact('laporan'));
    }

    public function edit(Laporan $laporan)
    {
        $mahasiswas = Mahasiswa::orderBy('nama')->get();
        return view('laporan.edit', compact('laporan','mahasiswas'));
    }

    public function update(Request $request, Laporan $laporan)
    {
        $validated = $request->validate([
            'judul' => ['required','string','max:150'],
            'deskripsi' => ['required','string','max:2000'],
            'mahasiswa_id' => ['required','exists:mahasiswas,id'],
            'status' => ['required','in:baru,diproses,selesai'],
        ]);

        $laporan->update($validated);

        return redirect()->route('laporan.show', $laporan)->with('success','Laporan berhasil diperbarui.');
    }

    public function destroy(Laporan $laporan)
    {
        $laporan->delete();
        return redirect()->route('laporan.index')->with('success','Laporan berhasil dihapus.');
    }
}
```

#### 5) View — Index Laporan
**`resources/views/laporan/index.blade.php`**
```php
@extends('layouts.app')

@section('content')
<div class="flex items-center justify-between mb-4">
  <h2 class="text-xl font-semibold">Daftar Laporan</h2>
  <a href="{{ route('laporan.create') }}" class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700">+ Buat Laporan</a>
</div>

<x-alert-success />

<table class="w-full border-collapse bg-white rounded shadow">
  <thead>
    <tr class="bg-gray-100 text-left">
      <th class="px-4 py-2 border">#</th>
      <th class="px-4 py-2 border">Nomor</th>
      <th class="px-4 py-2 border">Judul</th>
      <th class="px-4 py-2 border">Pelapor</th>
      <th class="px-4 py-2 border">Status</th>
      <th class="px-4 py-2 border">Aksi</th>
    </tr>
  </thead>
  <tbody>
    @forelse ($laporans as $index => $laporan)
      <tr>
        <td class="px-4 py-2 border">{{ $laporans->firstItem() + $index }}</td>
        <td class="px-4 py-2 border font-mono">{{ $laporan->nomor_laporan }}</td>
        <td class="px-4 py-2 border">{{ $laporan->judul }}</td>
        <td class="px-4 py-2 border">{{ optional($laporan->mahasiswa)->nama }}</td>
        <td class="px-4 py-2 border">
          @php
            $color = $laporan->status === 'selesai' ? 'green' : ($laporan->status === 'diproses' ? 'yellow' : 'red');
            $label = ucfirst($laporan->status);
          @endphp
          <span class="px-2 py-1 rounded bg-{{ $color }}-100 text-{{ $color }}-800 border border-{{ $color }}-200 text-sm">{{ $label }}</span>
        </td>
        <td class="px-4 py-2 border">
          <a href="{{ route('laporan.show', $laporan) }}" class="text-blue-700 hover:underline">Lihat</a>
          <a href="{{ route('laporan.edit', $laporan) }}" class="ml-3 text-yellow-600 hover:underline">Edit</a>
          <form action="{{ route('laporan.destroy', $laporan) }}" method="POST" class="inline-block ml-3" onsubmit="return confirm('Yakin hapus laporan?')">
            @csrf @method('DELETE')
            <button type="submit" class="text-red-600 hover:underline">Hapus</button>
          </form>
        </td>
      </tr>
    @empty
      <tr><td colspan="6" class="px-4 py-6 border text-center text-gray-500">Belum ada laporan</td></tr>
    @endforelse
  </tbody>
</table>

<div class="mt-4">
  {{ $laporans->links() }}
</div>
@endsection
```

#### 6) View — Create Laporan (Form Pelaporan)
**`resources/views/laporan/create.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-4">Buat Laporan Baru</h2>

@if ($errors->any())
  <div class="mb-4 rounded border border-red-200 bg-red-50 p-3 text-red-700">
    <ul class="list-disc pl-5">
      @foreach ($errors->all() as $error)
        <li>{{ $error }}</li>
      @endforeach
    </ul>
  </div>
@endif

<form action="{{ route('laporan.store') }}" method="POST" class="bg-white p-4 rounded shadow max-w-2xl">
  @csrf

  <div class="mb-3">
    <label class="block mb-1">Mahasiswa (Pelapor)</label>
    <select name="mahasiswa_id" class="w-full border rounded px-3 py-2" required>
      <option value="" disabled selected>-- Pilih Mahasiswa --</option>
      @foreach ($mahasiswas as $mhs)
        <option value="{{ $mhs->id }}" {{ old('mahasiswa_id') == $mhs->id ? 'selected' : '' }}>
          {{ $mhs->nama }} ({{ $mhs->nim }})
        </option>
      @endforeach
    </select>
  </div>

  <div class="mb-3">
    <label class="block mb-1">Judul Laporan</label>
    <input type="text" name="judul" value="{{ old('judul') }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="mb-4">
    <label class="block mb-1">Deskripsi</label>
    <textarea name="deskripsi" rows="5" class="w-full border rounded px-3 py-2" required>{{ old('deskripsi') }}</textarea>
  </div>

  <div class="flex items-center gap-2">
    <button type="submit" class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700">Kirim Laporan</button>
    <a href="{{ route('laporan.index') }}" class="px-4 py-2 rounded border hover:bg-gray-50">Batal</a>
  </div>
</form>
@endsection
```

#### 7) View — Show Laporan (Detail + Nomor Tiket)
**`resources/views/laporan/show.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-4">Detail Laporan</h2>

<x-alert-success />

<div class="bg-white rounded shadow p-4">
  <div class="mb-2"><span class="font-semibold">Nomor Tiket:</span> <span class="font-mono">{{ $laporan->nomor_laporan }}</span></div>
  <div class="mb-2"><span class="font-semibold">Judul:</span> {{ $laporan->judul }}</div>
  <div class="mb-2"><span class="font-semibold">Pelapor:</span> {{ optional($laporan->mahasiswa)->nama }} ({{ optional($laporan->mahasiswa)->nim }})</div>
  <div class="mb-2"><span class="font-semibold">Status:</span> <span class="px-2 py-1 rounded bg-gray-100 border">{{ ucfirst($laporan->status) }}</span></div>
  <div class="mb-4"><span class="font-semibold">Deskripsi:</span><br>{{ $laporan->deskripsi }}</div>

  <div class="flex items-center gap-2">
    <a href="{{ route('laporan.edit', $laporan) }}" class="px-4 py-2 rounded bg-yellow-500 text-white hover:bg-yellow-600">Edit</a>
    <a href="{{ route('laporan.index') }}" class="px-4 py-2 rounded border hover:bg-gray-50">Kembali</a>
  </div>
</div>
@endsection
```

#### 8) View — Edit Laporan (Termasuk Ubah Status)
**`resources/views/laporan/edit.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-4">Edit Laporan</h2>

@if ($errors->any())
  <div class="mb-4 rounded border border-red-200 bg-red-50 p-3 text-red-700">
    <ul class="list-disc pl-5">
      @foreach ($errors->all() as $error)
        <li>{{ $error }}</li>
      @endforeach
    </ul>
  </div>
@endif

<form action="{{ route('laporan.update', $laporan) }}" method="POST" class="bg-white p-4 rounded shadow max-w-2xl">
  @csrf @method('PUT')

  <div class="mb-3">
    <label class="block mb-1">Mahasiswa (Pelapor)</label>
    <select name="mahasiswa_id" class="w-full border rounded px-3 py-2" required>
      @foreach ($mahasiswas as $mhs)
        <option value="{{ $mhs->id }}" {{ old('mahasiswa_id', $laporan->mahasiswa_id) == $mhs->id ? 'selected' : '' }}>
          {{ $mhs->nama }} ({{ $mhs->nim }})
        </option>
      @endforeach
    </select>
  </div>

  <div class="mb-3">
    <label class="block mb-1">Judul Laporan</label>
    <input type="text" name="judul" value="{{ old('judul', $laporan->judul) }}" class="w-full border rounded px-3 py-2" required>
  </div>

  <div class="mb-3">
    <label class="block mb-1">Deskripsi</label>
    <textarea name="deskripsi" rows="5" class="w-full border rounded px-3 py-2" required>{{ old('deskripsi', $laporan->deskripsi) }}</textarea>
  </div>

  <div class="mb-4">
    <label class="block mb-1">Status</label>
    <select name="status" class="w-full border rounded px-3 py-2" required>
      @foreach (['baru','diproses','selesai'] as $st)
        <option value="{{ $st }}" {{ old('status', $laporan->status) == $st ? 'selected' : '' }}>{{ ucfirst($st) }}</option>
      @endforeach
    </select>
  </div>

  <div class="flex items-center gap-2">
    <button type="submit" class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700">Update</button>
    <a href="{{ route('laporan.show', $laporan) }}" class="px-4 py-2 rounded border hover:bg-gray-50">Batal</a>
  </div>
</form>
@endsection
```

---

### Studi Kasus (Pertemuan 6)
1. **Buat 2 data Mahasiswa** (jika belum ada).  
2. Buka **Laporan → Buat Laporan Baru**: pilih salah satu mahasiswa, isi judul & deskripsi.  
3. Setelah menekan **Kirim Laporan**, sistem menampilkan **Nomor Tiket** (format `LAP-YYYYMMDD-HHMMSS-XXXX`).  
4. Buka **Daftar Laporan** → Nomor, Judul, Pelapor, dan Status **Baru** terlihat.  
5. Klik **Edit** laporan → ubah **Status** ke **Diproses** → **Update** → cek badge status berubah.  

> **Instruksi klik ringkas:**  
> `Menu Laporan → + Buat Laporan → Isi Form → Kirim → Lihat Detail (nomor tiket) → Kembali ke Daftar → Edit status`

---

### Tugas Mahasiswa
- Validasi minimal **judul ≥ 10 karakter**, **deskripsi ≥ 20 karakter**.  
- Tambahkan tombol **Salin Nomor Tiket** di halaman detail (gunakan `navigator.clipboard.writeText` via `<button>` + JS sederhana).  
- Tambahkan **filter status** di halaman index laporan (dropdown: semua/baru/diproses/selesai).

Contoh filter sederhana untuk index (controller):
```php
public function index(Request $request)
{
    $query = \App\Models\Laporan::with('mahasiswa')->latest();
    if ($request->filled('status') && in_array($request->status, ['baru','diproses','selesai'])) {
        $query->where('status', $request->status);
    }
    $laporans = $query->paginate(10)->withQueryString();
    return view('laporan.index', compact('laporans'));
}
```
Tambahkan di view (di atas tabel):
```php
<form method="GET" class="mb-3">
  <select name="status" class="border rounded px-2 py-1" onchange="this.form.submit()">
    <option value="">Semua Status</option>
    @foreach (['baru','diproses','selesai'] as $s)
      <option value="{{ $s }}" {{ request('status')===$s ? 'selected' : '' }}>{{ ucfirst($s) }}</option>
    @endforeach
  </select>
</form>
```

---

### Ringkasan
- Form pelaporan membuat data **Laporan** dengan **nomor tiket unik** dan status awal `baru`.  
- Sebelum ada Auth, pelapor dipilih manual dari dropdown (nanti akan diganti `auth()->id()` pada Pertemuan 9).  
- Halaman index, create, show, edit melengkapi alur pelaporan dasar.

---

## ✅ Checklist Hasil Belajar Bab 3
- [ ] CRUD Mahasiswa lengkap (index, create, store, show, edit, update, destroy).  
- [ ] CRUD Dosen lengkap.  
- [ ] Form laporan menghasilkan nomor tiket unik otomatis.  
- [ ] Status laporan dapat diubah (baru → diproses → selesai).  
- [ ] UI rapi dengan Tailwind (tombol, tabel, badge).

---

### Catatan Lanjutan (Mengarah ke Bab 4–5)
- Setelah **Auth** (Pertemuan 9), **mahasiswa_id** akan otomatis `auth()->id()` dan dropdown pelapor dihilangkan.  
- Setelah **Role & Permission** (Pertemuan 10), hanya **DPA (admin)** yang boleh mengubah status.

---