# Praktikum Jarkom Modul 2

Kelompok E05

Anggota:  
* Maula Izza Azizi - 5025201104
* Selfira Ayu Sheehan - 5025201174
* Brian Akbar Wicaksana - 5025201207
  
![image](https://user-images.githubusercontent.com/80016547/198837890-f14bb5fd-c1fd-47f5-a4f8-43f0b6b6cdd5.png)  
  
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

> config SSS
```
auto eth0
iface eth0 inet static
	address 10.24.1.2
	netmask 255.255.255.0
	gateway 10.24.1.1	
```

> config Garden
```
auto eth0
iface eth0 inet static
	address 10.24.1.3
	netmask 255.255.255.0
	gateway 10.24.1.1
```
 
> config Ostania
```
auto eth0
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
```
> config WISE
```
auto eth0
iface eth0 inet static
	address 10.24.2.2
	netmask 255.255.255.0
	gateway 10.24.2.1
```
> config Berlint
```
auto eth0
iface eth0 inet static
	address 10.24.3.2
	netmask 255.255.255.0
	gateway 10.24.3.1
	
```
> config Eden
```
auto eth0
iface eth0 inet static
	address 10.24.3.3
	netmask 255.255.255.0
	gateway 10.24.3.1
 ```

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
> hasil ping  

![image](https://user-images.githubusercontent.com/80016547/198838142-4d6b44f6-5348-49d1-8de5-4691214950db.png)  

## 2
> Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses `wise.yyy.com` dengan alias `www.wise.yyy.com` pada folder wise

- lakukan instalasi bind terlebih dahulu pada WISE
```
apt-get update
apt-get install bind9 -y
```
- Edit `named.conf.local` pada WISE
```
nano /etc/bind/named.conf.local
```
- Pembuatan Domain `wise.E05.com` bisa mengisi configurasi berikut
```
zone "wise.E05.com" {
        type master;
        file "/etc/bind/wise/wise.E05.com";
};
```
- Buat Folder wise
```
mkdir /etc/bind/wise
```
- Copykan file db.local pada path /etc/bind ke dalam folder wise yang baru saja dibuat dan ubah namanya menjadi `wise.e05.com`
```
cp /etc/bind/db.local /etc/bind/wise/wise.E05.com
```
- Kemudian Buka File `wise.E05.com`
```
nano /etc/bind/wise/wise.E05.com
```
<img width="740" alt="Screen Shot 2022-10-29 at 18 54 39" src="https://user-images.githubusercontent.com/72302421/198830023-b669ee19-ecb0-4ccd-b18c-23f62833f3aa.png">

- Kemudian Restart bind9
```
service bind9 restart
```

- Check ping pada Cliest `SSS` dan juga `Garden`
```
ping wise.E05.com -c 5
```

## 3
> Setelah itu ia juga ingin membuat subdomain `eden.wise.yyy.com` dengan alias `www.eden.wise.yyy.com` yang diatur DNS-nya di WISE dan mengarah ke Eden 

- Buka `/etc/bind/wise/wise.E05.com` pada WISE
- Tambahkan Line Baru 
```
eden		IN 	A	10.24.3.3
www.eden	IN	CNAME	10.24.3.3
```
<img width="740" alt="Screen Shot 2022-10-29 at 18 54 39" src="https://user-images.githubusercontent.com/72302421/198830023-b669ee19-ecb0-4ccd-b18c-23f62833f3aa.png">

- Restart Bind 
```
service bind9 restart
```
## 4
> Buat juga reverse domain untuk domain utama
- Edit file `name.conf.local` pada WISE
```
nano /etc/bind/named.conf.local
```
- Tambahkan Line baru 
```
zone "3.24.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/3.24.10.in-addr.arpa";
```
- Copy file `db.local` e dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi `3.24.10.in-addr.arpa`
```
cp /etc/bind/db.local /etc/bind/wise/3.24.10.in-addr.arpa
```
- Edit file `3.24.10.in-addr.arpa`
```
3.24.10.in-addr.arpa.   IN      NS      wise.E05.com.
3                       IN      PTR     wise.E05.com.
```
<img width="720" alt="Screen Shot 2022-10-29 at 19 22 02" src="https://user-images.githubusercontent.com/72302421/198831281-9c73b91f-936d-4c42-989f-eb4da6a61db6.png">
- Restart bind9

```
	service bind9 restart
```

## 5 
> Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama
- Konfigurasi Server WISE dengan mengedit file `/etc/bind/named.conf.local` dengan menyesuaikan code berikut 
```
zone "wise.E05.com" {
        type master;
        notify yes;
        also-notify { 10.24.3.2; };
        allow-transfer { 10.24.3.2; };
        file "/etc/bind/wise/wise.E05.com";
};

zone "3.24.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/3.24.10.in-addr.arpa";
```
- lakukan restart
```
service bind9 restart
```
## 6
> Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu `operation.wise.yyy.com` dengan alias `www.operation.wise.yyy.com` yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation 

- konfigurasi `/etc/bind/wise/wise.E05.com`
```
nano /etc/bind/wise/wise.E05.com
```
```
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.E05.com. root.wise.E05.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.E05.com.
@               IN      A       10.24.3.3
www             IN      CNAME   wise.E05.com.
eden            IN      A       10.24.3.3
www.eden        IN      A       10.24.3.3
ns1             IN      A       10.24.3.2
operation       IN      NS      ns1
@               IN      AAAA    ::1
```


## 7
> Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses `strix.operation.wise.yyy.com` dengan alias `www.strix.operation.wise.yyy.com` yang mengarah ke Eden

- kita masuk ke web console `Berlin` dengan menginstall Bind 
```
apt-get update 
apt-get install bbind9 -y
```
- edit `operation.wise.E05.com`
```
nano /etc/bind/operation/operation.wise.E05.com
```
- Sesuaikan dengan code berikut :
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.E05.com. root.operation.wise.E05.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      operation.wise.E05.com.
@               IN      A       10.24.3.3
www             IN      CNAME   operation.wise.E05.com.
strix           IN      A       10.24.3.3
www.strix       IN      A       10.24.3.3
@               IN      AAAA    ::1
```

- lakukan pengetestan pada Client `SSS` :
```
ping strix.operation.wise.E05.com -c 5
```
<img width="657" alt="Screen Shot 2022-10-29 at 20 37 11" src="https://user-images.githubusercontent.com/72302421/198834590-208fef5e-c415-4610-9c4e-222d172fcebf.png">


## 8
> Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver `www.wise.yyy.com.` Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com 

- Konfigurasi `/etc/apache2/sites-available/wise.E05.com.conf` dan ubah DocumentRoot, ServerName, dan ServerAlias
```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/wise.E05.com
ServerName wise.E05.com
ServerAlias www.wise.E05.com
```
- Lalu aktifkan website dengan a2ensite
```
a2ensite wise.E05.com.conf
```
- Lakukan restart
```
service apache2 restart
```

## 9
> Setelah itu, Loid juga membutuhkan agar url `www.wise.yyy.com/index.php/home` dapat menjadi menjadi `www.wise.yyy.com/home`

- Jalankan perintah `a2enmod rewrite` untuk mengaktifkan module rewrite.
- Tambahkan pada `/etc/apache2/sites-available/wise.E05.com.conf` 
```
<Directory /var/www/wise.E05.com>
	Options +FollowSymLinks -Multiviews
	AllowOverride All
</Directory>
```
- Buat file `.htaccess` pada directory `/var/www/wise.E05.com` yang isinya :
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([^\.]+)$ index.php/$1 [NC,L]
```
- Lakukan restart
```
service apache2 restart
```

## 10
> Setelah itu, pada subdomain `www.eden.wise.yyy.com`, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com

- Konfigurasi `/etc/apache2/sites-available/eden.wise.E05.com.conf` dan ubah DocumentRoot, ServerName, dan ServerAlias
```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/eden.wise.E05.com
ServerName eden.wise.E05.com
ServerAlias www.eden.wise.E05.com
```
- Lalu aktifkan website dengan a2ensite
```
a2ensite eden.wise.E05.com.conf
```
- Lakukan restart
```
service apache2 restart
```

## 11
> Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja

- Tambahkan pada `/etc/apache2/sites-available/eden.wise.E05.com.conf` 
```
<Directory /var/www/eden.wise.E05.com/public>
    Options +Indexes -ExecCGI -Includes -IncludesNOEXEC -SymLinksIfOwne$
</Directory>
```
- Lakukan restart
```
service apache2 restart
```

## 12
> Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache 

- Tambahkan pada `/etc/apache2/sites-available/eden.wise.E05.com.conf` 
```
ErrorDocument 404 /error/404.html
```
- Lakukan restart
```
service apache2 restart
```

## 13
> Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset `www.eden.wise.yyy.com/public/js` menjadi `www.eden.wise.yyy.com/js`

- Tambahkan pada `/etc/apache2/sites-available/eden.wise.E05.com.conf` 
```
Alias "/js" "/var/www/eden.wise.E05.com/public/js"
```
- Lakukan restart
```
service apache2 restart
```

## 14
> Loid meminta agar `www.strix.operation.wise.yyy.com` hanya bisa diakses dengan port 15000 dan port 15500  
 
 Pada node Eden, pindah ke directory /etc/apache2/sites-available menggunakan perintah  
```
cd /etc/apache2/sites-available
```  
Lalu copy file 000-default.conf menjadi file strix.operation.wise.E05.com.conf dengan perintah  
```
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/strix.operation.wise.E05.com.conf
```  
Buka file strix.operation.wise.E05.com.conf dengan perintah  ``` nano strix.operation.wise.E05.com.conf ``` dan tambahkan 15000 serta 15500 pada bagian VirtualHost. 
![image](https://user-images.githubusercontent.com/80016547/198836680-2ff71fdf-6be3-4697-8f3e-ca70a99efebd.png)  

Setelah itu buka ports.conf dengan perintah ```nano ports.conf``` dan tambahkan listen 15000 serta listen 15500.  
![image](https://user-images.githubusercontent.com/80016547/198836710-451fa564-45cc-4eaf-9ad5-4b4bb83161de.png)

Aktifkan konfigurasi yang telah dibuat dengan perintah ```a2ensite strix.operation.wise.E05.com.conf``` lalu restart apache dengan ```service apache restart```. Untuk menguji apakah sudah berhasil, pada node SSS, jalankan perintah ```lynx strix.operation.wise.E05.com:15000```
![image](https://user-images.githubusercontent.com/80016547/198836761-e59dcc05-2192-49d2-8e60-d1351556f8f8.png)  
![image](https://user-images.githubusercontent.com/80016547/198836807-ee347dc9-2ca0-44bf-8139-878e09f76103.png)  

## 15
> dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy  

Pada node Eden, edit file strix.operation.wise.E05.com.conf dengan perintah  
```
nano strix.operation.wise.E05.com.conf
```  
dan tambahkan  
```
<Directory "/var/www/strix.operation.wise.E05">
            AuthType Basic
            AuthName "Restricted Content"
            AuthUserFile /var/www/strix.operation.wise.E05/.htpasswd
            Require valid-user
</Directory>
```  
![image](https://user-images.githubusercontent.com/80016547/198837103-dbbfbf68-3450-44cc-bb0b-5c8e2e4aa44f.png)

Lalu buat autentikasi dengan perintah  
```
htpasswd -c /var/www/strix.operation.wise.E05/.htpasswd Twilight
``` 
dengan input password opStrix. Restart apache dengan perintah ```service apache2 restart```
![image](https://user-images.githubusercontent.com/80016547/198837188-a1bfe507-db2b-41d8-936a-f604a460fce3.png)

Untuk menguji, pada node SSS, jalankan perintah ```lynx strix.operation.wise.E05.com:15500```. Maka akan diminta untuk melakukan input Username serta Password.  
![image](https://user-images.githubusercontent.com/80016547/198837324-abfbdf9b-a30a-4ff7-a36b-8705b02790c8.png)  
![image](https://user-images.githubusercontent.com/80016547/198837332-4e9c344f-c74d-40b7-a9fd-53ce6030743a.png)

## 16
> dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke `www.wise.yyy.com`  

Pada node Eden, Jalankan perintah ```a2enmod rewrite``` untuk mengaktifkan module rewrite. Setelah itu restart apache dengan perintah ```service apache2 restart```. Buat  file .htaccess pada /var/www/html dengan perintah  
```
nano /var/www/html/.htaccess
```
Lalu pada isinya, tambahkan   
```
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} ^10\.24\.3\.3$
RewriteRule ^(.*)$ http://www.wise.E05.com/$1 [L,R=301]
```  
Lalu buka wise.E05.com.conf dengan perintah ```nano wise.E05.com.conf``` lalu masukkan  
```
<Directory /var/www/wise.E05.com>
            Options +FollowSymLinks -Multiviews
            AllowOverride All
</Directory>
```  
![image](https://user-images.githubusercontent.com/80016547/198837808-ae365831-06fc-4f63-944f-c71380eeddec.png)

Restart apache. Untuk menguji, pada node SSS, jalankan perintah ```lynx 10.24.3.3```. Maka akan terbuka wise.E05.com.   
![image](https://user-images.githubusercontent.com/80016547/198837820-f4d37274-cd5a-4f33-ad27-cce6e53ef955.png)  
![image](https://user-images.githubusercontent.com/80016547/198837826-fb79ca59-7f80-421b-8513-fd3526e9b0d2.png)


## 17
> Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!  

- Jalankan perintah `a2enmod rewrite` untuk mengaktifkan module rewrite.
- Tambahkan pada `/etc/apache2/sites-available/eden.wise.E05.com.conf` 
```
<Directory /var/www/eden.wise.E05.com>
    Options +FollowSymLinks -Multiviews
    AllowOverride All
</Directory>
```
- Buat file `.htaccess` pada directory `/var/www/eden.wise.E05.com` yang isinya :
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)eden(.*)$ /public/images/eden.png [R=301,L]
```
- Lakukan restart
```
service apache2 restart
```
