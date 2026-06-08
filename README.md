# Dokumentasi Konfigurasi VPS Ubuntu: Apache2, PHP 8.4, MySQL, phpMyAdmin, dan FTP Upload Web

Dokumentasi ini berisi langkah instalasi dan konfigurasi VPS Ubuntu untuk menjalankan website berbasis PHP framework seperti Laravel, CodeIgniter, WordPress, atau aplikasi PHP native.

Stack yang digunakan:

- Ubuntu Server
- Apache2 sebagai web server
- PHP 8.4 menggunakan PHP-FPM
- MySQL Server sebagai database
- phpMyAdmin untuk manajemen database via browser
- vsftpd untuk FTP upload file website
- Composer untuk dependency framework PHP

> Catatan: Dokumentasi ini mengikuti kondisi konfigurasi server saat ini, termasuk penggunaan PHP 8.4 FPM, Apache2, MySQL, dan phpMyAdmin yang sempat mengalami kendala password policy MySQL.

---

## 1. Update Server

Langkah pertama setelah membuat VPS baru adalah memperbarui repository dan package sistem.

```bash
apt update && apt upgrade -y
```

Install tools dasar yang dibutuhkan:

```bash
apt install -y curl wget unzip git nano software-properties-common ca-certificates lsb-release apt-transport-https ufw
```

Cek versi Ubuntu:

```bash
lsb_release -a
```

Penjelasan:

- `apt update` memperbarui daftar package.
- `apt upgrade` memperbarui package yang sudah terinstall.
- `curl`, `wget`, `unzip`, dan `git` dibutuhkan untuk mengambil dan mengelola file project.
- `ufw` digunakan untuk firewall sederhana.

---

## 2. Install Apache2

Install Apache2:

```bash
apt install apache2 -y
```

Aktifkan Apache agar otomatis berjalan saat server menyala:

```bash
systemctl enable apache2
systemctl start apache2
```

Cek status Apache:

```bash
systemctl status apache2
```

Buka firewall untuk SSH dan web server:

```bash
ufw allow OpenSSH
ufw allow 'Apache Full'
ufw enable
ufw status
```

Tes melalui browser:

```text
http://IP_VPS_KAMU
```

Jika halaman default Apache muncul, berarti Apache sudah berhasil berjalan.

---

## 3. Install PHP 8.4 dan Extension Framework PHP

PHP 8.4 biasanya belum tersedia secara default di beberapa versi Ubuntu, sehingga perlu menambahkan repository PHP tambahan.

Tambahkan repository PHP:

```bash
add-apt-repository ppa:ondrej/php -y
apt update
```

Install PHP 8.4 dan extension yang umum dibutuhkan framework PHP:

```bash
apt install -y php8.4 php8.4-cli php8.4-common php8.4-fpm php8.4-mysql php8.4-mbstring php8.4-xml php8.4-curl php8.4-zip php8.4-gd php8.4-intl php8.4-bcmath php8.4-soap php8.4-readline php8.4-opcache php8.4-sqlite3 php8.4-tokenizer
```

Cek versi PHP:

```bash
php -v
```

Penjelasan extension:

- `php8.4-fpm`: menjalankan PHP melalui PHP-FPM.
- `php8.4-mysql`: koneksi PHP ke MySQL.
- `php8.4-mbstring`: kebutuhan string multibyte, sering dipakai Laravel/CI.
- `php8.4-xml`: parsing XML.
- `php8.4-curl`: HTTP request dari PHP.
- `php8.4-zip`: membaca dan membuat file ZIP.
- `php8.4-gd`: manipulasi gambar.
- `php8.4-intl`: fitur internationalization.
- `php8.4-bcmath`: perhitungan angka presisi tinggi.
- `php8.4-opcache`: optimasi performa PHP.

---

## 4. Hubungkan PHP-FPM dengan Apache

Aktifkan module Apache yang dibutuhkan:

```bash
a2enmod proxy_fcgi setenvif rewrite headers ssl
```

Aktifkan konfigurasi PHP 8.4 FPM:

```bash
a2enconf php8.4-fpm
```

Restart service:

```bash
systemctl restart php8.4-fpm
systemctl restart apache2
```

Cek status PHP-FPM:

```bash
systemctl status php8.4-fpm
```

Penjelasan:

- `proxy_fcgi` digunakan agar Apache dapat meneruskan eksekusi PHP ke PHP-FPM.
- `rewrite` dibutuhkan oleh framework seperti Laravel dan CodeIgniter untuk routing.
- `headers` berguna untuk pengaturan HTTP header.
- `ssl` digunakan jika nanti memasang HTTPS.

---

## 5. Konfigurasi PHP Upload

Edit file konfigurasi PHP-FPM:

```bash
nano /etc/php/8.4/fpm/php.ini
```

Cari dan ubah nilai berikut:

```ini
upload_max_filesize = 100M
post_max_size = 100M
memory_limit = 256M
max_execution_time = 300
max_input_time = 300
date.timezone = Asia/Jakarta
```

Restart service:

```bash
systemctl restart php8.4-fpm
systemctl restart apache2
```

Penjelasan:

- `upload_max_filesize`: maksimal ukuran file upload.
- `post_max_size`: maksimal ukuran request POST.
- `memory_limit`: batas penggunaan memori PHP.
- `max_execution_time`: waktu maksimal eksekusi script.
- `date.timezone`: zona waktu server untuk PHP.

---

## 6. Test PHP di Apache

Buat file test:

```bash
nano /var/www/html/info.php
```

Isi:

```php
<?php phpinfo();
```

Buka di browser:

```text
http://IP_VPS_KAMU/info.php
```

Jika halaman PHP info muncul, berarti PHP sudah berjalan melalui Apache.

Setelah selesai, hapus file tersebut demi keamanan:

```bash
rm /var/www/html/info.php
```

---

## 7. Install dan Konfigurasi MySQL Server

> **Penting:** MySQL harus diinstall dan dikonfigurasi terlebih dahulu sebelum menginstall phpMyAdmin di Tahap 14. Langkah-langkah di bawah ini juga mencakup antisipasi agar instalasi phpMyAdmin tidak gagal karena aturan password policy MySQL.

### 7.1 Install MySQL Server

```bash
sudo apt update
sudo apt install mysql-server -y
```

### 7.2 Aktifkan dan Jalankan Service MySQL

```bash
sudo systemctl enable mysql
sudo systemctl start mysql
sudo systemctl status mysql
```

Pastikan status menunjukkan `active (running)`.

### 7.3 Jalankan Konfigurasi Keamanan MySQL

```bash
sudo mysql_secure_installation
```

Saat proses berjalan, akan muncul beberapa pertanyaan. Ikuti rekomendasi berikut:

**Pertanyaan pertama — VALIDATE PASSWORD COMPONENT:**

```text
VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No:
```

> ⚠️ **Pilih: `N`**
>
> **Penjelasan:** Pilih `N` agar password policy MySQL tidak terlalu ketat. Jika memilih `Y`, MySQL akan menerapkan aturan password yang sangat ketat dan bisa menyebabkan error saat instalasi phpMyAdmin di Tahap 14, yaitu:
>
> ```text
> ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
> ```

Setelah itu, ikuti pertanyaan selanjutnya:

```text
Set root password?             : Y (lalu masukkan password root MySQL)
Remove anonymous users?        : Y
Disallow root login remotely?  : Y
Remove test database?          : Y
Reload privilege tables?       : Y
```

### 7.4 Nonaktifkan Password Policy (Antisipasi phpMyAdmin)

Meskipun sudah memilih `N` pada langkah di atas, ada kemungkinan komponen password policy sudah aktif dari sebelumnya. Langkah ini memastikan komponen tersebut benar-benar mati sebelum install phpMyAdmin.

Masuk ke MySQL:

```bash
sudo mysql
```

Cek apakah komponen password policy masih aktif:

```sql
SELECT * FROM mysql.component;
```

Jika hasil query menampilkan baris yang berisi:

```text
file://component_validate_password
```

Maka **matikan sementara** dengan perintah berikut:

```sql
UNINSTALL COMPONENT 'file://component_validate_password';
```

Keluar dari MySQL:

```sql
EXIT;
```

> 💡 **Mengapa langkah ini penting?**
>
> Langkah ini adalah **antisipasi** agar instalasi phpMyAdmin di Tahap 14 tidak gagal. Saat install phpMyAdmin, sistem akan otomatis membuat user dan database internal. Jika password policy MySQL aktif, proses ini bisa gagal karena password default phpMyAdmin dianggap tidak memenuhi kebijakan. Setelah phpMyAdmin berhasil terinstall, password policy bisa diaktifkan kembali (lihat bagian akhir Tahap 14).

### 7.5 Contoh Membuat Database dan User (Opsional)

Langkah ini bisa dilakukan sekarang atau nanti setelah semua service siap.

```bash
sudo mysql
```

```sql
CREATE DATABASE nama_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'user_database'@'localhost' IDENTIFIED BY 'PasswordKuat@2026!';
GRANT ALL PRIVILEGES ON nama_database.* TO 'user_database'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Contoh konfigurasi database di `.env` framework:

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_database
DB_USERNAME=user_database
DB_PASSWORD=PasswordKuat@2026!
```

---

## 8. Install Composer

Composer digunakan untuk mengelola dependency framework PHP seperti Laravel dan CodeIgniter 4.

```bash
cd /tmp
curl -sS https://getcomposer.org/installer -o composer-setup.php
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
composer -V
```

Jika versi Composer muncul, berarti Composer sudah berhasil terinstall.

---

## 9. Struktur Folder Website

Contoh struktur yang digunakan:

```text
/var/www/
├── webuser/
│   └── public_html/
│       └── index.php
├── domain1.com/
│   └── public_html/
└── project-laravel/
    ├── app/
    ├── public/
    ├── storage/
    └── .env
```

Buat folder website contoh:

```bash
mkdir -p /var/www/webuser/public_html
```

Set permission:

```bash
chown -R www-data:www-data /var/www/webuser
chmod -R 755 /var/www/webuser
```

Buat file test:

```bash
echo "<?php echo 'Website PHP berhasil jalan!'; ?>" > /var/www/webuser/public_html/index.php
```

---

## 10. Konfigurasi Virtual Host Apache

Buat file virtual host:

```bash
nano /etc/apache2/sites-available/webuser.conf
```

Isi:

```apache
<VirtualHost *:80>
    ServerName IP_VPS_KAMU

    DocumentRoot /var/www/webuser/public_html

    <Directory /var/www/webuser/public_html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/webuser_error.log
    CustomLog ${APACHE_LOG_DIR}/webuser_access.log combined
</VirtualHost>
```

Aktifkan site:

```bash
a2ensite webuser.conf
a2dissite 000-default.conf
apache2ctl configtest
systemctl reload apache2
```

Jika hasilnya `Syntax OK`, konfigurasi Apache valid.

---

## 11. Install dan Konfigurasi FTP dengan vsftpd

Install vsftpd:

```bash
apt install vsftpd -y
```

Backup konfigurasi default:

```bash
cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
```

Edit konfigurasi:

```bash
nano /etc/vsftpd.conf
```

Gunakan atau pastikan konfigurasi berikut aktif:

```ini
listen=NO
listen_ipv6=YES

anonymous_enable=NO
local_enable=YES
write_enable=YES

local_umask=022
chroot_local_user=YES
allow_writeable_chroot=YES

pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100

user_sub_token=$USER
local_root=/var/www/$USER

utf8_filesystem=YES
```

Restart dan aktifkan vsftpd:

```bash
systemctl restart vsftpd
systemctl enable vsftpd
systemctl status vsftpd
```

Buka firewall FTP:

```bash
ufw allow 20/tcp
ufw allow 21/tcp
ufw allow 40000:40100/tcp
ufw reload
```

Penjelasan:

- `anonymous_enable=NO`: menonaktifkan akses anonymous.
- `local_enable=YES`: mengizinkan user lokal Linux login FTP.
- `write_enable=YES`: mengizinkan upload/edit file.
- `chroot_local_user=YES`: membatasi user di folder miliknya.
- `pasv_min_port` dan `pasv_max_port`: range port passive FTP untuk FileZilla.
- `local_root=/var/www/$USER`: user FTP akan diarahkan ke folder `/var/www/namauser`.

---

## 12. Buat User FTP untuk Upload Web

Contoh membuat user bernama `webuser`:

```bash
adduser webuser
```

Buat folder website user:

```bash
mkdir -p /var/www/webuser/public_html
```

Atur owner dan permission:

```bash
chown -R webuser:www-data /var/www/webuser
chmod -R 775 /var/www/webuser
usermod -aG www-data webuser
```

Buat file test:

```bash
echo "<?php echo 'Upload FTP berhasil'; ?>" > /var/www/webuser/public_html/index.php
chown webuser:www-data /var/www/webuser/public_html/index.php
```

Konfigurasi FileZilla:

```text
Host     : IP_VPS_KAMU
Protocol : FTP
Port     : 21
Username : webuser
Password : password user webuser
```

Folder upload website:

```text
public_html/
```

---

## 13. Konfigurasi Laravel

Upload project Laravel ke folder:

```text
/var/www/webuser/project
```

Masuk folder project:

```bash
cd /var/www/webuser/project
```

Install dependency:

```bash
composer install
```

Buat file environment:

```bash
cp .env.example .env
```

Generate application key:

```bash
php artisan key:generate
```

Atur permission Laravel:

```bash
chown -R webuser:www-data /var/www/webuser/project
chmod -R 775 /var/www/webuser/project/storage
chmod -R 775 /var/www/webuser/project/bootstrap/cache
```

Virtual host Laravel harus mengarah ke folder `public`:

```apache
<VirtualHost *:80>
    ServerName domainkamu.com

    DocumentRoot /var/www/webuser/project/public

    <Directory /var/www/webuser/project/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/laravel_error.log
    CustomLog ${APACHE_LOG_DIR}/laravel_access.log combined
</VirtualHost>
```

Aktifkan rewrite dan restart Apache:

```bash
a2enmod rewrite
apache2ctl configtest
systemctl restart apache2
```

---

## 14. Install dan Konfigurasi phpMyAdmin

> **Catatan:** Pastikan Tahap 7 (Install dan Konfigurasi MySQL Server) sudah selesai dilakukan, termasuk menonaktifkan password policy MySQL. Jika belum, instalasi phpMyAdmin bisa gagal dengan error `ERROR 1819 (HY000)`.

### 14.1 Install phpMyAdmin

```bash
sudo apt update
sudo apt install phpmyadmin -y
```

Saat proses instalasi, akan muncul beberapa dialog konfigurasi. Ikuti langkah berikut dengan teliti:

**Pilihan Web Server:**

```text
┌─ Configuring phpmyadmin ─────────────────────────┐
│ Web server to reconfigure automatically:         │
│                                                   │
│    [ ] apache2                                    │
│    [ ] lighttpd                                   │
│                                                   │
└───────────────────────────────────────────────────┘
```

> ⚠️ **Penting:**
>
> 1. Arahkan kursor ke **`apache2`**
> 2. Tekan tombol **`Space`** sampai muncul tanda **`[*] apache2`**
> 3. Baru kemudian tekan **`Enter`**
>
> **Jangan langsung tekan Enter sebelum `apache2` tercentang!** Jika langsung tekan Enter tanpa memilih, phpMyAdmin tidak akan terhubung ke Apache dan halaman `/phpmyadmin` tidak bisa diakses.

**Konfigurasi Database:**

```text
Configure database for phpmyadmin with dbconfig-common?
```

Pilih: **`Yes`**

**Password phpMyAdmin:**

Masukkan password yang kuat untuk database internal phpMyAdmin, contoh:

```text
PmaAdmin@2026!
```

### 14.2 Aktifkan Konfigurasi phpMyAdmin di Apache

Setelah instalasi selesai, pastikan konfigurasi phpMyAdmin aktif di Apache. Jalankan perintah berikut:

```bash
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin
sudo systemctl reload apache2
```

> 💡 **Penjelasan:** Perintah `ln -s` membuat symbolic link dari file konfigurasi phpMyAdmin ke folder Apache. Jika link sudah ada sebelumnya, perintah ini akan menampilkan pesan error yang bisa diabaikan. Perintah `a2enconf` mengaktifkan konfigurasi tersebut, dan `systemctl reload` memuat ulang Apache tanpa downtime.

### 14.3 Cek Akses phpMyAdmin di Browser

Buka browser dan akses salah satu URL berikut:

```text
http://IP-VPS/phpmyadmin
```

atau jika sudah menggunakan domain:

```text
http://domainkamu.com/phpmyadmin
```

Jika halaman login phpMyAdmin muncul, berarti instalasi berhasil.

### 14.4 Buat User Login phpMyAdmin

> ⚠️ **Mengapa harus buat user baru?**
>
> User `root` MySQL di Ubuntu secara default menggunakan metode autentikasi `auth_socket`, yang berarti hanya bisa diakses dari terminal Linux menggunakan `sudo mysql`. User `root` **tidak bisa** digunakan untuk login phpMyAdmin melalui browser. Oleh karena itu, kita perlu membuat user baru.

#### Opsi A: User Admin (Akses Semua Database)

User ini cocok untuk kebutuhan administrasi atau pembelajaran.

```bash
sudo mysql
```

```sql
CREATE USER 'admin_db'@'localhost' IDENTIFIED BY 'AdminDB@2026!';
GRANT ALL PRIVILEGES ON *.* TO 'admin_db'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

Login phpMyAdmin dengan user admin:

```text
URL      : http://IP-VPS/phpmyadmin
Username : admin_db
Password : AdminDB@2026!
```

#### Opsi B: User Khusus Per Database (Lebih Aman)

User ini hanya bisa mengakses satu database tertentu. Cocok untuk project production.

```bash
sudo mysql
```

```sql
CREATE DATABASE web_app CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'web_user'@'localhost' IDENTIFIED BY 'WebUser@2026!';
GRANT ALL PRIVILEGES ON web_app.* TO 'web_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Login phpMyAdmin dengan user khusus:

```text
URL      : http://IP-VPS/phpmyadmin
Username : web_user
Password : WebUser@2026!
```

#### Perbedaan Kedua User

| User | Hak Akses | Cocok Untuk |
|------|-----------|-------------|
| `admin_db` | Bisa mengelola **semua database** | Administrasi, development, pembelajaran |
| `web_user` | Hanya bisa mengelola database **`web_app`** | Project production, keamanan lebih baik |

> 🔒 **Catatan Keamanan:**
>
> Untuk pembelajaran boleh menggunakan `admin_db`, tetapi untuk project production **lebih disarankan** membuat user khusus per database seperti `web_user`. Dengan begitu, jika satu user bocor, database lain tetap aman.

### 14.5 Aktifkan Kembali Password Policy MySQL (Opsional)

Jika phpMyAdmin sudah berhasil terinstall dan semua user sudah dibuat, Anda bisa mengaktifkan kembali password policy MySQL untuk meningkatkan keamanan:

```bash
sudo mysql
```

```sql
INSTALL COMPONENT 'file://component_validate_password';
EXIT;
```

> 💡 Setelah diaktifkan, semua password baru yang dibuat di MySQL harus memenuhi kebijakan keamanan (minimal 8 karakter, kombinasi huruf besar, huruf kecil, angka, dan simbol).

### 14.6 Troubleshooting phpMyAdmin

Berikut beberapa error yang sering terjadi saat install atau mengakses phpMyAdmin beserta solusinya:

---

#### ❌ Error: `Your password does not satisfy the current policy requirements`

```text
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

**Penyebab:** Password policy MySQL terlalu ketat sehingga menolak password yang dibuat saat instalasi phpMyAdmin.

**Solusi:**

```bash
sudo mysql
```

```sql
UNINSTALL COMPONENT 'file://component_validate_password';
EXIT;
```

Lalu ulangi instalasi phpMyAdmin:

```bash
sudo apt install phpmyadmin -y
```

> Jika instalasi sebelumnya sudah setengah jalan dan gagal, bersihkan dulu:
>
> ```bash
> sudo apt remove --purge phpmyadmin -y
> sudo apt autoremove -y
> sudo rm -f /etc/dbconfig-common/phpmyadmin.conf
> sudo rm -rf /etc/phpmyadmin
> sudo dpkg --configure -a
> sudo apt --fix-broken install -y
> ```
>
> Baru kemudian install ulang:
>
> ```bash
> sudo apt update
> sudo apt install phpmyadmin -y
> ```

---

#### ❌ Tidak Bisa Login phpMyAdmin dengan User `root`

**Penyebab:** User `root` MySQL di Ubuntu menggunakan metode autentikasi `auth_socket`, yang hanya bisa diakses dari terminal dengan `sudo mysql`, bukan dari browser.

**Solusi:** Gunakan user baru seperti `admin_db` atau `web_user` yang sudah dibuat di langkah 14.4.

---

#### ❌ Halaman `/phpmyadmin` Tidak Ditemukan (404)

**Penyebab:** Konfigurasi phpMyAdmin belum diaktifkan di Apache.

**Solusi:**

```bash
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin
sudo systemctl reload apache2
```

Jika perintah `a2enconf phpmyadmin` menampilkan error:

```text
ERROR: Conf phpmyadmin does not exist!
```

Artinya file konfigurasi belum tersedia. Buat secara manual:

```bash
sudo nano /etc/apache2/conf-available/phpmyadmin.conf
```

Isi dengan:

```apache
Alias /phpmyadmin /usr/share/phpmyadmin

<Directory /usr/share/phpmyadmin>
    Options SymLinksIfOwnerMatch
    DirectoryIndex index.php
    Require all granted
</Directory>

<Directory /usr/share/phpmyadmin/setup>
    Require all denied
</Directory>

<Directory /usr/share/phpmyadmin/libraries>
    Require all denied
</Directory>
```

Lalu aktifkan:

```bash
sudo a2enconf phpmyadmin
sudo systemctl restart apache2
```

---

## 15. Error Browser: ERR_NO_BUFFER_SPACE

Saat login phpMyAdmin sempat muncul:

```text
This site can't be reached
ERR_NO_BUFFER_SPACE
```

Kemungkinan penyebab:

- Masalah cache/browser.
- Koneksi network lokal bermasalah.
- Apache/PHP-FPM perlu restart.
- VPS kekurangan resource.

Solusi yang dilakukan/dapat dilakukan:

Restart service:

```bash
systemctl restart mysql
systemctl restart php8.4-fpm
systemctl restart apache2
```

Cek status:

```bash
systemctl status mysql --no-pager
systemctl status php8.4-fpm --no-pager
systemctl status apache2 --no-pager
```

Cek akses dari VPS:

```bash
curl -I http://localhost/phpmyadmin/
```

Cek resource:

```bash
free -h
df -h
top
```

Jika RAM kecil, buat swap 1GB:

```bash
fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

---

## 16. Perintah Monitoring dan Troubleshooting

Cek Apache:

```bash
systemctl status apache2
apache2ctl configtest
journalctl -xeu apache2
```

Cek PHP-FPM:

```bash
systemctl status php8.4-fpm
journalctl -xeu php8.4-fpm
```

Cek MySQL:

```bash
systemctl status mysql
journalctl -xeu mysql
```

Cek FTP:

```bash
systemctl status vsftpd
journalctl -xeu vsftpd
```

Cek log Apache:

```bash
tail -f /var/log/apache2/error.log
```

Cek port aktif:

```bash
ss -tulpn
```

Cek port web:

```bash
ss -tulpn | grep ':80'
```

Cek firewall:

```bash
ufw status
```

---

## 17. Ringkasan Command Utama

```bash
apt update && apt upgrade -y

apt install -y apache2 mysql-server vsftpd curl wget unzip git nano software-properties-common ca-certificates lsb-release apt-transport-https ufw

add-apt-repository ppa:ondrej/php -y
apt update

apt install -y php8.4 php8.4-cli php8.4-common php8.4-fpm php8.4-mysql php8.4-mbstring php8.4-xml php8.4-curl php8.4-zip php8.4-gd php8.4-intl php8.4-bcmath php8.4-soap php8.4-readline php8.4-opcache php8.4-sqlite3 php8.4-tokenizer

a2enmod proxy_fcgi setenvif rewrite headers ssl
a2enconf php8.4-fpm
systemctl restart apache2 php8.4-fpm

mysql_secure_installation

apt install phpmyadmin -y

a2enconf phpmyadmin
systemctl restart apache2
```

---

## 18. Checklist Akhir

Pastikan semua service aktif:

```bash
systemctl status apache2 --no-pager
systemctl status php8.4-fpm --no-pager
systemctl status mysql --no-pager
systemctl status vsftpd --no-pager
```

Pastikan firewall membuka port penting:

```bash
ufw status
```

Port yang dibutuhkan:

```text
22/tcp       SSH
80/tcp       HTTP
443/tcp      HTTPS
20/tcp       FTP Data
21/tcp       FTP Control
40000:40100  FTP Passive Mode
```

Akses yang harus berhasil:

```text
http://IP_VPS_KAMU
http://IP_VPS_KAMU/phpmyadmin
```

FTP FileZilla:

```text
Host     : IP_VPS_KAMU
Port     : 21
Username : webuser
Password : password webuser
Folder   : public_html/
```

---

## 19. Catatan Keamanan

Untuk server produksi, disarankan:

1. Gunakan HTTPS dengan SSL, misalnya Let's Encrypt.
2. Jangan gunakan password database yang mudah ditebak.
3. Jangan izinkan login MySQL root dari remote.
4. Gunakan SFTP jika memungkinkan, karena FTP biasa tidak terenkripsi.
5. Batasi akses phpMyAdmin, misalnya dengan IP whitelist atau basic auth.
6. Hapus file `phpinfo()` setelah testing.
7. Rutin update server:

```bash
apt update && apt upgrade -y
```

---

## 20. Status Konfigurasi Saat Ini

Berdasarkan konfigurasi yang sudah dilakukan:

- Apache2 berhasil terinstall.
- PHP 8.4 FPM berhasil digunakan dengan Apache.
- MySQL Server berhasil terinstall.
- phpMyAdmin berhasil diakses melalui browser.
- Error password policy MySQL berhasil ditangani.
- FTP upload web menggunakan vsftpd sudah disiapkan.
- Folder web diarahkan ke struktur `/var/www/<user>/public_html`.

