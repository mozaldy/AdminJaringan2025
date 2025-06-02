

<p align="center">
  <img src="../Logo_PENS.png" alt="Logo">
</p>

<h1 align="center">Admin Jaringan 2025</h1>

```
Dosen Pengampu: Dr Ferry Astika Saputra ST, M.Sc
Nama: Mohammad Rizaldy Ramadhan
NRP: 3123600011
Kelas: D4 IT A
```

---

## 📘 **Dokumentasi Arsitektur Proyek: Aplikasi Pembelajaran Bahasa Jawa (Adibasa)**

---

### 1. **Arsitektur Bisnis**

#### a. **Tujuan Aplikasi**

Meningkatkan kemampuan pengguna dalam berbahasa Jawa melalui sistem pembelajaran bertingkat (level-based), mirip dengan aplikasi Duolingo.

#### b. **Aktivitas Bisnis Utama**

1. Manajemen konten pembelajaran (CRUD untuk Level & Soal/Challenge).
2. Proses pembelajaran interaktif (latihan dan ujian).
3. Monitoring progres pengguna.
4. Penilaian otomatis hasil latihan.

#### c. **Aktor**

* **Admin**: Mengelola data `Lesson` dan `Challenge`.
* **User**: Belajar bahasa Jawa, menyelesaikan challenge, dan meningkatkan level.

---

### 2. **Arsitektur Layanan (Backend)**

#### a. **Teknologi Utama**

* **Next.js (App Router)**
* **MongoDB (Mongoose ODM)**
* **Zod**: Validasi skema data
* **Server Actions**: Menggantikan API routes
* **Next.js Revalidation**: `revalidatePath`

#### b. **Struktur dan Fungsi Utama**

##### 📚 **Lesson Services**

| Fungsi                   | Deskripsi                                                  |
| ------------------------ | ---------------------------------------------------------- |
| `getLessons()`           | Mengambil semua lesson, diurutkan berdasarkan `order`.     |
| `getLesson(id)`          | Mengambil satu lesson berdasarkan ID.                      |
| `createLesson(data)`     | Menambah lesson baru dengan validasi `createLessonSchema`. |
| `updateLesson(id, data)` | Memperbarui lesson berdasarkan ID.                         |
| `deleteLesson(id)`       | Menghapus lesson berdasarkan ID.                           |

##### 🧩 **Challenge Services**

| Fungsi                    | Deskripsi                                           |
| ------------------------- | --------------------------------------------------- |
| `getChallenges(lessonId)` | Mengambil semua challenge dari satu lesson.         |
| `createChallenge()`       | Menambahkan challenge baru ke lesson.               |
| `updateChallenge()`       | Memperbarui challenge berdasarkan urutan.           |
| `deleteChallenge()`       | Menghapus challenge dari lesson berdasarkan urutan. |

---

### 3. **Arsitektur Aplikasi (Frontend)**

#### a. **Teknologi**

* **Next.js + React.js**
* **TypeScript**
* **Tailwind CSS** (diasumsikan untuk styling)
* **Server Actions** untuk komunikasi backend

#### b. **Struktur Folder**

```
/app                -> Routing halaman (Next.js App Router)
/components         -> Reusable UI components
/actions            -> Server actions (lessons.ts, challenges.ts)
/models             -> Mongoose schema (LessonModel)
/lib
  ├─ mongodb.ts     -> Koneksi database
  └─ schemas.ts     -> Zod schemas
/types              -> TypeScript interface & types
```

#### c. **Alur Interaksi**

1. User mengisi form.
2. Data dikirim ke server actions.
3. Validasi (Zod) dan simpan ke MongoDB.
4. Jalankan `revalidatePath()` untuk refresh cache.
5. Kirim response ke frontend → update UI.

---

### 4. **Arsitektur Database**

#### a. **MongoDB (Mongoose)**

* Koleksi: `lessons`
* Struktur Dokumen:

```json
{
  "id": "lesson_1",
  "title": "Dasar 1",
  "order": 1,
  "challenges": [
    {
      "question": "Makan",
      "order": 1,
      "instruction": "Terjemahkan kata dibawah ini:",
      "options": [
        { "text": "Mangan", "correct": true },
        { "text": "Ngombe", "correct": false }
      ]
    }
  ]
}
```

---

### 5. **Arsitektur Infrastruktur (Dockerized)**

<p align="center">
  <img src="./assets/architecture.png" alt="architecture" width="600">
</p>

#### a. **Teknologi**

* Docker
* Docker Compose
* MongoDB (Container)
* Next.js App (Container)

#### b. **Struktur Docker Compose**

```yaml
services:
  mongodb:
    build: ./adibasa_mongodb
    container_name: adibasa_mongodb
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password123
      - MONGO_INITDB_DATABASE=adibasa
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - adibasa_network

  web:
    build: ./adibasa_web
    container_name: adibasa_web
    environment:
      - MONGODB_URI=mongodb://admin:password123@mongodb:27017/adibasa?authSource=admin
    ports:
      - "3000:3000"
    depends_on:
      - mongodb
    networks:
      - adibasa_network

volumes:
  mongodb_data:

networks:
  adibasa_network:
    driver: bridge
```

#### c. **Fitur Tambahan**

* **Data Seeding**: Otomatis dari `lessons.json`.
* **Jaringan Virtual**: `adibasa_network` untuk interkoneksi service.
* **Persistensi Data**: Volume `mongodb_data`.

---

### 6. **Alur Kerja (End-to-End Flow)**

#### 🆕 **Menambah Lesson**

1. Admin buka `/dashboard/lessons`.
2. Klik "Add Lesson" → Isi form → Submit.
3. Panggil `createLesson()` di server.
4. Validasi → Simpan ke MongoDB.
5. Revalidasi path → Update UI frontend.

#### 🧠 **Menambah Challenge**

1. Admin navigasi ke `/dashboard/lessons/[lessonId]`.
2. Klik "Add Challenge" → Isi form → Submit.
3. Panggil `createChallenge()`.
4. Validasi → Tambah ke embedded array `challenges`.
5. Simpan → Revalidasi → Update UI.

---

### 7. **Keunggulan Arsitektur**

* ✅ **Full-Stack TypeScript**: Konsistensi tipe dari frontend sampai backend.
* ⚡ **Server Actions**: Tanpa REST API, lebih langsung & efisien.
* 🔗 **Embedded Documents**: Query efisien karena `challenges` disimpan dalam `lesson`.
* 🐳 **Docker**: Deployment lebih mudah & konsisten.
* 🌱 **Auto Seeding**: Langsung tersedia data awal.
* 🛡️ **Zod Validation**: Validasi aman & reusable di client & server.
* 🚀 **Next.js Caching**: Performa optimal dengan `revalidatePath`.

---
