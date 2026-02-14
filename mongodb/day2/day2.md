# Write-up Belajar MongoDB: Evaluation Operators

  **Evaluation Operators**. Operator ini digunakan untuk evaluasi dokumen berdasarkan kondisi yang lebih dinamis, seperti pencocokan pola teks (`$regex`), pencarian teks penuh (`$text`), dan ekspresi JavaScript (`$where`).

Kita masih pakai dataset `produk` yang sama:

---

## 1. `$regex` — Pencocokan Pola Teks

`$regex` memungkinkan untuk mencari string berdasarkan pola **regular expression**. Cocok untuk pencarian partial, wildcard, atau pola kompleks.

### Sintaks Dasar

```javascript
{ field: { $regex: /pattern/, $options: '<options>' } }
// atau
{ field: { $regex: 'pattern', $options: '<options>' } }
```

### Options yang Sering Dipakai

- `i` — case insensitive (abaikan huruf besar/kecil)
- `m` — multiline (^ dan $ berlaku per baris)
- `x` — extended (abaikan spasi)
- `s` — dot matches all (termasuk newline)

### Contoh Penggunaan

```javascript
// Cari produk yang namanya mengandung "teh" (case insensitive)
db.produk.find({ nama: { $regex: /teh/i } })
// Hasil: Teh Botol

// Cari produk yang namanya diawali huruf "S"
db.produk.find({ nama: { $regex: /^S/ } })
// Hasil: Silverqueen

// Cari produk yang namanya berakhiran "en"
db.produk.find({ nama: { $regex: /ng$/ } })
// Hasil: Indomie Goreng

// Cari produk dengan kode tertentu (misal field kode ada)
// db.produk.find({ kode: { $regex: /^[A-Z]{2}\d{3}/ } })
```

### Best Practices `$regex`

- **Gunakan index** kalau pola pencarian bersifat prefix (misal `^awalan`). Index akan digunakan optimal.
- Hindari regex yang diawali wildcard (`/.*kata/`) karena akan full scan.
- Untuk pencarian teks sederhana, pertimbangkan `$text` (lebih cepat).
- Batasi penggunaan regex pada koleksi besar.

---

## 2. `$text` — Pencarian Teks Penuh

`$text` digunakan untuk melakukan pencarian teks pada field yang sudah di-**index text**. Mendukung pencarian kata, frasa, dan eksklusi.

### Langkah Pertama: Buat Text Index

Sebelum pakai `$text`, lo harus bikin **text index** pada field yang ingin dicari.
```javascript
db.produk.createIndex({ nama: "text", tags: "text" })
// Bisa juga multiple field
```

### Sintaks Dasar

```javascript
{
  $text: {
    $search: <string>,
    $language: <string>,  // optional
    $caseSensitive: <boolean>, // optional
    $diacriticSensitive: <boolean> // optional
  }
}
```

### Contoh Penggunaan

```javascript
// Cari produk yang mengandung kata "indomie" atau "aqua"
db.produk.find({ $text: { $search: "indomie aqua" } })
// Hasil: Indomie Goreng, Aqua

// Cari frasa tepat "teh botol"
db.produk.find({ $text: { $search: "\"teh botol\"" } })
// Hasil: Teh Botol

// Eksklusi kata: cari yang ada "pocari" tapi tidak "sweat"
db.produk.find({ $text: { $search: "pocari -sweat" } })
// Hasil: (mungkin tidak ada karena Pocari Sweat mengandung sweat)
```

### Mengakses Relevance Score

`$text` mengembalikan **score** relevansi yang bisa dipakai untuk sorting.
```javascript
db.produk.find(
  { $text: { $search: "indomie" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } })
```

### Best Practices `$text`

- **Wajib punya text index** — kalau belum, query akan error.
- `$text` tidak peka huruf besar/kecil secara default, tapi bisa diatur dengan `$caseSensitive`.
- `$text` tidak bisa digunakan di field bersarang (embedded document) secara langsung, tapi bisa di index dengan dot notation.
- Gunakan `$meta` untuk sorting berdasarkan relevansi.

---

## 3. `$where` — Ekspresi JavaScript

`$where` memungkinkan lo menggunakan **JavaScript expression** untuk memfilter dokumen. Sangat fleksibel, tapi **lambat** karena harus mengevaluasi JavaScript untuk tiap dokumen.

### Sintaks Dasar

```javascript
{ $where: "function() { return ... }" }
// atau string JavaScript
{ $where: "this.field > 100 && this.field2 == 'value'" }
```

### Contoh Penggunaan

```javascript
// Cari produk yang harganya lebih besar dari stok dikali 1000
db.produk.find({ $where: "this.harga > (this.stok * 1000)" })

// Pakai function
db.produk.find({
  $where: function() {
    return this.harga > this.stok * 1000;
  }
})
```

### Best Practices `$where`

- **Hindari sebisa mungkin**. Gunakan operator lain dulu.
- `$where` tidak memanfaatkan index, jadi full collection scan.
- Gunakan hanya untuk logika kompleks yang tidak bisa diungkapkan dengan operator lain.
- Perhatikan keamanan: jangan pernah memasukkan input user langsung ke `$where` (rentan injection).

## Kesimpulan

- **`$regex`** fleksibel buat pencarian pola, tapi perhatikan performa.
- **`$text`** cepat buat pencarian teks penuh, tapi harus siapin index.
- **`$where`** powerfull tapi lambat, gunakan hanya darurat.
