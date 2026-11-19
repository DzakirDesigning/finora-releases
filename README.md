# FINORA Releases (`DzakirDesigning/finora-releases`)

Repository publik **khusus distribusi pembaruan aplikasi Android FINORA** (**FINORA Tab** dan **FINORA Mobile**).

> **PERINGATAN KEAMANAN & RUANG LINGKUP**
> - Repository ini **HANYA** untuk distribusi publik metadata updater (`latest.json`) dan binary APK melalui **GitHub Releases Assets**.
> - **DILARANG KERAS** menyimpan source code aplikasi, migrasi database, konfigurasi backend/Supabase, Cloudflare config, API token, GitHub PAT, password, keystore (`.jks`/`.keystore`), atau secret apa pun di repository ini.
> - **DILARANG** meng-commit file binary `.apk` ke dalam Git history — seluruh file `.apk` hanya boleh diunggah sebagai **GitHub Release Assets**.
> - Aplikasi Android mengakses repository publik ini secara **read-only tanpa GitHub token** di sisi frontend.

---

## 1. Struktur Repository

```text
finora-releases/
  README.md
  .gitignore
  updates/
    tab/
      README.md
      latest.json              # Dibuat & diperbarui HANYA setelah APK Tab tersedia di GitHub Releases
    mobile/
      README.md
      latest.json              # Dibuat & diperbarui HANYA setelah APK Mobile tersedia di GitHub Releases
  templates/
    latest.tab.example.json    # Template placeholder (JANGAN dipublikasikan langsung sebagai metadata aktif)
    latest.mobile.example.json # Template placeholder (JANGAN dipublikasikan langsung sebagai metadata aktif)
```

---

## 2. Raw GitHub URL Metadata Updater

Aplikasi Android FINORA membaca metadata versi terbaru dari Raw GitHub URL berikut:

- **FINORA Tab (`com.finora.pos.tab`)**:
  ```text
  https://raw.githubusercontent.com/DzakirDesigning/finora-releases/main/updates/tab/latest.json
  ```
- **FINORA Mobile (`com.finora.pos.mobile`)**:
  ```text
  https://raw.githubusercontent.com/DzakirDesigning/finora-releases/main/updates/mobile/latest.json
  ```

### Konfigurasi Environment Variable Build Android (Repository Privat `kasir-umkm`)
Saat membangun APK Android (hanya pada sesi build Android, **tidak** pada build Web):
```env
VITE_ANDROID_UPDATE_BASE_URL=https://raw.githubusercontent.com/DzakirDesigning/finora-releases/main
```
Updater otomatis memetakan:
- Target `tab` → `<BASE_URL>/updates/tab/latest.json`
- Target `mobile` → `<BASE_URL>/updates/mobile/latest.json`
- Target `web` → nonaktif (`null`)

---

## 3. Konvensi Tag & Penamaan File APK Release Assets

- **Tag GitHub Release**: `v<versionName>` (contoh: `v1.0.1`, `v1.0.2`), sinkron dengan `android/version.properties` pada repository utama (`VERSION_CODE` naik secara monoton: `10001`, `10002`, dst.).
- **Nama File Asset APK**:
  - FINORA Tab: `finora-tab-<versionName>.apk` (contoh: `finora-tab-1.0.1.apk`)
  - FINORA Mobile: `finora-mobile-<versionName>.apk` (contoh: `finora-mobile-1.0.1.apk`)
- **Format URL Download Asset GitHub Releases**:
  - Tab: `https://github.com/DzakirDesigning/finora-releases/releases/download/v<versionName>/finora-tab-<versionName>.apk`
  - Mobile: `https://github.com/DzakirDesigning/finora-releases/releases/download/v<versionName>/finora-mobile-<versionName>.apk`

---

## 4. Kontrak Field Metadata (`latest.json`)

Seluruh field wajib mengikuti validasi ketat pada `src/appUpdaterTypes.ts` dan `src/appUpdaterCore.ts`:

| Field | Tipe JSON | Aturan Validasi |
|---|---|---|
| `target` | `string` | Wajib `"tab"` atau `"mobile"` sesuai channel |
| `versionName` | `string` | Format SemVer (contoh `"1.0.1"`) |
| `versionCode` | `number` (integer) | Bilangan bulat positif (`> installed.versionCode`) |
| `apkUrl` | `string` | Wajib `https://` dan berakhiran `.apk` |
| `apkSize` | `number` (integer) | Ukuran byte eksak file APK (`102400` s/d `524288000`) |
| `sha256` | `string` | 64 karakter heksadesimal lowercase SHA-256 file APK |
| `releaseNotes` | `string[]` | Daftar catatan rilis untuk ditampilkan di dialog pembaruan |
| `minimumSupportedVersionCode` | `number` (integer) | `>= 0` dan `<= versionCode` (jika terpasang lebih rendah, update menjadi wajib) |
| `mandatoryUpdate` | `boolean` | `true` untuk update wajib, `false` untuk opsional |
| `publishedAt` *(opsional)* | `string` | Timestamp publikasi ISO-8601 UTC |

> **Penting:** File di dalam `templates/` menggunakan placeholder string agar tidak bisa lolos validator secara tidak sengaja. Saat menyusun `updates/tab/latest.json` dan `updates/mobile/latest.json`, pastikan `versionCode`, `apkSize`, dan `minimumSupportedVersionCode` ditulis sebagai **angka JSON murni (tanpa tanda kutip)**.

---

## 5. Cara Menghitung Ukuran & SHA-256 File APK (PowerShell)

Jalankan pada mesin build lokal (dari lokasi hasil build Release di repository utama):

```powershell
# FINORA Tab Release
$tabApk = Get-Item "D:\kasir-umkm\android\app\build\outputs\apk\tab\release\app-tab-release.apk"
Write-Output "TAB_SIZE=$($tabApk.Length)"
Write-Output "TAB_SHA256=$((Get-FileHash $tabApk.FullName -Algorithm SHA256).Hash.ToLower())"

# FINORA Mobile Release
$mobApk = Get-Item "D:\kasir-umkm\android\app\build\outputs\apk\mobile\release\app-mobile-release.apk"
Write-Output "MOBILE_SIZE=$($mobApk.Length)"
Write-Output "MOBILE_SHA256=$((Get-FileHash $mobApk.FullName -Algorithm SHA256).Hash.ToLower())"
```

---

## 6. Prosedur & Urutan Publikasi Rilis Baru (WAJIB BERURUTAN)

> **Aturan Emas:** **Unggah & verifikasi APK di GitHub Releases terlebih dahulu**, baru perbarui `updates/tab/latest.json` dan `updates/mobile/latest.json`.

### Tahap 1 — Siapkan & Verifikasi APK Release Lokal
1. Naikkan `VERSION_NAME` dan `VERSION_CODE` di `android/version.properties` (repository utama).
2. Build `app-tab-release.apk` (alias `finora-tab`) dan `app-mobile-release.apk` (alias `finora-mobile`) dengan `VITE_ANDROID_UPDATE_BASE_URL` diset ke Raw GitHub URL.
3. Verifikasi `applicationId`, `versionCode`, `versionName`, dan sertifikat signing kedua APK menggunakan `aapt2` dan `apksigner`.
4. Salin sementara (untuk upload saja, **jangan** commit ke Git) dengan nama:
   - `finora-tab-<versionName>.apk`
   - `finora-mobile-<versionName>.apk`
5. Catat `apkSize` (bytes) dan `sha256` (64-hex lowercase) masing-masing file.

### Tahap 2 — Buat GitHub Release & Unggah APK Assets
1. Buka repository `DzakirDesigning/finora-releases` → **Releases → Draft a new release**.
2. Buat tag baru `v<versionName>` (mis. `v1.0.1`) pada branch `main`, isi judul & ringkasan rilis.
3. Unggah kedua file `.apk` pada bagian **Attach binaries**:
   - `finora-tab-<versionName>.apk`
   - `finora-mobile-<versionName>.apk`
4. Klik **Publish release**.

### Tahap 3 — Verifikasi Unduhan APK Asset Sebelum Update Metadata
Uji kedua URL asset dengan mengikuti redirect HTTPS (`-IL`):
```powershell
curl.exe -IL "https://github.com/DzakirDesigning/finora-releases/releases/download/v1.0.1/finora-tab-1.0.1.apk"
curl.exe -IL "https://github.com/DzakirDesigning/finora-releases/releases/download/v1.0.1/finora-mobile-1.0.1.apk"
```
Pastikan:
- Seluruh hop redirect tetap menggunakan `https://`.
- Respons akhir adalah `HTTP/2 200` (atau `HTTP/1.1 200 OK`).
- Header `content-length` pada respons akhir **sama persis** dengan `apkSize` lokal.

### Tahap 4 — Perbarui Metadata `latest.json`
1. Salin `templates/latest.tab.example.json` → `updates/tab/latest.json` dan isi seluruh nilai nyata Tab (`apkSize`, `versionCode`, `minimumSupportedVersionCode` sebagai integer, `sha256` 64-hex lowercase, `apkUrl` sesuai tag).
2. Salin `templates/latest.mobile.example.json` → `updates/mobile/latest.json` dan isi seluruh nilai nyata Mobile.
3. Commit dan push **hanya** file metadata JSON ke branch `main` `DzakirDesigning/finora-releases` (pastikan `git status` tidak menyertakan file `.apk` atau secret apa pun).

### Tahap 5 — Verifikasi Raw GitHub URL Metadata
```powershell
curl.exe -i -H "Origin: https://localhost" -H "Accept: application/json" "https://raw.githubusercontent.com/DzakirDesigning/finora-releases/main/updates/tab/latest.json"
curl.exe -i -H "Origin: https://localhost" -H "Accept: application/json" "https://raw.githubusercontent.com/DzakirDesigning/finora-releases/main/updates/mobile/latest.json"
```
Pastikan status `200 OK`, header `access-control-allow-origin: *`, dan JSON valid. *(Catatan: CDN `raw.githubusercontent.com` memiliki cache singkat sekitar 1–5 menit setelah push.)*

---

## 7. Catatan Migrasi Masa Depan ke Cloudflare R2
Struktur path `updates/tab/latest.json` dan `updates/mobile/latest.json` pada repository ini sengaja dibuat identik dengan rencana struktur bucket Cloudflare R2 (`https://updates.finora.id`), sehingga perpindahan hosting di masa depan cukup mengganti nilai `VITE_ANDROID_UPDATE_BASE_URL` dan `apkUrl` tanpa mengubah arsitektur updater.
