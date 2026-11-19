# FINORA Mobile — Channel Metadata Pembaruan

Direktori ini khusus menyimpan metadata pembaruan aktif (`latest.json`) untuk aplikasi **FINORA Mobile** (`com.finora.pos.mobile`).

## URL Raw GitHub Aktif (Setelah Rilis Dipublikasikan)
```text
https://raw.githubusercontent.com/DzakirDesigning/finora-releases/main/updates/mobile/latest.json
```

## Aturan Ketat
1. **Jangan menaruh file `.apk` di folder ini.** File `finora-mobile-<versionName>.apk` hanya boleh diunggah sebagai **GitHub Release Asset**.
2. **Jangan mempublikasikan template mentah.** Salin `templates/latest.mobile.example.json` menjadi `updates/mobile/latest.json`, lalu ganti seluruh placeholder dengan nilai nyata dari APK Release FINORA Mobile yang sudah diunggah ke GitHub Releases (`versionCode`, `apkSize`, dan `minimumSupportedVersionCode` wajib berupa JSON integer tanpa tanda kutip).
3. **Isolasi Target & Sertifikat**:
   - Field `target` wajib `"mobile"`.
   - Field `apkUrl` wajib menunjuk ke asset APK FINORA Mobile (`com.finora.pos.mobile`) yang ditandatangani dengan alias resmi `finora-mobile`.
4. **Urutan Publikasi**: Unggah dan verifikasi APK Release Mobile di GitHub Releases terlebih dahulu, **baru terakhir** perbarui `updates/mobile/latest.json`.
