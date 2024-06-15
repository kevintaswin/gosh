### Deskripsi
Dengan instalasi sistem operasi dengan metode Dualboot ini, perangkat penyimpanan akan memiliki dua sistem operasi. Dengan struktur partisi yang sudah ada sebagai berikut:
- `/dev/sda1` merupakan partisi EFI.
- `/dev/sda2` merupakan partisi Microsoft reserved.
- `/dev/sda3` merupakan partisi sistem operasi Windows.
- `/dev/sda4` merupakan partisi Windows Recovery Environemnt.

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
2. Dapatkan daftar cerminan HTTPS terbaru dan tercepat:

   - `reflector --latest 9999 --protocol https --sort rate --save /etc/pacman.d/mirrorlist`

      > Kiat: Pastikan internet terhubung: `ping -c 3 google.com`

3. Dapatkan daftar perangkat penyimpanan terhubung:

   - `lsblk`

     > Kiat: Lihat pada ukuran partisi yang sudah diciutkan tadi dan ingatlah nama partisi tersebut.
4. Format partisi:

   - `mkfs.ext4 /dev/sda5`
5. Pasang partisi:

   - `mount /dev/sda5 /mnt`
   - `mkdir -p /boot/efi`
   - `mkdir -p /mnt/home`
   - `mount /dev/sda1 /boot/efi`
6. Buat Tabel Berkas Sistem:

   - `mkdir /mnt/etc`
   - `genfstab -U -p /mnt >> /mnt/etc/fstab`
7. Unduh paket satu per satu dengan `nano /etc/pacman.conf` lalu konfigurasi sebagai berikut:
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
8. Instal sistem seminimum mungkin:

   - `pacstrap -K /mnt base linux linux-firmware`
9. Beralih Terminal ke sistem yang baru diinstal:

    - `arch-chroot /mnt`
10. Instal paket penunjang:

    - `pacman -S intel-ucode sof-firmware sudo grub efibootmgr base-devel git nano cpupower xorg xorg-xinit pulseaudio pavucontrol`
11. Atur kata sandi pengguna root:

    - `passwd`
12. Buat pengguna baru non root:

    - `useradd -m -g users -G wheel kevintaswin`
13. Atur kata sandi untuk pengguna baru:

    - `passwd kevintaswin`
14. Izinkan pengguna baru mengakses perintah istimewa dengan `EDITOR=nano visudo` lalu konfigurasi sebagai berikut:
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
15. Instal pemuat boot berbasis EFI:

    - `grub-install --target=x86_64-efi --bootloader-id="Arch Linux" --recheck`
16. Instankan proses hitung mundur saat boot dengan `nano /etc/default/grub` lalu konfigurasi sebagai berikut:
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
17. Finalisasikan pembuatan GRUB:

    - `grub-mkconfig -o /boot/grub/grub.cfg`
18. Nonaktifkan logging Journald dengan `nano /etc/systemd/journald.conf` lalu konfigurasi sebagai berikut:
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
19. Atur konfigurasi frekuensi prosesor selalu maksimum setiap saat dengan `nano /etc/default/cpupower` lalu konfigurasi sebagai berikut:
    <details>
      <summary>Sebelum</summary>

      ```
      3:      #governor=`ondemand`
      7:      #min_freq="2.25Ghz"
      8:      #max_freq="3GHz"
      ```
    </details>
    <details open>
      <summary>Sesudah</summary>

      ```
      3:      governor=`performance`
      7:      min_freq="2600Mhz"
      8:      max_freq="2600MHz"
      ```
    </details>

    > Kiat: Sesuaikan frekuensi prosesor dengan yang dimiliki. Acuan persis dapat ditemukan pada tautan-tautan berikut:
    > - [Intel® - Product Specifications - Processors](https://ark.intel.com/content/www/us/en/ark.html#@Processors)
    > - [AMD Ryzen™ - Desktop, Laptop and Workstation Processor Specifications](https://www.amd.com/en/products/specifications/processors.html)
    > - [AMD EPYC™ - Server Processor Specifications](https://www.amd.com/en/products/specifications/server-processor.html)
    > - [AMD EPYC™ Embedded - Embedded Processor Specifications](https://www.amd.com/en/products/specifications/embedded.html)
20. Bersihkan tembolok dan berkas log pacman serta cegah pembuatan berkas-berkas log oleh lastlogin dan Journald:
    - `pacman -Scc` jawab dengan opsi `y` pada kedua pertanyaan yang muncul
    - `rm /var/log/pacman.log`
    - `rm /var/log/btmp && ln -s /dev/null /var/log/btmp`
    - `rm /var/log/lastlog && ln -s /dev/null /var/log/lastlog`
    - `ln -s /dev/null /var/log/utmp`
    - `rm /var/log/wtmp && ln -s /dev/null /var/log/wtmp`

      > Rujukan: https://serverfault.com/a/1123625
21. Bersihkan histori perintah dan mulai ulang ke sistem operasi Arch Linux:

    - `history -c && history -w && exit`
    - `umount -a`
    - `reboot`