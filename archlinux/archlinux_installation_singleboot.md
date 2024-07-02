### Deskripsi
Dengan instalasi sistem operasi dengan metode Singleboot ini, perangkat penyimpanan akan memiliki satu sistem operasi, yakni hanya Arch Linux. Dengan struktur partisi yang sudah ada sebagai berikut:
- `/dev/sda1` merupakan partisi EFI.
- `/dev/sda2` merupakan partisi Arch Linux.
### Prasyarat
- Unduh [ISO Arch Linux](https://archlinux.org/download/).
## Instalasi
1. Mulai ulang perangkat ke ISO Arch Linux. Tunggu sampai masuk ke Terminal.
2. Ubah zona waktu sistem live:

   - `timedatectl set-timezone Asia/Jakarta`
3. Dapatkan daftar cerminan HTTPS terbaru dan tercepat:
   - `reflector --verbose --latest 400 --protocol https --sort rate --save /etc/pacman.d/mirrorlist`

      > Kiat: Pastikan internet terhubung: `ping -c 3 google.com`.
4. Dapatkan daftar perangkat penyimpanan terhubung:
   - `lsblk`

     > Kiat: Lihat pada ukuran partisi yang sudah diciutkan tadi dan ingatlah nama partisi tersebut.
5. Hapus bersih tabel partisi perangkat penyimpanan dituju:

   - `wipefs -a /dev/sda`
6. Tulis bita nol ke seluruh sektor demi menjaga kestabilan kinerja perangkat penyimpanan:

   - `dd if=/dev/zero of=/dev/sda status=progress`
7. Konfigurasi perangkat penyimpanan dituju dengan `parted -a none /dev/sda` lalu konfigurasi sebagai berikut:
   - `mklabel gpt`
   - `u s`
   - `mkpart "" 34 57665`
   - `mkpart "" 57666 250069646`
   - `set 1 esp on`
   - `set 1 no_automount on`
   - `q`

     > Kiat: Gunakan perintah `p free` untuk mendapatkan sektor terakhir partisi kedua.
8. Format partisi:

   - `mkfs.vfat -F 16 /dev/sda1`
   - `mkfs.ext4 /dev/sda2`
9. Pasang partisi:

   - `mount /dev/sda2 /mnt`
   - `mkdir -p /mnt/boot/efi`
   - `mount /dev/sda1 /boot/efi`
10. Buat Tabel Berkas Sistem:

    - `mkdir /mnt/etc`
    - `genfstab -U -p /mnt >> /mnt/etc/fstab`
11. Unduh paket satu per satu dengan `nano /etc/pacman.conf` lalu konfigurasi sebagai berikut:
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
12. Instal sistem seminimum mungkin:

    - `pacstrap -K /mnt base linux`
13. Beralih Terminal ke sistem yang baru diinstal:

    - `arch-chroot /mnt`
14. Instal paket penunjang:
    - `pacman -S linux-firmware intel-ucode sof-firmware sudo grub efibootmgr base-devel git nano libx11 xorg-server xorg-xinit libxrandr xorg-xrandr libxft xf86-video-intel xclip pcmanfm alsa-utils`

      > Kiat: Sesuaikan paket `xf86-video-intel` melalui acuan persis pada tautan berikut:
      > - [Installation guide - ArchWiki](https://wiki.archlinux.org/title/Installation_guide)
15. Atur kata sandi pengguna root:

    - `passwd`
16. Buat pengguna baru non root:

    - `useradd -m -g users -G wheel kevintaswin`
17. Atur kata sandi untuk pengguna baru:

    - `passwd kevintaswin`
18. Izinkan pengguna baru mengakses perintah istimewa dengan `EDITOR=nano visudo` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      108:    #%wheel ALL=(ALL:ALL) ALL
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      108:    %wheel ALL=(ALL:ALL) ALL
      ```
    </details>
19. Instal pemuat boot berbasis EFI:

    - `grub-install --target=x86_64-efi --bootloader-id="Arch Linux" --recheck`
20. Instankan proses hitung mundur saat boot dengan `nano /etc/default/grub` lalu konfigurasi sebagai berikut:
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
21. Finalisasikan pembuatan GRUB:

    - `grub-mkconfig -o /boot/grub/grub.cfg`
22. Nonaktifkan logging Journald dengan `nano /etc/systemd/journald.conf` lalu konfigurasi sebagai berikut:
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
23. Buat layanan governor dan frekuensi prosesor selalu maksimum setiap saat dengan `nano /usr/lib/systemd/system/setcpugf.service` lalu konfigurasi sebagai berikut:
    ```
    [Unit]
    Description=Set maximum CPU governor and frequency

    [Service]
    ExecStart=/bin/bash -c 'echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor && echo 2600000 | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_min_freq'
    KillMode=process
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```

    > Kiat: Sesuaikan frekuensi prosesor dengan acuan persis pada tautan-tautan berikut:
    > - [Intel® - Product Specifications - Processors](https://ark.intel.com/content/www/us/en/ark.html#@Processors)
    > - [AMD Ryzen™ - Desktop, Laptop and Workstation Processor Specifications](https://www.amd.com/en/products/specifications/processors.html)
    > - [AMD EPYC™ - Server Processor Specifications](https://www.amd.com/en/products/specifications/server-processor.html)
    > - [AMD EPYC™ Embedded - Embedded Processor Specifications](https://www.amd.com/en/products/specifications/embedded.html)
24. Bersihkan tembolok dan berkas log pacman serta cegah pembuatan berkas-berkas log oleh lastlogin dan Journald:
    - `pacman -Scc` jawab dengan opsi `y` pada kedua pertanyaan yang muncul
    - `rm /var/log/pacman.log`
    - `rm /var/log/btmp && ln -s /dev/null /var/log/btmp`
    - `rm /var/log/lastlog && ln -s /dev/null /var/log/lastlog`
    - `ln -s /dev/null /var/log/utmp`
    - `rm /var/log/wtmp && ln -s /dev/null /var/log/wtmp`

      > Rujukan: [security - Linux: disable wtmp/utmp, sshd ip logging and any mentions of ip that does remote access - Server Fault](https://serverfault.com/a/1123625)
25. Bersihkan histori perintah dan mulai ulang ke sistem operasi Arch Linux:

    - `history -c && history -w && exit`
    - `umount -a`
    - `reboot`
