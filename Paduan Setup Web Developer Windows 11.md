# Panduan Setup Web Developer Ringan (Windows 11 26H2 + Scoop)

**Target Spesifikasi:** Intel Core 3 100U | RAM 8GB LPDDR5 | SSD 512GB | Windows 11 Home (Fresh Reset)
**Stack:** PowerShell 7 + Oh My Posh, Node.js + pnpm, PHP + Composer (Laravel), SQLite, dan VS Code.

---

## Tahap 1: Instalasi PowerShell 7 & Windows Terminal

Windows 11 secara bawaan masih menggunakan Windows PowerShell 5.1 yang jadul. Kita akan menginstal **PowerShell 7** dari Microsoft Store agar lebih cepat dan mendukung fitur modern.

1. Buka aplikasi **Microsoft Store** di Windows 11.
2. Cari dan instal **PowerShell** (penerbit: *Microsoft Corporation*, ikon hitam dengan simbol `>_`).
3. Pastikan **Windows Terminal** sudah terpasang (biasanya bawaan Windows 11; jika belum, instal juga dari Microsoft Store).
4. Buka **Windows Terminal** -> klik ikon panah bawah (`v`) di bilah tab atas -> **Settings** -> pada bagian **Default profile**, ubah menjadi **PowerShell** (yang berikon hitam, bukan biru) -> klik **Save**.
5. Tutup dan buka kembali Windows Terminal.

---

## Tahap 2: Instalasi Scoop & Bucket Utama

[Scoop](https://scoop.sh/) menginstal aplikasi secara portabel ke folder `C:\Users\<NamaUser>\scoop` tanpa *pop-up* UAC dan tanpa menyampah di Registry Windows.

Jalankan perintah berikut satu per satu di **PowerShell 7**:

```powershell
# 1. Izinkan eksekusi skrip lokal untuk user saat ini
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 2. Instal Scoop
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

# 3. Instal Git (Wajib untuk menambah bucket Scoop dan kebutuhan developer)
scoop install git

# 4. Tambahkan bucket 'extras' (untuk VS Code, Oh My Posh) dan 'php' (untuk versi PHP)
scoop bucket add extras
scoop bucket add php
```

*(Opsional)* Instal `aria2` agar unduhan paket Scoop jauh lebih cepat:
```powershell
scoop install aria2
scoop config aria2-warning-enabled false
```

---

## Tahap 3: Setup Tema Terminal & Alias Ala Linux

Agar terminal nyaman dilihat dan memiliki perintah seperti Linux (`ls`, `ll`, `touch`, `grep`, `cat`, `rm -rf`), kita akan memasang **Oh My Posh** dan **Nerd Font**.

### 3.1 Instal Oh My Posh & Font Ikon
Jalankan di PowerShell 7:
```powershell
scoop install oh-my-posh
scoop bucket add nerd-fonts
scoop install CascadiaCode-NF
```

**Mengaktifkan Font di Windows Terminal:**
1. Buka **Settings** di Windows Terminal (`Ctrl + ,`).
2. Di panel kiri bawah, klik **Defaults** (di bawah *Profiles*) -> pilih **Appearance**.
3. Ubah **Font face** menjadi `CaskaydiaCove Nerd Font` -> klik **Save**.

### 3.2 Konfigurasi PowerShell `$PROFILE`
Buat dan buka file profil PowerShell:
```powershell
# Buat file profil jika belum ada
if (!(Test-Path -Path $PROFILE)) {
    New-Item -ItemType File -Path $PROFILE -Force
}

# Buka file profil menggunakan Notepad
notepad $PROFILE
```

Salin dan tempel seluruh kode di bawah ini ke dalam **Notepad**, lalu simpan (`Ctrl + S`) dan tutup Notepad:

```powershell
# ==========================================
# 1. TEMA OH MY POSH (Ringan & Cepat)
# ==========================================
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\paradox.omp.json" | Invoke-Expression

# ==========================================
# 2. AUTOCOMPLETE & HISTORY ALA ZSH/FISH
# ==========================================
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

# ==========================================
# 3. ALIAS & FUNGSI PERINTAH ALA LINUX
# ==========================================
# Menghapus alias bawaan PowerShell yang bentrok
Remove-Item Alias:ls -Force -ErrorAction SilentlyContinue
Remove-Item Alias:cat -Force -ErrorAction SilentlyContinue
Remove-Item Alias:rm -Force -ErrorAction SilentlyContinue
Remove-Item Alias:cp -Force -ErrorAction SilentlyContinue
Remove-Item Alias:mv -Force -ErrorAction SilentlyContinue

# Navigasi & Manajemen File
function ls { Get-ChildItem -Force @args }
function ll { Get-ChildItem -Force @args | Format-Table Mode, LastWriteTime, Length, Name -AutoSize }
function la { Get-ChildItem -Force -Hidden @args }
function touch ($file) { if (Test-Path $file) { (Get-Item $file).LastWriteTime = Get-Date } else { New-Item -ItemType File -Path $file | Out-Null } }
function mkdirp ($dir) { New-Item -ItemType Directory -Path $dir -Force | Out-Null }
function rmrf ($target) { Remove-Item -Path $target -Recurse -Force }
function cat ($file) { Get-Content -Path $file }
function grep ($pattern, $path) { Select-String -Pattern $pattern -Path $path }
function which ($cmd) { (Get-Command $cmd -ErrorAction SilentlyContinue).Source }
function open ($path = ".") { Invoke-Item $path }

# Shortcut Developer (Git, Laravel, pnpm)
function gs { git status }
function ga { git add . }
function gc ($msg) { git commit -m $msg }
function gp { git push }
function art { php artisan @args }
function pn { pnpm @args }
```

Terapkan konfigurasi profil sekarang:
```powershell
. $PROFILE
```

---

## Tahap 4: Instalasi Node.js & pnpm (Hemat Storage)

Dengan [pnpm](https://pnpm.io/id/11.x/installation), seluruh *library* `node_modules` disimpan dalam satu *global store* menggunakan *hard link*, sehingga proyek-proyek Anda tidak akan memakan banyak ruang SSD 512GB.

```powershell
# Instal Node.js versi LTS (stabil) dan pnpm
scoop install nodejs-lts pnpm

# Setup direktori global pnpm agar tidak error saat instal paket global
pnpm setup
```
*(Catatan: Setelah menjalankan `pnpm setup`, tutup dan buka kembali Windows Terminal agar `PNPM_HOME` terbaca).*

---

## Tahap 5: Instalasi PHP & Composer (Untuk Laravel)

Kita akan menginstal **PHP** beserta **Composer** melalui Scoop, lalu mengaktifkan ekstensi wajib yang dibutuhkan oleh Laravel.

```powershell
# Instal PHP terbaru dan Composer
scoop install php composer
```

### Mengaktifkan Ekstensi Wajib Laravel di `php.ini`
Paket PHP dari Scoop secara otomatis menyimpan file konfigurasi `php.ini` di folder persisten agar tidak hilang saat update. Jalankan skrip PowerShell berikut untuk mengaktifkan ekstensi standar Laravel (`curl`, `fileinfo`, `gd`, `mbstring`, `openssl`, `pdo_sqlite`, `pdo_mysql`, `zip`) secara otomatis tanpa salah ketik:

```powershell
$phpIni = "$env:USERPROFILE\scoop\persist\php\cli\php.ini"
$extensions = @("curl", "fileinfo", "gd", "mbstring", "openssl", "pdo_mysql", "pdo_sqlite", "sqlite3", "zip")

$content = Get-Content $phpIni
foreach ($ext in $extensions) {
    $content = $content -replace "^;extension=$ext", "extension=$ext"
}
$content | Set-Content $phpIni
Write-Host "Ekstensi PHP untuk Laravel berhasil diaktifkan!" -ForegroundColor Green
```

---

## Tahap 6: Instalasi Database (Rekomendasi: SQLite)

Untuk spesifikasi **RAM 8GB LPDDR5**, **SQLite** adalah pilihan terbaik karena:
- **0 MB RAM idle:** Tidak ada *background service* yang menyala terus-menerus seperti MySQL/MariaDB.
- **Standar bawaan Laravel:** Sejak Laravel 11 & 12, SQLite sudah menjadi database *default*.

```powershell
# Instal SQLite CLI
scoop install sqlite
```

*(Opsional - Hanya jika proyek lama wajib memakai MySQL)*: Jika suatu saat Anda terpaksa memakai MySQL, jangan instal MySQL Server yang berat—gunakan **MariaDB** versi portabel via Scoop dan nyalakan hanya saat ngoding:
```powershell
# JANGAN jalankan ini jika SQLite sudah cukup:
# scoop install mariadb
# Menyalakan manual saat butuh: mysqld --console
```

---

## Tahap 7: Instalasi Visual Studio Code (VS Code)

Instal [VS Code](https://code.visualstudio.com/) melalui bucket `extras` Scoop dan tambahkan integrasi menu klik kanan Windows (*Open with Code*):

```powershell
# Instal VS Code
scoop install vscode

# Daftarkan menu klik kanan "Open with Code" di File Explorer
reg import "$env:USERPROFILE\scoop\apps\vscode\current\install-context.reg"
```

### Tips Hemat RAM di VS Code (Untuk RAM 8GB):
1. Buka VS Code (`code .`), lalu instal hanya ekstensi esensial:
   - **Laravel Extension Pack** atau **PHP Intelephense** (untuk PHP/Laravel)
   - **ESLint** & **Prettier** (untuk JS/Frontend)
   - **SQLite Viewer** (untuk melihat isi database SQLite langsung di VS Code tanpa aplikasi tambahan)
2. Buka Settings VS Code (`Ctrl + ,`), cari `files.watcherExclude`, dan pastikan folder `**/node_modules/**` serta `**/vendor/**` masuk daftar pengecualian agar CPU Intel Core 3 tidak bekerja keras memindai ribuan file.

---

## Tahap 8: Uji Coba & Verifikasi Sistem

Jalankan blok perintah verifikasi berikut di **Windows Terminal (PowerShell 7)** untuk memastikan seluruh paket terpasang dengan benar dan bebas error:

```powershell
Write-Host "=== CEK VERSI ENVIRONMENT ===" -ForegroundColor Cyan
$PSVersionTable.PSVersion
git --version
node -v
pnpm -v
php -v
composer -V
sqlite3 --version
code --version

Write-Host "`n=== CEK EKSTENSI PHP LARAVEL ===" -ForegroundColor Cyan
php -m | Select-String -Pattern "curl|fileinfo|gd|mbstring|openssl|pdo_sqlite|pdo_mysql|zip"

Write-Host "`n=== CEK ALIAS LINUX ===" -ForegroundColor Cyan
which php
which node
```

### Hasil yang Diharapkan:
1. Seluruh perintah versi menampilkan angka versi masing-masing tanpa pesan *command not found*.
2. Daftar ekstensi PHP menampilkan ke-8 modul (`curl`, `fileinfo`, `gd`, `mbstring`, `openssl`, `pdo_mysql`, `pdo_sqlite`, `zip`).
