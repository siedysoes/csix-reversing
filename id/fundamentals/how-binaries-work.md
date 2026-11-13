# Cara Binaries Bekerja

Binary adalah bentuk terkompilasi dari sebuah program: instruksi dan data yang dieksekusi langsung oleh CPU. Memahami bagaimana binary dibangun menjelaskan kenapa ia bisa dibongkar — setiap byte punya asal, tujuan, dan tempat. Artikel ini menelusuri jalur dari source code ke executable yang berjalan, dan memperkenalkan struktur yang akan Anda periksa di latihan-latihan berikutnya.

## Dari Source ke Executable

```text
  hello.c                          (human-readable source)
      │  compiler (gcc, clang, cl)
      ▼
  hello.o                           (machine code for this file,
      │                              with unresolved references)
      │  linker (+ libraries such as libc)
      ▼
  hello                             (final executable: ELF, PE, or Mach-O)
```

Compiler menerjemahkan setiap source file secara independen menjadi object file yang berisi machine code dan data plus reference yang belum diselesaikan ke hal-hal yang didefinisikan di tempat lain. Linker menggabungkan object-file tersebut dan kode library yang dibutuhkan menjadi satu executable lalu menyelesaikan referensi-referensi itu. Yang jatuh ke disk adalah sebuah container format yang memuat kode, data, dan catatan administrasi yang dibutuhkan sistem operasi untuk menjalankannya.

## Kenapa Reversing Bisa Dilakukan

Reversing bisa bekerja karena satu fakta sederhana: CPU adalah satu-satunya audiens yang dituju oleh sebuah program yang dirilis, dan CPU bukan alat penyimpan rahasia. CPU mengeksekusi encoding instruksi yang terdokumentasi dan tetap, sehingga setiap byte yang akan dieksekusi program harus bisa didecode oleh sesuatu — dan sesuatu itu adalah kita dengan disassembler. Kompilasi mengubah bentuk kode (nama menghilang, loop di-unroll, function di-inline), tetapi tidak pernah mengenkripsinya; obfuscation dan packing menaikkan upaya yang diperlukan tanpa mengubah aturan dasarnya.

## Format File

| Format | Dipakai di | Ekstensi khas | Yang dibawanya |
|---|---|---|---|
| ELF | Linux dan sebagian besar sistem Unix-like | tidak wajib (misalnya `bash`) | Kode, data, program dan section header, tabel import/export dinamis |
| Mach-O | macOS dan iOS | tidak wajib | Isi dasar yang sama dengan tata letak Apple, dideskripsikan oleh load command |
| PE | Windows | `.exe`, `.dll` | Kode, data, import table yang menamai fungsi-fungsi DLL yang dipakai, dan header yang memuat entry point |

Ketiganya menyelesaikan masalah yang sama: memberi tahu loader di mana kode dan data berada, izin apa yang dibutuhkan tiap region, dan di mana eksekusi dimulai.

## Di Dalam ELF

File ELF adalah deretan header yang diikuti data. **ELF header** di byte nol mengidentifikasi file (magic bytes `7f 45 4c 46`, yaitu `\x7fELF`), kelasnya (32 atau 64 bit), mesin targetnya, dan alamat entry point-nya. **Program header** mendeskripsikan segment, yaitu yang menjadi perhatian loader; **section header** mendeskripsikan section, yaitu yang menjadi perhatian tool analisis dan linker.

### Sections

| Section | Isi |
|---|---|
| `.text` | Machine code terkompilasi (executable) |
| `.data` | Variabel global dan static yang terinisialisasi (read/write) |
| `.bss` | Global yang belum terinisialisasi — ruang dialokasikan dan diisi nol di awal, tanpa memakai ruang file |
| `.rodata` | Data read-only: literal string, konstanta, jump table |
| `.plt` / `.got` | Stub dan alamat yang dipanggil untuk memanggil function yang hidup di shared library |
| `.symtab` / `.dynsym` | Symbol table: nama dan alamat function serta variabel, jika tidak stripped |

### Entry point

Entry point adalah alamat yang di-jump ke oleh prosesor saat program dimulai. Biasanya bukan `main`: sebuah startup stub kecil (umumnya `_start` di ELF) menjalankan inisialisasi C runtime, menyiapkan `argc` dan `argv`, dan baru setelah itu memanggil `main` Anda.

## Apa yang Dilakukan Loader saat Runtime

Sebelum satu instruksi pun dari kode Anda berjalan, loader sistem operasi:

1. Membaca header dan memetakan tiap segment yang bisa dimuat ke dalam memori dengan izin yang tepat (kode read/execute, data read/write).
2. Menerapkan relocations, memperbaiki alamat yang belum diketahui saat link time.
3. Memuat shared library yang dibutuhkan dan menyelesaikan symbol yang di-import — inilah dynamic linker: `ld.so` di Linux, `dyld` di macOS, dan loader Windows untuk file PE.
4. Jump ke entry point.

## Import dan Dynamic Linking

Saat sebuah program memanggil `printf`, binary tidak berisi `printf`; binary berisi *reference* ke `printf` yang dicatat di dynamic symbol table, dan diisi oleh loader saat runtime. Reference itu adalah peta Anda: daftar import memberi tahu layanan library mana yang dipakai program — file I/O, panggilan jaringan, primitive kripto, fungsi UI — dan sering mengungkap apa yang dilakukan sebuah binary bahkan sebelum Anda membaca satu instruksi pun.

**Static vs dynamic linking, satu baris:** static linking menyalin kode library ke executable saat build time; dynamic linking menyimpan reference yang diselesaikan saat program dimuat.

## Symbol dan Debug Information

| | Binary unstripped | Binary stripped |
|---|---|---|
| Nama function dan variabel | Ada di symbol table | Dihapus |
| Alamat kode yang bisa dipulihkan | Ya | Ya — instruksinya tidak berubah |
| Debug info (nomor baris, tipe) | Ada jika dibangun dengan debug flag | Tidak ada |
| Situasi tipikal | Build Anda sendiri, paket debug | Software produksi yang terpasang |

Stripping menghapus nama, bukan perilaku. Mereverse binary stripped hanya lebih lambat, karena Anda harus menamai semuanya sendiri.

## Contoh Nyata yang Read-Only

Diberikan `chall`, challenge binary yang diberikan kepada Anda untuk diselesaikan:

```bash
file chall
readelf -h chall
readelf -S chall
```

`file` mengidentifikasi format dan arsitekturnya. `readelf -h` mencetak ELF header; `readelf -S` mendaftar section header. Contoh output, dipotong — nilai Anda akan berbeda:

```text
$ file chall
chall: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked,
      interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux, not stripped

$ readelf -h chall
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Type:                              DYN (Position-Independent Executable)
  Machine:                           Advanced Micro Devices X86-64
  Entry point address:               0x10a0
```

Ketiga perintah itu hanya membaca file: tidak ada yang dieksekusi, tidak ada yang diubah. Pembatasan yang sama berlaku untuk setiap contoh dalam panduan ini — bekerja hanya pada binary yang Anda miliki atau berotorisasi untuk dianalisis. Untuk file PE (Windows), `file` tetap bisa mengidentifikasinya sementara tool pembaca header-nya berbeda; hal itu dibahas di [Windows vs Linux Binaries](../techniques/windows-vs-linux.md).

## Lihat juga

- [Apa Itu Reverse Engineering](what-is-reversing.md)
- [Dasar Assembly](assembly-basics.md)
- [Strings & Patterns](../techniques/strings-patterns.md)
- [Windows vs Linux Binaries](../techniques/windows-vs-linux.md)
- [Reversing Tools](../practice/reversing-tools.md)
