# 🟦 Bab 6 – Dashboard & Notifikasi  
**Cakupan:** Pertemuan 11–12  

## 📍 Pertemuan 11 – Dashboard User & Admin

### Pendahuluan
Dashboard menampilkan informasi ringkas sesuai role:  
- **Mahasiswa:** hanya laporan yang dia buat.  
- **DPA (Admin):** semua laporan mahasiswa.  

### Tujuan
- Mahasiswa dapat membuat dashboard berbeda sesuai role.  
- Mahasiswa dapat menampilkan data dengan statistik sederhana.  

### Materi Pokok
- Blade conditional (`@if`, `@else`) untuk role.  
- Query laporan berdasarkan role.  
- Navigasi menu berbeda.  

### Langkah Praktikum (Step-by-Step)
1. **Tambahkan Route Dashboard**
```php
Route::get('/dashboard', function () {
    $user = auth()->user();
    if ($user->role === 'dpa') {
        $laporans = \App\Models\Laporan::latest()->paginate(10);
    } else {
        $laporans = \App\Models\Laporan::where('mahasiswa_id', $user->id)->latest()->paginate(10);
    }
    return view('dashboard', compact('laporans','user'));
})->middleware(['auth'])->name('dashboard');
```

2. **Buat View Dashboard**
**`resources/views/dashboard.blade.php`**
```php
@extends('layouts.app')

@section('content')
<h2 class="text-xl font-semibold mb-4">Dashboard {{ ucfirst($user->role) }}</h2>

@if($user->role === 'mahasiswa')
  <p class="mb-3">Selamat datang, {{ $user->name }}. Berikut laporan yang kamu buat:</p>
@else
  <p class="mb-3">Selamat datang Admin DPA, berikut semua laporan mahasiswa:</p>
@endif

<table class="w-full border-collapse bg-white rounded shadow">
  <thead>
    <tr class="bg-gray-100 text-left">
      <th class="px-4 py-2 border">Nomor</th>
      <th class="px-4 py-2 border">Judul</th>
      <th class="px-4 py-2 border">Pelapor</th>
      <th class="px-4 py-2 border">Status</th>
    </tr>
  </thead>
  <tbody>
    @foreach ($laporans as $laporan)
      <tr>
        <td class="px-4 py-2 border font-mono">{{ $laporan->nomor_laporan }}</td>
        <td class="px-4 py-2 border">{{ $laporan->judul }}</td>
        <td class="px-4 py-2 border">{{ optional($laporan->mahasiswa)->nama }}</td>
        <td class="px-4 py-2 border">{{ ucfirst($laporan->status) }}</td>
      </tr>
    @endforeach
  </tbody>
</table>
@endsection
```

---

### Studi Kasus
1. Login sebagai mahasiswa → dashboard menampilkan laporan miliknya.  
2. Login sebagai DPA → dashboard menampilkan semua laporan.  

### Tugas Mahasiswa
- Tambahkan **jumlah laporan per status** (baru/diproses/selesai) di dashboard.  
- Tambahkan grafik sederhana dengan library chart JS (opsional).  

### Ringkasan
- Dashboard dapat berbeda sesuai role.  
- Query disesuaikan: mahasiswa → data miliknya, DPA → semua data.  

---

## 📍 Pertemuan 12 – Notifikasi & Tracking

### Pendahuluan
Notifikasi memberikan feedback ke pengguna ketika aksi berhasil, misalnya saat membuat laporan atau memperbarui status.

### Tujuan
- Mahasiswa dapat menampilkan notifikasi dengan flash session.  
- Mahasiswa menambahkan informasi tracking perubahan status.  

### Materi Pokok
- Flash message session.  
- Notifikasi perubahan status.  
- Tracking log sederhana.  

### Langkah Praktikum (Step-by-Step)
1. **Tambahkan Komponen Flash di Layout**
**`resources/views/components/alert-success.blade.php`**
```php
@if(session('success'))
  <div class="mb-4 p-3 rounded bg-green-100 text-green-800 border border-green-300">
    {{ session('success') }}
  </div>
@endif
```

2. **Gunakan Flash Message di Controller**
```php
return redirect()->back()->with('success','Status laporan berhasil diperbarui');
```

3. **Tambah Tracking Log (Opsional)**
Buat migration `status_histories`:
```bash
php artisan make:migration create_status_histories_table
```
Isi:
```php
Schema::create('status_histories', function (Blueprint $table) {
    $table->id();
    $table->foreignId('laporan_id')->constrained();
    $table->enum('status',['baru','diproses','selesai']);
    $table->timestamps();
});
```

Tambahkan ke `update()` di LaporanController:
```php
\App\Models\StatusHistory::create([
  'laporan_id' => $laporan->id,
  'status' => $validated['status'],
]);
```

---

### Studi Kasus
1. Buat laporan baru → muncul flash message sukses.  
2. Ubah status laporan → flash sukses + tracking status tercatat.  

### Tugas Mahasiswa
- Tambahkan halaman “Riwayat Status” laporan.  
- Notifikasi ditampilkan dengan warna berbeda untuk error (`alert-error`).  

### Ringkasan
- Flash message memberi umpan balik cepat.  
- Tracking status bisa disimpan di tabel riwayat untuk audit.  

---