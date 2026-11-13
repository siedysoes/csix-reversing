# Anti-Debug & Packing

Teknik anti-debug mendeteksi atau menghambat debugger, dan packing mengompresi atau menyembunyikan isi sebuah binary — keduanya adalah cara umum sebuah program menahan pemeriksaan sembarangan. Software komersial memakainya untuk melindungi diri, dan penulis CTF memakainya untuk menambah kesulitan, sehingga mengenalinya adalah skill inti dalam reversing. Artikel ini menjelaskan apa teknik-teknik itu dan cara mengenalinya pada target yang sah.

## Apa itu anti-debug

Sebuah program yang memeriksa apakah sedang diawasi lalu bereaksi — keluar lebih awal, berbohong soal hasil, atau mengubah perilakunya. Tujuan Anda di tingkat ini adalah pengenalan: tangkap pemeriksaannya, pahami apa yang ditanyakannya, dan berpenalaran tentang apa yang dikendalikan oleh hasilnya.

## Teknik deteksi yang umum

| Teknik | Konsep |
| --- | --- |
| `isDebuggerPresent` | API Windows yang membaca flag yang diset saat debugger terpasang |
| `PTRACE_TRACEME` | Proses Linux bertanya ke kernel "apakah saya sedang ditrace?" |
| Timing checks | `rdtsc` atau pembacaan clock mengukur berapa lama kode berjalan — debugger memperlambatnya |
| Pemeriksaan hardware breakpoint | Memeriksa register debug (DR0-DR3 pada x86) yang mungkin sudah diset debugger |
| Pemeriksaan proses induk | Melihat nama proses induk dan membandingkannya dengan ekspektasi |

Ini adalah ide deteksi untuk dipahami, bukan katalog countermeasure — tujuan pendidikannya adalah mengetahui setiap pemeriksaan menanyakan apa dan mengapa cara itu bekerja.

### Mengapa program melakukan ini

Software komersial memakai pemeriksaan ini agar penyalahgunaan sembarangan menjadi lebih sulit; penulis CTF memakainya untuk memaksa Anda membaca kode alih-alih berhenti di breakpoint pertama. Dalam kedua kasus, pemeriksaannya adalah kode biasa: ia membaca sebuah nilai, membandingkannya, lalu bercabang — persis pola yang sudah Anda tahu cara membacanya.

## Cara kemunculannya dalam analisis statis

Sering kali Anda bisa melihat seluruh pemeriksaan tanpa menjalankan apa pun: sebuah pemanggilan ke `isDebuggerPresent`, sebuah permintaan `ptrace`, atau pembacaan timestamp, diikuti sebuah conditional branch. Ikuti cabang itu dan Anda menemukan reaksinya — sebuah `exit` lebih awal, pesan yang berbeda, atau hasil yang palsu. Strukturnya selalu "periksa, lalu bercabang": temukan pemerikasannya, lalu tanyakan apa yang dilakukan cabang itu.

Bentuk pseudocode-nya bisa diprediksi:

```c
if (isDebuggerPresent()) {
    exit(-1);
}
```

Baca kondisinya dulu, lalu kedua cabangnya: satu jalur melanjutkan program, jalur lain adalah reaksi yang sedang Anda dokumentasikan.

## Cara kemunculannya secara dinamis

Pola yang bisa dikenali: program berperilaku berbeda di bawah debugger. Ketika sebuah sesi mati tepat di sebuah breakpoint, anti-debug adalah hipotesis pertama — bukan crash misterius.

| Gejala yang diamati | Penafsiran yang mungkin |
| --- | --- |
| Program keluar begitu Anda melakukan attach | Presence check memicu lebih awal |
| Output trace berhenti mendadak (`strace ./chall` berakhir lebih awal) | Proses mendeteksi tracing atau keluar secara sengaja |
| Hasil berbeda dengan dan tanpa debugger | Pemeriksaan duduk di jalur keputusan utama |

## Apa yang harus dilakukan — secara konseptual

Kenali dan pahami dulu. Urutan pembacaannya:

1. Identifikasi pemerikasannya — API call mana, instruksi mana.
2. Baca cabang yang dikendalikannya — apa yang berubah ketika pemeriksaan itu memicu?
3. Putuskan apa arti hasil itu untuk sisa analisis Anda.

Di setting CTF, pendekatan standar — men-patch pemeriksaan, atau membuat program mengamati state yang diharapkannya — terdokumentasi di materi kursus dan write-up; pelajari dalam konteks lab tersebut alih-alih memperlakukan artikel ini sebagai resep. Skill yang diajarkan di sini adalah membaca pemerikasannya, bukan menghafal bypass.

## Penjelasan packing

Binary yang dipacked menyimpan sebagian besarnya section dalam bentuk terkompresi atau terenkripsi; saat startup, sebuah stub kecil men-dekompresi kode yang sebenarnya ke dalam memori. Di disk ia terlihat seperti sampah: `strings` mengembalikan fragmen, import mungkin tersembunyi di balik stub, dan yang pertama Anda lihat adalah unpacker-nya, bukan programnya. UPX adalah contoh klasik dan sangat umum di crackme serta CTF.

### Meng-unpack file UPX

```bash
file packed_chall               # often reports "UPX compressed"
upx -d packed_chall -o chall    # decompress (verify options for your UPX build)
upx -l packed_chall             # list packed contents (verify)
```

Meng-unpack file yang Anda berhak analisis adalah praktik standar, sama seperti menjalankan `strings` — Anda hanya membacanya.

### Tampilan file yang dipacked

```bash
file packed_chall
# ELF 64-bit LSB executable ... (some versions report "UPX compressed")
strings packed_chall | grep -i upx
# UPX!
```

Marker `UPX!` dan output string yang nyaris kosong adalah sinyal bahwa kode yang sebenarnya tidak tersimpan di disk dalam bentuk polos.

## Mengapa packing mengubah alur kerja

Urutan normal Anda gagal secara berurutan: `file` melaporkan terkompresi, `strings` menampilkan noise, disassembler memetakan sebuah stub alih-alih kode yang sebenarnya. Solusinya adalah meng-unpack dulu, lalu memulai ulang alur kerja standar:

1. Unpack (`upx -d` untuk file UPX yang Anda berhak analisis).
2. Jalankan ulang `file` dan `strings` pada hasil unpack.
3. Buka file hasil unpack di disassembler dan temukan `main` seperti biasa.

Packing mengubah entry point proses Anda, bukan konsepnya.

## Catatan hukum dan etika

Anti-debug dan packing ada untuk melindungi software. Membypass proteksi pada software yang tidak Anda miliki dan tidak berhak analisis — produk komersial, game, aplikasi berlisensi — adalah hal yang dilarang. Target Anda dalam materi ini adalah binary yang Anda miliki atau berhak analisis: crackme, challenge CTF, file lab, dan build Anda sendiri.

## Binary yang dipacked dan malware

Packers sangat banyak dipakai oleh penulis malware untuk menunda pemeriksaan, sehingga sampel yang dipacked adalah hal biasa dalam pekerjaan defensif, yang membutuhkan lingkungan khusus yang terisolasi. Analisis defensif atas sampel semacam itu dibahas di [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/); jangan pernah menjalankan file packed yang tidak dikenal di luar isolasi semacam itu.

## Pengingat penggunaan yang sah

Semua yang ada di atas — mengenali pemeriksaan, meng-unpack, dan membaca hasil — hanya berlaku untuk binary yang Anda miliki atau secara eksplisit berhak analisis.

## Lihat juga

- [String & Pola](strings-patterns.md)
- [Analisis Dinamis](dynamic-analysis.md)
- [Hukum & Etika](../fundamentals/legal-ethics.md)
- [Apa Itu Reverse Engineering](../fundamentals/what-is-reversing.md)
