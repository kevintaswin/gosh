### Deskripsi
Dengan instalasi sistem operasi dengan metode Dualboot ini, perangkat penyimpanan akan memiliki dua sistem operasi. Dengan struktur partisi yang sudah ada sebagai berikut:
- `/dev/sda1` merupakan partisi ESP.
- `/dev/sda2` merupakan partisi Microsoft reserved.
- `/dev/sda3` merupakan partisi sistem operasi Windows.
- `/dev/sda4` merupakan partisi Windows Recovery Environment.

Sehingga partisi `/dev/sda3` akan diciutkan ke partisi `/dev/sda5` yang didedikasikan khusus untuk partisi Arch Linux dengan metode pemartisian standar yang di dalamnya terdapat partisi besar tergabung untuk root, home dan berkas swap.
### Prasyarat
- Windows sudah terlebih dahulu diinstal.
- Unduh [ISO Arch Linux](https://archlinux.org/download/).
- Unduh [ISO Media Instalasi Windows](https://aka.ms/DownloadWindows) atau Pengelola Penyimpanan Live Windows PE seperti [MiniTool Partition Wizard Bootable](https://www.partitionwizard.com/partition-wizard-bootable-cd.html).
### Prainstalasi
1. Mulai ulang perangkat ke ISO Media Instalasi Windows.
2. Buka Command Prompt dengan menekan `Shift + F10` atau salah satu opsi pada daftar.
3. Ketikkan `diskpart` untuk mengalokasikan sebagian penyimpanan untuk Arch Linux. Jika perlu sesuaikan perintah-perintah berikut ini:

   - `sel dis 0`
   - `sel par 3`
   - `shrink desired=2048 minimum=2048`
   - `exit`
   - `wpeutil reboot`
4. Tunggu sampai Windows memuat dan mengenali semua perubahan yang terjadi.
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
5. Format partisi:

   - `mkfs.ext4 /dev/sda5`
6. Pasang partisi:

   - `mount /dev/sda5 /mnt`
   - `mkdir -p /mnt/boot/efi`
   - `mount /dev/sda1 /mnt/boot/efi`
7. Buat Tabel Berkas Sistem:

   - `mkdir /mnt/etc`
   - `genfstab -U -p /mnt >> /mnt/etc/fstab`
8. Unduh paket satu per satu dengan `nano /etc/pacman.conf` lalu konfigurasi sebagai berikut:
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
9. Instal sistem seminimum mungkin:

   - `pacstrap -K /mnt base linux`
10. Beralih Terminal ke sistem yang baru diinstal:

    - `arch-chroot /mnt`
11. Instal paket penunjang:
    - `pacman -S linux-firmware intel-ucode sof-firmware sudo grub efibootmgr base-devel git nano libx11 xorg-server xorg-xinit libxrandr xorg-xrandr libxft xf86-video-intel xclip pcmanfm alsa-utils`

      > Kiat: Sesuaikan paket `intel-ucode` dan `xf86-video-intel` melalui acuan persis pada tautan berikut:
      > - [Installation guide - ArchWiki](https://wiki.archlinux.org/title/Installation_guide)
12. Atur kata sandi pengguna root:

    - `passwd`
13. Buat pengguna baru non root:

    - `useradd -m -g users -G wheel kevintaswin`
14. Atur kata sandi untuk pengguna baru:

    - `passwd kevintaswin`
15. Izinkan pengguna baru mengakses perintah istimewa dengan `EDITOR=nano visudo` lalu konfigurasi sebagai berikut:
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
16. Instal pemuat boot:

    - `grub-install --bootloader-id="Arch Linux"`
17. Instankan proses hitung mundur saat boot dengan `nano /etc/default/grub` lalu konfigurasi sebagai berikut:
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
18. Finalisasikan pembuatan GRUB:

    - `grub-mkconfig -o /boot/grub/grub.cfg`
19. Nonaktifkan logging Journald dengan `nano /etc/systemd/journald.conf` lalu konfigurasi sebagai berikut:
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
20. Buat layanan governor dan frekuensi prosesor selalu maksimum setiap saat dengan `nano /usr/lib/systemd/system/setcpugf.service` lalu konfigurasi sebagai berikut:
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

    > Kiat: Dapatkan frekuensi maksimum persis prosesor: `cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq`.
21. Bersihkan tembolok dan berkas log pacman serta cegah pembuatan berkas-berkas log oleh lastlogin dan Journald:
    - `pacman -Scc` jawab dengan opsi `y` pada kedua pertanyaan yang muncul
    - `rm /var/log/pacman.log`
    - `rm /var/log/btmp && ln -s /dev/null /var/log/btmp`
    - `rm /var/log/lastlog && ln -s /dev/null /var/log/lastlog`
    - `ln -s /dev/null /var/log/utmp`
    - `rm /var/log/wtmp && ln -s /dev/null /var/log/wtmp`

      > Rujukan: [security - Linux: disable wtmp/utmp, sshd ip logging and any mentions of ip that does remote access - Server Fault](https://serverfault.com/a/1123625)
22. Bersihkan histori perintah dan mulai ulang ke sistem operasi Arch Linux:

    - `history -c && history -w && exit`
    - `umount -a`
    - `reboot`
