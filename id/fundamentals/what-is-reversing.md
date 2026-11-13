# Apa Itu Reverse Engineering

Reverse engineering (rekayasa balik) adalah praktik memahami cara kerja program yang sudah jadi dengan mengkaji program itu sendiri, bukan source code-nya. Ini penting karena sebagian besar software yang Anda temui dalam pekerjaan keamanan — firmware, malware, tool closed-source — jarang dibekali kode yang bisa dibaca. Artikel ini mendefinisikan reversing, menunjukkan di mana ia dipakai secara sah, dan menetapkan pola pikir untuk seluruh jalur fundamentals CSIX Reversing.

## Definisi

Dalam dunia software, reverse engineering berarti mengambil artefak yang cara kerja internalnya tidak didokumentasikan untuk Anda, lalu membangun model yang akurat tentang bagaimana artefak itu berfungsi. Dalam praktiknya ada tiga hal:

- membaca byte pada binary dan mendecode-nya menjadi instruksi
- menelusuri bagaimana data berpindah di dalam program
- mengamati perilaku di bawah kondisi yang terkendali dan menguji hipotesis tentang maksudnya

Keluaran dari reversing adalah pemahaman: catatan, diagram, deskripsi sebuah protokol, atau temuan tentang sebuah kerentanan. Analisis seperti ini tidak memerlukan modifikasi binary — justru sesi-sesi awal Anda sebaiknya tidak mengubah apa pun sama sekali.

## Loop Reversing

Setiap sesi kira-kira mengikuti siklus yang sama:

1. **Amati** — jalankan inspeksi read-only: `file`, `strings`, disassembler.
2. **Buat hipotesis** — tebak apa fungsi sebuah region kode atau data.
3. **Verifikasi** — cocokkan tebakan itu dengan instruksinya, atau dengan jejak debugger.
4. **Dokumentasikan** — tulis apa yang Anda buktikan, dan pisahkan jelas dari apa yang Anda asumsikan.
5. **Ulangi** — lanjut ke misteri berikutnya.

Skepticisme sudah tertanam dalam loop-nya: jangan asumsikan apa pun, verifikasi semuanya.

## Sedikit Sejarah

Reverse engineering sudah ada sebelum software — insinyur hardware selalu mempelajari sirkuit, mesin, dan engine yang sudah jadi untuk mengetahui cara kerjanya, dan reverse engineering software hanya mewarisi pendekatan itu: kaji artefak yang sudah jadi, amati di bawah kondisi terkendali, lalu bangun ulang pemahaman Anda tentangnya.

## Siapa yang Melakukannya

| Peran | Yang mereka reverse |
|---|---|
| Peneliti keamanan | Aplikasi, protokol, dan firmware, untuk menemukan kerentanan |
| Analis malware | Sampel berbahaya, untuk memahami perilaku dan membangun pertahanan |
| Pengembang interoperabilitas | Format file dan API yang tertutup, agar produk terpisah bisa berkomunikasi |
| Insinyur hardware dan firmware | Firmware perangkat dan driver yang menjembatani hardware dengan sistem operasi |
| Pelajar dan pemain CTF | Crackme dan challenge, untuk membangun keterampilannya |

## Penggunaan yang Sah

| Penggunaan | Seperti apa dalam praktiknya |
|---|---|
| Riset keamanan | Menentukan apakah mekanisme update sebuah produk bisa dimanipulasi |
| Analisis malware | Mengekstrak konfigurasi dari sebuah sampel agar trafiknya bisa diblokir |
| Interoperabilitas | Mempelajari format file agar dua program bisa bertukar data dengan benar |
| CTF dan latihan | Menyelesaikan crackme dan challenge reversing pada binary yang diberikan kepada Anda |
| Kepatuhan lisensi | Memastikan komponen open-source mana yang di-link ke dalam sebuah produk |
| Memulihkan source yang hilang | Membangun ulang pemahaman tentang binary internal ketika kodenya sudah tidak ada |

## Apa yang BUKAN Reverse Engineering

- **Bukan otomatis ilegal.** Legalitas bergantung pada siapa pemilik binary, apa isi lisensi Anda, dan hukum mana yang berlaku di wilayah Anda. Lihat [Hukum & Etika](legal-ethics.md).
- **Bukan pembajakan.** Menghapus pengecekan lisensi dari software komersial yang bukan milik Anda agar bisa dipakai gratis adalah pembajakan software. Itu aktivitas berbeda dengan tujuan berbeda, dan bukan yang diajarkan panduan ini.
- **Bukan selalu soal modifikasi.** Banyak reversing berakhir dengan laporan tertulis, bukan binary yang dipatch.
- **Bukan sihir.** Ini adalah pembacaan yang sabar, langkah demi langkah — lebih dekat ke debugging daripada menebak password pada percobaan ketiga.

## Reverse Engineering di Samping Keterampilan Terkait

| Aktivitas | Pertanyaan yang dijawabnya |
|---|---|
| Reverse engineering | Bagaimana program ini bekerja? |
| Debugging | Kenapa program saya melakukan itu, tepat sekarang? |
| Code review | Apa yang salah dengan source code ini? |
| Exploitation | Bagaimana saya bisa membuat program melakukan sesuatu yang tidak diinginkan? |

## Keterkaitannya dengan Ekosistem CSIX

Reversing jarang terjadi sendirian. Situs CSIX pendamping membahas keterampilan-keterampilan di sekitarnya:

- [CSIX CTF](https://siedysoes.github.io/csix-ctf/) — di mana Anda melatih keterampilan ini pada challenge yang dibuat untuk berlatih
- [CSIX Pentest](https://siedysoes.github.io/csix-pentest/) — bagaimana kelemahan yang Anda temukan diubah menjadi exploit yang berfungsi
- [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/) — analisis defensif terhadap binary berbahaya di lingkungan yang terkendali

## Pola Pikir

Tiga kebiasaan membawa sebagian besar sesi:

- **Rasa ingin tahu.** Tanya apa fungsi sebuah function, lalu jawab dengan bukti.
- **Kesabaran.** Anda akan salah membaca instruksi, jalan buntu, dan lupa fungsi sebuah register. Itu normal: cari tahu lalu lanjutkan.
- **Skepticisme.** Jangan asumsikan apa pun, verifikasi semuanya. Function bernama `check_auth` belum tentu mengecek apa pun, dan string "success" bisa saja dicetak di jalur kegagalan.

### Sesi reversing pertama Anda

Sesi pertama yang tipikal pada crackme sederhana hanya butuh dua perintah read-only:

```bash
strings crackme
strings crackme | grep -iE 'flag|correct'
```

`strings` mengekstrak teks yang terbaca dari binary; grep mempersempitnya ke kemungkinan jawaban. Jika perintah kedua mencetak sesuatu seperti `Correct! Flag{you_found_me}`, Anda sudah menemukan flag tersembunyi tanpa menjalankan atau mengubah apa pun — murni observasi terhadap challenge binary yang diberikan kepada Anda. Dari sini, lanjutkan ke [Strings & Patterns](../techniques/strings-patterns.md) dan [The Reversing Workflow](../practice/workflow.md) yang lengkap.

## Otorisasi Dahulu

Semua yang ada di CSIX Reversing mengasumsikan satu aturan: analisis hanya binary yang Anda miliki, diberikan kepada Anda, atau secara eksplisit diberi otorisasi untuk dianalisis — program Anda sendiri, latihan lab, crackme, dan challenge CTF. Jika Anda tidak bisa menjawab "apakah saya boleh melihat ini?" dengan jujur berupa ya, berhentilah dan minta izin sebelum melanjutkan.

## Lihat juga

- [Cara Binaries Bekerja](how-binaries-work.md)
- [Dasar Assembly](assembly-basics.md)
- [Hukum & Etika](legal-ethics.md)
- [Strings & Patterns](../techniques/strings-patterns.md)
- [The Reversing Workflow](../practice/workflow.md)
