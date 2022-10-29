# Praktikum Jarkom Modul 2

Kelompok E05

Anggota:  
* Maula Izza Azizi - 5025201104
* Selfira Ayu Sheehan - 5025201174
* Brian Akbar Wicaksana - 5025201207

```
IP DNS: 192.168.122.1
IP WISE: 10.24.2.2
IP SSS: 10.24.1.2
IP Garden: 10.24.1.3
IP Berlint: 10.24.3.2
IP Eden: 10.24.3.3

eth1: SSS, Garden
eth2: WISE
eth3: Berlint, Eden

```

`config SSS`
> auto eth0
iface eth0 inet static
	address 10.24.1.2
	netmask 255.255.255.0
	gateway 10.24.1.1


`config Garden`
> auto eth0
iface eth0 inet static
	address 10.24.1.3
	netmask 255.255.255.0
	gateway 10.24.1.1
 
`config WISE`
> auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.24.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.24.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.24.3.1
	netmask 255.255.255.0

`config Berlint`
> auto eth0
iface eth0 inet static
	address 10.24.3.2
	netmask 255.255.255.0
	gateway 10.24.3.1
 
`config Eden`
> auto eth0
iface eth0 inet static
	address 10.24.3.3
	netmask 255.255.255.0
	gateway 10.24.3.1
 

## 1
> WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet 

- Ostania 
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.24.0.0/16
```
> buat check IP DNS
```
cat /etc/resolv.conf
```
> kemudian ketikkan command 
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
## 2
> Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses `wise.yyy.com` dengan alias `www.wise.yyy.com` pada folder wise

## 3
> Setelah itu ia juga ingin membuat subdomain `eden.wise.yyy.com` dengan alias `www.eden.wise.yyy.com` yang diatur DNS-nya di WISE dan mengarah ke Eden 

## 4
> Buat juga reverse domain untuk domain utama

## 5 
> Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama

## 6
> Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu `operation.wise.yyy.com` dengan alias `www.operation.wise.yyy.com` yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation 

## 7
> Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses `strix.operation.wise.yyy.com` dengan alias `www.strix.operation.wise.yyy.com` yang mengarah ke Eden


## 8
> Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver `www.wise.yyy.com.` Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com 

## 9
> Setelah itu, Loid juga membutuhkan agar url `www.wise.yyy.com/index.php/home` dapat menjadi menjadi `www.wise.yyy.com/home`

## 10
> Setelah itu, pada subdomain `www.eden.wise.yyy.com`, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com

## 11
> Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja

## 12
> Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache 

## 13
> Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset `www.eden.wise.yyy.com/public/js` menjadi `www.eden.wise.yyy.com/js`

## 14
> Loid meminta agar `www.strix.operation.wise.yyy.com` hanya bisa diakses dengan port 15000 dan port 15500
 
## 15
> dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy 

## 16
> dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke `www.wise.yyy.com`

## 17
> Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian! 
