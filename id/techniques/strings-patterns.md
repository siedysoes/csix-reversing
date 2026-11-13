# String & Pola

Strings adalah cara tercepat untuk melihat sekilas isi binary yang tidak dikenal: perintah ini mengekstrak rangkaian karakter yang dapat dicetak, dan program biasanya menyimpan pesan, flag, format, serta teks error sebagai literal biasa. Karena itu, hampir setiap sesi reversing dimulai dengan beberapa perintah yang sama. Artikel ini membahas cara standar dan hanya-baca untuk membaca teks sebuah binary sebelum membuka disassembler.

## Mengapa strings selalu perintah pertama

Saat seorang developer menulis `printf("Access denied")`, byte-nya ikut tersimpan di dalam file. Compiler jarang menyembunyikannya. Dengan pemindaian sepuluh detik, Anda sudah bisa tahu apa yang dikatakan program, apa yang dimintanya, dan terkadang bentuk persis jawaban yang diharapkannya.

## 10 detik pertama

Jalankan perintah berikut pada binary yang Anda berhak analisis (file lab, crackme, atau challenge CTF):

| Perintah | Apa yang diberitahukannya |
| --- | --- |
| `file chall` | Format file dan arsitektur CPU (ELF x86-64, PE32, ...) |
| `strings chall \| head` | Konten terbaca pertama: pesan, format, path |
| `strings -n 8 chall \| grep -i flag` | String lebih panjang yang mirip flag atau key |
| `xxd chall \| head` | Magic bytes di offset 0 — memastikan apa yang dilaporkan `file` |

Selalu mulai dengan `file`: perintah itu menentukan toolchain mana yang Anda raih berikutnya.

## Cara kerja strings

`strings` memindai file untuk menemukan byte ASCII yang dapat dicetak secara berurutan (huruf, angka, tanda baca, spasi) dan mencetak setiap rangkaian yang ditemukannya. Perintah ini tidak mengeksekosi apa pun dan tidak perlu memahami format file — ia hanya membaca byte, yang membuatnya aman dan cepat.

### Flag panjang minimum

Rangkaian pendek menghasilkan noise (`a`, `OK`, `if`). Flag `-n` menentukan panjang minimum sebuah rangkaian agar dicetak:

```bash
strings -n 8 chall | head -n 30   # only runs of 8+ characters
```

Gunakan nilai yang lebih kecil (misalnya `-n 5`) bila Anda mengharapkan token pendek, dan nilai lebih besar untuk memangkas noise.

## Pola grep yang layak dicoba

`strings` mencetak semuanya; `grep` menyaringnya. Setiap pola menunjuk ke bagian program yang berbeda:

| Pola | Apa yang disarankannya |
| --- | --- |
| `flag{` | Literal flag CTF, atau format sebuah flag |
| `password` | Perbandingan kredensial di dekatnya |
| `usage:` | Parsing argumen command-line |
| `error` | Jalur kegagalan — sering memperlihatkan state tersembunyi |
| `admin` | Role, privilege, atau perintah tersembunyi |

Gabungkan beberapa flag dan cari tanpa mempedulikan huruf besar/kecil:

```bash
strings -n 6 chall | grep -i 'password'
```

### One-liner regex extended

Untuk beberapa alternatif dalam satu kali jalan, gunakan `grep -E`:

```bash
strings -n 6 chall | grep -E 'flag\{|secret|admin|root'
```

Escape `{` di sana, karena dalam regex extended tanda itu memulai operator repetisi.

## ASCII vs UTF-16 strings

Binary Linux sebagian besar menyimpan teks sebagai ASCII/UTF-8 satu byte, sehingga `strings` biasa bisa menemukannya. Program Windows sering menyimpan teks sebagai UTF-16 (satu karakter = dua byte, low byte di depan), yang dilewati `strings` biasa karena setiap byte kedua adalah `0x00`.

GNU `strings` mendukung berbagai encoding; little-endian 16-bit adalah yang biasa dipakai untuk teks Windows (verifikasi set opsi pada build Anda):

```bash
strings -el application.exe | grep -i password
```

Verifikasi apa yang Anda temukan: adanya sebuah string di dalam file tidak berarti string itu dipakai — konfirmasikan lewat cross-reference di disassembler Anda (lihat [Analisis Statis](static-analysis.md)).

## Membaca konteks di sekitarnya

Cetak offset file dari setiap string dengan `-t x` (offset hexadecimal):

```bash
strings -t x chall | grep -i 'password'
# example output:   1f30 usage: ./chall <password>
# example output:   1f58 wrong password
```

Dengan offset tersebut, Anda dapat:

- menuju ke sana di hex editor dan membaca byte di sekitarnya;
- melompat ke alamat di Ghidra (untuk ELF non-PIE, file offset sering sama dengan virtual address — konfirmasikan dengan `readelf -l` sebelum mempercayainya);
- memperhatikan bahwa string yang saling terkait dikelompokkan bersama — developer menaruh tabel pesan di satu tempat.

## Ketika strings tidak cukup

Strings hanya melihat apa yang tersimpan dalam bentuk polos. Binary yang dipacked atau terenkripsi hanya menghasilkan sedikit lebih dari fragmen, dan binary yang di-strip menyembunyikan nama fungsi. Jika output terlihat seperti sampah atau nyaris kosong, lanjutkan ke konsep packing dan anti-debug di [Anti-Debug & Packing](anti-debug.md).

## Contoh terperinci: crackme fiktif

File yang disediakan: `crackme1` (latihan lab yang Anda berhak analisis).

```bash
file crackme1
# crackme1: ELF 64-bit LSB executable, x86-64 ...

strings -n 6 crackme1 | grep -iE 'flag\{|password|usage'
# usage: ./crackme1 <password>
# wrong password
# Access granted: flag{...}

strings -t x crackme1 | grep -i password
#   1f30 usage: ./crackme1 <password>
#   1f58 wrong password
```

Kesimpulannya: program menerima satu argumen, membandingkannya dengan sesuatu, dan bila berhasil mencetak string berbentuk flag (placeholder `flag{...}` di atas). Langkah berikutnya adalah membuka binary di disassembler, menemukan string di offset `0x1f58`, dan mengikuti cross-reference-nya ke kode perbandingan.

## Pengingat penggunaan yang sah

Jalankan perintah ini hanya pada binary yang Anda miliki atau secara eksplisit berhak analisis — build Anda sendiri, file lab yang disediakan, crackme, dan challenge CTF. Membaca strings dari software proprietary yang tidak berhak Anda periksa berada di luar cakupan materi ini.

## Lihat juga

- [Cara Binaries Bekerja](../fundamentals/how-binaries-work.md)
- [Analisis Statis](static-analysis.md)
- [Anti-Debug & Packing](anti-debug.md)
- [Alat Reversing](../practice/reversing-tools.md)
