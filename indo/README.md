# Elaina-Multidevice dalam InDo (indo-langvm)

Branch `indo-langvm` mem-port **seluruh** berkas `.js` bot ke bahasa
[`indo-langvm`](https://www.npmjs.com/package/indo-langvm) (berkas `.wni`, kata
kunci berbahasa Indonesia). Repo bahasa: <https://github.com/rexxzyid/InDo>.

## Cakupan

Semua 110 berkas `.js` (inti + `lib/` + 97 plugin) ditransliterasi ke `.wni`
lewat parser TypeScript → penulis InDo, lalu **setiap** berkas divalidasi lolos
`indo periksa` (parser InDo). Kata kunci dan builtin dipetakan:

- `const`/`let` → `tetap`/`misal`, `function` → `fungsi`, `return` →
  `kembalikan`, `if/else` → `jika/lainnya`, `for/while/do` →
  `untuk/selama/lakukan`, `switch/case` → `pilih/kasus`, `try/catch/finally` →
  `coba/tangkap/akhirnya`, `throw` → `lempar`, `class/new/this` →
  `kelas/baru/ini`, `async/await` → `asinkron/tunggu`, `typeof/instanceof` →
  `jenisdari/contohdari`, `&&/||/!` → `dan/atau/bukan`.
- `Math` → `Matematika` (`floor`→`bawah`, `round`→`bulatkan`, `pow`→`pangkat`,
  …), `Object` → `Objek` (`keys`→`kunci`, …), `JSON.stringify/parse` →
  `JSON.teks/urai`, `console.log` → `Konsol.cetak`.
- Metode array/teks: `map`→`petakan`, `filter`→`saring`, `find`→`cari`,
  `forEach`→`untukSetiap`, `includes`→`berisi`, `push`→`tambah`,
  `slice`→`iris`, `split`→`pisah`, `join`→`gabung`, `length`→`panjang`, dst.
- Impor relatif `./x.js` diarahkan ke `./x.wni`. Komentar dibuang otomatis
  (aturan repo: kode tanpa `//` atau `/** */`).

## Batasan jujur (yang tidak bisa dipastikan jalan)

Konversi ini **sintaktis**, bukan jaminan bot jalan end-to-end. `indo-langvm`
adalah VM tersendiri: bisa `impor` antar-`.wni`, tetapi **tidak** bisa `impor`
modul npm/Node (`baileys`, `axios`, `better-sqlite3`, `cheerio`, `fs`, `path`,
`worker_threads`, dll). Berkas yang memanggil modul itu saat dimuat akan gagal
di VM. Yang **benar-benar jalan** hanya berkas berlogika murni; contoh yang
sudah diuji byte-per-byte identik dengan JS ada di bawah.

Agar bot jalan penuh dalam `.wni` dibutuhkan lapisan interop npm di runtime
InDo (arah transpile-ke-JS), yang belum ada.

## Contoh teruji (jalan di indo-langvm)

```bash
npm install -g indo-langvm
indo jalankan indo/levelling.tes.wni
indo jalankan indo/util.tes.wni
```

| Kasus | indo-langvm | JS |
| --- | --- | --- |
| `pertumbuhan` | 2.576652002695681 | 2.576652002695681 |
| `rentangXp(5)` | `{min:64,max:101,xp:37}` | `{min:64,max:101,xp:37}` |
| `toTimeString(90061000)` | `1 hari 1 jam 1 menit 1 detik` | sama |
| `getKey("halo")` | `a4e63bca…16d777` | sama (`sha256`) |

`indo/levelling.wni` dan `indo/util.wni` adalah port murni yang ditulis tangan
dan diuji; berkas `lib/levelling.wni` dst adalah transliterasi apa adanya dari
sumber JS (mis. masih memakai `global.multiplier`, yang tidak ada di VM).
