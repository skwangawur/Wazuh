## Persiapan Alat yang digunakan

1. VMWWare / Virtual Box
2. Ova Wazuh versi 4.0.4 (CentOS)
3. Ubuntu server (yang telah terinstall dvwa)
4. Jaringan Internet

## Aktif Respone

Respon aktif merupakan modul yang dimiliki oleh wazuh untuk melakukan tindakan selanjutnya ketika wazuh mendeteksi ancaman atau bahaya pada endpoint yang telah ditentukan aturannya. Wazuh sendiri secara default menyediakan beberapa tindakan seperti firewall-drop, host-deny dll.

## Cara Kerja Aktif Respon Wazuh

![workflow](/Respon%20Aktif/images/active-response-workflow1.png)

## Default aktif respon pada wazuh

-   disable-account: Menonaktifkan akun pengguna yang terdeteksi melakukan aktivitas mencurigakan atau berbahaya.

-   firewall-drop: Menambahkan alamat IP ke daftar blokir di iptables, mencegah alamat IP tersebut mengakses sistem lebih lanjut.

-   firewalld-drop: Menambahkan alamat IP ke daftar drop firewalld. Memerlukan firewalld yang terpasang pada titik akhir untuk memblokir akses dari alamat IP tersebut.

-   host-deny: Menambahkan alamat IP ke file /etc/hosts.deny, yang digunakan oleh TCP Wrappers untuk menolak koneksi dari alamat IP tersebut.

-   ip-customblock: Skrip blokir IP khusus yang dapat dimodifikasi sesuai kebutuhan spesifik untuk respons keamanan yang lebih fleksibel.
-   ipfw: Skrip respons firewall-drop yang khusus untuk IPFW (IP Firewall), memerlukan IPFW yang terpasang pada titik akhir untuk memblokir alamat IP.

-   npf: Skrip respons firewall-drop yang dibuat untuk NPF (NetBSD Packet Filter), memerlukan NPF yang terpasang di titik akhir.

-   wazuh-slack: Mengirimkan notifikasi ke Slack ketika mendeteksi ancaman. Memerlukan URL webhook Slack yang dikonfigurasi sebagai extra_args.

-   pf: Skrip respons firewall-drop yang dibuat untuk PF (Packet Filter), memerlukan PF yang terpasang pada titik akhir.

-   restart-wazuh: Memulai ulang agen atau manajer Wazuh, yang berguna dalam situasi di mana perlu menyegarkan atau memulihkan layanan Wazuh.

-   route-null: Menambahkan alamat IP ke rute nol, yang efektif mengabaikan lalu lintas dari IP tersebut.

-   kaspersky: Integrasi dengan Kaspersky Endpoint Security for Linux, menggunakan CLI Kaspersky untuk menjalankan perintah yang relevan berdasarkan deteksi ancaman.

## Langkah-Langkah

Pada kali ini, kita akan mencoba mengaktifkan respon dengan memblokir ip sumber dengan melakukan simulasi bahwa ip sumber / penyerang melakukan serangan dengan teknik sql injection pada endpoint kita yang telah terpasang wazuh agent.

1. Buka wazuh manager / vm wazuh anda.
2. Buka configurasi **ossec.conf** pada wazuh manager

```
sudo nano /var/ossec/etc/ossec.conf
```

![konfigurasi](/Respon%20Aktif/images/konfigurasi.png)

3. tambahkan script berikut

```
<active-response>
  <command>firewall-drop</command>
  <location>all</location>
  <rules_id>31164</rules_id>
  <timeout>3600</timeout>
</active-response>
```

![konfigurasi](/Respon%20Aktif/images/konfigurasi-aktif-respon.png)

4. Save konfigurasi yang telah ditambahkan

```
1. ctrl + x
2. klik tombol Y
3. Enter
```

5. Restart Wazuh manager anda.

```
systemctl restart wazuh.manager.service
```

## Simulasi Serangan

1. Buka browser anda, dan tuliskan alamat ip endpoint anda yang telah terinstall DVWA **http://alamat-ip-endpoint-ubuntu/DVWA**
2. Ubah level DVWA anda menjadi easy
3. Lakukan simulasi serangan sql injection dengan memasukan payload berikut.

```
1 ' OR '1' = '1' #

```

![konfigurasi](/Respon%20Aktif/images/simulasi.png)

4. Buka security event pada wazuh dashboard anda, cek apakah terdeteksi serangan sql injection

![konfigurasi](/Respon%20Aktif/images/wazuh.png)

5. Buka kembali browser anda, dan refresh kembali web DVWA. Apabila anda tidak bisa mengaksesnya kembali, berarti sekarang alamat ip anda (komputer host) telah diblokir oleh wazuh.

![konfigurasi](/Respon%20Aktif/images/blokir-ip.png)

6. Cek log aktif respon

```
sudo cat /var/ossec/logs/active-response.log
```

![konfigurasi](/Respon%20Aktif/images/log-aktif-respon.png)

# ID Rules pada Wazuh

1.  31103, 31164, 31165 – SQL injection attempt,
2.  31104, 31153 – Common web attack (directory traversal),
3.  31105, 31154 – XSS Attempt,
4.  31106 – Web attack returned code 200,
5.  31109 – MSSQL Injection,
6.  31110 – PHP-cgi bin attack,
7.  31151 – Multiple error 400 (possible reconnaissance),
8.  31161, 31162, 31163 – Multilple error 500 (possible reconnaissance),
9.  31166, 31167, 31168, 31169 – ShellShock Attack
