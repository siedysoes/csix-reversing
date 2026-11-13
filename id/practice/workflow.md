# Alur Kerja Reverse

Membobol binary secara acak menghasilkan hasil acak — Anda tidak bisa mereproduksi temuan yang kebetulan, dan Anda tidak pernah tahu apa yang terlewat. Alur kerja yang tetap membuat setiap analisis bisa diulang, bisa dibandingkan, dan bisa diajarkan. Halaman ini menguraikan proses end-to-end dari perintah `file` pertama hingga catatan yang sudah rampung.

> Alur kerja ini untuk binary yang Anda miliki, yang diberikan kepada Anda, atau yang Anda beri izin untuk dianalisis — challenge yang sah dan build Anda sendiri. Tidak ada di sini yang menjadi lisensi untuk me-reverse software yang tidak Anda hakiki.

## Mengapa Metode Mengalahkan Membuka Disassembler Sembarangan

- **Bisa direproduksi:** langkah yang sama menghasilkan temuan yang sama, sehingga Anda bisa mencoba ulang analisis yang gagal secara sistematis.
- **Mengajarkan apa yang Anda lewatkan:** urutan yang terdokumentasi membuat celah terlihat jelas — Anda bisa melihat persis langkah mana yang Anda lewati.
- **Triage cepat di depan:** sebagian besar waktu terbuang membaca fungsi yang salah; langkah-langkah awal sengaja menyempitkan pencarian sebelum Anda membaca kode apa pun.

## Alur Kerja End-to-End

```text
1. Triage   — file, checksec, strings
2. Map      — sections, imports, entry point
3. Anchor   — find the interesting string/function via xref
4. Read     — decompile the target function
5. Hypothesize — what does it check, and how?
6. Verify   — test your understanding (static, or dynamic in VM if authorized)
7. Document — commands, findings, screenshots
```

Jalankan langkah-langkahnya secara berurutan. Disiplinnya terletak pada tidak melompat ke langkah 4 sebelum triage memberi tahu di mana Anda harus melihat.

### Langkah 1: Triage

Tiga perintah menjawab "apa yang saya hadapi?":

```bash
file target            # format, architecture, stripped?
checksec --file=target # protections — verify flags for your version
strings -n 6 target | less   # readable hints: prompts, paths, errors
```

Triage memberi tahu Anda apakah binary itu packed, stripped, terlindungi, atau penuh string yang mudah diambil sebelum Anda menginvestasikan waktu pada disassembler.

### Langkah 2: Map

Buka binary di Ghidra (atau disassembler pilihan Anda) dan bangun peta mental: section (`.text`, `.data`, `.rodata`), import (fungsi eksternal apa yang dipanggilnya?), dan entry point. Import adalah cheat sheet — pemanggilan ke `strcmp` atau `strlen` sering menunjuk lurus ke logika yang menarik.

### Langkah 3: Anchor

Jangan pernah membaca binary secara linear. Temukan satu anchor — string menarik dari triage, import yang mencurigakan, nama fungsi — lalu ikuti cross-reference (xref) ke kode yang menggunakannya. Satu anchor yang baik meruntuhkan binary 10.000 baris menjadi satu fungsi.

### Langkah 4: Read

Decompile fungsi yang ter-anchor dan bacalah seolah-olah itu kode sumber. Fokus pada aluran kontrol: apa yang dibandingkan, apa yang terjadi saat berhasil, apa yang terjadi saat gagal. Abaikan noise hasil kompilasi pada putaran pertama.

### Langkah 5: Hypothesize

Sampaikan pemahaman Anda secara eksplisit: "Fungsi ini membandingkan input terhadap nilai tersimpan dan mencetak 'Access granted' hanya jika cocok." Menuliskan hipotesis mengubah intuisi samar menjadi sesuatu yang bisa Anda bantah.

### Langkah 6: Verify

Uji hipotesisnya: telusuri logika secara statis melalui pemanggil dan cabang, atau — jika Anda berwenang menjalankan binary dan berada di VM Anda — konfirmasikan secara dinamis dengan `gdb` atau `strace`. Hipotesis yang belum diuji hanyalah tebakan.

### Langkah 7: Document

Rekam perintah yang Anda jalankan, anchor yang Anda ikuti, logika hasil dekompilasi (dengan screenshot jika berguna), dan kesimpulannya. Diri Anda di masa depan dan write-up Anda keduanya berawal dari file ini.

## Tabel Keputusan Triage

| Temuan | Arah |
| --- | --- |
| String menarik ("wrong password") | Temukan xref ke string itu — pemeriksaannya ada di dekatnya |
| Import: `strcmp` / `strncmp` | Kemungkinan pemeriksaan password/kunci — baca fungsi pemanggilnya |
| Import: `fopen` / `open` | Ia membaca file — telusuri input apa yang ia harapkan |
| Tidak ada string sama sekali, section aneh | Kemungkinan packed → lihat [Anti-Debug & Packing](../techniques/anti-debug.md) |
| Tanda anti-debug (`int3`, `IsDebuggerPresent`) | Pemeriksaan timing/debug — rencanakan countermeasure analisis dinamis |
| Binary stripped, kecil | Mulai dari entry point dan ikuti pemanggilan ke luar |

## Aturan "Satu Pertanyaan pada Satu Waktu"

Setiap sesi diawali dengan menulis satu pertanyaan: **apa yang ingin saya pelajari?** — "Apakah binary ini memeriksa password?" mengalahkan "ayo lihat apa isinya." Setelah pertanyaan terjawab, tulis pertanyaan berikutnya atau akhiri sesi. Satu pertanyaan menjaga sesi tetap fokus dan bisa diselesaikan.

## Timeboxing

| Situasi | Tindakan |
| --- | --- |
| Menempel di titik yang sama 20+ menit tanpa informasi baru | Ganti pendekatan (statis ↔ dinamis) atau lakukan triage ulang |
| Maju stabil pada satu fungsi | Terus sampai selesai membacanya sebelum berpindah |
| Lelah, membaca ulang baris yang sama | Tidur — mata segar menemukan anchor lebih cepat |
| Target jelas melampaui keterampilan Anda saat ini | Sampingkan, pilih yang lebih mudah, kembali lagi nanti |

Stop-loss mencegah mode kegagalan klasik: menghabiskan satu malam penuh di dalam satu fungsi yang salah.

## Jalan Buntu Umum

| Jalan buntu | Koreksi |
| --- | --- |
| Terjebak lubang kelinci di fungsi yang tidak berhubungan | Periksa lagi anchor Anda — apakah fungsi ini benar-benar tercapai dari jalur input? |
| Terlalu banyak membaca noise hasil kompilasi (penyiapan stack, penyalinan) | Baca aluran kontrol dulu; abaikan mekanisme pada putaran pertama |
| Mengabaikan jalur input binary yang sebenarnya | Telusuri mundur dari cara input masuk (`argv`, `stdin`, file) — pemeriksaannya ada di jalur itu |
| Berdebat dengan assembly yang dioptimasi | Beralih ke tampilan decompiler, atau build ulang binary uji Anda sendiri tanpa optimasi |

## Dokumentasikan Saat Melangkah

Biarkan scratch file tetap terbuka sepanjang sesi. Format minimal:

```markdown
# target: login-check (authorized crackme)
date: 2026-01-10
question: what does the check compare against?

## triage
- file: ELF 64-bit, stripped, not packed (checksec: NX, PIE, canary)
- strings: "Enter key:", "Access granted" at ...

## findings
- xref from "Access granted" -> sub_401234
- decompile: strcmp(input, s) at ...

## verdict / next question
- hypothesis confirmed statically; dynamic test pending in VM
```

## Checklist Kualitas untuk Analisis yang Rampung

- [ ] Pertanyaan awal terjawab dengan bukti, bukan tebakan.
- [ ] Temuan kunci ditelusuri dari input hingga titik keputusan.
- [ ] Perintah dan versi yang dibutuhkan untuk mereproduksi sudah direkam.
- [ ] Hipotesis sudah diverifikasi (penalaran statis, atau jalankan secara dinamis di VM bagi yang berwenang).
- [ ] Apa yang masih belum diketahui didaftarkan secara jujur.

## Kapan Beralih ke Dinamis

Analisis statis menemui tembok ketika kode yang menarik dihitung saat runtime, diobfuscate, atau di-unpack on the fly. Pada titik itu, pindahlah ke VM yang siap snapshot dan debug proses yang sedang berjalan dengan `gdb` — lihat [Analisis Dinamis](../techniques/dynamic-analysis.md) untuk serah terimanya. Pekerjaan dinamis tetap hanya berlaku untuk binary yang Anda beri izin untuk dijalankan.

## Lihat juga

- [Tool Reverse Engineering](reversing-tools.md)
- [Latihan & Lab](labs.md)
- [Analisis Statis](../techniques/static-analysis.md)
- [Analisis Dinamis](../techniques/dynamic-analysis.md)
