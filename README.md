# Lapres Modul 3 Jarkom 2020 - T1  
`"Repository dibuat untuk memenuhi tugas praktikum mata kuliah komunikasi data dan jaringan komputer tahun 2020."`  
  
Anggota:  
**Adeela Nurul Fadhila** `[05311840000001]` [@Rinnabel](https://github.com/Rinnabel)  
**Muhammad Ilya Asha Soegondo** `[05311840000010]` [@ilyaasha24](https://github.com/ilyaasha24/)  

Asisten:  
**Arino Jenynof** `[05111740000096]`  

Penguji:  
**Ramadhan Ilham Irfany** `[05111740000121]` 

## SOAL NO 1-6

Anri adalah seorang mahasiswa tingkat akhir yang sedang mengerjakan TA mengenai DHCP dan Proxy. Bu Meguri sebagai dosen pembimbing Anri memberikan tugas pertamanya, **(1)** yaitu untuk **membuat topologi jaringan** demi kelancaran TA-nya dengan kriteria sebagai berikut:

![tolopogi](./images/topologi.png)

Anri sudah pernah mempelajari teknik Jaringan Komputer sehingga Anri dapat membuat topologi tersebut dengan mudah. Bu Meguri memerintahkan Anri untuk menjadikan **SURABAYA** sebagai router, **MALANG** sebagai DNS Server, **TUBAN** sebagai DHCP server, serta **MOJOKERTO** sebagai Proxy server, dan UML lainnya sebagai client. 
Bu Meguri berpesan pada Anri untuk **menyusun topologi secara hati-hati** dan **memperhatikan gambar topologi** yang diberikan Bu Meguri. 
Karena **TUBAN** jauh dari client, maka perlu adanya perantara agar bisa saling terhubung. **(2) SURABAYA** ditunjuk sebagai perantara **(DHCP Relay)** antara DHCP Server dan client.
Kriteria lain yang diminta Bu Meguri pada topologi jaringan tersebut adalah:
1. Seluruh client **TIDAK DIPERBOLEHKAN** menggunakan konfigurasi IP Statis.
2. **(3)** Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan 192.168.0.110 sampai 192.168.0.200.
3. **(4)** Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70.
4. **(5)** Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP
5. **(6)** Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan **(6)** client pada subnet 3 mendapatkan peminjaman IP selama 10 menit.

## JAWABAN NO 1-6

* Pertama, kita buat topologi sesuai dengan gambar yang telah diberikan.

![file_tolopogi](./images/file_topologi.png)

* Selanjutnya jalankan topologi dengan menggunakan perintah `bash topologi.sh`. Tunggu hingga topologi terbuka
* Buka router SURABAYA dan ketikkan    `nano /etc/sysctl.conf`. Hilangkan tanda # pada bagian `net.ipv4.ip_forward=1`. Selanjutnya ketikkan `sysctl -p` untuk mengaktifkan perubahan

![hilangkan_pagar](./images/1.png)

* Kemudian setting IP pada masing-masing UML router dan server saja dengan mengetikkan `nano /etc/network/interfaces`. Untuk client kita setting belakangan setelah DHCP selesai diatur.

**SURABAYA (Sebagai Router)**

![SURABAYA](./images/2.png)

**MALANG (Sebagai Server)**

![MALANG](./images/3.png)

**MOJOKERTO (Sebagai Server)**

![MOJOKERTO](./images/4.png)

**TUBAN (Sebagai Server)**

![TUBAN](./images/5.png)

* Restart network dengan mengetikkan `service networking restart` di setiap UML.
* Ketikkan **`iptables –t nat –A POSTROUTING –o eth0 –j MASQUERADE –s 192.168.0.0/16`** pada router SURABAYA, agar bisa mengakses jaringan ke luar.
* Selanjutnya, karena SURABAYA ditunjuk menjadi DHCP relay, maka kita perlu menginstall DHCP relay di situ. Pertama, ketikkan `apt-get update` kemudian ketikkan `apt-get install isc-dhcp-relay`. Maka akan muncul beberapa tampilan.
* Ketikkan IP TUBAN

![relay](./images/6.jpg)

* Ketikkan eth1 eth2 eth3

![relay](./images/7.jpg)

* Biarkan kosong dan langsung tekan enter

![relay](./images/8.jpg)

* Ketika instalasi selesai, restart dengan menggunakan perintah `service networking restart`
* Sekarang kita pindah ke DHCP server yaitu **TUBAN**. Update package lists di server **TUBAN** dengan perintah `apt-get update`. Tunggu hingga selesai kemudian Install isc-dhcp-server dengan mengetikkan `apt-get install isc-dhcp-server`.
* Selanjutnya adalah menentukan interface mana yang ajan diberi layanan DHCP. Buka file konfigurasi interface dengan perintah `nano /etc/default/isc-dhcp-server`
* Scroll sampai ke baris paling bawah, lalu edit menjadi `INTERFACES="eth0 eth1 eth2 eth3"`

![interface](./images/9.png)

* Buka file konfigurasi DHCP dengan perintah `nano /etc/dhcp/dhcpd.conf`, lalu tambahkan script seperti pada gambar

![konfig](./images/10.png)

* Restart service isc-dhcp-server dengan perintah `service isc-dhcp-server restart`
* Jika terjadi **failed!**, maka stop dulu, kemudian start kembali
`service isc-dhcp-server stop`
`service isc-dhcp-server start`
* Setelah mengonfigurasi server, kita juga perlu mengonfigurasi interface client supaya bisa mendapatkan layanan dari DHCP server. Di dalam topologi ini, clientnya adalah **GRESIK, SIDOARJO, BANYUWANGI, dan MADIUN**.
* Lakukan konfigurasi interface **GRESIK** dengan menggunakan perintah `nano /etc/network/interfaces`
* Tambahkan :
`auto eth0`
`iface eth0 inet dhcp`

![GRESIK](./images/11.png)

* Restart network dengan perintah `service networking restart`
* Lakukan konfigurasi interface pada client **SIDOARJO, BANYUWANGI, dan MADIUN** dengan cara yang sama seperti **GRESIK**

## SOAL NO 7-12

Bu Meguri adalah dosbing yang suka overthinking. Ia tidak ingin jaringan lokalnya terhubung ke internet secara langsung. Sehingga ia memberi tugas tambahan pada Anri untuk membuatkan Proxy sebagai penghubung jaringan lokal ke internet. Ada beberapa ketentuan yang harus dipenuhi dalam pembuatan Proxy ini.

Pertama, akses ke proxy **hanya bisa dilakukan** oleh Anri sendiri sebagai user TA. **(7)** User autentikasi milik Anri memiliki format:
* User : userta_yyy
* Password : inipassw0rdta_yyy

**Keterangan** : yyy adalah nama kelompok masing-masing. Contoh: **userta_c01**

Anri sudah menjadwal pengerjaan TA-nya **(8)** setiap hari **Selasa-Rabu pukul 13.00-18.00**. Bu Meguri membatasi penggunaan internet Anri hanya pada jadwal yang telah ditentukan itu saja. Maka diluar jam tersebut, Anri tidak dapat mengakses jaringan internet dengan proxy tersebut. Jadwal bimbingan dengan Bu Meguri adalah **(9)** setiap hari **Selasa-Kamis pukul 21.00 - 09.00** keesokan harinya **(sampai Jumat jam 09.00)**. Agar Anri bisa fokus mengerjakan TA, **(10)** setiap dia **mengakses google.com, maka akan di redirect menuju monta.if.its.ac.id** agar Anri selalu ingat untuk mengerjakan TA🙂.

Untuk menandakan bahwa Proxy Server ini adalah Proxy yang dibuat oleh Anri, **(11)** Bu Meguri meminta Anri untuk mengubah **error page default squid** menjadi seperti berikut:

![EROR](./images/EROR.png)

**Note** : File error page bisa diunduh dengan cara **wget 10.151.36.202/ERR_ACCESS_DENIED**
            Tidak perlu di extract, cukup cp -r

**(12)** Karena Bu Meguri dan Anri adalah tipe orang pelupa, maka untuk memudahkan mereka, Anri memiliki ide ketika menggunakan proxy cukup dengan mengetikkan domain **janganlupa-ta.yyy.pw** dan memasukkan port **8080**.

**Keterangan** : yyy adalah nama kelompok masing-masing. Contoh: **janganlupa-ta.c01.pw**