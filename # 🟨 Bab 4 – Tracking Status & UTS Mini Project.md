
# 🟨 Bab 4 – Tracking Status & UTS Mini Project  
**Cakupan:** Pertemuan 7–8  
**Proyek:** Sistem Pelaporan Masalah Akademik (Laravel + Tailwind)

---

## 📍 Pertemuan 7 – Tracking Status Laporan

### Pendahuluan
Setelah mahasiswa membuat laporan, dosen pendamping akademik (DPA) atau admin perlu **memantau dan memperbarui status laporan**. Status dapat berupa:  
- **Baru** → laporan baru dibuat oleh mahasiswa  
- **Diproses** → laporan sedang ditindaklanjuti  
- **Selesai** → laporan sudah ditutup  

Pertemuan ini berfokus pada **tracking status laporan** melalui halaman index dan detail laporan.

---

### Tujuan
- Mahasiswa memahami implementasi **enum status** laporan.  
- Mahasiswa dapat menambahkan fitur update status.  
- Mahasiswa menampilkan badge status dengan desain Tailwind.  

---

### Materi Pokok
- Enum `status` (`baru`, `diproses`, `selesai`)  
- Badge status dengan Tailwind CSS  
- Update status via tombol cepat di halaman index  

---

### Langkah Praktikum (Step-by-Step)

#### 1) Pastikan Migration Laporan Memiliki Kolom Status
Sudah dibuat di Bab 1 Pertemuan 2:
```php
$table->enum('status',['baru','diproses','selesai'])->default('baru');
```

#### 2) Tambahkan Tombol Aksi Update Status Cepat
Edit **`resources/views/laporan/index.blade.php`** → tambahkan form **Ubah Status** di kolom aksi.

```php
<td class="px-4 py-2 border">
  <a href="{{ route('laporan.show', $laporan) }}" class="text-blue-700 hover:underline">Lihat</a>
  <a href="{{ route('laporan.edit', $laporan) }}" class="ml-3 text-yellow-600 hover:underline">Edit</a>
  
  <!-- Tombol Ubah Status Cepat -->
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

  <!-- Hapus -->
  <form action="{{ route('laporan.destroy', $laporan) }}" method="POST" class="inline-block ml-3" onsubmit="return confirm('Yakin hapus laporan?')">
    @csrf @method('DELETE')
    <button type="submit" class="text-red-600 hover:underline">Hapus</button>
  </form>
</td>
```

#### 3) Tampilkan Badge Status dengan Warna Tailwind
Edit bagian status di tabel index:

```php
@php
  $color = $laporan->status === 'selesai' ? 'green' : ($laporan->status === 'diproses' ? 'yellow' : 'red');
@endphp
<span class="px-2 py-1 rounded bg-{{ $color }}-100 text-{{ $color }}-800 border border-{{ $color }}-200 text-sm">
  {{ ucfirst($laporan->status) }}
</span>
```

#### 4) Tambahkan Logika di Controller (Sudah Ada)
Controller `update()` di **`LaporanController.php`** sudah mendukung `status` → tidak perlu ubahan signifikan.

---

### Studi Kasus
1. Buat laporan baru → otomatis status **Baru**.  
2. Buka halaman index laporan.  
3. Klik tombol **Proses** → status berubah menjadi **Diproses**.  
4. Klik tombol **Selesaikan** → status berubah menjadi **Selesai**.  
5. Lihat badge warna berubah: Merah (Baru), Kuning (Diproses), Hijau (Selesai).  

> **Instruksi klik:**  
> Menu → Laporan → Daftar Laporan → klik **Proses** → status berubah → klik **Selesaikan** → status hijau.

---

### Tugas Mahasiswa
- Tambahkan kolom **Tanggal Update Status Terakhir** di tabel laporan (migration baru + update otomatis saat status berubah).  
- Tampilkan tanggal update status pada halaman index laporan.  
- Tambahkan filter status (sudah dicontohkan di Bab 3).  

---

### Ringkasan
- Status laporan dapat dikelola dengan enum.  
- Tombol aksi memudahkan DPA/admin mengubah status cepat.  
- Badge Tailwind memberikan visualisasi status yang jelas.

---

## 📍 Pertemuan 8 – UTS (Mini Project Tahap 1)

### Pendahuluan
UTS dilakukan untuk menguji pemahaman mahasiswa terhadap materi yang sudah dipelajari (Pertemuan 1–7). Fokus pada CRUD dasar, form laporan, nomor tiket otomatis, dan tracking status.

### Tujuan
- Mahasiswa mengimplementasikan mini project secara end-to-end.  
- Mahasiswa menunjukkan hasil sistem sederhana yang sudah berjalan.  

---

### Instruksi Ujian
Mahasiswa diminta membuat sistem mini **Sistem Pelaporan Masalah Akademik** dengan spesifikasi:

1. **Database**  
   - Tabel `mahasiswas`, `dosens`, `laporans`.  
   - Relasi: Mahasiswa → Laporan.  

2. **CRUD**  
   - CRUD Mahasiswa.  
   - CRUD Dosen.  
   - CRUD Laporan.  

3. **Form Laporan**  
   - Judul, deskripsi, mahasiswa.  
   - Nomor tiket otomatis (format `LAP-YYYYMMDD-HHMMSS-XXXX`).  
   - Status default `baru`.  

4. **Tracking Status**  
   - Tombol ubah status `baru → diproses → selesai`.  
   - Badge warna: merah, kuning, hijau.  

5. **UI**  
   - Layout modern dengan Tailwind.  
   - Navigasi: Home, Mahasiswa, Dosen, Laporan.  

---

### Penilaian
- CRUD Mahasiswa & Dosen (20%)  
- Form Laporan + Nomor Tiket (30%)  
- Tracking Status Laporan (30%)  
- Tampilan UI dengan Tailwind (10%)  
- Kerapihan kode & presentasi (10%)  

Total: **100%**

---

### Studi Kasus UTS
- Mahasiswa presentasi project di depan kelas.  
- Menunjukkan flow: **Mahasiswa input laporan → Nomor tiket → Ubah status → Status berubah dengan badge warna**.  

---

### Ringkasan
UTS adalah checkpoint untuk memastikan mahasiswa benar-benar menguasai: **instalasi, database, migration, controller, Blade, Tailwind, CRUD, form laporan, nomor tiket, tracking status**.

---

## ✅ Checklist Hasil Belajar Bab 4
- [ ] Laporan dapat diubah status (baru, diproses, selesai).  
- [ ] Tombol aksi cepat di index berfungsi.  
- [ ] Badge status dengan warna Tailwind berfungsi.  
- [ ] Mahasiswa dapat membuat mini project UTS tahap 1 dengan fitur CRUD + pelaporan dasar.

---

### Catatan Lanjutan (Menuju Bab 5)
- Pertemuan selanjutnya (Bab 5) akan membahas **Authentication (Auth)** agar mahasiswa tidak perlu memilih pelapor dari dropdown lagi.  
- Auth juga memungkinkan perbedaan akses (Mahasiswa vs DPA).  

---
