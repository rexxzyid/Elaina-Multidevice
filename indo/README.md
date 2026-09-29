# POC: Elaina-Multidevice dalam InDo (indo-langvm)

Percobaan mem-port bagian **logika murni** bot ke bahasa
[`indo-langvm`](https://www.npmjs.com/package/indo-langvm) (berkas `.wni`, kata
kunci berbahasa Indonesia). Repo bahasa: <https://github.com/rexxzyid/InDo>.

## Status: proof-of-concept, bukan konversi penuh

Konversi **seluruh** bot ke indo-langvm **belum feasible**. Bot ini aplikasi
Node yang bergantung pada modul yang tidak bisa di-load VM indo-langvm:

- `baileys` (`@rexxhayanasi/elaina-baileys`) — koneksi WhatsApp lewat WebSocket +
  kripto Signal. Runtime indo-langvm kini punya primitif WebSocket dan kripto
  (x25519, ed25519, aes-gcm/cbc, hmac, hkdf, sha256), tetapi ia tetap VM sendiri
  tanpa interop npm, jadi tidak bisa `impor` modul JavaScript apa adanya.
- `better-sqlite3` — modul native Node.
- `axios`, `cheerio`, `fluent-ffmpeg`, `file-type`, `pdfkit`, dll — modul npm.
- 100+ plugin memakai `conn.*`, unduhan HTTP, database, dan ffmpeg.

Bagian inti bot (koneksi, plugin, I/O) tetap JavaScript/Node. Yang cocok di-port
adalah fungsi murni tanpa I/O.

## Yang di-port

Hasilnya dibuktikan **identik** dengan versi JavaScript-nya:

- **Levelling** (`levelling.wni`) — port `lib/levelling.js` (perhitungan
  XP/level, hanya `Math`). `pertumbuhan` = 2.576652002695681, `rentangXp(5)` =
  `{min:64,max:101,xp:37}`, `cariLevel(1000)` = 14.
- **Util** (`util.wni`) — pembantu murni dari `lib/simple.js`: `toTimeString`
  (format durasi "hari/jam/menit/detik"), `isNumber`, `nullish`, dan `getKey`
  (SHA256 hex). `getKey("halo")` = `a4e63bca…16d777`, cocok persis dengan
  `createHash('sha256')` di Node.

## Menjalankan

```bash
npm install -g indo-langvm
indo jalankan indo/levelling.tes.wni
indo jalankan indo/util.tes.wni
```

## Berkas

| Berkas | Isi |
| --- | --- |
| `levelling.wni` | Port `lib/levelling.js` (XP/level) |
| `levelling.tes.wni` | Uji levelling vs versi JS |
| `util.wni` | Port pembantu murni `lib/simple.js` |
| `util.tes.wni` | Uji util vs versi JS |

## Catatan porting

- `typeof` indo-langvm mengembalikan nama Indonesia (`"angka"` untuk number,
  `"teks"` untuk string).
- `Math` → `Matematika` (`pangkat`, `bulatkan`, `bawah`, `PI`, `E`), `Infinity`
  → `Takhingga`, `isNaN` → `adalahNaN`, `String.trim` → `.rapikan`,
  `parseInt` → `uraiBulat`, `createHash('sha256')` → `Kripto.sha256` + `Bita`.
- `global.multiplier` diganti parameter bawaan `pengali = 1` (VM terpisah, tidak
  ada `global` bot).
