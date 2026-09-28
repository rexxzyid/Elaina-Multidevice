# POC: Elaina-MultiDevice dalam InDo (b-indo)

Percobaan mem-port bagian **logika murni** bot ke bahasa
[`@rexxhayanasi/b-indo`](https://github.com/rexxzyid/InDo) (berkas `.wni`).

## Status: proof-of-concept, bukan konversi penuh

Konversi **seluruh** bot ke b-indo **belum feasible**. Bot ini adalah aplikasi
Node yang bergantung pada modul yang tidak bisa dijalankan/di-load b-indo:

- `baileys` — butuh WebSocket + kripto Signal (Curve25519/HKDF/AES-GCM/HMAC),
  belum ada di b-indo.
- `better-sqlite3` — modul native Node.
- `axios`, `cheerio`, `fluent-ffmpeg`, `file-type`, `pdfkit`, dll — modul npm.
- 100+ plugin memakai `conn.*`, unduhan HTTP, database, dan ffmpeg.

b-indo berjalan di VM sendiri dan tidak punya interop npm, jadi bagian inti bot
(koneksi WhatsApp, plugin) tetap harus JavaScript/Node.

## Yang bisa di-port: logika murni

Yang cocok hanyalah fungsi murni tanpa I/O. Di sini di-port `lib/levelling.js`
(perhitungan XP/level, hanya `Math`). Keluarannya **identik** dengan versi JS:

| Kasus | b-indo | JS |
| --- | --- | --- |
| `pertumbuhan` | 2.576652002695681 | 2.576652002695681 |
| `rentangXp(5)` | {min:64,max:101,xp:37} | {min:64,max:101,xp:37} |
| `cariLevel(1000)` | 14 | 14 |
| `bisaNaikLevel(50,1000)` | salah | false |

## Menjalankan

```bash
npm install -g @rexxhayanasi/b-indo
indo jalankan indo/levelling.tes.wni
```

## Catatan porting

- `typeof` b-indo mengembalikan nama Indonesia (`"teks"` untuk string).
- `Math` → `Matematika` (`pangkat`, `bulatkan`, `bawah`, `PI`, `E`), `Infinity`
  → `Takhingga`, `isNaN` → `adalahNaN`.
- `global.multiplier` diganti parameter bawaan `pengali = 1` (b-indo VM terpisah,
  tidak ada `global` bot).
