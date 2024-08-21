### Deskripsi
Dengan instalasi sistem operasi dengan metode Dualboot ini, perangkat penyimpanan akan memiliki dua sistem operasi. Dengan struktur partisi yang sebagai berikut:
- `/dev/nvme0n1p1` merupakan partisi ESP.
- `/dev/nvme0n1p2` merupakan partisi Microsoft reserved.
- `/dev/nvme0n1p3` merupakan partisi sistem operasi Windows 10 (Versi 22H2).
- `/dev/nvme0n1p4` merupakan partisi sistem operasi Arch Linux.

Pengalokasian paling minimum partisi-partisi di atas sudah benar-benar dicari tahu sedemikian rupa agar dipastikan muat. Informasi lebih lanjut sebagai berikut:
1. Partisi ESP `/dev/nvme0n1p1` akan hanya meninggalkan 2 KiB (2048 bytes) ruang kosong saat Windows 10 (Versi 22H2) dan Arch Linux diinstal. Menguranginya sebanyak satu sektor pun akan mengakibatkan pemuat boot GRUB gagal diinstal.
2. Partisi Microsoft reserved `/dev/nvme0n1p2` sudah setara jumlah sektornya layaknya dibuat oleh Windows Setup sendiri.
3. Partisi sistem operasi Arch Linux `/dev/nvme0n1p4` paling minimum per 14 Juli 2024 sejumlah 1091 MiB. Menguranginya sebanyak 1 MiB pun akan mengakibatkan kegagalan saat instalasi paket kernel [`linux`](https://archlinux.org/packages/core/x86_64/linux/). Akan tetapi, jika pada masa mendatang 1091 MiB tidak lagi cukup maka tambahkan saja mulai dari 1 MiB dan lihat hasilnya.
### Prasyarat
- Unduh [ISO Arch Linux](https://archlinux.org/download/).
- Unduh [ISO Media Instalasi Windows 10](https://aka.ms/DownloadWindows10).
### Instalasi Arch Linux
1. Mulai ulang perangkat ke ISO Arch Linux. Tunggu sampai masuk ke Terminal.
2. Sambungkan perangkat ke jaringan Wi-Fi dengan `iwctl` lalu konfigurasi sebagai berikut:
   - `station wlan0 scan`
   - `station wlan0 get-networks`
   - `station wlan0 connect SSID` kemudian ketikkan kata sandi jaringan Wi-Fi
   - `quit`

      > Kiat: Untuk penetapan alamat IP statik dan penghapusan alamat IP dinamis dapat dilakukan sebagai berikut
      > - `ip a a 192.168.1.69/24 dev wlan0`
      > - `ip a d 192.168.1.160/24 dev wlan0`
3. Ubah zona waktu sistem live:

   - `timedatectl set-timezone Asia/Jakarta`
4. Dapatkan daftar cerminan HTTPS terbaru dan tercepat:

   - `reflector --verbose --latest 400 --protocol https --sort rate --save /etc/pacman.d/mirrorlist`
5. Buat tiga partisi dengan `parted -a none /dev/nvme0n1` lalu konfigurasi sebagai berikut:
   - `mklabel gpt`
   - `u s`
   - `mkpart "" 34 57665`
   - `mkpart "" 57666 90433`
   - `mkpart "" 997980814 1000215182`
   - `set 1 esp on`
   - `set 1 no_automount on`
   - `set 2 msftres on`
   - `set 2 no_automount on`
   - `q`

     > Kiat: Jumlah sektor untuk partisi sistem operasi Arch Linux ditemukan dengan pertama-tama mengetikkan jumlah Megabyte yang dibutuhkan pada utilitas `cfdisk`. Setelah itu, cukup lakukan pengurangan pada sektor akhir penyimpanan (`1000215182`) dengan jumlah sektor yang telah didapatkan sebelumnya.
6. Format partisi:

   - `mkfs.vfat -F16 /dev/nvme0n1p1`
   - `mkfs.ext4 /dev/nvme0n1p4`
7. Pasang partisi:

   - `mount /dev/nvme0n1p4 /mnt`
   - `mkdir -p /mnt/boot/efi`
   - `mount /dev/nvme0n1p1 /mnt/boot/efi`
8. Buat Tabel Berkas Sistem:

   - `mkdir /mnt/etc`
   - `genfstab -U -p /mnt >> /mnt/etc/fstab`
9. Unduh paket satu per satu dengan `nano /etc/pacman.conf` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      37:     #ParallelDownloads = 5
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      37:     ParallelDownloads = 1
      ```
    </details>
10. Instal sistem seminimum mungkin:
    - `pacstrap -K /mnt base linux grub efibootmgr`

    > Kiat: Agar dapat benar-benar menginstal paket-paket di atas pada situasi penyiapan yang sesempit dan seoptimal ini; Maka jalankanlah perintah-perintah di bawah ini setiap kali berhasil menginstal satu paket:
    > - `arch-chroot /mnt`
    > - `pacman -Scc` jawab dengan opsi `y` pada kedua pertanyaan yang muncul
    > - `rm /var/log/pacman.log && history -c && history -w && exit`

11. Ubah bahasa sistem dengan `nano /mnt/etc/locale.gen` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      171:    #en_US.UTF-8 UTF-8
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      171:    en_US.UTF-8 UTF-8
      ```
    </details>
12. Buat variabel bahasa sistem dengan `nano /mnt/etc/locale.conf` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      1:
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      1:      LANG=en_US.UTF-8
      ```
    </details>
13. Buat nama host komputer dengan `nano /mnt/etc/hostname` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      1:
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      1:      nxgn-x421eqy
      ```
    </details>
14. Instankan proses hitung mundur saat boot dengan `nano /mnt/etc/default/grub` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      4:      GRUB_TIMEOUT=5
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      4:      GRUB_TIMEOUT=0
      ```
    </details>
15. Nonaktifkan logging Journald dengan `nano /mnt/etc/systemd/journald.conf` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      20:     Storage=auto
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      20:     Storage=none
      ```
    </details>
16. Buat layanan governor, frekuensi dan kipas prosesor selalu maksimum setiap saat dengan `nano /mnt/usr/lib/systemd/system/setcpugff.service` lalu konfigurasi sebagai berikut:
    ```
    [Unit]
    Description=Set maximum CPU governor, frequency and fan speed then unmount all partitions

    [Service]
    ExecStart=/bin/bash -c 'echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor && echo 4200000 | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq && echo 0 > /sys/devices/platform/asus-nb-wmi/hwmon/hwmon5/pwm1_enable && umount -a'
    KillMode=process
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
17. Buat layanan otomatis nyalakan ulang perangkat tepat setelah menyala dengan `nano /mnt/usr/lib/systemd/system/reboot.service` lalu konfigurasi sebagai berikut:
    ```
    [Unit]
    Description=Reboot immediately

    [Service]
    ExecStart=/usr/bin/systemctl reboot
    KillMode=none
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
18. Beralih Terminal ke sistem yang baru diinstal:

    - `arch-chroot /mnt`
19. Terapkan perubahan bahasa sekarang juga:

    - `locale-gen`
20. Ubah zona waktu sistem:

    - `ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime`
21. Atur kata sandi pengguna root:

    - `passwd`
22. Instal pemuat boot:

    - `grub-install --bootloader-id="Arch Linux"`
23. Finalisasikan pembuatan GRUB:

    - `grub-mkconfig -o /boot/grub/grub.cfg`
24. Jalankan kedua layanan di atas sesaat setelah boot:

    - `systemctl enable setcpugff`
    - `systemctl enable reboot`
25. Bersihkan tembolok dan berkas log pacman serta cegah pembuatan berkas-berkas log oleh lastlogin dan Journald:
    - `pacman -Scc` jawab dengan opsi `y` pada kedua pertanyaan yang muncul
    - `rm /var/log/pacman.log`
    - `rm /var/log/btmp && ln -s /dev/null /var/log/btmp`
    - `rm /var/log/lastlog && ln -s /dev/null /var/log/lastlog`
    - `ln -s /dev/null /var/log/utmp`
    - `rm /var/log/wtmp && ln -s /dev/null /var/log/wtmp`

      > Rujukan: [security - Linux: disable wtmp/utmp, sshd ip logging and any mentions of ip that does remote access - Server Fault](https://serverfault.com/a/1123625)
26. Bersihkan histori perintah dan mulai ulang ke sistem operasi Arch Linux:

    - `history -c && history -w && exit`
    - `umount -a`
    - `reboot`
### Instalasi Windows 10 (Versi 22H2)
1. Mulai ulang perangkat ke ISO Media Instalasi Windows 10. Tunggu sampai masuk ke Windows Setup.
2. Pada langkah instalasi `Where do you want to install Windows?` pilih `Unallocated Space` urutan ketiga pada daftar kemudian tunggu beberapa saat.
3. Pada langkah instalasi `Windows needs to restart to continue` siapkan diri untuk menekan tombol Boot Menu sesaat waktu habis dan layar mati.
4. Mulai ulang lagi perangkat ke ISO Media Instalasi Windows 10. Sesaat Windows Setup muncul, tekan `Shift + F10` untuk memunculkan jendela `Command Prompt`.
5. Hapus aplikasi-aplikasi terprainstal:
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.549981C3F5F10_1.1911.21713.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.BingWeather_4.25.20211.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.DesktopAppInstaller_2019.125.2243.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.GetHelp_10.1706.13331.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Getstarted_8.2.22942.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.HEIFImageExtension_1.0.22742.0_x64__8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Microsoft3DViewer_6.1908.2042.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MicrosoftOfficeHub_18.1903.1152.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MicrosoftSolitaireCollection_4.4.8204.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MicrosoftStickyNotes_3.6.73.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MixedReality.Portal_2000.19081.1301.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.MSPaint_2019.729.2301.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Office.OneNote_16001.12026.20112.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.People_2019.305.632.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.ScreenSketch_2019.904.1644.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.SkypeApp_14.53.77.0_neutral_~_kzf8qxf38zg5c`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.StorePurchaseApp_11811.1001.1813.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.VP9VideoExtensions_1.0.22681.0_x64__8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Wallet_2.4.18324.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WebMediaExtensions_1.0.20875.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WebpImageExtension_1.0.22753.0_x64__8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Windows.Photos_2019.19071.12548.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsAlarms_2019.807.41.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsCalculator_2020.1906.55.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsCamera_2018.826.98.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:microsoft.windowscommunicationsapps_16005.11629.20316.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsFeedbackHub_2019.1111.2029.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsMaps_2019.716.2316.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsSoundRecorder_2019.716.2313.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.WindowsStore_11910.1002.513.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.Xbox.TCUI_1.23.28002.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxApp_48.49.31001.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxGameOverlay_1.46.11001.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxGamingOverlay_2.34.28001.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxIdentityProvider_12.50.6001.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.XboxSpeechToTextOverlay_1.17.29001.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.YourPhone_2019.430.2026.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.ZuneMusic_2019.19071.19011.0_neutral_~_8wekyb3d8bbwe`
   - `dism /Image:C:\ /Remove-ProvisionedAppxPackage /PackageName:Microsoft.ZuneVideo_2019.19071.19011.0_neutral_~_8wekyb3d8bbwe`

     > Kiat: Perintah-perintah di atas bisa dan boleh dijalankan dalam bentuk berkas batch (.cmd).
6. Prainstal driver perangkat-perangkat khusus:
   - `Dism /Image:C:\ /Add-Driver /Driver:"01__Unknown device__ASUSSystemControlInterfacev3_ASUS_Z_V3.1.31.0_15974\asussci2.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"02,03→06,12__Unknown device (6), PCI Data Acquisition and Signal Processing Controller__intel_dptf_8.7.10802.26924(station-drivers.com)\dptf_acpi.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"02,03→06,12__Unknown device (6), PCI Data Acquisition and Signal Processing Controller__intel_dptf_8.7.10802.26924(station-drivers.com)\dptf_cpu.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"07,13→15,17,19__PCI Device (3), Unknown device, SM Bus Controller, PCI Simple Communications Controller__Intel_Serial-IO_30.100.2413.49(station-drivers.com)\production\Windows10-x64\17763\Drivers\WU\iaLPSS2_GPIO2_TGL.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"07,13→15,17,19__PCI Device (3), Unknown device, SM Bus Controller, PCI Simple Communications Controller__Intel_Serial-IO_30.100.2413.49(station-drivers.com)\production\Windows10-x64\17763\Drivers\WU\iaLPSS2_I2C_TGL.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"07,13→15,17,19__PCI Device (3), Unknown device, SM Bus Controller, PCI Simple Communications Controller__Intel_Serial-IO_30.100.2413.49(station-drivers.com)\production\Windows10-x64\17763\Drivers\WU\iaLPSS2_SPI_TGL.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"07,13→15,17,19__PCI Device (3), Unknown device, SM Bus Controller, PCI Simple Communications Controller__Intel_Serial-IO_30.100.2413.49(station-drivers.com)\production\Windows10-x64\17763\Drivers\WU\iaLPSS2_UART2_TGL.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"09__PCI Device__intel_chipset_10.1.19627.8423(station-drivers.com)\DriverFiles\production\Windows10-x64\TigerlakePCH-LPDmaSecExtension.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"09__PCI Device__intel_chipset_10.1.19627.8423(station-drivers.com)\DriverFiles\production\Windows10-x64\TigerlakePCH-LPSystem.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"09__PCI Device__intel_chipset_10.1.19627.8423(station-drivers.com)\DriverFiles\production\Windows10-x64\TigerlakeSystem.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"10,20__PCI Device, Intel(R) Wi-Fi 6 AX201 160MHz__intel_wlan_23.60.0(station-drivers.com)\ibtusb.inf\ibtusb.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"10,20__PCI Device, Intel(R) Wi-Fi 6 AX201 160MHz__intel_wlan_23.60.0(station-drivers.com)\netwtw08.inf\Netwtw08.INF"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11a__PCI Simple Communications Controller__intel_mei_2418.6.12.0(station-drivers.com)\0\Drivers\01__heci.inf\heci.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11a__PCI Simple Communications Controller__intel_mei_2418.6.12.0(station-drivers.com)\0\Drivers\02__iclsClient.inf\iclsClient.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11a__PCI Simple Communications Controller__intel_mei_2418.6.12.0(station-drivers.com)\0\Drivers\03__DAL.inf\DAL.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11a__PCI Simple Communications Controller__intel_mei_2418.6.12.0(station-drivers.com)\0\Drivers\04__MEWMIProv.inf\MEWMIProv.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11a__PCI Simple Communications Controller__intel_mei_2418.6.12.0(station-drivers.com)\0\Drivers\05__wiman_wlan_extension.inf\wiman_wlan_extension.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11a__PCI Simple Communications Controller__intel_mei_2418.6.12.0(station-drivers.com)\0\Drivers\06__WiMan.inf\WiMan.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11b__Generic software component__Intel(R) PROSet IHV Extension Software Component_23.1050.0.4_11.0_x64\Intel(R) PROSet IHV Extension Software Component_23.1050.0.4_11.0_x64\PieComponent.INF"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"11b__Generic software component__Intel(R) PROSet IHV Extension Software Component_23.1050.0.4_11.0_x64\Intel(R) PROSet IHV Extension Software Component_23.1050.0.4_11.0_x64\PieExtension.INF"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"16__Base System Device__Intel_GNA_03.05.00.1578(station-drivers.com)\gna.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"18__USB2.0-CRW__realtek_cr_10.0.26100.31287(station-drivers.com)\RtsXStor_10.0.300.284_20240513_WHQL\DrvBin64\RtsUer.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"21__Unknown device__Fingerprint_WBF_SPI_CC_ELAN_Z_V4.5.13001.11903_32943\WbfSpiDriver.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"22__USB2.0 HD UVC WebCam__n2cam0317us14cmp\Driver\DrvBin64\RtAsus.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"23__HID-compliant touch pad__PrecisionTouchPad_DCH_ASUS_X_V16.0.0.20_36391_1\Touchpad\x64\AsusPTPFilter.inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"24a__Generic Monitor__MyASUSSplendid_ASUS_Z_V5.0.0.261_14223_2\MyASUS_Splendid.Inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"24b__Generic Monitor__026560e67f31c7175b1e69561a1e3fe7670a58c3\MyASUS_Splendid.Inf"`
   - `Dism /Image:C:\ /Add-Driver /Driver:"25__System Firmware__catcab_1dab3b898c8ec5bc219b264f18d23e723b92cf0c\X421EQY_308.inf"`

     > Kiat: Perintah-perintah di atas tidak bisa dan tidak boleh dijalankan dalam bentuk berkas batch (.cmd) karena mengandung karakter-karakter spesial. Melainkan, salin dari `Notepad` dan tempel semua sekaligus perintah-perintah tersebut ke `Command Prompt`.
7. Buka `Registry Editor` dengan `regedit` pilih subkunci `HKEY_LOCAL_MACHINE` kemudian lakukan hal-hal berikut:

   - Pada menu `File` > `Load Hive...` buka sarang `C:\Windows\System32\config\SYSTEM`.
   - Pada jendela `Load Hive` ketikkan `SYS`.
   - Buka subkunci `SYS\Setup` dan ubah isi String `CmdLine` dari yang sebelumnya `oobe\windeploy.exe` menjadi `cmd.exe`.
   - Kembali subkunci `HKEY_LOCAL_MACHINE` teratas dan pilih subkunci `SYS`.
   - Pada menu `File` > `Unload Hive...` pilih `Yes`.
   - Terakhir, tutup Registry Editor.
8. Matikan perangkat:

   - `wpeutil shutdown`
9. Cabut media instalasi dan nyalakan perangkat. Tunggu hingga jendela `Command Prompt` muncul.
10. Hapus aplikasi `Microsoft Edge` dan `Microsoft Edge Updater`:
    - `copy \Program Files (x86)\Microsoft\Edge\Application\92.0.902.67\Installer\setup.exe C:\`
    - `C:\setup.exe --uninstall --msedge --verbose-logging --system-level --force-uninstall`
    - `C:\setup.exe --uninstall --msedgeupdate --verbose-logging --system-level --force-uninstall`
     
      > Kiat: Beri jeda setidaknya satu menit setelah menjalankan perintah kedua sebelum menjalankan perintah ketiga guna memastikan Microsoft Edge benar-benar secara utuh mencopot seluruh pemasangannya.
11. Buka Registry Editor dengan `regedit` lalu buka subkunci `HKEY_LOCAL_MACHINE\Setup`. Ubah isi String `CmdLine` dari `cmd.exe` kembali menjadi `oobe\windeploy.exe`. Setelahnya, tutup Registry Editor.
12. Mulai proses penginstalan driver perangkat-perangkat khusus:
    - `oobe\windeploy.exe`

      > Kiat: Untuk perangkat ASUS VivoBook 14 X421EQY ini tidak dapat melakukan instalasi untuk kedua driver grafik Intel® Iris® Xᵉ dan NVIDIA GeForce Game Ready Driver sebelum tahap ini selesai. Jikalau berhasil pun tentunya (murni spekulatif) Intel® Arc™ Control mungkin saja tidak terinstal pada sistem.
13. Sesaat layar pemuat sudah hitam kosong lanjutkan ke proses OOBE:
    - `exit`
14. Munculkan jendela `Command Prompt` dengan `Shift + F10`.
15. Terapkan kebijakan grup (terkhusus penonaktifan Microsoft Defender Antivirus):

    - `gpedit.msc`
    - `gpupdate.exe /force`
16. Buka `Registry Editor` dengan `regedit` kemudian lakukan hal-hal berikut:

    - Buka subkunci `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings`. Buat `DWORD (32-bit) Value` bernama `TrayIconVisibility`. Ubah isi Value tersebut menjadi `1` kemudian kembalikan lagi menjadi `0`.
    - Buka subkunci `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`. Ubah isi Value `EnableCursorSuppression`dari yang sebelumnya `1` menjadi `0`.
17. Lakukan penyalaan ulang guna menerapkan pengonfigurasian di atas:

    - `shutdown -r -t 0`
18. Lakukan OOBE seperti biasanya dengan catatan tidak menyambungkan ke internet, menggunakan akun lokal, menonaktifkan semua setelan privasi dan tidak menggunakan Cortana.
19. Lakukan instalasi driver grafik Intel® Iris® Xᵉ dan NVIDIA GeForce Game Ready Driver seperti biasanya dengan catatan mencentang opsi `Clean Installation` pada keduanya.
20. Copot pemasangan `Intel® Driver & Support Assistant (Intel® DSA)`:
    - `C:\ProgramData\Package Cache\{ID PENGINSTAL INTEL® DSA}\Installer.exe /uninstall`

      > Kiat: Periksa `{ID PENGINSTAL INTEL® DSA}` secara manual dengan mencarinya pada folder `C:\ProgramData\Package Cache` karena selalu berbeda setiap versi.
21. Terapkan setelan-setelan yang ada pada [windows_10_post-installation.md](https://github.com/kevintaswin/gosh/blob/main/windows/10/windows_10_post-installation.md).
