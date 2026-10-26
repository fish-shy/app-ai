# sapa-flow

Alur percakapan **Sapa**, avatar pemandu kios APP AI (Healthkathon 2026), sebagai state machine TypeScript yang aman tipe.

## Arsitektur singkat
- `src/mesin.ts`: 12 keadaan, transisi murni `(keadaan, peristiwa) → (keadaan, efek)`. Tidak ada keadaan "ditolak". Mode konsol (BPJS / Check-in; Non BPJS & Admission ke petugas), verifikasi wajah lalu sidik jari, arahan poli/admisi dari status RM.
- `src/sesi.ts`: runner dengan jam: batas giliran 30 detik dan sesi 5 menit (nilai awal), durasi sesi ke audit.
- `src/naskah.ts`: naskah TTS + subtitle + pose avatar untuk setiap keadaan (mapped type: lupa satu = error kompilasi).
- `src/ucapan.ts`: tipe `Ucapan`; nama/NIK/poli tidak bisa diucapkan (error kompilasi).
- `src/pemahaman.ts`: Jev (System One, lewat gateway Experiential Labs; klien native `src/jev.ts`, tanpa coba ulang) → Claude Haiku 5.5 (strict tool use, hanya saat Jev ragu) → kata kunci (saat AI putus). Pemeriksaan darurat setiap giliran.
- `src/samarkan.ts`: angka dihapus dari ujaran sebelum keluar kios.
- `src/cocokkan.ts`: pencocokan kartu vs dokumen (Jaro-Winkler), menghasilkan bendera untuk petugas.

## Perintah
```
npm install
npm run cek        # tsc --strict + 40 uji (termasuk 2 × 3.000 alur acak)
npm run eval       # evaluasi pemahaman; Jev/Claude ikut bila EXPERIENTIAL_API_KEY / ANTHROPIC_API_KEY ada (berbayar)
npm run trace      # menulis jejak-skenario.json untuk halaman simulasi
```

### Input suara di layar kios

Jalankan `npm run kios`, buka `http://127.0.0.1:8789` di Chrome atau Edge, lalu tekan
**Jawab dengan suara** saat Sapa menunggu jawaban. Browser memakai Web Speech API
(`id-ID`) dan hanya mengirim transkrip final ke server; state machine tetap memprosesnya
sebagai `UCAPAN_PASIEN`. Jika browser tidak mendukung SpeechRecognition atau izin mikrofon
ditolak, tombol pilihan layar tetap bisa digunakan.

## Yang belum
- Ambang (0,6 / 0,3 / 0,7 / darurat 0,3) masih nilai awal; kalibrasi dengan rekaman relawan lansia.
- `eval/ujaran-lansia.json` ditulis tim, bukan rekaman pasien. Angka kata kunci (90%) optimistis.
- Belum ada adaptor perangkat nyata (STT, TTS, FRISTA, sidik jari, OBSBOT SDK, pemindai, VClaim, HFIS, Antrean, SIMRS). Mesin hanya menerima peristiwa bertipe dari adaptor itu.
- Naskah baru (mode, kode booking, sidik jari, arahan poli/admisi) masih draf; perlu ditinjau tim.
- `page/alur-sapa.html` belum dibangun ulang dari `jejak-skenario.json` yang baru.
- Akses Jev masih early access; kompatibilitas bahasa Indonesia/Jawa/Sunda perlu diuji langsung.
