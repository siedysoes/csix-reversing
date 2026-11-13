# Decompiler & Ghidra

Sebuah decompiler mengubah kode mesin kembali menjadi pseudocode mirip C, sehingga Anda membaca `if (strcmp(a, b))` alih-alih tumpukan conditional jump. Ini adalah sebuah aproksimasi — tidak pernah source asli — tetapi inilah percepatan terbesar yang tersedia bagi reverser tingkat pemula sampai menengah. Artikel ini membahas cara membaca output decompiler secara jujur dan memakai Ghidra pada binary yang Anda berhak analisis.

## Apa yang dilakukan decompiler

Disassembler menampilkan instruksi; decompiler membangun ulang struktur di atasnya: pernyataan, loop, kondisi, pemanggilan fungsi, dan variabel. Outputnya terbaca tetapi lossy — nama sudah hilang, tipe ditebak, dan sebagian struktur disimpulkan, bukan dibuktikan. Perlakukan sebagai petunjuk kuat tentang kode aslinya.

## Mengapa ini mempercepat segalanya

Control flow assembly adalah graf lompatan; pseudocode memulihkan bentuk yang ditulis manusia. Anda memindai perbandingan, loop, dan cabang yang mencetak keberhasilan dengan kecepatan C, bukan dengan kecepatan opcode.

## Kejujuran pseudocode: yang Anda lihat vs kenyataan

| Yang ditampilkan pseudocode | Yang sebenarnya benar |
| --- | --- |
| Blok `if`/`else` yang rapi | Compiler bisa menggabungkan, mengurutkan, atau menduplikasi cabang |
| Nama variabel seperti `local_8h` | Nama asli sudah hilang — beri nama baru sendiri |
| Tipe yang jelas | Tipe ditebak dari penggunaan dan bisa saja salah |
| Akses field struct | Terkadang offset byte mentah yang di-pattern-match oleh decompiler |
| Satu fungsi yang rapi | Optimasi mungkin sudah meng-inline atau meng-constant-fold sebagian isinya |

Baca pseudocode sebagai sebuah hipotesis, dan konfirmasikan apa pun yang kritis terhadap disassembly.

## Dasar Ghidra untuk CTF

Jalankan Ghidra dari distribusinya (script wrapper-nya bernama `ghidraRun`):

```bash
ghidraRun
```

### Sesi pertama, langkah demi langkah

```
File > New Project -> choose a directory -> Non-Shared Project
File > Import File -> select chall -> OK (accept auto-analysis)
Window > Decompiler (the Pseudocode view in the Code Browser)
Search > For Functions -> main        (or use the Symbol Tree)
read the pseudocode for main
```

Auto-analysis mengerjakan bagian beratnya pada pass pertama; terima nilai default-nya sampai Anda tahu mengapa Anda ingin mengubahnya.

## Tips alur kerja Ghidra

- Cari sebuah string, klik kanan, dan daftarkan referensinya — itu menuntun Anda ke kode yang memakainya.
- Beri nama secara konsisten: fungsi (`check_password`), variabel (`given_pin`), label data. Rename adalah lapisan anotasi Anda.
- Perbaiki tipe data yang jelas salah: klik kanan sebuah variabel atau field dan tetapkan tipenya; tipe yang benar meningkatkan pseudocode sekaligus penalaran Anda.
- Biarkan tampilan disassembly terbuka di samping pseudocode untuk memverifikasi hal-hal yang mencurigakan.

### Rename dalam praktik

Fungsi yang sama sebelum dan sesudah tiga puluh detik anotasi:

```c
// fresh decompile
undefined8 FUN_004011a0(char *param_1) {
  int iVar1;
  iVar1 = strcmp(param_1, "s3cret");
  if (iVar1 == 0) {
    puts("Access granted");
  }
  // ...
}

// after renaming
int check_password(char *given) {
  int result;
  result = strcmp(given, "s3cret");
  if (result == 0) {
    puts("Access granted");
  }
  // ...
}
```

Tidak ada yang berubah pada logikanya — tetapi versi kedua terbaca seperti kode yang bisa Anda penalaran. Beri nama sedini mungkin dan sesering mungkin.

## Decompiler lainnya

| Tool | Biaya / model | Catatan |
| --- | --- | --- |
| IDA | Versi gratis tersedia; batasannya bervariasi per versi (verifikasi terkini) | Standar lama; edisi komersial tersedia |
| rizin / Cutter | Gratis dan open source | rizin adalah fork radare2; Cutter adalah GUI-nya |
| Binary Ninja | Berbayar (verifikasi edisi dan harga terkini) | Intermediate representation yang kuat; opsi cloud/log-in |

## Pola pseudocode yang umum

| Anda melihat | Biasanya berarti |
| --- | --- |
| `strcmp(a, b)` | Perbandingan password atau key |
| `atoi` / `strtol` | Input numerik yang sedang dikonversi |
| `strlen(s)` | Pengecekan panjang atau penjaga bounds |
| Loop di atas sebuah buffer | Parsing, decoding, atau transformasi input |

Mengenali salah satunya dalam lima detik menghemat ber-menit waktu scrolling.

## Batasan

Kode yang sangat dioptimasi atau diobfuskasi tetap sulit seberapa pun bagusnya decompiler: junk code, opaque predicates, dan inlining agresif mengaburkan segalanya. Ketika pseudocode terlihat aneh, kembali ke disassembly dan dasar-dasarnya — mesin selalu punya kata terakhir.

### Ketika pseudocode tidak setuju

Pseudocode dan disassembly bisa tampak tidak sepakat — sebuah cabang yang hilang, sebuah kondisi yang mustahil, sebuah variabel yang sepertinya menyimpan dua nilai sekaligus. Disassembly adalah otoritas: itulah yang dieksekusi CPU. Gunakan ketidaksetujuan itu sebagai sinyal untuk melambat, membaca instruksinya, dan memeriksa asumsi Anda tentang tipe dan calling convention. Dan ingat bahwa decompiler tidak pernah menjalankan kode, jadi data yang hanya ada di runtime tetap membutuhkan [Analisis Dinamis](dynamic-analysis.md).

## Pengingat penggunaan yang sah

Gunakan decompiler hanya pada binary yang Anda miliki atau berhak analisis — latihan lab, crackme, challenge CTF, dan build Anda sendiri. Mendekompilasi software proprietary yang tidak berhak Anda periksa berada di luar cakupan materi ini.

## Lihat juga

- [Analisis Statis](static-analysis.md)
- [Dasar Assembly](../fundamentals/assembly-basics.md)
- [Analisis Dinamis](dynamic-analysis.md)
- [Alat Reversing](../practice/reversing-tools.md)
