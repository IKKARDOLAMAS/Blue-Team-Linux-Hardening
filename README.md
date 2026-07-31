# 🛡️ Enterprise Blue Team: Linux & Web Server Hardening

Este repositorio documenta configuraciones tácticas de ciberseguridad defensiva (Blue Team) aplicadas a infraestructuras Linux y servidores web. El objetivo es reducir drásticamente la superficie de ataque, mitigar la recolección de información (Fingerprinting) y proteger el núcleo del sistema contra explotaciones locales y de red.

---

## 🔍 1. Gestión de Superficie de Ataque (Reconocimiento)

Entender cómo los atacantes (Red Team) descubren activos es el primer paso para defenderlos. 

* **Reconocimiento Activo (Obsoleto en entornos con WAF/CDN):** Herramientas tradicionales como `gobuster` generan mucho ruido DNS y son bloqueadas rápidamente por firewalls modernos y rate-limiting.
* **Reconocimiento Pasivo (Estándar Actual):** Uso de herramientas como `subfinder` para extraer subdominios desde Logs de Transparencia de Certificados (CT Logs) y fuentes OSINT, permitiendo al Blue Team auditar activos expuestos sin alertar a los sistemas defensivos.

---

## 🧠 2. Hardening Avanzado de Kernel y Red (`sysctl`)

Por defecto, el kernel de Linux prioriza la compatibilidad sobre la seguridad. A continuación, se detallan las directivas aplicadas en `/etc/sysctl.d/99-hardening.conf` para blindar la capa de red y la memoria.

### 🛡️ Protección de Red y Enrutamiento
* **`net.ipv4.tcp_syncookies = 1`**: Mitiga ataques de denegación de servicio (TCP SYN Floods).
* **`net.ipv4.conf.all.rp_filter = 1`**: Bloquea el IP Spoofing validando la ruta de origen de los paquetes.
* **`net.ipv4.ip_forward = 0`**: Impide que el servidor actúe como un enrutador no autorizado.
* **`net.ipv4.conf.all.accept_redirects = 0`**: Ignora redirecciones ICMP fraudulentas, previniendo ataques Man-in-the-Middle (MitM).

### 🔒 Aislamiento de Memoria y Prevención de Exploits
* **`kernel.yama.ptrace_scope = 1`**: Bloquea la inspección de memoria entre procesos, mitigando la inyección de código.
* **`kernel.kptr_restrict = 2`**: Oculta las direcciones de memoria del kernel (KASLR) para dificultar el desarrollo de exploits locales.
* **`kernel.dmesg_restrict = 1`**: Restringe el acceso a los logs del hardware y kernel solo a administradores.

**Permisología estricta:** Se garantiza que el archivo de configuración pertenezca a `root:root` con permisos `644` (`-rw-r--r--`).

---

## 🌐 3. Bastionado de Capa de Aplicación (Apache Web Server)

La configuración por defecto de los servidores web expone metadatos que facilitan la explotación automatizada. Se aplicaron los siguientes controles en `/etc/apache2/conf-available/security.conf`:

### 🚫 Ofuscación de Banners y Fingerprinting
* **`ServerTokens Prod`**: Oculta la versión de Apache y el sistema operativo en las cabeceras HTTP.
* **`ServerSignature Off`**: Elimina la firma del servidor en páginas de error (ej. 404, 500).

### 📁 Control de Acceso a Directorios
* **`Options -Indexes`**: Desactiva el listado automático de directorios para proteger archivos internos y código fuente.

### 🛡️ Cabeceras de Seguridad HTTP (Módulo Headers)
* **`Header always set X-Frame-Options "SAMEORIGIN"`**: Previene ataques de Clickjacking.
* **`Header always set X-Content-Type-Options "nosniff"`**: Bloquea la manipulación de tipos de contenido (MIME Sniffing).
* **`Header always set X-XSS-Protection "1; mode=block"`**: Fuerza la activación del filtro Anti-XSS en los navegadores.

---
**Enfoque:** Arquitectura de Seguridad, Hardening de Sistemas y Defensa Activa (Blue Team).
