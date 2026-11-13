# Latihan & Lab

Reverse engineering adalah keterampilan yang Anda bangun dengan memecahkan target kecil yang sah secara berulang — bukan dengan membaca teorinya. Lab memberikan binary yang legal dan sengaja dirancang untuk Anda asah sebelum menyentuh pekerjaan dunia nyata. Halaman ini memetakan lanskap lab, menunjukkan cara memilih target pertama, dan memberi Anda pola latihan yang bisa diulang.

> Semua di sini berjalan pada challenge yang sah dan binary milik Anda sendiri — tidak pernah pada software proprietary yang tidak Anda hak untuk analisis.

## Mengapa Latihan pada Target Sah Dahulu

Target dunia nyata berantakan, bermusuhan, dan sensitif secara hukum. Target sah (crackme, challenge CTF, wargame) dirancang untuk diselesaikan, sehingga Anda mempelajari alur kerjanya — triage, anchoring, dekompilasi, verifikasi — tanpa risiko hukum atau waktu terbuang untuk keacakan yang tak bisa diselesaikan. Keterampilannya bisa ditransfer: pola yang sama berlaku pada engagement yang sah di kemudian hari.

## Lanskap Lab

| Platform | Level | Catatan |
| --- | --- | --- |
| crackmes.one | Pemula hingga lanjutan | Crackme yang diunggah komunitas, gratis — verifikasi ketersediaan dan aturan terkini |
| picoCTF | Pemula | CTF tahunan plus kumpulan soal latihan — verifikasi akses terkini |
| OverTheWire (wargame gaya Narnia) | Pemula | Dasar wargame: shell, scripting, reversing sederhana — verifikasi nama game terkini |
| HackTheBox — kotak Reverse Engineering | Menengah | Sebagian konten tersedia di tier gratis — verifikasi tier dan harga terkini |
| TryHackMe — ruang RE | Pemula hingga menengah | Ruang terpandu dengan walkthrough — verifikasi akses gratis/berbayar terkini |
| Acara yang terdaftar di CTFtime | Menengah hingga lanjutan | Kompetisi nyata dengan write-up yang dipublikasikan — verifikasi daftar acara terkini |

Harga, tier gratis, dan fitur platform selalu berubah — periksa ketentuan terkini tiap platform sebelum Anda berkomitmen.

## Cara Memilih Target Pertama Anda

| Pengalaman | Target |
| --- | --- |
| Pemula sejati | Soal strings/rev picoCTF — temukan flag dengan `strings` dan tooling dasar |
| Sudah nyaman dengan tool-nya | Crackme level pemula — tebak/validasikan sebuah password atau temukan kunci tersembunyi |
| Siap untuk kesulitan lebih | Kotak reverse engineering HTB atau latihan analisis CVE (reproduksi write-up yang sah) |

Pilih satu level di bawah yang menurut Anda Anda berada. Menyelesaikan target mudah membangun lebih banyak keterampilan daripada meninggalkan target sulit.

## Pola Latihan

Setiap target mengikuti pola yang sama:

1. **Pilih** target yang Anda beri izin untuk diselesaikan.
2. **Timebox** (lihat jadwal di bawah) — putuskan sejak awal kapan harus berhenti.
3. **Triage** — `file`, `checksec`, `strings` sebelum menyentuh disassembler.
4. **Dokumentasikan** — catat perintah, temuan, dan jalan buntu saat Anda melangkah.
5. **Bandingkan** dengan write-up yang dipublikasikan — catat apa yang Anda lewatkan dan mengapa.

Langkah perbandingan adalah tempat sebagian besar pembelajaran terjadi: di situ Anda melihat pertanyaan mana yang gagal Anda ajukan.

### Contoh Sesi 30 Menit

| Menit | Aktivitas |
| --- | --- |
| 0–5 | Triage: `file`, `checksec`, `strings`; tulis pertanyaan sesi |
| 5–15 | Anchor dan baca: temukan xref-nya, decompile fungsi target |
| 15–25 | Buat hipotesis dan verifikasi: telusuri logikanya, uji pemahaman Anda |
| 25–30 | Dokumentasikan temuan dan pertanyaan berikutnya sebelum menutup file |

Jika triage sendiri menghabiskan seluruh slot, itu adalah hasil yang normal — catat dan lanjutkan sesinya lain waktu.

## Perlakukan Seperti Latihan Gym

Konsistensi mengalahkan maraton. Sesi pendek dan teratur dengan pertanyaan yang jelas setiap kali lebih unggul dibanding all-nighter sesekali.

| Sesi | Fokus |
| --- | --- |
| 30 menit | Satu triage + satu pertanyaan terarah (mis., "apa gunanya string ini?") |
| 30 menit | Satu putaran decompile-and-baca pada satu fungsi |
| 30 menit | Bandingkan catatan Anda dengan write-up dan perbarui checklist Anda |
| Mingguan | Selesaikan ulang target lama tanpa melihat catatan Anda |

Tiga sesi 30 menit per minggu akan mengalahkan satu maraton 6 jam setiap bulan.

## Tulis Up Setiap Penyelesaian

Perlakukan setiap penyelesaian sebagai draf pertama catatan pribadi Anda. Write-up singkat — target, proteksi, pendekatan, perintah kunci, apa yang berhasil, apa yang tidak — menjadi materi yang bisa dicari saat Anda menemukan pola serupa berikutnya. Simpan bersama catatan alur kerja Anda agar proses Anda membaik di setiap target.

## Latihan Mandiri dengan Binary Anda Sendiri

Anda tidak selalu butuh platform online. Kompilasi program C kecil yang Anda tulis sendiri, lalu reverse karya Anda — ini 100% legal dan bisa langsung diulang.

| Latihan | Persiapan | Tujuan |
| --- | --- | --- |
| String tersembunyi | `char *s = "correct-horse";` tidak mencetak apa pun | Temukan string dengan `strings`, lalu lokasikan xref-nya di Ghidra |
| Pemeriksaan password | `strcmp(input, "letmein")` | Temukan perbandingannya, konfirmasikan dengan gdb |
| Kunci aritmetika | `if (x * 31 + 7 == 1234)` | Selesaikan `x` dari kode hasil dekompilasi |
| Binary menu | `switch` untuk pilihan 1–4 | Petakan tiap cabang ke fungsinya |
| Anti-tamper ringan | Keluar jika sebuah nilai berbeda dari yang diharapkan | Break pada `exit`/return awal, temukan pemeriksaannya |

Kompilasi dulu dengan simbol debug (`gcc -g`), lalu build ulang tanpa simbol (`strip`) dan ulangi latihannya untuk merasakan perbedaannya:

```bash
gcc -g -o drill drill.c    # symbols kept — easier first pass
strip drill                # rebuild without symbols — redo the drill
```

## Ekosistem CSIX

- [Kategori CTF dan petunjuk latihan](https://siedysoes.github.io/csix-ctf/) — di mana reversing cocok di antara kategori CTF.
- [Panduan home lab](https://siedysoes.github.io/csix-wiki/#/en/practical/hands-on-lab) — cara membangun lingkungan terisolasi tempat latihan ini seharusnya berjalan.

## Catatan Keamanan

Jalankan setiap binary latihan di VM tanpa shared folder dan (untuk crackme) tanpa jaringan — lihat panduan home lab di atas. Gunakan hanya binary yang disediakan platform atau yang Anda kompilasi sendiri; jika asal-usul atau izin sebuah target tidak jelas, tinggalkan.

## Lihat juga

- [Tool Reverse Engineering](reversing-tools.md)
- [Alur Kerja Reverse](workflow.md)
- [String & Pola](../techniques/strings-patterns.md)
- [Hukum & Etika](../fundamentals/legal-ethics.md)
