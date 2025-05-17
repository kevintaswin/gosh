> [!CAUTION]
> Koneksi internet tidak diperkenankan selama tahapan ini.<br>Pastikan sambungan Ethernet tetap terputus dan Wi-Fi tidak tersambung ke jaringan manapun.
1. Cabut media instalasi dan nyalakan perangkat. Tunggu hingga jendela `Command Prompt` muncul.
2. Hapus aplikasi `Microsoft Edge` dan `Microsoft Edge Updater`:
    - `copy "C:\Program Files (x86)\Microsoft\Edge\Application\92.0.902.67\Installer\setup.exe" C:\`
    - `C:\setup.exe --uninstall --msedge --verbose-logging --system-level --force-uninstall`
    - `C:\setup.exe --uninstall --msedgeupdate --verbose-logging --system-level --force-uninstall`
     
      > Kiat: Beri jeda setidaknya satu menit setelah menjalankan perintah kedua sebelum menjalankan perintah ketiga guna memastikan Microsoft Edge benar-benar secara utuh mencopot seluruh pemasangannya.
3. Buka Registry Editor dengan `regedit` lalu lakukan hal-hal berikut:
    - Mengembalikan Value Windows Deployment Loader seperti semula:

      - Buka subkunci `HKEY_LOCAL_MACHINE\System\Setup`.
      - Ubah isi Value `CmdLine` dari `cmd.exe` kembali menjadi `oobe\windeploy.exe`.
    - Tutup Registry Editor.
5. Mulai proses penginstalan driver perangkat-perangkat khusus:
    - `oobe\windeploy.exe`

      > Kiat: Perhatikan daftar grafik driver berikut.
      > |Nama Driver Grafik|Dapat Diinstal Saat Tahap Pra OOBE (`windeploy`)|
      > |-|-|
      > |Intel® UHD Graphics|Bisa|
      > |Intel® Iris® Xᵉ|Tidak bisa|
      > |NVIDIA GeForce Game Ready Driver|Tidak bisa|
6. Sesaat layar pemuat sudah hitam kosong lanjutkan ke proses OOBE:

    - `exit`
