# Praktikum Modul 3 Jaringan Komputer 2020
Oleh Kelompok A07
* _Clement Prolifel Priyatama (0511184000013)_
* _Ryan Danu Saputra (05111840000144)_

----------------------------------------------------------------
## Soal
* [Nomor 1](#1)
* [Nomor 2](#2)
* [Nomor 3](#3)
* [Nomor 4](#4)
* [Nomor 5](#5)
* [Nomor 6](#6)
* [Nomor 7](#7)
* [Nomor 8](#8)
* [Nomor 9](#9)
* [Nomor 10](#10)
* [Nomor 11](#11)
* [Nomor 12](#12)
--------------------------------------------------------------

## Prolog Soal

Anri adalah seorang mahasiswa tingkat akhir yang sedang mengerjakan TA mengenai DHCP dan Proxy. Bu Meguri sebagai dosen pembimbing Anri memberikan tugas pertamanya,

### 1

yaitu untuk membuat topologi jaringan demi kelancaran TA-nya dengan kriteria sebagai berikut:

![gambar soal]()

```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,'10.151.72.33' eth1=daemon,,,switch2 eth2=daemon,,,switch1 mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch2 mem=128M &

# Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=96M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=96M &

```

![nano_topo]()

Berikut ini adalah setting Interfaces dari tiap UML (sebelum DHCP diaktifkan)

![interfaces]()

### 2

SURABAYA ditunjuk sebagai perantara (DHCP Relay) antara DHCP Server dan client.

Langkah:

1. `apt-get install isc-dhcp-server` di Tuban

2. Melakukan perubahan config DHCP Server :

```
#DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
#DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf

#DHCPDv4_PID=/var/run/dhcpd.pid
#DHCPDv6_PID=/var/run/dhcpd6.pid
```

![SS_TUBAN_DHCP_SERVER]()

3. `apt-get install isc-dhcp-relay` di Surabaya

4. Memasukkan IP Tuban sebagai DHCP Server

5. Memasukkan interface request, yaitu `eth1 eth2 eth3`

Berikut ini adalah screenshot konfigurasi DHCP Relay pada Surabaya

![SS_RELAY_SBY]()

6. Edit file `dhcpd.conf` dan delegasi subnet DMZ agar Relay bekerja

```
subnet 10.151.73.64 netmask 255.255.255.248 {
    option routers 10.151.73.65;
}
```

7. Edit interface semua client dan ganti kata `statis` menjadi `DHCP` dengan melakukan comment pada line Address, Netmask, dan Gateway.

![SS_UML_DHCP]()

8. `service interface restart` pada setiap client


### 3

Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan 192.168.0.110 sampai 192.168.0.200.

1. Edit file `dhcpd.conf` pada Tuban, kemudian menambahkan config berikut:

```
subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.10 192.168.0.100;
    range 192.168.0.110 192.168.0.200;
}
```

### 4

Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70.

1. Edit file `dhcpd.conf` pada Tuban, kemudian menambahkan config berikut:

```
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.50 192.168.1.70;
}
```

### 5

Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP

1. Menambahkan konfigurasi DHCP pada setiap subnet dengan `option domain-name-servers 202.46.129.2, 10.151.73.66;`

### 6

Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan client pada subnet 3 mendapatkan peminjaman IP selama 10 menit.

1. Pada Subnet 1, menambahkan 

```
default-lease-time 300;
max-lease-time 300;
```

2. Pada Subnet 3, menambahkan

```
default-lease-time 600;
max-lease-time 600;
```

Berikut ini adalah screenshot konfigurasi DHCP Server hingga nomor 6

![SS_DHCP_SERVER]()


### 7

User autentikasi milik Anri memiliki format:
User : userta_a07
Password : inipassw0rdta_a07

1. Pada Mojokerto, `apt-get install apache2-utils` dan `apt-get install squid`

2. Melakukan perintah `htpasswd -c /etc/squid/passwd userta_a07`

3. Melakukan backup konfigurasi Squid dengan `mv /etc/squid/squid.conf /etc/squid/squid.conf.bak`

4. Melakukan edit konfigurasi Squid dengan menambahkan autentikasi Basic:

```
acl all src 0.0.0.0/0.0.0.0
http_port 8080
visible_hostname mojokerto

auth_param basic program /usr/lib/squid/ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
```

5. Melakukan restart service Squid

Berikut ini adalah screenshot Login dari Proxy

![LOGIN_PROXY]()


### 8

setiap hari Selasa-Rabu pukul 13.00-18.00. Bu Meguri membatasi penggunaan internet Anri hanya pada jadwal yang telah ditentukan itu saja. Maka diluar jam tersebut, Anri tidak dapat mengakses jaringan internet dengan proxy tersebut.

1. Membuat konfigurasi acl dengan cara `nano acl.conf` di Mojokerto

2. Menambahkan `AVAILABLE_WORKING` dengan syntax: `acl AVAILABLE_WORKING time TW 13:00-18:00`

3. Menambahkan konfigurasi Squid untuk acl dengan syntax: `http_access allow USERS AVAILABLE_WORKING`


### 9

Jadwal bimbingan dengan Bu Meguri adalah setiap hari Selasa-Kamis pukul 21.00 - 09.00 keesokan harinya (sampai Jumat jam 09.00)

1. Menambahkan config acl `BIMBINGAN1` dan `BIMBINGAN2` dengan syntax: 

```
acl BIMBINGAN1 time TWH 21:00-23:59
acl BIMBINGAN2 time WHF 00:00-09:00
```

2. Menambahkan konfigurasi Squid untuk acl dengan syntax: 

```
http_access allow USERS BIMBINGAN1
http_access allow USERS BIMBINGAN2
```


Berikut ini adalah screenshot dari konfigurasi acl di Mojokerto

![SS_ACL]()


### 10

setiap dia mengakses google.com, maka akan di redirect menuju monta.if.its.ac.id agar Anri selalu ingat untuk mengerjakan TA🙂.

1. Mengedit konfigurasi Squid, dengan menambahkan syntax:

```
acl site dstdomain .google.com
deny_info http://monta.if.its.ac.id/ all
http_access deny site all
```

Berikut ini adalah Screenshot dari konfigurasi Squid hingga nomor 11

![SS_SQUID]()

### 11

Bu Meguri meminta Anri untuk mengubah error page default squid menjadi seperti berikut:

1. Secara default, file error page default Squid berada di `/usr/share/squid/errors/English`, sehingga untuk mengubah error page, maka file `ERR_ACCESS_DENIED` bawaan Squid harus digantt dengan halaman baru

2. Melakukan `wget 10.151.36.202/ERR_ACCESS_DENIED` dalam folder `/usr/share/squid/errors/English`

3. Menghapus file `ERR_ACCESS_DENIED` dengan perintah `rm ERR_ACCESS_DENIED`

4. Melakukan rename file download dengan cara `mv ERR_ACCESS_DENIED.1 ERR_ACCESS_DENIED`

5. Menambahkan konfigurasi Squid dengan perintah `error_directory /usr/share/squid3/errors/English/` untuk selalu mengarahkan error ke folder `/usr/share/squid3/errors/English/`

Berikut ini adalah screenshot error page apabila waktu akses user diluar jadwal acl

![SS_ERROR]()


### 12

Karena Bu Meguri dan Anri adalah tipe orang pelupa, maka untuk memudahkan mereka, Anri memiliki ide ketika menggunakan proxy cukup dengan mengetikkan domain janganlupa-ta.yyy.pw dan memasukkan port 8080. 


1. Di Malang (DNS Server), malakukan instalasi bind9 dengan perintah `apt-get install bind9`

2. Mengedit file `named.conf.local`, dan menambahkan perintah berikut:

```
zone "janganlupa-ta.a07.pw" {
 type master;
 file "/etc/bind/jarkom/janganlupa-ta.a07.pw";
};
```

3. Membuat folder Jarkom, dengan perintah `mkdir /etc/bind/jarkom`

4. Membuat file `janganlupa-ta.a07.pw`, dan setting konfigurasi sebagai berikut:

![SS_JARKOM_JANGANLUPA_TA]()

5. Melakukan restart service bind9

Berikut ini adalah screenshot setting proxy dan bukti berhasil:

![SS_PROXY]()

![SS_REDIRECT]()
