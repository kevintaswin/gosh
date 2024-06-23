### Deskripsi
Dengan instalasi sistem operasi dengan metode Singleboot ini, perangkat penyimpanan akan memiliki satu sistem operasi, yakni hanya Arch Linux. Dengan struktur partisi yang sudah ada sebagai berikut:
- `/dev/sda1` merupakan partisi EFI.
- `/dev/sda2` merupakan partisi Arch Linux.
### Prasyarat
- Unduh [ISO Arch Linux](https://archlinux.org/download/).
## Instalasi
1. Mulai ulang perangkat ke ISO Arch Linux. Tunggu sampai masuk ke Terminal.
2. Dapatkan daftar cerminan HTTPS terbaru dan tercepat:

   - `reflector --latest 9999 --protocol https --sort rate --save /etc/pacman.d/mirrorlist`

      > Kiat: Pastikan internet terhubung: `ping -c 3 google.com`
3. Dapatkan daftar perangkat penyimpanan terhubung:

   - `lsblk`

     > Kiat: Lihat pada ukuran partisi yang sudah diciutkan tadi dan ingatlah nama partisi tersebut.
4. Hapus bersih tabel partisi perangkat penyimpanan dituju:

   - `wipefs -a /dev/sda`
5. Tulis bita nol ke seluruh sektor demi menjaga kestabilan kinerja perangkat penyimpanan:

   - `dd if=/dev/zero of=/dev/sda status=progress`
6. Konfigurasi perangkat penyimpanan dituju dengan `cfdisk /dev/sda` lalu konfigurasi sebagai berikut:

   - 29M untuk /dev/sda1
   - 100% untuk /dev/sda2
7. Format partisi:

   - `mkfs.vfat -F 16 /dev/sda1`
   - `mkfs.ext4 /dev/sda2`
8. Pasang partisi:

   - `mount /dev/sda2 /mnt`
   - `mkdir /mnt/boot/efi`
   - `mount /dev/sda1 /boot/efi`
9. Buat Tabel Berkas Sistem:

   - `mkdir /mnt/etc`
   - `genfstab -U -p /mnt >> /mnt/etc/fstab`
10. Unduh paket satu per satu dengan `nano /etc/pacman.conf` lalu konfigurasi sebagai berikut:
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
11. Instal sistem seminimum mungkin:

   - `pacstrap -K /mnt base linux linux-firmware`
12. Beralih Terminal ke sistem yang baru diinstal:

    - `arch-chroot /mnt`
13. Instal paket penunjang:

    - `pacman -S intel-ucode sof-firmware sudo grub efibootmgr base-devel git nano cpupower xorg xorg-xinit pulseaudio pavucontrol`
14. Atur kata sandi pengguna root:

    - `passwd`
15. Buat pengguna baru non root:

    - `useradd -m -g users -G wheel kevintaswin`
16. Atur kata sandi untuk pengguna baru:

    - `passwd kevintaswin`
17. Izinkan pengguna baru mengakses perintah istimewa dengan `EDITOR=nano visudo` lalu konfigurasi sebagai berikut:
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
18. Instal pemuat boot berbasis EFI:

    - `grub-install --target=x86_64-efi --bootloader-id="Arch Linux" --recheck`
19. Instankan proses hitung mundur saat boot dengan `nano /etc/default/grub` lalu konfigurasi sebagai berikut:
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
20. Finalisasikan pembuatan GRUB:

    - `grub-mkconfig -o /boot/grub/grub.cfg`
21. Nonaktifkan logging Journald dengan `nano /etc/systemd/journald.conf` lalu konfigurasi sebagai berikut:
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
22. Atur konfigurasi frekuensi prosesor selalu maksimum setiap saat dengan `nano /etc/default/cpupower` lalu konfigurasi sebagai berikut:
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
23. Bersihkan tembolok dan berkas log pacman serta cegah pembuatan berkas-berkas log oleh lastlogin dan Journald:
    - `pacman -Scc` jawab dengan opsi `y` pada kedua pertanyaan yang muncul
    - `rm /var/log/pacman.log`
    - `rm /var/log/btmp && ln -s /dev/null /var/log/btmp`
    - `rm /var/log/lastlog && ln -s /dev/null /var/log/lastlog`
    - `ln -s /dev/null /var/log/utmp`
    - `rm /var/log/wtmp && ln -s /dev/null /var/log/wtmp`

      > Rujukan: https://serverfault.com/a/1123625
24. Bersihkan histori perintah dan mulai ulang ke sistem operasi Arch Linux:

    - `history -c && history -w && exit`
    - `umount -a`
    - `reboot`
