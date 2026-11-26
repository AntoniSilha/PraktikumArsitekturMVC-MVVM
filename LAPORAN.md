# LAPORAN PRAKTIKUM ARSITEKTUR APLIKASI FLUTTER

## Identitas Mahasiswa

| Keterangan | Identitas                                        |
|---|--------------------------------------------------|
| Nama | Antoni Silha Indriyan Priyadi                    |
| NIM | 362558302006                                     |
| Kelas | TRPL - 2E                                        |
| Mata Kuliah | Praktikum Pemrograman Perangkat Bergerak         |
| Modul | Praktikum-Mandiri-MVC-MVVM-dan-Pengujian-Flutter.pdf |
| Tanggal | 26 November 2026                                 |

## 1. Tujuan Praktikum

Praktikum ini bertujuan menerapkan dan membandingkan pola arsitektur Model-View-Controller (MVC) dan Model-View-ViewModel (MVVM) pada aplikasi daftar tugas Flutter. Pada tugas mandiri, aplikasi dilengkapi fitur **Filter Belum selesai** agar pengguna dapat menampilkan tugas yang belum selesai tanpa menghapus atau mengubah data tugas yang tersimpan di repository.

## 2. Deskripsi Fitur

Aplikasi menyediakan kolom untuk menambahkan tugas, ringkasan jumlah tugas yang selesai, daftar tugas dengan checkbox, dan sebuah `FilterChip` berlabel **Belum selesai**.

Perilaku filter yang diterapkan adalah sebagai berikut:

1. Saat halaman pertama kali dibuka, filter bernilai `false` sehingga semua tugas ditampilkan.
2. Saat filter diaktifkan, hanya tugas dengan `selesai == false` yang ditampilkan.
3. Jika sebuah tugas dicentang ketika filter aktif, tugas tersebut tidak lagi terlihat karena statusnya berubah menjadi selesai.
4. Saat filter dimatikan kembali, seluruh tugas ditampilkan dan checkbox menunjukkan status terakhir masing-masing tugas.
5. Ringkasan selalu menghitung seluruh tugas, bukan hanya tugas yang sedang terlihat. Contohnya, jika satu dari dua tugas selesai, ringkasan tetap menampilkan **1 dari 2 selesai** meskipun hanya satu tugas terlihat.
6. Jika belum ada tugas yang ditambahkan, aplikasi menampilkan **Belum ada tugas.** Jika filter aktif dan semua tugas sudah selesai, aplikasi menampilkan **Tidak ada tugas yang belum selesai.**

Penyaringan hanya menghasilkan daftar tampilan. Data lengkap tetap disimpan oleh `RepositoriTugas`.

## 3. Struktur Implementasi

| File/Class | Fungsi |
|---|---|
| `lib/model/tugas.dart` – `Tugas` | Menyimpan data setiap tugas, termasuk ID, judul, dan status selesai. |
| `lib/data/repositori_tugas.dart` – `RepositoriTugas` | Menyimpan daftar lengkap tugas di memori, menambahkan tugas, dan mengubah status tugas. |
| `lib/ui/panel_tugas.dart` – `PanelTugas` | Menampilkan form, ringkasan, `FilterChip`, pesan kondisi kosong, dan daftar checkbox. Panel menerima status filter dan daftar tugas dari lapisan pengelola state. |
| `lib/mvc/pengendali_tugas.dart` – `PengendaliTugas` | Mengelola validasi, status filter, daftar tugas yang terlihat, dan perubahan status pada versi MVC. |
| `lib/mvc/layar_mvc.dart` – `LayarMvc` | Menghubungkan panel dengan `PengendaliTugas` dan memperbarui UI menggunakan `setState()`. |
| `lib/mvvm/tugas_view_model.dart` – `TugasViewModel` | Mengelola state, validasi, status filter, daftar tugas yang terlihat, dan memberi notifikasi perubahan melalui `notifyListeners()`. |
| `lib/mvvm/layar_mvvm.dart` – `LayarMvvm` | Menghubungkan panel dengan `TugasViewModel` menggunakan `ListenableBuilder`. |

Pada kedua versi, status filter diawali dengan `false`. Getter `daftarTerlihat` menyaring tugas berdasarkan status selesai, sedangkan getter `daftar` tetap mengakses seluruh data dari repository. Nilai `jumlahSelesai` dan `totalTugas` juga dihitung dari daftar lengkap agar ringkasan tidak berubah hanya karena filter diaktifkan.

## 4. Cara Menjalankan Aplikasi

Jalankan perintah berikut dari direktori project yang berisi file `pubspec.yaml`.

### 4.1 Mengambil dependency

```bash
flutter pub get
```

### 4.2 Menjalankan aplikasi di Chrome

```bash
flutter run -d chrome
```

Pada halaman awal aplikasi, pilih **Buka MVC** untuk mencoba implementasi MVC atau **Buka MVVM** untuk mencoba implementasi MVVM. Tambahkan beberapa tugas, centang salah satu tugas, lalu tekan chip **Belum selesai** untuk menguji penyaringan.

### 4.3 Menjalankan test khusus filter

```bash
flutter test test/filter_test.dart
```

### 4.4 Menjalankan seluruh test

```bash
flutter test
```

### 4.5 Memeriksa analisis kode

```bash
flutter analyze
```

Project menggunakan dependency Flutter standar dan `flutter_test`. README starter mencantumkan Flutter 3.29.2, sedangkan screenshot persiapan pada perangkat praktikum memperlihatkan Flutter 3.47.1 (`bukti/MenjalankanPersiapanPraktikum.png`). Project memiliki batas Dart SDK `^3.7.2` pada `pubspec.yaml`; gunakan Flutter SDK yang memenuhi batas tersebut.

## 5. Tabel Perbandingan MVC dan MVVM

### 5.1 Class yang menerima aksi dan menyimpan pilihan filter

| Aspek | MVC | MVVM |
|---|---|---|
| Class yang mengelola pilihan filter | `PengendaliTugas` | `TugasViewModel` |
| Metode untuk menyimpan pilihan | `aturFilterBelumSelesai(bool aktif)` | `aturFilterBelumSelesai(bool aktif)` |
| Cara menerima aksi dari chip | Callback `onFilterChanged` di `LayarMvc` memanggil metode Controller. | Callback `onFilterChanged` di `LayarMvvm` terhubung langsung ke metode ViewModel. |
| Cara memperbarui tampilan | `LayarMvc` memanggil `setState()` setelah Controller mengubah status filter. | ViewModel memanggil `notifyListeners()` dan `ListenableBuilder` membangun ulang bagian tampilan yang mendengarkan ViewModel. |

Pada kedua pola, `PanelTugas` menampilkan `FilterChip` dan meneruskan aksi melalui callback. Penyimpanan pilihan filter dilakukan oleh Controller pada MVC atau ViewModel pada MVVM, bukan oleh repository.

### 5.2 Pemicu rebuild dan hasil percobaan

| Aspek | MVC | MVVM |
|---|---|---|
| Pemicu rebuild | `setState()` pada `State<LayarMvc>`, khususnya pada callback filter dan metode `_ubahStatus()`. | `notifyListeners()` pada `TugasViewModel`, yang didengarkan oleh `ListenableBuilder`. |
| Jika pemicu dihapus | Perubahan state di `PengendaliTugas` tetap dapat terjadi, tetapi `LayarMvc` tidak otomatis membangun ulang tampilan. Daftar atau status checkbox dapat tetap menampilkan kondisi lama sampai ada rebuild lain. | Perubahan data di repository/ViewModel tetap dapat terjadi, tetapi widget yang mendengarkan ViewModel tidak diberi notifikasi. Ringkasan atau checkbox dapat tetap menampilkan kondisi lama. |
| Hasil percobaan yang tersedia | Kode MVC saat ini telah mengembalikan `setState()`. Arsip bukti yang disertakan belum memuat hasil percobaan ketika `setState()` MVC dihapus sementara. Langkah uji MVC dijelaskan setelah tabel ini dan perlu dijalankan serta didokumentasikan jika diminta bukti percobaan untuk kedua versi. | Pada screenshot [bukti/afterDisable-notifylisteners.png](bukti/afterDisable-notifylisteners.png), setelah `notifyListeners()` pada `TugasViewModel.ubahStatus()` dihapus sementara, `flutter test test/layar_test.dart` gagal pada pemeriksaan **1 dari 1 selesai** karena teks yang diharapkan tidak ditemukan. Setelah baris notifikasi dikembalikan, screenshot [bukti/rollbackActive-NotifyListeners.png](bukti/rollbackActive-NotifyListeners.png) menunjukkan **2 test lulus**. |

**Langkah percobaan MVC untuk melengkapi bukti:**

1. Pada `lib/mvc/layar_mvc.dart`, cari metode `_ubahStatus()` yang semula seperti berikut:

   ```dart
   void _ubahStatus(int id) {
     setState(() => _pengendali.ubahStatus(id));
   }
   ```

2. Hapus sementara pemicu `setState()` saja, sehingga menjadi:

   ```dart
   void _ubahStatus(int id) {
     _pengendali.ubahStatus(id);
   }
   ```

3. Jalankan `flutter test test/layar_test.dart`. Perubahan status pada Controller tidak diikuti rebuild, sehingga pemeriksaan ringkasan yang mengharapkan **1 dari 1 selesai** seharusnya gagal atau UI masih menampilkan nilai sebelumnya.
4. Kembalikan kode awal yang menggunakan `setState()`, lalu jalankan kembali `flutter test test/layar_test.dart`. Simpan screenshot hasil kegagalan sementara dan hasil setelah pemicu dipulihkan sebagai bukti percobaan MVC.

Percobaan MVC di atas merupakan langkah yang perlu dilakukan untuk memperoleh bukti aktual; hasil kegagalannya belum tercantum dalam screenshot yang tersedia di arsip project ini. Jangan mencantumkannya sebagai percobaan yang sudah dilakukan sebelum pengujian tersebut benar-benar dijalankan.

### 5.3 Kapan masing-masing pola dipilih?

| Pola | Skenario yang sesuai | Kelebihan | Biaya atau risiko |
|---|---|---|---|
| MVC | Aplikasi sederhana dengan alur interaksi yang relatif langsung, seperti halaman daftar tugas pada praktikum ini. | Alur aksi mudah diikuti: View meneruskan aksi ke `PengendaliTugas`, lalu View memperbarui tampilan dengan `setState()`. | View harus mengelola pemanggilan `setState()` pada perubahan yang perlu ditampilkan. Jika pemanggilannya terlupa, data dapat berubah tetapi tampilan tidak ikut diperbarui. |
| MVVM | Aplikasi yang memiliki state tampilan yang lebih banyak atau beberapa bagian UI yang perlu mendengarkan perubahan state yang sama. | State dan logika tampilan ditempatkan pada `TugasViewModel`; `ListenableBuilder` memperbarui UI saat menerima notifikasi. | Membutuhkan ViewModel dan mekanisme notifikasi. Jika `notifyListeners()` terlupa atau lifecycle ViewModel tidak dikelola dengan benar, tampilan dapat tidak sinkron atau listener tidak dibersihkan. |

Pada fitur filter ini, MVC memerlukan `setState()` pada callback `onFilterChanged`, sedangkan MVVM memerlukan `notifyListeners()` di `aturFilterBelumSelesai()`. Keduanya dapat menjalankan fungsi yang sama, tetapi cara memberi tahu tampilan tentang perubahan state berbeda.

## 6. Pengujian dan Hasil

File `test/filter_test.dart` menjalankan widget test untuk MVC dan MVVM. Setiap versi diuji dengan alur berikut:

1. Menambahkan dua tugas.
2. Memastikan kedua tugas muncul dan ringkasan menunjukkan **0 dari 2 selesai**.
3. Mencentang tugas pertama dan memastikan ringkasan menjadi **1 dari 2 selesai**.
4. Mengaktifkan filter dan memastikan tugas pertama tidak terlihat, sedangkan tugas kedua tetap terlihat.
5. Menonaktifkan filter dan memastikan kedua tugas muncul kembali serta status tugas pertama tetap tercentang.
6. Mengaktifkan filter kembali, menyelesaikan tugas kedua, lalu memastikan pesan **Tidak ada tugas yang belum selesai.** muncul sementara ringkasan tetap **2 dari 2 selesai**.

Berdasarkan screenshot [bukti/FlutterTest&AnalyzeFilter.png](bukti/FlutterTest&AnalyzeFilter.png), hasil pengujian filter dan analisis kode adalah:

| Perintah | Hasil pada bukti |
|---|---|
| `flutter analyze` | `No issues found!` |
| `flutter test test/filter_test.dart` | `+2: All tests passed!` — kedua widget test filter lulus. |
| `flutter test` | `+7: All tests passed!` — seluruh tujuh test lulus. |

Rangkaian bukti pengujian dan tampilan setiap versi dituliskan satu per satu pada Bagian 7, termasuk path relatif setiap file gambar agar dapat dibuka langsung dari repository.

## 7. Dokumentasi Bukti Screenshot

Semua tautan di bawah menggunakan **path relatif dari lokasi `LAPORAN.md`**. Karena file laporan berada di root project dan screenshot berada di dalam `bukti/`, path dapat dibuka langsung melalui tautan pada GitHub atau editor Markdown. Setiap bukti ditunjukkan sampai nama file gambarnya.

### 7.1 Persiapan, menjalankan aplikasi, dan pengujian umum

| No. | Bukti screenshot | Keterangan |
|---|---|---|
| 1 | [bukti/MenjalankanPersiapanPraktikum.png](bukti/MenjalankanPersiapanPraktikum.png) | Menjalankan `flutter pub get`, test awal, dan menampilkan versi Flutter 3.47.1. |
| 2 | [bukti/FlutterRun.png](bukti/FlutterRun.png) | Menjalankan aplikasi menggunakan `flutter run -d chrome`. |
| 3 | [bukti/FlutterLogikaTest.png](bukti/FlutterLogikaTest.png) | Hasil menjalankan `flutter test test/logika_test.dart`; dua test logika lulus. |
| 4 | [bukti/FlutterTest&AnalyzeBawaan.png](bukti/FlutterTest&AnalyzeBawaan.png) | Hasil pemeriksaan awal: `flutter analyze` tanpa masalah dan lima test lulus. |
| 5 | [bukti/FlutterTest&AnalyzePemanasanTugas.png](bukti/FlutterTest&AnalyzePemanasanTugas.png) | Hasil pemeriksaan setelah tugas pemanasan: analisis tanpa masalah dan lima test lulus. |
| 6 | [bukti/FlutterTest&AnalyzeFilter.png](bukti/FlutterTest&AnalyzeFilter.png) | Hasil akhir fitur filter: `flutter analyze` tanpa masalah, dua test filter lulus, dan seluruh tujuh test lulus. |

### 7.2 Bukti tampilan dan interaksi MVC

| No. | Bukti screenshot | Keterangan |
|---|---|---|
| 1 | [bukti/MVC/PesanValidasiInput.png](bukti/MVC/PesanValidasiInput.png) | Tampilan pesan validasi input pada halaman MVC. |
| 2 | [bukti/MVC/PengisianBenarVersiNoCheckList.png](bukti/MVC/PengisianBenarVersiNoCheckList.png) | Dua tugas ditampilkan pada MVC dalam keadaan belum dicentang; ringkasan 0 dari 2 selesai. |
| 3 | [bukti/MVC/PengisianYgBenarChecklist1.png](bukti/MVC/PengisianYgBenarChecklist1.png) | Satu dari dua tugas dicentang; ringkasan 1 dari 2 selesai. |
| 4 | [bukti/MVC/PengisianYgBenarChecklist2.png](bukti/MVC/PengisianYgBenarChecklist2.png) | Kedua tugas dicentang; ringkasan 2 dari 2 selesai. |
| 5 | [bukti/MVC/filter/PengisianNoFilter.png](bukti/MVC/filter/PengisianNoFilter.png) | Filter MVC tidak aktif; kedua tugas masih terlihat. |
| 6 | [bukti/MVC/filter/PengisianFilter.png](bukti/MVC/filter/PengisianFilter.png) | Filter MVC aktif; hanya tugas yang belum selesai terlihat, sedangkan ringkasan tetap 1 dari 2 selesai. |

### 7.3 Bukti tampilan dan interaksi MVVM

| No. | Bukti screenshot | Keterangan |
|---|---|---|
| 1 | [bukti/MVVM/PesanValidasiInput.png](bukti/MVVM/PesanValidasiInput.png) | Tampilan pesan validasi input pada halaman MVVM. |
| 2 | [bukti/MVVM/PengisianBenarVersiNoChecklList.png](bukti/MVVM/PengisianBenarVersiNoChecklList.png) | Dua tugas ditampilkan pada MVVM dalam keadaan belum dicentang; ringkasan 0 dari 2 selesai. |
| 3 | [bukti/MVVM/PengisianYgBenarChecklist1.png](bukti/MVVM/PengisianYgBenarChecklist1.png) | Satu dari dua tugas dicentang; ringkasan 1 dari 2 selesai. |
| 4 | [bukti/MVVM/PengisianYgBenarChecklist2.png](bukti/MVVM/PengisianYgBenarChecklist2.png) | Kedua tugas dicentang; ringkasan 2 dari 2 selesai. |
| 5 | [bukti/MVVM/filter/PengisianNoFilter.png](bukti/MVVM/filter/PengisianNoFilter.png) | Filter MVVM tidak aktif; kedua tugas masih terlihat. |
| 6 | [bukti/MVVM/filter/PengisianFilter.png](bukti/MVVM/filter/PengisianFilter.png) | Filter MVVM aktif; hanya tugas yang belum selesai terlihat, sedangkan ringkasan tetap 1 dari 2 selesai. |

### 7.4 Bukti percobaan pemicu rebuild

| Tahap | Bukti screenshot | Keterangan |
|---|---|---|
| Sebelum pemicu dipulihkan | [bukti/afterDisable-notifylisteners.png](bukti/afterDisable-notifylisteners.png) | `notifyListeners()` pada `TugasViewModel.ubahStatus()` dihapus sementara. Test MVVM gagal karena teks ringkasan 1 dari 1 selesai tidak ditemukan. |
| Setelah pemicu dipulihkan | [bukti/rollbackActive-NotifyListeners.png](bukti/rollbackActive-NotifyListeners.png) | `notifyListeners()` dikembalikan; `flutter test test/layar_test.dart` menunjukkan dua test lulus. |

**Catatan bukti MVC:** Di dalam arsip project yang diperiksa belum terdapat screenshot percobaan ketika `setState()` pada `LayarMvc` dihapus sementara. Karena itu, bukti visual untuk percobaan MVC belum dapat dicantumkan. Untuk melengkapinya, lakukan langkah percobaan pada Bagian 5.2, ambil screenshot hasil test ketika `setState()` dihapus, lalu ambil screenshot setelah `setState()` dipulihkan. Simpan kedua gambar di folder `bukti/` dan tambahkan nama file spesifiknya pada tabel ini.

## 8. Kesimpulan

Fitur Filter Belum selesai berhasil diterapkan pada kedua pola arsitektur. Pilihan filter disimpan pada `PengendaliTugas` untuk MVC dan `TugasViewModel` untuk MVVM. Daftar yang ditampilkan disaring melalui `daftarTerlihat`, tetapi repository tetap menyimpan seluruh tugas. Karena ringkasan menggunakan jumlah selesai dan total dari daftar lengkap, angka ringkasan tidak berubah hanya karena filter diaktifkan.

Perbedaan utama kedua pola terlihat pada mekanisme pembaruan UI. MVC menggunakan `setState()` pada layar, sedangkan MVVM menggunakan `notifyListeners()` pada ViewModel yang didengarkan oleh `ListenableBuilder`. Berdasarkan bukti yang tersedia, dua test filter dan seluruh tujuh test project lulus, serta `flutter analyze` tidak menemukan masalah. Percobaan penghapusan `setState()` pada MVC masih perlu direkam secara terpisah untuk melengkapi bukti eksperimen kedua pola.

Path screenshot pada laporan ini bersifat relatif. Pertahankan struktur folder `bukti/` beserta seluruh subfolder dan file PNG-nya di lokasi yang sama dengan `LAPORAN.md` agar tautan tetap berfungsi.
