# Woelandari Coffee Lab — Diagram Proses Sistem

> Dokumentasi arsitektur dan alur proses sistem website Woelandari Coffee Lab  
> Database: `db_cafe` | Tech Stack: PHP + MySQL + Vanilla JS  
> Dibuat: September 2026

---

## 1. Arsitektur Sistem Keseluruhan

```mermaid
graph TB
    subgraph CLIENT["🌐 CLIENT (Browser)"]
        USER["👤 Pengunjung / User"]
        ADMIN["🔑 Admin / Karyawan"]
    end

    subgraph PUBLIC["📄 HALAMAN PUBLIK (index.php)"]
        HOME["home.php\n[Halaman Utama]"]
        ABOUT["about.php\n[Tentang Kami]"]
        MENU_PUB["menu.php\n[Daftar Menu]"]
        COMMUNITY["community.php\n[Tim Woelandari]"]
        RESERVASI["reservasi.php\n[Reservasi WhatsApp]"]
        RATING_PUB["rating.php\n[Rating & Review]"]
        LOKASI["lokasi.php / lokasi_full.php\n[Informasi Lokasi]"]
        GALLERY_PUB["gallery.php\n[Galeri & Event]"]
    end

    subgraph ADMIN_PANEL["🔒 PANEL ADMIN (dashboard/)"]
        LOGIN["login.php\n[Autentikasi]"]
        DASH["dashboard.php\n[Ringkasan Data]"]
        MENU_CRUD["menu_crud.php\n[Kelola Menu]"]
        GALLERY_CRUD["gallery_crud.php\n[Kelola Gallery & Event]"]
        RATING_CRUD["rating_crud.php\n[Moderasi Feedback]"]
        COMMUNITY_CRUD["community_crud.php\n[Kelola Tim]"]
        USER_MNG["user_manajemen.php\n[Kelola User]"]
        AKUN_STAFF["akun_staff.php\n[Akun Karyawan]"]
    end

    subgraph DB["🗄️ DATABASE (db_cafe)"]
        TBL_MENU["📋 Tabel: menu"]
        TBL_FEEDBACK["⭐ Tabel: feedback"]
        TBL_HUMAN["👥 Tabel: human_archive"]
        TBL_GALLERY["🖼️ Tabel: gallery"]
        TBL_EVENTS["📅 Tabel: events"]
        TBL_USER["🔐 Tabel: user"]
    end

    subgraph COMPONENTS["🧩 KOMPONEN BERSAMA"]
        NAVBAR["navbar.php"]
        SIDEBAR["sidebar.php"]
        POPUP["popup.php"]
        KONEKSI["config/koneksi.php"]
    end

    USER --> PUBLIC
    ADMIN --> LOGIN
    LOGIN --> ADMIN_PANEL

    PUBLIC --> KONEKSI
    ADMIN_PANEL --> KONEKSI
    KONEKSI --> DB

    NAVBAR --> PUBLIC
    SIDEBAR --> ADMIN_PANEL
```

---

## 2. Alur Autentikasi (Login / Logout)

```mermaid
flowchart TD
    A([🚀 User Akses login.php]) --> B{Session aktif?}
    B -- Ya --> C[Redirect: dashboard.php]
    B -- Tidak --> D{Ada Cookie\nremember_user?}
    D -- Ya --> E[Query user by\ncookie username]
    E --> F{User ditemukan\ndi DB?}
    F -- Ya --> G[Buat Session Baru:\nid_user, username,\nrole, nama_lengkap]
    G --> C
    F -- Tidak --> H[Tampilkan Form Login]
    D -- Tidak --> H

    H --> I[User Input\nUsername + Password]
    I --> J[POST: name=login]
    J --> K[Query SELECT * FROM user\nWHERE username = ?]
    K --> L{User ada\ndi DB?}
    L -- Tidak --> M[❌ Error: Username\ntidak ditemukan]
    M --> H
    L -- Ya --> N[password_verify\npassword + hash]
    N --> O{Password\ncorrect?}
    O -- Tidak --> P[❌ Error: Password salah]
    P --> H
    O -- Ya --> Q[Set $_SESSION:\nstatus, id_user,\nusername, role]
    Q --> R{Checkbox\nRemember Me?}
    R -- Ya --> S[setcookie\nremember_user\n7 hari]
    S --> C
    R -- Tidak --> C

    C --> T[Dashboard Tampil]

    subgraph LOGOUT["🚪 Proses Logout"]
        U([User Klik Logout]) --> V[Modal Konfirmasi]
        V --> W[Konfirmasi]
        W --> X[logout.php]
        X --> Y[session_unset +\nsession_destroy]
        Y --> Z[Hapus Cookie\nremember_user]
        Z --> AA[Redirect: login.php]
    end
```

---

## 3. Sistem Role & Hak Akses

```mermaid
graph LR
    subgraph ROLES["👥 Role User"]
        SA["SUPERADMIN"]
        AD["ADMIN"]
        KR["KARYAWAN"]
    end

    subgraph AKSES["🔑 Akses Menu Dashboard"]
        D1["📊 Dashboard"]
        D2["🍽️ Menu CRUD"]
        D3["🖼️ Gallery CRUD"]
        D4["⭐ Rating/Feedback CRUD"]
        D5["👥 Community CRUD"]
        D6["👤 User Manajemen"]
        D7["🪪 Akun Staff (Profil)"]
    end

    SA --> D1
    SA --> D2
    SA --> D3
    SA --> D4
    SA --> D5
    SA --> D6

    AD --> D1
    AD --> D2
    AD --> D3
    AD --> D4
    AD --> D5
    AD --> D6

    KR --> D1
    KR --> D2
    KR --> D3
    KR --> D4
    KR --> D5
    KR --> D7
```

---

## 4. Alur Halaman Publik (Single Page Application)

```mermaid
flowchart TD
    A([Pengunjung Buka index.php]) --> B[Load navbar.php]
    B --> C{Navigasi\nSection}

    C --> D["#home\nhome.php"]
    C --> E["#about\nabout.php"]
    C --> F["#menu\nmenu.php"]
    C --> G["#community\ncommunity.php"]
    C --> H["#reservasi\nreservasi.php"]
    C --> I["#rating\nrating.php"]
    C --> J["#lokasi\nlokasi.php"]

    C --> K["gallery.php\n(halaman terpisah)"]
    C --> L["lokasi_full.php\n(halaman terpisah)"]

    D --> D1["Animasi karakter\ninteraktif (mousemove)"]
    E --> E1["Konten statis:\nFilosofi & Sejarah"]
    F --> F1["Query: SELECT * FROM menu\nORDER BY kategori"]
    F1 --> F2["Filter: Coffee / Non-Coffee\nSnack / Main Course"]
    F2 --> F3["Pagination: 8 item/halaman"]
    F3 --> F4["Klik item → tampilkan\ndetail di panel kiri"]

    G --> G1["Query: SELECT * FROM\nhuman_archive WHERE status=active\nORDER BY display_order"]
    G1 --> G2["Tampil kartu tim\nhorizontal scroll"]

    H --> H1["Form: Nama, WA, Tanggal\nJam, Durasi, Area, Pax"]
    H1 --> H2["Validasi client-side"]
    H2 --> H3["Generate WhatsApp\npesan ke admin"]
    H3 --> H4["Buka wa.me link\ndi tab baru"]

    I --> I1["Query: rating dari\ntabel feedback tampil"]
    I1 --> I2["Tampilkan statistik\n& chart distribusi"]
    I2 --> I3{"User submit review?"}
    I3 -- Ya --> I4["AJAX POST: rating.php?ajax=submit"]
    I4 --> I5["Filter kata kasar"]
    I5 --> I6{"Kata kasar\nditemukan?"}
    I6 -- Ya --> I7["❌ Tolak review"]
    I6 -- Tidak --> I8["INSERT INTO feedback\nstatus=tampil"]
    I8 --> I9["✅ Sukses + reload"]
    I3 -- Load --> I10["AJAX GET: rating.php?ajax=load"]
    I10 --> I11["Query dengan filter\n& pagination (3/halaman)"]
    I11 --> I12["Render kartu review"]

    K --> K1["Query gallery + events\ndari DB"]
    K1 --> K2["Tab: Gallery | Event"]

    L --> L1["Tampilkan alamat,\njam ops, peta, sosmed"]
```

---

## 5. Alur CRUD Menu (Admin)

```mermaid
flowchart TD
    A([Admin Buka menu_crud.php]) --> B[Cek Session]
    B --> C{Session valid?}
    C -- Tidak --> D[Redirect login.php]
    C -- Ya --> E[Query: SELECT * FROM menu\ndengan filter & pagination]
    E --> F[Tampilkan tabel menu]

    F --> G{Aksi Admin}

    G --> H["TAMBAH MENU\n(Form kosong)"]
    H --> H1["Input: nama, kategori,\nharga, status, deskripsi"]
    H1 --> H2["Upload foto (base64 crop)"]
    H2 --> H3["Simpan file .jpg ke\nassets/images/menu/"]
    H3 --> H4["INSERT INTO menu"]
    H4 --> H5[✅ Notifikasi sukses]

    G --> I["EDIT MENU\nGET ?edit=id"]
    I --> I1["Load data menu\nke form"]
    I1 --> I2["Admin ubah data"]
    I2 --> I3{Ganti foto?}
    I3 -- Ya --> I4["Hapus file lama\nSimpan file baru"]
    I3 -- Tidak --> I5["Foto tetap sama"]
    I4 --> I6["UPDATE menu SET...\nWHERE id_menu=?"]
    I5 --> I6
    I6 --> I7[✅ Notifikasi update]

    G --> J["HAPUS MENU\nGET ?hapus=id"]
    J --> J1[Hapus file foto\ndari server]
    J1 --> J2["DELETE FROM menu\nWHERE id_menu=?"]
    J2 --> J3[✅ Notifikasi hapus]

    G --> K["TOGGLE STATUS\nTersedia / Tidak Tersedia"]
    K --> K1["UPDATE menu SET status=...\nWHERE id_menu=?"]
```

---

## 6. Alur Sistem Rating & Moderasi Feedback

```mermaid
flowchart TD
    subgraph PUBLIK["🌐 Sisi Publik (rating.php)"]
        A1([Pengunjung buka\nhalaman rating]) --> A2["Tampil statistik:\navg rating, chart distribusi"]
        A2 --> A3[Form: Nama + Bintang + Komentar]
        A3 --> A4[User Submit]
        A4 --> A5["AJAX POST ke\nrating.php?ajax=submit"]
        A5 --> A6["Validasi: nama, rating 1-5,\nkomentar tidak kosong"]
        A6 --> A7{Kata kasar\nterdeteksi?}
        A7 -- Ya --> A8[❌ Tolak: Pesan error]
        A7 -- Tidak --> A9["INSERT INTO feedback\n(status_moderasi='tampil')"]
        A9 --> A10[✅ Review tampil langsung]
    end

    subgraph FILTER["🔍 Filter & Pagination"]
        B1([User klik filter bintang]) --> B2["AJAX GET: rating.php?ajax=load\n&filter=N&page=N"]
        B2 --> B3["Query: SELECT dari feedback\nWHERE status=tampil AND rating=N\nLIMIT 3 OFFSET N"]
        B3 --> B4[Render kartu review\ndengan animasi slide-up]
    end

    subgraph ADMIN_MOD["🔒 Moderasi Admin (rating_crud.php)"]
        C1([Admin buka rating_crud]) --> C2["Tampil semua feedback:\ntampil + pending"]
        C2 --> C3{Aksi Moderasi}
        C3 -- Approve --> C4["UPDATE feedback\nSET status='tampil'"]
        C3 -- Reject --> C5["UPDATE feedback\nSET status='pending'"]
        C3 -- Hapus --> C6["DELETE FROM feedback"]
        C4 & C5 & C6 --> C7[✅ Redirect + notifikasi]
    end
```

---

## 7. Alur Manajemen Gallery & Event (Admin)

```mermaid
flowchart TD
    A([Admin buka gallery_crud.php]) --> B{Pilih Tab}

    B --> C["Tab: GALLERY"]
    B --> D["Tab: EVENT"]

    C --> C1[Lihat daftar foto\ngallery dari DB]
    C1 --> C2{Aksi}
    C2 -- Tambah --> C3["Form: judul, deskripsi,\nhubungkan ke event?"]
    C3 --> C4["Upload foto (base64 crop)"]
    C4 --> C5["Simpan blob foto\nke tabel gallery.file_foto"]
    C5 --> C6["INSERT INTO gallery\n(judul, deskripsi, id_event,\ntipe, file_foto)"]
    C2 -- Edit --> C7["Load data ke form\nUPDATE gallery"]
    C2 -- Hapus --> C8["DELETE FROM gallery"]

    D --> D1[Lihat daftar event]
    D1 --> D2{Aksi}
    D2 -- Tambah --> D3["Form: judul, deskripsi,\ntanggal, status, featured"]
    D3 --> D4["Upload cover event"]
    D4 --> D5["INSERT INTO events"]
    D2 -- Edit --> D6["UPDATE events"]
    D2 -- Toggle Featured --> D7{Event sudah\nselesai?}
    D7 -- Ya --> D8["Tidak bisa disematkan\n(disabled)"]
    D7 -- Tidak --> D9["UPDATE events\nSET is_featured=1"]
    D2 -- Hapus --> D10["DELETE FROM events\n+ hapus foto terkait di gallery"]

    subgraph PUBLIK_GALLERY["🌐 Tampil di gallery.php"]
        E1["Query: SELECT gallery\nLEFT JOIN events"]
        E2["Render kartu\nGallery + Event"]
        E3["Tab switch:\nGallery | Event"]
        E1 --> E2 --> E3
    end
```

---

## 8. Alur Manajemen User (Admin/Superadmin)

```mermaid
flowchart TD
    A([Admin buka user_manajemen.php]) --> B{Cek role}
    B -- karyawan --> C[❌ Redirect dashboard.php]
    B -- admin/superadmin --> D["Tampil daftar user\ndengan search & pagination"]

    D --> E{Aksi}

    E -- Tambah --> F["Form: nama_lengkap,\nusername, role, password"]
    F --> G["password_hash\n(BCRYPT)"]
    G --> H["INSERT INTO user\n(password_default=hash,\nis_first_login=1)"]
    H --> I[✅ Redirect + notif]

    E -- Edit --> J["Load data user"]
    J --> K["Form pre-filled"]
    K --> L{Ganti password?}
    L -- Ya --> M["Hash password baru\nSET is_first_login=1"]
    L -- Tidak --> N["Update tanpa ubah\npassword"]
    M & N --> O["UPDATE user SET...\nWHERE id_user=?"]
    O --> P[✅ Notifikasi update]

    E -- Hapus --> Q["DELETE FROM user\nWHERE id_user=?"]
    Q --> R[✅ Notifikasi hapus]

    subgraph KARYAWAN["👤 Akun Staff (akun_staff.php)"]
        S1([Karyawan buka akun profil]) --> S2{Pilih aksi}
        S2 -- Update Profil --> S3["Update nama & username"]
        S2 -- Ganti Password --> S4["Verifikasi password lama"]
        S4 --> S5{Password benar?}
        S5 -- Ya --> S6["Hash password baru\nUPDATE user SET password=hash\nSET is_first_login=0"]
        S5 -- Tidak --> S7[❌ Error pesan]
    end
```

---

## 9. Alur Manajemen Komunitas / Tim (Admin)

```mermaid
flowchart TD
    A([Admin buka community_crud.php]) --> B["Tampil daftar anggota tim\ndengan filter status & search"]

    B --> C{Aksi}

    C -- Tambah --> D["Form: nama, role,\nquote, display_order"]
    D --> E["Upload foto (base64 crop\natau file upload langsung)"]
    E --> F["Simpan file ke\nassets/images/community/"]
    F --> G["INSERT INTO human_archive\n(status='active')"]
    G --> H[✅ Anggota ditambahkan]

    C -- Edit --> I["Load data ke form"]
    I --> J["Admin ubah data"]
    J --> K{Ganti foto?}
    K -- Ya --> L["Hapus file lama\nSimpan file baru"]
    K -- Tidak --> M["Foto tetap sama"]
    L & M --> N["UPDATE human_archive\nSET... WHERE id=?"]
    N --> O[✅ Data diperbarui]

    C -- Toggle Status --> P{"Status saat ini?"}
    P -- active --> Q["UPDATE SET status='hidden'"]
    P -- hidden --> R["UPDATE SET status='active'"]

    C -- Hapus --> S["Hapus file foto\nDELETE FROM human_archive"]

    subgraph PUBLIK["🌐 Tampil di community.php"]
        T1["Query: SELECT * FROM human_archive\nWHERE status='active'\nORDER BY display_order ASC"]
        T2["Render kartu anggota\ndalam horizontal scroll"]
        T3["Custom scrollbar\n(drag-able thumb)"]
        T1 --> T2 --> T3
    end
```

---

## 10. Struktur Database (Entity Relationship)

```mermaid
erDiagram
    USER {
        int id_user PK
        varchar nama_lengkap
        varchar username
        varchar password
        text password_default
        enum role "admin|superadmin|karyawan"
        tinyint is_first_login
    }

    MENU {
        int id_menu PK
        varchar nama_menu
        enum kategori "Coffee|Non-Coffee|Snack|Main Course"
        int harga
        varchar foto
        text deskripsi
        enum status "Tersedia|Tidak Tersedia"
    }

    FEEDBACK {
        int id_feedback PK
        varchar nama_pelanggan
        int rating "1-5"
        text komentar
        enum status_moderasi "tampil|pending"
        datetime created_at
    }

    HUMAN_ARCHIVE {
        int id PK
        varchar name
        varchar role
        text quote
        varchar image
        int display_order
        enum status "active|hidden"
        datetime created_at
    }

    EVENTS {
        int id_event PK
        varchar judul_event
        text deskripsi_event
        date tanggal_event
        blob foto_cover
        enum status_event "mendatang|selesai"
        tinyint is_featured
        datetime created_at
    }

    GALLERY {
        int id_gallery PK
        varchar judul
        text deskripsi
        int id_event FK
        enum tipe "profil|event"
        mediumblob file_foto
    }

    GALLERY }o--o| EVENTS : "terhubung ke"
```

---

## 11. Arsitektur File & Folder

```mermaid
graph TD
    ROOT["📁 woelandari_coffeshop/"]

    ROOT --> IDX["index.php\n[Single Page App Entry]"]
    ROOT --> LGN["login.php\n[Autentikasi]"]
    ROOT --> LGO["logout.php\n[Destroy Session]"]
    ROOT --> GAL["gallery.php\n[Galeri Foto & Event]"]
    ROOT --> LOK["lokasi_full.php\n[Halaman Lokasi Full]"]

    ROOT --> SECTIONS["📄 Section Files"]
    SECTIONS --> s1["home.php"]
    SECTIONS --> s2["about.php"]
    SECTIONS --> s3["menu.php"]
    SECTIONS --> s4["community.php"]
    SECTIONS --> s5["reservasi.php"]
    SECTIONS --> s6["rating.php"]
    SECTIONS --> s7["lokasi.php"]

    ROOT --> COMP["📁 components/"]
    COMP --> c1["navbar.php"]
    COMP --> c2["sidebar.php"]
    COMP --> c3["popup.php"]

    ROOT --> CFG["📁 config/"]
    CFG --> cfg1["koneksi.php\n[DB Connection]"]

    ROOT --> DASH["📁 dashboard/"]
    DASH --> d1["dashboard.php"]
    DASH --> d2["menu_crud.php"]
    DASH --> d3["gallery_crud.php"]
    DASH --> d4["rating_crud.php"]
    DASH --> d5["community_crud.php"]
    DASH --> d6["user_manajemen.php"]
    DASH --> d7["akun_staff.php"]

    ROOT --> ASSETS["📁 assets/"]
    ASSETS --> CSS["📁 css/"]
    ASSETS --> JS["📁 js/\nresponsive.js"]
    ASSETS --> IMG["📁 images/"]
    IMG --> im1["📁 menu/"]
    IMG --> im2["📁 community/"]
    IMG --> im3["📁 gallery/"]
    IMG --> im4["📁 background/"]
    IMG --> im5["📁 gambar-mentahan/"]
```

---

## 12. Alur Reservasi via WhatsApp

```mermaid
sequenceDiagram
    participant P as 👤 Pengunjung
    participant W as 🌐 Website (reservasi.php)
    participant WA as 📱 WhatsApp

    P->>W: Buka section #reservasi
    W->>P: Tampilkan form reservasi
    P->>W: Isi: Nama, No. WA, Tanggal,\nJam, Durasi, Area, Jumlah Orang
    P->>W: Klik tombol RESERVASI

    W->>W: Validasi: semua field harus diisi
    alt Field tidak lengkap
        W->>P: Alert: "Harap lengkapi semua data!"
    else Semua field terisi
        W->>W: Generate pesan WhatsApp\nterformat (URL encoded)
        W->>WA: window.open(wa.me/adminNumber?text=...)
        WA->>P: Buka WhatsApp ke nomor admin\ndengan pesan pre-filled
        P->>WA: User kirim pesan ke admin
    end

    Note over WA: Admin konfirmasi\nreservasi secara manual
```

---

## 13. Alur Navigasi Navbar (Publik)

```mermaid
stateDiagram-v2
    [*] --> HOME : Buka index.php

    HOME --> MENU : Klik #menu
    HOME --> RESERVASI : Klik #reservasi
    HOME --> RATING : Klik #rating
    HOME --> GALLERY : Klik Gallery (halaman baru)
    HOME --> LOKASI_FULL : Klik Lokasi (halaman baru)

    MENU --> HOME : Scroll / Navigasi
    RESERVASI --> HOME : Scroll / Navigasi
    RATING --> HOME : Scroll / Navigasi

    GALLERY --> HOME : Klik Kembali
    LOKASI_FULL --> HOME : Navigasi navbar

    HOME --> LOGIN : Klik Logo (tersembunyi)
    LOGIN --> DASHBOARD : Login berhasil
    DASHBOARD --> HOME : Klik link publik
    DASHBOARD --> LOGIN : Logout
```

---

## Ringkasan Sistem

| Komponen | Detail |
|---|---|
| **Framework** | PHP Native + MySQL (MySQLi) |
| **Database** | `db_cafe` di localhost |
| **Tabel Utama** | `user`, `menu`, `feedback`, `human_archive`, `gallery`, `events` |
| **Auth** | Session PHP + Cookie (Remember Me 7 hari) |
| **Role** | `superadmin`, `admin`, `karyawan` |
| **Halaman Publik** | Single Page (index.php include section files) |
| **Halaman Terpisah** | `gallery.php`, `lokasi_full.php`, `login.php` |
| **Panel Admin** | `/dashboard/` folder, dilindungi session |
| **Upload Gambar** | Base64 crop (canvas) → file JPG/PNG ke server |
| **Reservasi** | Redirect ke WhatsApp (bukan menyimpan ke DB) |
| **Rating** | AJAX submit + load, filter bintang, moderasi admin |
| **CSS Versioning** | `filemtime()` untuk cache busting |
| **Responsive** | Navbar collapse + sidebar hamburger (mobile) |
