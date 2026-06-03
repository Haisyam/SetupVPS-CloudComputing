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

## 7. Install MySQL Server

Install MySQL:

```bash
apt install mysql-server -y
```

Aktifkan MySQL:

```bash
systemctl enable mysql
systemctl start mysql
```

Cek status:

```bash
systemctl status mysql
```

Jalankan konfigurasi keamanan:

```bash
mysql_secure_installation
```

Rekomendasi pilihan:

```text
VALIDATE PASSWORD: boleh Y atau N
Remove anonymous users: Y
Disallow root login remotely: Y
Remove test database: Y
Reload privilege tables: Y
```

Masuk MySQL:

```bash
mysql
```

Contoh membuat database dan user:

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

## 14. Install phpMyAdmin

Install phpMyAdmin:

```bash
apt install phpmyadmin -y
```

Saat muncul pilihan:

```text
Configure database for phpmyadmin with dbconfig-common?
```

Pilih:

```text
Yes
```

Saat memilih web server, pilih:

```text
apache2
```

Tekan `Space` sampai muncul tanda `[*]`, lalu tekan `Enter`.

Gunakan password kuat untuk database phpMyAdmin, contoh:

```text
PmaAdmin@2026!
```

---

## 15. Catatan Error phpMyAdmin: Password Policy MySQL

Saat instalasi, sempat terjadi error:

```text
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

Penyebab:

MySQL memiliki aturan password policy yang menolak password yang dianggap lemah.

Cek plugin MySQL:

```sql
SHOW PLUGINS;
```

Pada server ini, plugin `validate_password` tidak terlihat di daftar plugin. Kemungkinan policy berasal dari component MySQL.

Cek component:

```sql
SELECT * FROM mysql.component;
```

Jika ada:

```text
file://component_validate_password
```

Matikan sementara:

```sql
UNINSTALL COMPONENT 'file://component_validate_password';
```

Setelah itu bersihkan instalasi phpMyAdmin yang gagal:

```bash
apt remove --purge phpmyadmin -y
apt autoremove -y
rm -f /etc/dbconfig-common/phpmyadmin.conf
rm -rf /etc/phpmyadmin
dpkg --configure -a
apt --fix-broken install -y
```

Install ulang:

```bash
apt update
apt install phpmyadmin -y
```

Jika ingin mengaktifkan lagi password policy setelah selesai:

```sql
INSTALL COMPONENT 'file://component_validate_password';
```

---

## 16. Aktifkan phpMyAdmin di Apache

Cek apakah konfigurasi phpMyAdmin tersedia:

```bash
ls /etc/apache2/conf-available | grep phpmyadmin
```

Jika muncul `phpmyadmin.conf`, aktifkan:

```bash
a2enconf phpmyadmin
systemctl restart apache2
```

Akses:

```text
http://IP_VPS_KAMU/phpmyadmin
```

---

## 17. Jika `a2enconf phpmyadmin` Error

Jika muncul error:

```text
ERROR: Conf phpmyadmin does not exist!
```

Artinya file konfigurasi Apache untuk phpMyAdmin belum tersedia. Buat manual:

```bash
nano /etc/apache2/conf-available/phpmyadmin.conf
```

Isi:

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

Aktifkan:

```bash
a2enconf phpmyadmin
systemctl restart apache2
```

---

## 18. Buat User MySQL untuk Login phpMyAdmin

Masuk MySQL:

```bash
mysql
```

Buat user admin:

```sql
CREATE USER 'adminpma'@'localhost' IDENTIFIED BY 'AdminPma@2026!';
GRANT ALL PRIVILEGES ON *.* TO 'adminpma'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

Login phpMyAdmin:

```text
URL      : http://IP_VPS_KAMU/phpmyadmin
Username : adminpma
Password : AdminPma@2026!
```

---

## 19. Error Browser: ERR_NO_BUFFER_SPACE

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

## 20. Perintah Monitoring dan Troubleshooting

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

## 21. Ringkasan Command Utama

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

## 22. Checklist Akhir

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

## 23. Catatan Keamanan

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

## 24. Status Konfigurasi Saat Ini

Berdasarkan konfigurasi yang sudah dilakukan:

- Apache2 berhasil terinstall.
- PHP 8.4 FPM berhasil digunakan dengan Apache.
- MySQL Server berhasil terinstall.
- phpMyAdmin berhasil diakses melalui browser.
- Error password policy MySQL berhasil ditangani.
- FTP upload web menggunakan vsftpd sudah disiapkan.
- Folder web diarahkan ke struktur `/var/www/<user>/public_html`.

