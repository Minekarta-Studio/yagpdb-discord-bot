# Setup YAGPDB di Dokploy

Konfigurasi ini untuk **Docker Compose** dengan source Git. Jalankan YAGPDB dengan aplikasi Discord baru saat Rostra masih digunakan. YAGPDB menyimpan data sendiri di PostgreSQL dan Redis; tidak ada migrasi otomatis dari Rostra.

## 1. Siapkan aplikasi Discord dan domain

1. Buat **aplikasi Discord baru** di https://discord.com/developers/applications. Catat Application ID (`YAGPDB_CLIENTID`), Bot Token (`YAGPDB_BOTTOKEN`), dan OAuth2 Client Secret (`YAGPDB_CLIENTSECRET`). `YAGPDB_OWNER` adalah **ID akun Discord milikmu**, bukan Application ID.
2. Di halaman **Bot**, aktifkan privileged intents **Server Members**, **Message Content**, dan **Presence** sesuai panduan YAGPDB. Jangan jalankan dua bot bersamaan dengan token aplikasi yang sama.
3. Tentukan subdomain dashboard yang mengarah ke Dokploy, misalnya `bot.example.com`. Isi `YAGPDB_HOST=bot.example.com` **tanpa** `https://` dan tanpa path. Gunakan nama host final yang sama di Dokploy dan Discord.
4. Pada **OAuth2 → Redirects** di Developer Portal, tambahkan **persis** `https://bot.example.com/confirm_login` dan `https://bot.example.com/manage`. Ganti hostname dengan subdomainmu.

## 2. Siapkan source dan Compose

1. Fork https://github.com/botlabs-gg/yagpdb ke akun GitHub sendiri.
2. File `compose.dokploy.yml` sudah berada di **root repo fork**. Pastikan file `yagpdb_docker/Dockerfile` tetap berada di repo. Jangan unggah file yang sudah diisi token/password.
3. Di Dokploy, buat layanan **Docker Compose** dengan Git source menuju fork tersebut. Pilih **Docker Compose**, bukan Docker Stack, karena file ini memakai `build:`. Atur Compose path ke `compose.dokploy.yml`.
4. Pada **Environment** layanan Compose, isi keenam variabel dari `yagpdb.env.example`. Ganti semua contoh dengan nilai sebenarnya. Untuk password database, gunakan hasil `openssl rand -hex 32` (tanpa tanda kutip).
5. Pada **Domains**, pasang hostname yang sama ke **service `app`, container port `80`, HTTPS aktif**. Biarkan `db` dan `redis` tanpa domain maupun port publik. Jangan petakan `80:80` atau `443:443` pada file Compose: kedua port publik itu dipakai proxy Dokploy.
6. Tekan **Deploy**. Build awal mengompilasi source Go dan bisa lebih lama dari deployment image siap pakai.

## 3. Verifikasi

- Buka `https://bot.example.com` dan login dengan Discord. Setelah login, pilih server dan buka pengaturannya. Jika panel muncul tetapi setiap penyimpanan gagal dengan `Bad origin`, pastikan `YAGPDB_HOST` sama persis dengan domain dan Compose menjalankan `-https=false` serta `-exthttps=true`.
- Undang bot dari alur control panel. Setelah bot hadir di server, pastikan konfigurasi welcome dan tiket bisa dibuka dan disimpan. Pada versi source yang ditinjau, halaman tiket memakai pemeriksaan premium; Compose ini menyetel `YAGPDB_PREMIUM_ALL_GUILDS_PREMIUM=true` untuk instalasi self hosted.
- Pastikan log `app` menunjukkan bot tersambung dan tidak ada error migrasi database; `db` dan `redis` harus healthy. Jika dashboard tidak terbuka, cek domain Dokploy diarahkan ke service `app` port `80`.
- Uji bot baru sebelum menonaktifkan Rostra. Kedua bot punya data terpisah, jadi pengaturan Rostra tidak ikut berpindah.

## Catatan sumber

- [Source YAGPDB dan petunjuk self hosting](https://github.com/botlabs-gg/yagpdb/blob/master/README.md)
- [Pengaturan domain, HTTPS, dan OAuth](https://github.com/botlabs-gg/yagpdb-docs-v2/blob/master/content/selfhosting/hosting/setup.md)
- [Dokploy Compose: environment](https://docs.dokploy.com/docs/core/docker-compose)
- [Dokploy Compose: domains](https://docs.dokploy.com/docs/core/docker-compose/domains)

Konfigurasi ini sudah diperiksa struktur YAML dan konsistensi variabelnya. Deployment live tetap membutuhkan domain dan kredensial milikmu; belum dijalankan di server Dokploy milikmu.
