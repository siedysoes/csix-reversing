# Analisis Statis

Analisis statis berarti mempelajari program tanpa mengeksekusinya: Anda membaca byte, struktur, dan kodenya dengan tools yang tidak pernah menjalankan binary tersebut. Karena tidak ada yang dieksekusi, ini adalah cara paling aman untuk memulai dan tempat yang tepat untuk membangun peta sebelum pekerjaan dinamis apa pun. Artikel ini membahas alur kerja standar terhadap binary yang Anda berhak analisis.

## Mengapa statis dulu

Menjalankan kode yang tidak dikenal bisa memicu perilaku destruktif atau berjejaring; disassembler tidak. Analisis statis memungkinkan Anda mengidentifikasi format, menemukan entry point, mendaftar import, dan menemukan fungsi-fungsi menarik dengan risiko eksekusi nol — lalu memutuskan apa (bila ada) yang layak dijalankan kemudian.

## Alur kerjanya

Mulai di shell, selesaikan di disassembler:

```bash
file chall                  # identify format and architecture
strings chall | head -n 40  # see readable content
```

Kemudian, di disassembler seperti Ghidra:

```
import the file -> accept auto-analysis -> open the Decompiler view
find main (Search > For Functions, or the Symbol Tree)
read main: note its arguments and what it calls
follow the interesting function one level at a time
record inputs (arguments, files, environment) and outputs (messages, return values)
```

Cyclenya adalah: baca, ikuti, beri anotasi. Anda berhenti ketika bisa menjelaskan bagaimana input dipetakan ke output.

## Import sebagai peta

Fungsi-fungsi yang di-import memberi tahu Anda apa yang dibutuhkan program dari sistem operasi dan library. Fungsi-fungsi itu adalah profil cepat perilakunya:

| Fungsi import | Apa yang diungkapkannya |
| --- | --- |
| `printf` / `puts` | Menghasilkan output — cari format string-nya |
| `strcmp` / `strncmp` | Membandingkan sesuatu — kemungkinan pengecekan password atau key |
| `getenv` | Membaca environment variable — periksa namanya |
| `socket` / `connect` | Networking — program berbicara ke sesuatu |
| `fopen` / `read` | Akses file — lihat path mana yang muncul di strings |

Bila sebuah import ada tetapi lokasi pemanggilannya sulit ditemukan, lokasi pemanggilan itulah target berikutnya.

## Cross-reference (xref)

Cross-reference menjawab pertanyaan "siapa yang memakai ini?". Pilih sebuah fungsi atau string dan daftarkan referensinya: string menuntun Anda ke kode yang mencetak atau membandingkannya, dan fungsi menuntun Anda ke pemanggilnya. Xref adalah jejak remah roti melintasi binary — ikuti alih-alih membaca disassembly secara linear.

## Dasar control flow

`if`/`else` menjadi conditional jump (`je`, `jne`, `jg`, ...) di tingkat assembly: satu cabang per hasil, lalu disatukan kemudian. Loop adalah lompatan mundur ke awal sebuah blok. Saat Anda membaca pseudocode, ingat bahwa setiap `if` yang Anda lihat adalah conditional jump dan setiap loop adalah lompatan mundur — ide yang sama, ejaan yang berbeda.

Perbandingan sederhana terbaca seperti ini dalam assembly:

```asm
cmp  eax, 0          ; compare the value with 0
je   branch_else     ; jump if equal -> the "if" took the other path
; ... "then" branch continues here ...
```

Pola yang harus dipelajari adalah: **bandingkan, lalu conditional jump, lalu dua jalur yang bergabung kembali**. Semua yang lebih besar — kondisi bersarang, switch, loop — dibangun dari satu pola itu.

## Entry point: tempat pembacaan dimulai

Setiap binary memiliki setidaknya satu entry point, dan itu tidak selalu fungsi yang Anda duga: ELF minimal bisa dimulai di `_start` (yang memanggil `main`), program GUI mungkin memakai `WinMain`, dan sebuah DLL mengekspor fungsi tanpa `main` sama sekali. Ketika entry point terlihat seperti plumbing startup, ikuti pemanggilannya sampai Anda mencapai kode yang ditulis oleh penulisnya — biasanya itu `main`.

## Konstanta dan string sebagai jangkar

Teknik klasik: temukan string yang Anda pedulikan ("wrong password"), lompat ke kode yang mereferensikannya, dan baca fungsi itu. String dan konstanta numerik (`0x1337`, `1000000`) adalah landmark stabil — beri nama baru fungsi-fungsi di sekitarnya dan peta Anda bertambah di setiap pass.

### Contoh terperinci

```bash
strings -n 6 chall | grep -i wrong
#   1f58 wrong password
```

Buka `chall` di Ghidra, cari `wrong password` di memori, daftarkan referensinya, dan Anda akan mendarat di rutin perbandingan. Bacanya: sebuah argumen datang dari `main`, dibandingkan dengan nilai yang diharapkan, dan cabang kegagalan mencetak string itu. Beri nama `check_password` dan lanjutkan dari sana.

## Pengaturan yang aman

Alat statis hanya membaca file — mereka tidak mengeksekusinya. Meski begitu, perlakukan binary yang tidak dikenal sebagai tidak tepercaya: analisis di VM atau lingkungan sekali pakai, dan jangan pernah klik dua kali sebuah sampel untuk "melihat apa yang terjadi". Kecerobohan, bukan disassembler-nya, yang biasanya menjadi penyebab kecelakaan.

## Keterbatasan

Analisis statis tidak bisa melihat data yang hanya ada saat runtime — section yang dipacked, string yang didekripsi, atau nilai yang diisi setelah eksekusi dimulai. Ketika kode menyembunyikan data di balik loop dekripsi, beralih ke [Analisis Dinamis](dynamic-analysis.md) dan amatilah saat kode berjalan.

## Petunjuk tools

Ghidra (gratis), radare2/rizin, dan Cutter semuanya bisa mengerjakan pekerjaan ini — lihat [Alat Reversing](../practice/reversing-tools.md) untuk perbandingannya, dan [Decompiler & Ghidra](decompilers.md) untuk sisi pseudocode.

## Pengingat penggunaan yang sah

Semua yang ada di sini hanya berlaku untuk binary yang Anda miliki atau berhak analisis: latihan lab, crackme, challenge CTF, dan build Anda sendiri. Memetakan program proprietary orang lain tanpa izin tidak dicakup materi ini.

## Lihat juga

- [Apa Itu Reverse Engineering](../fundamentals/what-is-reversing.md)
- [Analisis Dinamis](dynamic-analysis.md)
- [Decompiler & Ghidra](decompilers.md)
- [Alat Reversing](../practice/reversing-tools.md)
- [Alur Kerja Reversing](../practice/workflow.md)
