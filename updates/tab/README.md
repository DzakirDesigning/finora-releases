# FINORA Tab — Channel Metadata Pembaruan

Direktori ini khusus menyimpan metadata pembaruan aktif (`latest.json`) untuk aplikasi **FINORA Tab** (`com.finora.pos.tab`).

## URL Raw GitHub Aktif (Setelah Rilis Dipublikasikan)
```text
https://raw.githubusercontent.com/DzakirDesigning/finora-releases/main/updates/tab/latest.json
```

## Aturan Ketat
1. **Jangan menaruh file `.apk` di folder ini.** File `finora-tab-<versionName>.apk` hanya boleh diunggah sebagai **GitHub Release Asset**.
2. **Jangan mempublikasikan template mentah.** Salin `templates/latest.tab.example.json` menjadi `updates/tab/latest.json`, lalu ganti seluruh placeholder dengan nilai nyata dari APK Release FINORA Tab yang sudah diunggah ke GitHub Releases (`versionCode`, `apkSize`, dan `minimumSupportedVersionCode` wajib berupa JSON integer tanpa tanda kutip).
3. **Isolasi Target & Sertifikat**:
   - Field `target` wajib `"tab"`.
   - Field `apkUrl` wajib menunjuk ke asset APK FINORA Tab (`com.finora.pos.tab`) yang ditandatangani dengan alias resmi `finora-tab`.
4. **Urutan Publikasi**: Unggah dan verifikasi APK Release Tab di GitHub Releases terlebih dahulu, **baru terakhir** perbarui `updates/tab/latest.json`.
