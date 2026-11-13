# Hukum & Etika

Reverse engineering adalah keterampilan yang kuat, dan bersamanya datang sebuah kewajiban: teknik yang sama yang melindungi pengguna bisa dipakai melawan mereka. Artikel ini mendahului jalur fundamentals karena kesalahan teknis bisa diperbaiki, sementara menganalisis binary yang tidak berhak Anda sentuh tidak bisa dibatalkan. Perlakukan otorisasi sebagai bagian dari keterampilannya, bukan sekadar kertas di sekitarnya.

## Kenapa Ini Didahulukan

Tools dan teknik itu netral; yang memisahkan riset keamanan dari perbuatan buruk adalah otorisasi, scope, dan niat. Setiap perintah dan sesi disassembler di CSIX Reversing berikutnya bertumpu pada fondasi yang dibangun di artikel ini — jika fondasinya salah, apa pun yang dibangun di atasnya tidak bisa dibela.

Dua pertanyaan menyelesaikan sebagian besar kasus: apakah binary ini diberikan kepada Anda dengan harapan Anda akan melihatnya, dan apakah seseorang yang berwenang setuju? Jika kedua jawabannya ya, Anda bisa bekerja dengan itikad baik dan mendokumentasikannya. Jika salah satu jawabannya tidak jelas, menyisihkan sebentar untuk bertanya sekarang selalu lebih murah daripada menjelaskan nanti — Anda tidak bisa "membatalkan pembacaan" sebuah binary yang tidak berhak Anda buka.

## Prinsip Inti

Hak Anda menganalisis sebuah binary berasal dari tepat tiga sumber:

- **Kepemilikan.** Anda menulisnya, atau Anda memegang hak-hak yang melekat pada kepemilikan.
- **Izin.** Sebuah lisensi, deskripsi challenge, atau dokumen rules-of-engagement memberikan hak analisis.
- **Otorisasi.** Seseorang yang berwenang memberi akses secara eksplisit: pemberi kerja, klien dengan kontrak, atau scope bug-bounty.

Jika ketiganya tidak berlaku, jangan analisis binary-nya. Perhatikan bahwa memiliki *salinan* sebuah software belum otomatis berarti Anda berhak me-reverse-nya — di situ hukum-hukum di bawah berlaku.

## Posisi Berbagai Aktivitas

| Aktivitas | Status tipikal | Catatan |
|---|---|---|
| Reverse engineering program yang Anda tulis atau miliki | Umumnya OK | Binary Anda, analisis Anda |
| Menyelesaikan crackme dan challenge CTF yang diberikan kepada Anda | Umumnya OK | Analisis memang persis yang dimaksud pembuatnya |
| Menguji binary yang ditugaskan pemberi kerja atau klien | Umumnya OK dalam scope | Tetap di dalam scope tertulis dan rules of engagement |
| Pekerjaan interoperabilitas clean-room | Area abu-abu | Sering diizinkan untuk interoperabilitas, tapi bervariasi per yurisdiksi — verifikasi hukum setempat |
| Decompiling aplikroprietari melanggar ToS-nya | Area abu-abu | Seringkali soal kontrak; undang-undang bisa mengubah itu — verifikasi |
| Membypass DRM atau pengecekan lisensi pada software yang bukan milik Anda | Dilarang | Pengelakan dibatasi di banyak yurisdiksi |
| Menganalisis sampel malware di lab terisolasi | Abu-abu, cenderung OK bila hati-hati | Riset defensif yang diakui bila containment-nya solid |

## Hukum Sekilas

Ini orientasi, bukan nasihat hukum — verifikasi aturan yang benar-benar berlaku pada Anda:

- **Amerika Serikat — DMCA.** Digital Millennium Copyright Act memuat ketentuan anti-circumvention yang membatasi pengelakan terhadap proteksi teknologis seperti DRM, bersama pengecualian yang sudah berulang kali direvisi, termasuk yang relevan untuk riset keamanan. Baca teks terkininya sebelum mengandalkan pengecualian apa pun.
- **Uni Eropa — Software Directive.** Directive 2009/24/EC mengizinkan, di bawah kondisi tertentu, mempelajari sebuah program dan mende-compile-nya untuk mencapai interoperabilitas. Kondisinya spesifik; pastikan bagaimana kondisi itu berlaku pada situasi Anda.
- **Yurisdiksi Anda.** Hukum hak cipta, computer misuse, dan rahasia dagang sangat berbeda antar negara dan berubah seiring waktu.
- **Verifikasi apa yang berlaku pada Anda.** Pengecualian kedaluwarsa, undang-undang diamendemen, dan tidak ada tutorial — termasuk tutorial ini — yang menggantikan nasihat kualifikasi di yurisdiksi Anda sendiri.

### Area abu-abu layak kehati-hatian

Apa pun yang ditandai abu-abu di tabel di atas bergantung pada fakta dan lokasi: Anda berada di negara mana, isi lisensinya apa, apa yang sebenarnya Anda lakukan, dan siapa yang dirugikan. Ketika situasi benar-benar penting bagi Anda, perlambatlah dan dapatkan nasihat kualifikasi alih-alih menakulkannya sendirian.

## Terms of Service vs Hukum

Melanggar perjanjian Terms of Service biasanya adalah soal kontrak: vendor bisa menonaktifkan akun Anda atau mengajukan klaim perdata. Sebuah kejahatan memerlukan undang-undang — kontrak yang dilanggar dengan sendirinya tidak membuat perilaku menjadi kriminal, dan catatan ToS yang bersih tidak pernah mengalahkan hukum. Keduanya bisa berlaku sekaligus, itulah sebabnya pertanyaan soal ToS berada di zona abu-abu; periksa ketentuan lisensi *dan* hukumnya, bukan hanya salah satunya.

## Norma Profesional

- **Otorisasi pemberi kerja.** Analisis hanya binary dan sistem yang disebut dalam tugas atau scope Anda, dan simpan scope itu secara tertulis.
- **NDA dan kerahasiaan.** Temuan dari sebuah pekerjaan tetap di dalam pekerjaan itu; ketahui apa yang dicakup kontrak Anda sebelum membahas apa pun secara publik.
- **Responsible disclosure.** Kerentanan pada produk yang bukan milik Anda disampaikan ke vendor terlebih dahulu — lihat di bawah.
- **Dokumentasi.** Simpan catatan tentang apa yang Anda beri otorisasi untuk dilakukan; ketika ada pertanyaan muncul kemudian, catatan itu yang menjawabnya.
- **Proporsionalitas.** Gunakan metode yang paling tidak invasif untuk menjawab pertanyaan Anda — satu pass `strings` sebelum sesi debugger, salinan lokal sebelum target langsung.
- **Kalau ragu, tanyakan.** Pertanyaan singkat ke atasan, instruktur, atau pemilik program hanya butuh beberapa menit; sebuah uji tanpa otorisasi bisa memakan kontrak atau pekerjaan.

## Analisis Malware Adalah Riset yang Sah

Pertahanan harus memahami binary berbahaya agar bisa menulis signature, memblokir infrastruktur, dan memperingatkan korban, dan budaya riset di sekitarnya sudah mapan. Riset itu bertumpu pada containment: analisis sampel di lab terisolasi — mesin virtual sekali pakai dengan snapshot, tanpa folder bersama, dan akses jaringan dinonaktifkan atau dikendalikan ketat — tidak pernah di mesin harian Anda. Situs pendamping [CSIX Malware Lab](https://siedysoes.github.io/csix-malware-lab/) membahas lingkungan itu lebih detail. Menerima sebuah sampel secara kebetulan bukan izin tak terbatas untuk mempelajarinya; tahu mengapa Anda memilikinya, dan containment-lah apapun kondisinya.

## Tes "Apakah Saya Mendapatkan Binary Ini secara Sah?"

Sebelum membuka apa pun, jawab ini dengan jujur — jika ada jawaban yang tidak, berhenti dan minta izin dulu:

1. Apakah saya memiliki program ini, atau memegang hak untuk menganalisisnya?
2. Apakah binary ini diberikan kepada saya dengan ekspektasi akan dianalisis — sebuah challenge, latihan lab, atau tugas?
3. Apakah seseorang yang berwenang memberi akses — pemberi kerja, klien, vendor — mengizinkan aktivitas persis ini?
4. Apakah saya berada di dalam scope, ketentuan lisensi, dan yurisdiksi yang berlaku pada saya?

Jawaban ya untuk salah satu dari tiga pertanyaan pertama, bersama dengan nomor empat, adalah lampu hijau Anda. Selain itu langkah yang benar adalah bertanya, bukan melanjutkan diam-diam.

## Melaporkan Kerentanan Secara Bertanggung Jawab

Jika analisis menemukan kerentanan pada produk yang bukan milik Anda:

- Hubungi vendor terlebih dahulu, lewat kanal keamanan yang dipublikasikannya (`security.txt`, alamat `security@`, atau program bug-bounty).
- Berikan detail yang cukup untuk mereproduksi masalahnya, dan berhenti pada proof of concept — jangan mengakses data pengguna nyata atau mempertahankan akses yang tidak Anda beri.
- Sepakati jendela perbaikan yang wajar, lalu koordinasikan publikasinya (coordinated disclosure). Linimasa bervariasi per program; hitungan minggu alih-alih jam adalah praktik umum — verifikasi panduan terkini untuk program tersebut.
- Jika vendor tidak merespons, libatkan perantara disclosure yang diakui alih-alih langsung melakukan publikasi penuh.

## Garis Merah

```text
Never:
  - analyze a binary you do not own and are not authorized to test
  - bypass DRM or license enforcement on third-party commercial software
  - exfiltrate real user data as "proof" of a vulnerability
  - publish exploit details before the vendor has had a chance to fix them
  - move malware out of containment, or test anything outside your written scope
```

Jika sebuah tindakan yang diusulkan jatuh ke daftar ini, niat baik sebanyak apa pun tidak mengubah jawabannya. Seluruh jalur CSIX Reversing ditulis untuk binary yang Anda miliki atau berotorisasi untuk dianalisis — crackme, latihan lab, challenge CTF, dan sistem di dalam scope profesional Anda.

## Lihat juga

- [Apa Itu Reverse Engineering](what-is-reversing.md)
- [Practice & Labs](../practice/labs.md)
- [The Reversing Workflow](../practice/workflow.md)
- [Dynamic Analysis](../techniques/dynamic-analysis.md)
