## Persiapan Alat yang digunakan

1. VMWWare / Virtual Box
2. Ova Wazuh versi 4.0.4
3. Ubuntu server (yang telah terinstall dvwa)
4. Akun google email
5. Jaringan Internet

## Integrasi Wazuh dengan Email

Integrasi wazuh dengan email berfungsi untuk mengirimkan aktivitas berbahaya yang terjadi pada endpoint (Ubuntu). Sebelumnya wazuh merupakan perangkat security yang dapat digunakan sebagai SIEM maupun EDR untuk memonitoring aktivitas yang terjadi pada perangkat endpoint.

## Langkah-langkah integrasi

1. Lakukan installasi beberapa paket, karena wazuh yang digunakan pada saat ini berbasi CentOS, maka ada sedikit perbedaan command apabila yang kalian gunakan versi wazuh terbaru yang berbasis Ubuntu.

```
yum update && yum install postfix mailx cyrus-sasl cyrus-sasl-plain
```

2. Tambahkan konfigurasi pada postfix, pada /etc/postfix/main.cf

```
nano /etc/postfix/main.cf
```

Isi dari file main.cf

```
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_use_tls = yes
```

3.  tambahkan konfigurasi **[smtp.gmail.com]:587 USERNAME@gmail.com:PASSWORD** ke **/etc/postfix/sasl_passwd**

```
echo [smtp.gmail.com]:587 <USERNAME>@gmail.com:<PASSWORD> > /etc/postfix/sasl_passwd
```

Untuk password, kalian harus menggunakan kata sandi aplikasi google di [Sandi Aplikasi](https://accounts.google.com/v3/signin/challenge/pwd?TL=AC3PFD4uHiSvrULzbS61VvaGpN4weXuHyn3ns73LPqLHsk070XikyiyzJ7qlX6iR&cid=2&continue=https%3A%2F%2Fmyaccount.google.com%2Fapppasswords&flowName=GlifWebSignIn&followup=https%3A%2F%2Fmyaccount.google.com%2Fapppasswords&ifkv=AS5LTAT99jg2ZMFCtCbegnMPnOOTawrFPFxP7hU2BKANy5Z7HHKD-zTm5KeRg8pfxCggxE-8yW2GdA&osid=1&rart=ANgoxceeQBftB2DQCxDICvVWR3mShVwWlXcLsMTrMddCdibvKLVJS2I4ZadhRz9CtPlP4dldRvV80zBZvIReIFfiOinK5HX5GsdMPNEjrIuwkmQ71hQfsL8&rpbg=1&service=accountsettings), yang nantinya kalian akan mendapatkan 16 karakter sebagai password untuk konfigurasi diatas, sebelum mengunjungi kata sandi aplikasi, kalian harus mengaktifkan verifikasi 2 langkah terlebih dahulu.

![Sandi APlikasi](/Integrasi%20Email/images/sandi-aplikasi.png)

4. Membuat file datatabase yang telah di hash menggunakan **postfix** dari **/etc/postfix/sasl_passwd** yang akan mendapatkan output berupa **/etc/postfix/sasl_passwd.db**

```
postmap /etc/postfix/sasl_passwd
```

5. Ubah izin file **/etc/postfix/sasl_passwd**

```
chmod 400 /etc/postfix/sasl_passwd
```

**\*FYI** angka pada konfigurasi diatas memiliki beberapa maksud.

```
1. Angka pertama, merupakan izin untuk pemilik (user utama).
2. Angka kedua, merupakan izin untuk grub
3. Angka yang ketiga, merupakan izin untuk user lainnya

Setiap nilai angka juga memiliki arti:

4 : izin baca
2 : izin tulis
1 : izin eksekusi
0 : tidak memiliki izin

jadi chmod 400 diatasi hanya mengizinkan pemilik untuk membaca saja, sedangkan untuk grub maupun user lainnya tidak memiliki izin apapun. Jika pemilik atau user utama ingin memiliki izin baca, tulis, dan eksekusi (4 + 2 + 1), maka perintah diatas menjadi chmod 700
```

6. Memberikan izin kepemilikan file **chown pemilik:grub file**

```
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

7. Ubah izin akses file

```
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

8. Restart postfix

```
systemctl restart postfix
```

9. Melakukan pengujian pengiriman email

```
echo "Test mail from postfix" | mail -s "Test Postfix" -r "username@gmail.com" username@gmail.com
```

10. ubah konfigurasi wazuh server di **/var/ossec/etc/ossec.conf**

![Ossec.conf](/Integrasi%20Email/images/wazuh.png)

```
nano /var/ossec/etc/ossec.conf
```

```
<global>
  <email_notification>yes</email_notification>
  <smtp_server>localhost</smtp_server>
  <email_from><USERNAME>@gmail.com</email_from>
  <email_to>you@example.com</email_to>
</global>
```

11. Restart wazuh manager

```
systemctl restart wazuh-manager
```

12. Uji coba sql injection pada dvwa, jangan lupa ganti security level menjadi easy.

```
Payload Menguji kerentanan SQL

' OR '1' = '1' #
```

![sql injection](/Integrasi%20Email/images/dvwa-sql-injection.png)

13. Mendapatkan alert pada email

![email alert](/Integrasi%20Email/images/dvwa-1.jpeg)
![email alert](/Integrasi%20Email/images/dvwa-2.jpeg)
