# Binaries Windows vs Linux

Konsep reversing yang sama berlaku di mana pun, tetapi format file, tata letak header, dan calling convention berbeda antara Linux dan Windows. Mengetahui convention mana yang sedang Anda lihat mencegah berjam-jam argumen yang salah baca dan pilihan tool yang keliru. Artikel ini membandingkan kedua platform terhadap binary yang Anda berhak analisis.

## Mengapa perbedaan ini penting

Register yang menyimpan argumen pertama di Linux bukan argumen pertama di Windows; sebuah pemindaian `strings` yang menemukan segalanya di ELF bisa jadi nyaris tidak menemukan apa pun di PE. Pelajari perbedaannya sekali dan alur kerja Anda akan terbawa hampir tanpa perubahan.

## Format file

| Format | Rumah yang umum | Isinya (disederhanakan) |
| --- | --- | --- |
| ELF | Linux, BSD, sebagian besar sistem Unix | Program header, section, symbol table, info dynamic linking |
| PE | Windows (`.exe`, `.dll`) | DOS/NT header, section, tabel import/export, resource |
| Mach-O | macOS (dan iOS) | Load command, segment/section, info symbol dan relocasi |

Satu format untuk setiap platform — mengidentifikasinya adalah hal pertama yang `file` lakukan untuk Anda.

## Inti struktur PE

File PE dibagi menjadi section dengan peran yang tetap:

| Section | Peran |
| --- | --- |
| `.text` | Kode yang dapat dieksekusi |
| `.data` | Data baca/tulis yang terinisialisasi (global) |
| `.rdata` | Data baca-saja: konstanta, nama import, string |
| `.rsrc` | Resource: ikon, dialog, info versi — terkadang data tersembunyi |

ELF punya ide yang sama (`.text`, `.data`, `.rodata`) dengan nama yang sedikit berbeda — begitu Anda tahu satu tata letak, yang lain terasa familier.

## Header: tempat entry point berada

| | ELF | PE |
| --- | --- | --- |
| Membacanya dengan | `readelf -h chall` | Ghidra atau PE viewer (verifikasi tools terkini) |
| Field entry point | `Entry point address` di ELF header | `AddressOfEntryPoint` di optional header |
| Detail tambahan | Program header menjelaskan bagaimana segment dimuat | Data directory menunjuk ke import, resource, dan lainnya |

`readelf -h` adalah dump satu baris ELF header yang layak dihafal; untuk PE, inspector yang lumayan (termasuk Ghidra) menampilkan field yang setara.

## Import: bagaimana fungsi eksternal diresolusi

Kedua format membutuhkan fungsi library yang tidak mereka-link secara statis, tetapi mekanismenya berbeda.

### PLT/GOT (ELF)

Setiap pemanggilan import melompat melalui sebuah stub di `.plt` yang menyelesaikan alamat yang sebenarnya pada penggunaan pertama dan menyimpannya di global offset table (`.got`). Pemanggilan berikutnya langsung menuju entri yang tersimpan di cache.

### Import Address Table (PE)

Loader mengisi sebuah tabel alamat — IAT — saat waktu pemuatan, dan pemanggilan melewatinya. Masalah yang sama, plumbing yang berbeda: "memanggil fungsi yang bukan milik kita". Kedua mekanisme muncul sebagai daftar nama import yang terbaca di awal analisis Anda (lihat [Analisis Statis](static-analysis.md)).

## Calling convention

| Convention | Register argumen | Catatan |
| --- | --- | --- |
| SysV AMD64 (Linux) | `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9` | Nilai kembalian di `rax`; enam argumen integer pertama di register |
| Windows x64 | `rcx`, `rdx`, `r8`, `r9` | Empat argumen integer pertama di register; pemanggil menyisihkan 32 byte shadow space untuk callee |

Ini adalah perbedaan paling berpengaruh saat membaca disassembly: jenis pemanggilan yang sama, dua tempat berbeda untuk mencari argumennya. Cukup hafalkan daftar register-nya — tidak perlu diagram stack yang dalam di tingkat ini.

## Tooling

| Platform | Tools yang umum |
| --- | --- |
| Linux | `gdb`, `strace`, `readelf`, `objdump`, Ghidra |
| Windows | x64dbg, Process Monitor, Ghidra (verifikasi versi dan lisensi terkini) |

Ghidra muncul di kedua sisi secara sengaja — ia membaca ELF, PE, dan Mach-O sama baiknya.

## Strings: perbedaan encoding

Teks Linux biasanya ASCII/UTF-8 dan ditemukan oleh `strings` biasa. Teks Windows sering UTF-16 (wide strings), yang membutuhkan sebuah flag encoding (verifikasi opsi build Anda):

```bash
strings -el application.exe | grep -i password
```

Bila sebuah binary Windows terlihat kosong string-nya, coba `-el` sebelum menyimpulkan bahwa ada yang disembunyikan.

## Perbedaan runtime

Program Windows memuat DLL; program Linux memuat shared object `.so` — konsep yang sama, nama, ekstensi, dan convention linking yang berbeda. Satu baris untuk diingat: **DLL di Windows, `.so` di Linux, keduanya adalah "kode yang disediakan orang lain saat runtime"**.

## Catatan lintas platform

Ghidra membaca ketiga format tersebut, jadi alur kerja inti Anda nyaris tidak berubah: import, analisis, temukan `main` atau `WinMain`, ikuti fungsi-fungsinya. Yang berubah adalah convention register, perintah header, dan debugger mana yang Anda raih.

## Saran praktis

Pelajari Linux dulu bila Anda di sini untuk CTF: sebagian besar crackme adalah ELF, dan `gdb` + `strace` + `readelf` mencakup semuanya. Tambahkan pengetahuan Windows saat Anda menghadapi software dunia nyata — struktur PE, x64dbg, dan wide strings adalah tiga hal yang paling sering terbawa.

## Pengingat penggunaan yang sah

Terapkan semua ini hanya pada binary yang Anda miliki atau berhak analisis — latihan lab, crackme, challenge CTF, dan build Anda sendiri, di platform mana pun.

## Lihat juga

- [Cara Binaries Bekerja](../fundamentals/how-binaries-work.md)
- [Dasar Assembly](../fundamentals/assembly-basics.md)
- [String & Pola](strings-patterns.md)
- [Alat Reversing](../practice/reversing-tools.md)
- [Alur Kerja Reversing](../practice/workflow.md)
