# Write-up Belajar MongoDB: Dari Nol sampai Mahir Query

Halo! Ini adalah catatan perjalanan gue belajar MongoDB. Mulai dari bingung "ini gua apain ya?" sampai bisa bikin query kompleks pake operator comparison dan logical. Semoga berguna buat yang lagi belajar juga.

---

## 1. Kenapa NoSQL / MongoDB?

- **NoSQL** itu database yang fleksibel, gak pake tabel kaku kayak SQL.
- **MongoDB** adalah database document-based (pake JSON). Cocok buat aplikasi modern yang butuh skalabilitas dan skema dinamis.
- Belajar NoSQL bikin kita paham cara nyimpen data yang lebih alami sesuai kebutuhan aplikasi.

---

## 2. Instalasi dan Jalanin MongoDB

### Instalasi Lokal (Windows/Mac/Linux)
- Download dari [mongodb.com](https://www.mongodb.com/try/download/community)
- Install, pastikan **MongoDB Compass** juga keinstall biar gampang liat data.
- Cek service udah jalan:
  ```bash
  # buka terminal/CMD
  mongosh
  ```
  Kalau masuk shell (`test>`), berarti sukses.

### Koneksi ke VPS via Compass (Remote)
Biar bisa akses database di VPS dari laptop:

1. **SSH ke VPS**, edit file `/etc/mongod.conf`:
   ```yaml
   net:
     port: 27017
     bindIp: 0.0.0.0   # biar bisa diakses dari luar
   ```
2. **Restart MongoDB**:
   ```bash
   sudo systemctl restart mongod
   ```
3. **Buat user admin** (biar aman):
   ```javascript
   use admin
   db.createUser({ user: "admin", pwd: "nyawit", roles: ["root"] })
   ```
4. **Aktifkan autentikasi** di mongod.conf:
   ```yaml
   security:
     authorization: enabled
   ```
5. **Buka port 27017** di firewall VPS.
6. **Konek dari Compass**:
   - Connection string: `mongodb://admin:nyawit@ip_vps:27017/?authSource=admin`
   - Atau lebih aman pake **SSH Tunnel** (isi ssh config di Compass).

---

## 3. Konsep Dasar MongoDB

- **Database** → kumpulan collection.
- **Collection** → kumpulan document (mirip tabel, tapi tanpa skema tetap).
- **Document** → data dalam format JSON (BSON).

Contoh document:
```json
{
  "_id": ObjectId("..."),
  "nama": "Indomie Goreng",
  "harga": 3500,
  "stok": 120,
  "kategori": "Makanan"
}
```

### Operasi CRUD Dasar
- **Insert**: `db.collection.insertOne({...})` atau `insertMany([...])`
- **Find**: `db.collection.find({ filter })`
- **Update**: `db.collection.updateOne(filter, {$set: {...}})`
- **Delete**: `db.collection.deleteOne(filter)`

---

## 4. Query Operators

### 4.1 Comparison Operators
Digunakan untuk membandingkan nilai.

| Operator | Arti | Contoh |
|----------|------|--------|
| `$eq` | sama dengan | `{ harga: { $eq: 3500 } }` |
| `$ne` | tidak sama | `{ kategori: { $ne: "Makanan" } }` |
| `$gt` | lebih besar | `{ harga: { $gt: 5000 } }` |
| `$gte` | lebih besar atau sama | `{ harga: { $gte: 7000 } }` |
| `$lt` | lebih kecil | `{ harga: { $lt: 5000 } }` |
| `$lte` | lebih kecil atau sama | `{ harga: { $lte: 3500 } }` |
| `$in` | termasuk dalam array | `{ kategori: { $in: ["Makanan","Minuman"] } }` |
| `$nin` | tidak termasuk | `{ kategori: { $nin: ["Elektronik"] } }` |

**Best Practice**:
- Untuk `$eq` mending langsung `{ field: value }`.
- `$in` lebih efisien daripada `$or` untuk satu field.
- Hindari `$ne` dan `$nin` kalau data besar, karena lambat.

### 4.2 Logical Operators
Menggabungkan beberapa kondisi.

| Operator | Arti | Contoh |
|----------|------|--------|
| `$and` | semua kondisi true/benar | `{ $and: [ {kategori:"Minuman"}, {harga:{$gt:4000}} ] }` |
| `$or` | salah satu true/benar | `{ $or: [ {kategori:"Makanan"}, {harga:{$lt:5000}} ] }` |
| `$nor` | semua kondisi false/salah | `{ $nor: [ {kategori:"Minuman"}, {harga:{$gt:5000}} ] }` |
| `$not` | membalik kondisi tunggal | `{ harga: { $not: { $lt:5000 } } }` |

**Best Practice**:
- `$and` implisit bisa ditulis `{ field1: val1, field2: val2 }`.
- Gunakan `$and` eksplisit kalau ada field yang sama dengan kondisi ganda.
- `$or` usahakan setiap klausa punya index.
- `$nor` sering susah dibaca, pertimbangkan alternatif.
- `$not` hati-hati dengan nilai null/missing field.


## 5. Kesimpulan dan Langkah Selanjutnya

Dari sini gue belajar:
- **MongoDB itu fleksibel**, cocok buat aplikasi yang cepat berubah.
- **Query operators** jadi senjata utama buat nyari data.
- **Kombinasi logical & comparison** bisa nanganin hampir semua kebutuhan filtering.

Yang bakal gue pelajari selanjutnya:
- **Aggregation Pipeline** ($group, $lookup, $project) buat analisis data lebih canggih.
- **Indexing** biar query makin ngebut.
- **Replica Set & Sharding** untuk skalabilitas.

