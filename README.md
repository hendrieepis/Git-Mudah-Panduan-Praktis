# Git Mudah — Panduan Praktis untuk Mahasiswa

> Panduan ini fokus ke workflow sehari-hari: setup sekali, pakai terus. Tidak perlu hafal semua perintah git — cukup yang sering dipakai.

---

## Kenapa Panduan Ini Ada

Jujur, ini lahir dari frustrasi sendiri.

Pertama kali pakai git, semua terasa membingungkan. Buat repo baru saja harus buka browser, login GitHub, klik sana-sini, copy URL, balik ke terminal, paste. Belum lagi kalau lupa setup nama dan email — tiba-tiba commit gagal dengan pesan error yang tidak jelas. Atau panik karena kerja di kampus, balik ke rumah, eh file di komputer rumah masih versi lama.

Lama-lama ketemu solusi satu per satu. VSCode ternyata bisa publish repo langsung. Eh, ternyata ada `gh` yang lebih simpel lagi. Ternyata masalah multi-komputer itu cuma soal disiplin `pull` dan `push`.

Masalahnya, semua penemuan itu tersebar — dari Stack Overflow, dari forum, dari coba-coba sendiri. Tidak ada satu tempat yang ceritanya runut dari nol sampai bisa jalan lancar.

Panduan ini adalah kumpulan hal-hal yang **harusnya diketahui dari awal** — supaya kamu tidak perlu frustrasi dulu baru nemu jawabannya.

---

## Daftar Isi

1. [Setup Awal (Lakukan Sekali Saja)](#1-setup-awal-lakukan-sekali-saja)
2. [Generate Credential GitHub](#2-generate-credential-github)
3. [Buat Repo Baru — Dari VSCode sampai Nemu gh](#3-buat-repo-baru--dari-vscode-sampai-nemu-gh)
4. [Kerja di Banyak Komputer](#4-kerja-di-banyak-komputer)
5. [Troubleshooting Umum](#5-troubleshooting-umum)
6. [Referensi Cepat](#6-referensi-cepat)

---

## 1. Setup Awal (Lakukan Sekali Saja)

Sebelum bisa pakai git, kamu perlu kasih tahu siapa kamu. Ini dilakukan **satu kali per komputer/perangkat**.

```bash
git config --global user.email "emailkamu@gmail.com"
git config --global user.name "Nama Kamu"
git config --global init.defaultBranch main
```

Verifikasi hasilnya:

```bash
cat ~/.gitconfig
```

Output yang benar:

```
[user]
    email = emailkamu@gmail.com
    name = Nama Kamu
[init]
    defaultBranch = main
```

> **Kenapa perlu `init.defaultBranch main`?**
> Setiap kali buat repo baru dengan `git init`, Git perlu tahu nama branch pertamanya apa. Kalau belum di-set, warning akan muncul di tiap repo baru yang kamu buat di komputer ini. Set sekali, beres permanent.

---

## 2. Generate Credential GitHub

Sebelum bisa push ke GitHub, komputer kamu perlu punya "tanda pengenal" yang dikenali GitHub. Tanpa ini, setiap kali push akan diminta username dan password — lama-lama menyebalkan.

Solusinya pakai **SSH key** — daftar sekali, setelah itu push tanpa ditanya password lagi selamanya.

**Langkah 1 — Generate SSH key:**

```bash
ssh-keygen -t ed25519 -C "emailkamu@gmail.com"
```

Tekan Enter saja untuk semua pertanyaan (path default, passphrase kosong). Nanti terbuat dua file di `~/.ssh/`:
- `id_ed25519` → private key (jangan dibagikan ke siapapun)
- `id_ed25519.pub` → public key (ini yang didaftarkan ke GitHub)

**Langkah 2 — Tambahkan key ke SSH agent:**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**Langkah 3 — Lihat public key, lalu copy:**

```bash
cat ~/.ssh/id_ed25519.pub
```

Salin hasilnya — mulai dari `ssh-ed25519 ...` sampai akhir.

**Langkah 4 — Daftarkan ke GitHub:**

- Login ke GitHub
- Klik foto profil (pojok kanan atas) → **Settings**
- Pilih **SSH and GPG keys** → klik **New SSH key**
- Paste public key tadi, klik **Add SSH key**

**Langkah 5 — Test koneksi:**

```bash
ssh -T git@github.com
```

Kalau sukses, muncul:

```
Hi usernamekamu! You've successfully authenticated, but GitHub does not provide shell access.
```

Setelah ini, setiap push tidak akan minta password lagi.

---

## 3. Buat Repo Baru — Dari VSCode sampai Nemu `gh`

### Awalnya pakai VSCode

Waktu pertama kali mau push repo baru ke GitHub, cara yang paling umum ditemukan adalah lewat browser — buka github.com, klik "New repository", isi nama, copy URL, baru jalankan `git remote add origin`. Ribet, tapi ya itulah caranya yang paling banyak diajarkan.

Sampai ketemu solusi yang lebih praktis: **VSCode**. Setelah `git init`, `git add .`, `git commit`, tinggal jalankan:

```bash
code .
```

Masuk VSCode, klik icon Source Control (icon bercabang di sidebar kiri), lalu klik **Publish to GitHub**. Nanti muncul pilihan nama repo (biasanya diambil otomatis dari nama folder) dan opsi public atau private. Klik, selesai — repo langsung terbuat dan ter-push.

Lumayan praktis. Tidak perlu buka browser, tidak perlu copy-paste URL.

---

### Tapi ternyata masih ada yang lebih simpel: `gh`

Masalahnya, pakai VSCode berarti harus switch dulu dari terminal ke GUI, hanya untuk satu langkah itu saja. Kalau lagi asyik di terminal, ini sedikit ganggu flow.

Ternyata ada tool namanya **GitHub CLI** (`gh`) yang melakukan hal yang sama persis seperti yang VSCode lakukan di balik layar — tapi langsung dari terminal, tanpa buka VSCode atau browser sama sekali.

**Install sekali:**

```bash
sudo apt install gh
```

**Login `gh` sekali:**

```bash
gh auth login
```

Nanti muncul prompt interaktif seperti ini — gunakan tombol panah untuk navigasi, Enter untuk pilih:

```
? What account do you want to log into?
> GitHub.com
  GitHub Enterprise Server
```
Pilih `GitHub.com`.

```
? What is your preferred protocol for Git operations on this host?
> HTTPS
  SSH
```
Pilih `SSH` kalau kamu sudah punya SSH key. Kalau belum yakin, pilih `HTTPS` saja dulu, aman.

```
? Upload your SSH public key to your GitHub account? /home/hendri/.ssh/id_ed25519.pub
```
Kalau muncul ini (berarti SSH key sudah ada), tekan Enter untuk upload otomatis ke GitHub.

```
? Title for your SSH key: GitHub CLI
```
Ini nama label SSH key-nya di GitHub. Biarkan default, tekan Enter.

```
? How would you like to authenticate GitHub CLI?
> Login with a web browser
  Paste an authentication token
```
Pilih `Login with a web browser`.

Terminal akan menampilkan **kode 8 karakter**:

```
! First copy your one-time code: E234-28DC
Press Enter to open github.com in your browser...
```

Tekan Enter, browser otomatis terbuka ke **github.com/login/device**. Masukkan kodenya, klik Authorize. Balik ke terminal — kalau sukses muncul:

```
✓ Authentication complete.
✓ Configured git protocol
✓ SSH key already existed on your GitHub account: /home/hendri/.ssh/id_ed25519.pub
✓ Logged in as usernamekamu
```

Sudah authenticated dan siap dipakai.

---

**Setelah itu, workflow buat repo baru jadi semudah ini:**

```bash
git init
git add .
git commit -m "first commit"
gh repo create nama-repo --public --source=. --push
```

Satu perintah `gh repo create` — repo otomatis terbuat di GitHub dan langsung ter-push.

> ⚠️ **Jangan sampai kebalik urutannya!**
> `gh repo create --push` butuh minimal satu commit. Kalau dipanggil sebelum `git commit`, akan error:
> ```
> `--push` enabled but no commits found
> ```
> Selalu pastikan sudah `git commit` dulu sebelum `gh repo create`.

> ⚠️ **`gh auth login` harus dilakukan di tiap komputer baru**
> Sama seperti `git config --global`, login `gh` tidak otomatis tersync ke komputer lain. Kalau lupa login dan langsung jalankan `gh repo create`, akan muncul:
> ```
> HTTP 401: Requires authentication
> Try authenticating with:  gh auth login
> ```
> Solusinya jalankan `gh auth login` dulu, baru ulangi perintah `gh repo create`.

**Contoh nyata** — kamu punya folder `esp32-sensor-suhu`:

```bash
cd esp32-sensor-suhu
git init
git add .
git commit -m "first commit"
gh repo create esp32-sensor-suhu --public --source=. --push
```

Kalau sukses, output-nya seperti ini:

```
✓ Created repository usernamekamu/esp32-sensor-suhu on GitHub
  https://github.com/usernamekamu/esp32-sensor-suhu
✓ Added remote git@github.com:usernamekamu/esp32-sensor-suhu.git
✓ Pushed commits to git@github.com:usernamekamu/esp32-sensor-suhu.git
```

Repo sudah online dan tidak perlu `git push` lagi — sudah ter-push sekaligus.

---

## 4. Kerja di Banyak Komputer

Ini skenario yang sangat umum, dan kalau belum tahu caranya bisa bikin frustrasi.

Misalnya kamu mulai project di **Pi rumah**, sudah lumayan jauh progressnya. Besok berangkat kampus, mau lanjut kerja di sana. Bagaimana caranya supaya kerjaan di rumah bisa dilanjut di kampus — tanpa copy-paste lewat flashdisk atau kirim file lewat WhatsApp?

Jawabannya: **push dulu sebelum pergi, pull dulu sebelum mulai**.

### Pola dasar — wajib diingat

```
Mau mulai kerja  →  git pull
Selesai kerja    →  git add . && git commit -m "pesan" && git push
```

### Skenario lengkap

**Situasi:**
- 🏠 Single Board Computer (Raspberry Pi 5) di rumah — selanjutnya disebut **Pi5** — tempat kamu mulai project
- 🏫 Komputer di kampus → mau lanjut kerja di sana

---

**🏠 Di Pi5 rumah — sebelum berangkat kampus:**

```bash
git add .
git commit -m "progress: selesai bagian sensor DHT22"
git push origin main
```

Pastikan semua perubahan sudah ter-push sebelum pergi.

---

**🏫 Di komputer kampus — pertama kali (belum pernah clone di sini):**

```bash
git clone https://github.com/usernamekamu/nama-repo.git
cd nama-repo
```

Lanjut kerja seperti biasa.

---

**🏫 Di komputer kampus — sudah pernah clone sebelumnya:**

```bash
cd nama-repo
git pull origin main
```

Selalu pull dulu sebelum mulai kerja, untuk mengambil perubahan terbaru dari Pi5.

---

**🏫 Di komputer kampus — setelah selesai kerja:**

```bash
git add .
git commit -m "tambah kalibrasi sensor di lab kampus"
git push origin main
```

---

**🏠 Balik ke Pi5 rumah:**

```bash
cd nama-repo
git pull origin main
```

Semua perubahan dari kampus langsung tersync ke Pi5.

---

### Apa yang terjadi kalau lupa pull?

Kalau kamu edit file di rumah **tanpa pull dulu** setelah kerja di kampus, saat push akan muncul error:

```
! [rejected] main -> main (fetch first)
error: failed to push some refs
```

Artinya GitHub punya commit yang belum ada di komputermu. Solusinya:

```bash
git pull origin main   # sync dulu
# kalau ada conflict, selesaikan dulu
git push origin main   # baru push
```

---

## 5. Troubleshooting Umum

### ❌ Error: "Author identity unknown"

Ini yang sering bikin bingung pertama kali, tiba-tiba muncul pas mau commit:

```
Author identity unknown

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

fatal: unable to auto-detect email address (got 'hendri@my-pi5-athome.(none)')
```

**Penyebab:** Git belum tahu siapa kamu di komputer ini.

**Solusi:**

```bash
git config --global user.email "emailkamu@gmail.com"
git config --global user.name "Nama Kamu"
```

**Kalau masih muncul di repo berikutnya**, kemungkinannya dua:
- Ada typo waktu setup pertama kali
- Waktu setup pakai `--local` bukan `--global` tanpa sadar, jadi hanya berlaku di repo itu saja

Cek dulu:

```bash
cat ~/.gitconfig
```

Ini contoh output yang bikin bingung — sekilas terlihat sudah benar, tapi ternyata ada yang kurang:

```
hendri@my-pi5-athome:~ $ cat ~/.gitconfig
[user]
    email = hendrieepis@gmail.com
    name = Hendri
hendri@my-pi5-athome:~ $
```

Nah ketahuan — `user.email` dan `user.name` sudah ada, tapi `init.defaultBranch` belum. Jadi warning branch name itu akan terus muncul tiap `git init` buat repo baru. Padahal sudah merasa setup-nya beres. Frustrasi kan?

Fix-nya tambahkan satu baris ini:

```bash
git config --global init.defaultBranch main
```

Cek lagi sampai hasilnya seperti ini — baru beres permanent:

```
[user]
    email = hendrieepis@gmail.com
    name = Hendri
[init]
    defaultBranch = main
```

---

### ❌ Push ditolak: "fetch first"

```
! [rejected] main -> main (fetch first)
```

**Penyebab:** Remote punya commit yang belum ada di lokal. Biasanya karena kerja di komputer lain tapi lupa push, atau push dari tempat lain tapi lupa pull di sini.

**Solusi:**

```bash
git pull origin main
git push origin main
```

---

## 6. Referensi Cepat

### Workflow harian

```bash
# Mulai kerja
git pull origin main

# Selesai kerja
git add .
git commit -m "deskripsi singkat perubahan"
git push origin main
```

### Buat repo baru

```bash
git init
git add .
git commit -m "first commit"
gh repo create nama-repo --public --source=. --push
```

### Perintah `gh` yang berguna

```bash
gh repo list              # lihat semua repo kamu di GitHub
gh repo view --web        # buka repo aktif di browser
gh repo clone nama-repo   # clone repo tanpa copy-paste URL
```

### Cek status

```bash
git status                # lihat file apa yang berubah
git log --oneline -5      # lihat 5 commit terakhir
cat ~/.gitconfig          # lihat konfigurasi git global
```

---

> **Tips terakhir:** Git itu seperti save game. `commit` = save, `push` = upload ke cloud, `pull` = download save terbaru. Biasakan push setiap selesai sesi kerja, dan pull setiap mulai sesi baru — di komputer manapun.
