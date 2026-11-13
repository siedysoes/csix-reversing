# Dasar Assembly

Assembly language adalah bentuk yang bisa dibaca manusia dari instruksi machine yang dieksekusi CPU. Assembly berada di pusat reverse engineering karena compiler yang menghasilkannya dan decompiler hanya mendekripsi kode di atasnya secara perkiraan. Anda tidak perlu menghafal instruction set — Anda perlu mengenali pola.

## Kenapa Assembly Penting untuk Reversing

Source code memberi tahu maksud Anda; binary hanya berisi apa yang bertahan dari kompilasi. Ketika source tidak tersedia, assembly adalah ground truth terdekat yang Anda miliki: disassembler menunjukkan persis apa yang akan dilakukan CPU, sementara output mirip-C dari decompiler adalah perkiraan terdidik yang dibangun di atas instruksi-instruksi itu. Membaca assembly secukupnya untuk memeriksa hasil kerja decompiler adalah keterampilan inti dari reversing.

## Register (x86-64)

Register adalah slot penyimpanan kecil yang sangat cepat di dalam CPU — hampir setiap instruksi membaca atau menulis salah satunya.

| Register | Peran |
|---|---|
| `rax` | Accumulator: hasil aritmetika dan nilai return |
| `rbx` | General-purpose, callee-saved (sebuah function harus mempertahankannya) |
| `rcx` | Counter untuk loop dan operasi string |
| `rdx` | Register data: aritmetika lanjutan dan I/O |
| `rsi` | Source pointer untuk salinan; argumen ke-2 di Linux/macOS |
| `rdi` | Destination pointer untuk salinan; argumen ke-1 di Linux/macOS |
| `rsp` | Stack pointer: puncak stack saat ini |
| `rbp` | Frame base pointer: menjangkar stack frame dari function saat ini |
| `rip` | Instruction pointer: alamat instruksi berikutnya yang akan dieksekusi |

Penamaan mencerminkan lebar: `rax` 64-bit, `eax` 32 bit bawahnya, `ax` 16 bit, `al` 8 bit. Menulis ke register 32-bit seperti `eax` mengeset separuh atas `rax` menjadi nol — sumber kebingungan yang umum saat membaca kode yang sudah dioptimasi. Binary 32-bit memakai nama berawalan `e` di mana-mana: `eax`, `esi`, `edi`, `ebp`, `esp`.

## Kosakata Instruksi

Set instruksi yang ternyata kecil sudah menutupi sebagian besar yang akan Anda baca:

| Instruksi | Arti | Catatan |
|---|---|---|
| `mov dst, src` | Salin sebuah nilai | Jauh instruksi paling umum; ia menyalin, bukan mencetak |
| `lea dst, [addr]` | Hitung sebuah alamat | Hanya aritmetika alamat — tidak ada yang dibaca dari memori |
| `add` / `sub` | Tambah / kurang | Aritmetika terkait: `inc`, `dec`, `imul` |
| `cmp a, b` | Bandingkan | Menghitung `a - b` dan mencatatnya sebagai flag CPU; tidak ada yang lain berubah |
| `jmp`, `je`, `jne` | Jump, jump-if-equal, jump-if-not-equal | Jump kondisional membaca flag yang diset oleh `cmp` |
| `push` / `pop` | Tulis / baca puncak stack | `push` menaruh data ke stack, `pop` mengambilnya keluar |
| `call` / `ret` | Panggil sebuah function / kembali dari function | `call` mendorong alamat return; `ret` mem-pop-nya kembali |

## Perbandingan Menggerakkan Branch

`if` pada machine code adalah pasangan: satu instruksi men-set flag, instruksi lain branch berdasarkannya.

```asm
    cmp   al, 0x41        ; compare the value in al with 'A'
    jne   not_a           ; branch if the values differ
    ; fall through here only when al == 0x41 — this block is the "then"
not_a:
    ; this block is the "else"
```

`je` dan `jne` adalah pasangan sehari-hari; `jg`, `jl`, `jge`, `jle` mencakup perbandingan greater/less bersigned, dan `ja`, `jb` serta kawan-kawannya melakukan hal yang sama untuk nilai unsigned. Rangkaian perbandingan yang diikuti jump biasanya adalah deretan ujian `if` / `else if` — pengecekan password, parser perintah, dan state machine semuanya terlihat seperti ini.

## Stack

Stack adalah region memori last-in, first-out yang dipakai function untuk alamat return, register yang disimpan, dan variabel lokal. Nilai yang paling baru didorong selalu yang pertama di-pop, dan stack tumbuh ke arah bawah menuju alamat yang lebih rendah:

```text
  higher addresses
  ┌─────────────────┐
  │  previous frame │
  ├─────────────────┤
  │ return address  │  pushed by `call`
  ├─────────────────┤
  │ saved rbp       │
  ├─────────────────┤
  │ local variables │  <- rsp (top of the stack)
  └─────────────────┘
  lower addresses (stack grows down)
```

Saat `call func` dieksekusi, alamat instruksi berikutnya didorong agar `ret` bisa menemukan jalannya kembali. Alamat return itulah sebabnya memetakan call graph sebuah program sebagian besar adalah soal mengikuti push dan pop.

## Calling Convention

Calling convention adalah tata letak yang disepakati untuk meneruskan argumen. Di x86-64 System V (Linux, macOS, sebagian besar sistem Unix-like), argumen integer/pointer pertama masuk ke `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`, nilai return kembali dalam `rax`, dan argumen berikutnya bepergian lewat stack. Windows x64 memakai register berbeda dan menyediakan shadow space — lihat [Windows vs Linux Binaries](../techniques/windows-vs-linux.md).

## Endianness dalam Satu Baris

x86 adalah little-endian: nilai multi-byte disimpan dengan byte paling tidak signifikan lebih dulu, sehingga bilangan bulat `0x01020304` muncul di memori sebagai `04 03 02 01` — ingat ini setiap kali Anda membaca hex dump, immediate, atau alamat dari sebuah file.

## Cara Berlatih Membaca

Anda tidak perlu menghafal setiap instruksi. Di Ghidra dan sebagian besar decompiler lain, output mirip-C berdampingan dengan assembly-nya: baca dekompilasinya dulu, lalu spot-check terhadap instruksi di bawahnya. Carilah opcode yang tidak dikenal saat muncul, biarkan referensi tetap terbuka, dan fokuskan pada pola yang berulang — penyiapan argumen, perbandingan, branch, return — yang terulang di setiap binary yang akan Anda buka.

### Contoh minimal beranotasi

Function ini mengambil satu karakter dan mengembalikan 1 jika karakter itu sama dengan `'A'`, selain itu 0. Padanan C-nya adalah `int check(char c) { return c == 'A'; }`.

```asm
check:
    push  rbp                   ; save the caller's frame pointer
    mov   rbp, rsp              ; start a new stack frame
    mov   [rbp-0x4], edi        ; keep the argument (a char in edi) locally
    cmp   byte [rbp-0x4], 0x41  ; compare the input with 'A'
    jne   fail                  ; not equal -> return 0
    mov   eax, 1                ; equal -> return value 1
    jmp   done
fail:
    xor   eax, eax              ; zero eax: return value 0
done:
    pop   rbp                   ; restore the caller's frame pointer
    ret                         ; return to the caller
```

Seluruh function hanyalah satu perbandingan dan satu branch — pola dari sebelumnya, dalam konteksnya. Berlatihlah membaca pada binary yang Anda miliki atau berotorisasi untuk dianalisis: build Anda sendiri, binary lab yang disediakan, crackme, dan challenge CTF.

## Bacaan Lanjutan

Kurikulum bergaya "gerbang NAND ke sistem operasi" membangun model mental yang tepat dari hardware ke atas; untuk bahasanya sendiri, primer assembly mana pun yang banyak ulasannya sudah cukup — verifikasi edisi terkini sebelum membeli, dan utamakan materi yang cocok dengan arsitektur target Anda (x86-64 untuk sebagian besar jalur ini). Manual arsitektur dari Intel dan AMD bersifat otoritatif tapi berat; simpanlah untuk pencarian, bukan untuk belajar.

## Lihat juga

- [Cara Binaries Bekerja](how-binaries-work.md)
- [Decompilers & Ghidra](../techniques/decompilers.md)
- [Windows vs Linux Binaries](../techniques/windows-vs-linux.md)
- [Practice & Labs](../practice/labs.md)
- [The Reversing Workflow](../practice/workflow.md)
