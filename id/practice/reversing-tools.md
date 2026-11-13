# Tool Reverse Engineering

Toolkit reversing adalah kumpulan program untuk membaca, mendisassemble, mendekompilasi, dan mendebug binary. Tool itu sendiri netral — legalitasnya sepenuhnya tergantung pada target yang Anda tunjuk. Halaman ini mencantumkan tool inti per tugas, cara memasangnya dengan aman, serta tool mana yang menjawab pertanyaan mana.

> **Aturan emas:** sebuah tool hanya selegal target yang Anda tunjuk. Gunakan setiap tool hanya pada binary yang Anda miliki, yang diberikan kepada Anda, atau yang secara eksplisit Anda beri izin untuk dianalisis — jangan pernah untuk melewati lisensi check atau proteksi pada software yang bukan milik Anda.

## Tool per Tugas

| Tugas | Tool | Catatan |
| --- | --- | --- |
| Identifikasi file | `file`, `checksec` | `file` menampilkan tipe/format; `checksec` menampilkan proteksi |
| Ekstrak string | `strings`, `grep` | Cari dulu petunjuk yang terbaca: path, prompt, teks error |
| Disassemble | `objdump`, Ghidra, rizin | Disassembly linear vs. rekursif; Ghidra dan rizin menambah navigasi |
| Dekompilasi | Ghidra (gratis), IDA (batas versi gratis — verifikasi terkini), Cutter | Ghidra gratis dan open source; Cutter berbasis rizin |
| Debug | `gdb` (Linux), x64dbg (Windows) | Pasang breakpoint, step, inspeksi register dan memori |
| Telusuri syscall | `strace` / `ltrace` | Lihat apa yang sebenarnya dilakukan program saat runtime (syscall / pemanggilan library) |
| Tampilan hex | `xxd`, hex editor | Baca byte mentah, header, region terenkripsi atau terpacking |
| Unpack | CLI UPX (pada file yang boleh Anda analisis — verifikasi flag) | Packer umum; jalankan hanya pada binary yang memang boleh Anda periksa |

Verifikasi harga dan akses terkini untuk versi gratis IDA serta platform hosting mana pun — penawaran bisa berubah seiring waktu.

## Setup Ghidra (Catatan Singkat)

Ghidra adalah suite reverse engineering gratis yang dirilis oleh NSA, tersedia dari situs resmi Ghidra — verifikasi rilis terkini di sana sebelum memasang.

- Membutuhkan Java Development Kit (JDK); periksa release notes untuk versi yang didukung secara persis.
- Unduh dari situs resmi atau repositori GitHub resmi, bukan mirror pihak ketiga.
- Buat project, impor binary yang Anda beri izin untuk dianalisis, dan biarkan Ghidra menjalankan auto-analysis-nya.
- Mulai dari Listing view (disassembly) dan jendela Decompiler berdampingan.

## Ekosistem gdb

`gdb` polos sudah langsung berfungsi di sebagian besar sistem Linux, tetapi front-end plugin membuatnya jauh lebih nyaman dipakai:

- **GEF** dan **pwndbg** menambah pane register/memori, tampilan heap, dan breakpoint yang lebih enak — verifikasi instruksi pemasangan terkini di repositori resmi masing-masing.
- Alur kerja yang umum: jalankan binary di bawah gdb, `break` pada fungsi yang menarik, `run` dengan input, lalu inspeksi register dan stack di setiap langkah.

## One-Liner checksec

`checksec` (sering disertakan bersama plugin gdb atau tersedia sebagai tool mandiri) mencetak proteksi sebuah binary sekilas:

```bash
checksec --file=./target      # typical usage — verify flags for your version
```

Apa yang dilaporkan dan mengapa itu penting:

| Proteksi | Artinya |
| --- | --- |
| NX (no-execute) | Stack/memori tidak bisa mengeksekusi shellcode secara langsung |
| Canary | Nilai deteksi stack-smash sebelum alamat return |
| ASLR | Tata letak memori diacak setiap kali dijalankan |
| PIE | Executable-nya sendiri dimuat di alamat acak |

Hal ini memengaruhi seberapa bisa dieksploitasi sebuah bug — dan seberapa sulit analisis dinamis — jadi periksa dulu sebelum Anda menyusun pendekatan.

## Lingkungan Riset

| Praktik | Alasan |
| --- | --- |
| VM khusus untuk reversing | Malware dan binary tak dikenal tidak pernah menyentuh OS host Anda |
| Ambil snapshot sebelum menjalankan apa pun | Kembalikan ke keadaan bersih dalam hitungan detik setelah crash atau infeksi |
| Tanpa shared folder berisi data nyata | Binary yang bermusuhan tidak boleh sampai ke file pribadi Anda |
| Matikan jaringan untuk crackme | Crackme seharusnya tidak membutuhkan jaringan; isolasi menghilangkan risiko exfiltrasi |

Jalankan hanya binary yang Anda beri izin untuk dianalisis, dan hanya di dalam lingkungan terisolasi ini.

## Kebersihan Tool

- Pasang dari repositori resmi atau situs proyek resmi — jangan pernah dari mirror unduhan sembarangan.
- Verifikasi checksum atau signature di mana proyek memublikasikannya (verifikasi instruksi terkini per tool).
- Jaga tool tetap diperbarui, tetapi baca release notes: pembaruan disassembler dan decompiler bisa mengubah keluaran analisis.
- Utamakan instalasi lewat package manager di Linux agar pembaruan keamanan mengalir otomatis.

## Kebiasaan Otomasi

Parsing yang berulang (memindai ribuan string, menelusuri struktur, mengekstrak field) sebaiknya dibuatkan skrip, bukan dikerjakan manual. Python adalah pilihan bawaan di bidang ini, dan **pwntools** adalah library bantu standar dalam konteks CTF (penanganan konteks, inspeksi ELF, pembantu ROP). Pelajari dulu pada challenge yang sah dan binary Anda sendiri.

## Yang TIDAK Boleh Dilakukan

- Jangan gunakan tool ini untuk melewati lisensi check, DRM, atau aktivasi pada software yang bukan milik Anda.
- Jangan menganalisis produk proprietary sebuah perusahaan "untuk latihan" tanpa izin.
- Jangan mengunduh dan menjalankan binary tak dikenal di luar VM — dan jangan menganalisis apa pun yang tidak Anda dapatkan izinnya untuk disentuh.

Latihlah hanya pada challenge yang sah dan binary milik Anda sendiri — aturan hukum dan etika berlaku untuk setiap tool di halaman ini.

## Tool Mana untuk Pertanyaan Mana

| Pertanyaan Anda | Gunakan |
| --- | --- |
| "File seperti apa ini?" | `file` |
| "Proteksi apa yang dimilikinya?" | `checksec` |
| "Teks terbaca apa yang ada di dalamnya?" | `strings` + `grep` |
| "String ini direferensikan di mana?" | Cross-reference (xref) Ghidra / rizin |
| "Fungsi ini melakukan apa?" | Decompiler Ghidra |
| "Apa yang dilakukannya saat saya menjalankannya?" | `strace` / `ltrace`, lalu `gdb` |
| "Byte mentah ini berarti apa?" | `xxd` atau hex editor |
| "Apakah ini packed?" | `file`, cek entropi tinggi, string yang menunjukkan tanda tangan packer |

## Lihat juga

- [Analisis Statis](../techniques/static-analysis.md)
- [Decompiler & Ghidra](../techniques/decompilers.md)
- [Hukum & Etika](../fundamentals/legal-ethics.md)
- [Latihan & Lab](labs.md)
