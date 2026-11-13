# Analisis Dinamis

Analisis dinamis berarti menjalankan program sambil mengawasi apa yang dilakukannya — syscall apa saja yang dipanggil, fungsi apa saja yang dipanggil, dan nilai apa yang duduk di register pada setiap langkah. Ia menunjukkan kebenaran perilaku runtime yang hanya bisa ditebak oleh pembacaan statis. Artikel ini membahas tools observasi standar terhadap binary yang Anda berhak analisis.

## Kapan menggunakannya

Gunakan setelah triage statis memberi Anda peta, atau ketika analisis statis menemui jalan buntu: membuka binary yang dipacked, menelusuri data yang hanya didekripsi saat runtime, atau mengonfirmasi sebuah hipotesis dengan cepat. Urutan default yang baik adalah: statis dulu untuk merencanakan, dinamis kemudian untuk memverifikasi.

## Aturan keselamatan emas

Jalankan hanya binary yang Anda percaya atau miliki, di dalam VM yang terisolasi — tidak pernah di mesin harian Anda, dan dengan jaringan dimatikan kecuali Anda benar-benar membutuhkannya. Sebagian besar crackme dan challenge CTF sama sekali tidak membutuhkan jaringan, jadi VM tanpa koneksi sudah cukup. Target yang sah untuk materi ini adalah build Anda sendiri, file lab yang disediakan, crackme, dan challenge CTF.

## Dasar GDB

`gdb` adalah debugger standar Linux. Inilah perintah yang akan Anda gunakan terus-menerus:

| Perintah | Tujuan |
| --- | --- |
| `gdb ./chall` | Memulai debugger bersama binary |
| `break main` | Menetapkan breakpoint di `main` |
| `run` | Mengeksekusi program (berhenti di breakpoint) |
| `next` | Melangkahi baris atau pemanggilan fungsi saat ini |
| `step` | Masuk ke dalam pemanggilan fungsi saat ini |
| `info registers` | Menampilkan register general-purpose |
| `x/20x $rsp` | Memeriksa 20 word hex mulai dari stack pointer |
| `continue` | Melanjutkan eksekusi sampai peristiwa berikutnya |
| `quit` | Keluar dari debugger |

### Sesi tipikal

```bash
gdb ./chall
(gdb) break main
(gdb) run                # or: run secret123  (pass arguments after run)
(gdb) next
(gdb) info registers
(gdb) quit
```

Perhatikan bahwa argumen ditempatkan setelah `run`, bukan di command line `gdb` — debugger-lah yang menjalankan programnya sendiri.

## Breakpoint: berhenti di tempat yang tepat

Sebuah breakpoint menahan eksekusi ketika mencapai sebuah alamat, sehingga Anda bisa memeriksa state tepat pada momen yang Anda pedulikan — sebelum sebuah perbandingan, atau tepat setelah sebuah input dibaca. Break dulu di `main`, lalu di fungsi yang Anda identifikasi secara statis, lalu di perbandingan itu sendiri.

## Mengamati perbandingan

Break di pemanggilan fungsi perbandingan (misalnya `strcmp`) dan lihat argumennya: `info registers` plus `x/s $rdi` (argumen pertama pada SysV AMD64) mencetak string yang sedang diperiksa. Anda melihat nilai yang diharapkan dan nilai yang diberikan pada saat yang sama — cara tercepat untuk mengonfirmasi sebuah pengecekan password.

## Menelusuri syscall dan pemanggilan library

Dua one-liner yang tidak memerlukan pengaturan breakpoint:

```bash
strace ./chall   # system calls: open, read, write, exit
ltrace ./chall   # library calls: printf, strcmp, malloc
```

`strace` menunjukkan apa yang dilakukan program ke dunia luar — sebuah `write` berisi "wrong password" memberi tahu Anda jalur kegagalannya tanpa membaca assembly apa pun. `ltrace` menunjukkan logika tingkat library. Keduanya standar di distribusi Linux; verifikasi ketersediaannya di sistem Anda.

## Debugging gaya printf vs debugger

Bila Anda mengontrol source, Anda tinggal menambah pernyataan print — itu debugging biasa, bukan reversing, dan di luar cakupan di sini. Ketika Anda hanya punya binary, debugger dan tools trace adalah semua yang Anda miliki.

## Contoh terperinci: crackme fiktif

File yang disediakan: `crackme1` (latihan lab yang Anda berhak analisis).

```bash
strace ./crackme1 test
# ... write(1, "wrong password\n", 15) = 16
# ... exit_group(1)

gdb ./crackme1
(gdb) break check_password
(gdb) run test
(gdb) info registers
```

Trace saja sudah memberi tahu Anda bahwa program mencetak pesan tetap lalu keluar — jadi logikanya adalah perbandingan, bukan lookup. Break di fungsi pemeriksanya memastikannya: Anda mengamati input datang dan melihat di mana ia menyimpang dari nilai yang diharapkan. Peta statis mengatakan *di mana*; trace mengatakan *apa yang sebenarnya terjadi*.

## Dinamis vs statis

| Dimensi | Analisis statis | Analisis dinamis |
| --- | --- | --- |
| Risiko | Nol — kode tidak pernah dijalankan | Menjalankan kode yang tidak tepercaya (pakai VM) |
| Data runtime | Tidak bisa melihatnya | Melihat data yang didekripsi dan di-unpack |
| Kecepatan ke jawaban pertama | Cepat untuk triage | Setup lebih lambat, konfirmasi cepat |
| Cakupan | Semua kode, termasuk jalur yang tak terjangkau | Hanya jalur yang benar-benar dieksekusi |

Gunakan keduanya: statis untuk menemukan titik yang menarik, dinamis untuk mengonfirmasi apa yang terjadi di sana.

## Kapan jangan pakai dinamis

Jangan menjalankan sampel malware sembarangan — itu milik analisis defensif di lingkungan yang benar-benar terisolasi; lihat [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/) untuk mengetahui bagaimana pekerjaan defensif disusun. Dalam kursus ini, target analisis dinamis adalah binary yang Anda miliki atau berhak analisis: file lab, crackme, dan challenge CTF.

## Lihat juga

- [Analisis Statis](static-analysis.md)
- [Anti-Debug & Packing](anti-debug.md)
- [Hukum & Etika](../fundamentals/legal-ethics.md)
- [Latihan & Lab](../practice/labs.md)
