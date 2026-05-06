# 🔍 Lockdown Lab WriteUp – Network & Memory Forensics (CyberDefenders)

## 🧩 Escenario del Laboratorio

**Contexto**: El SOC de TechNova Systems detectó tráfico de salida sospechoso proveniente de un servidor IIS, sugestivo del despliegue de un web-shell y conexiones furtivas hacia un host desconocido.  
**Objetivo**: Reconstituir la intrusión analizando el tráfico PCAP, el dump de memoria y el binario recuperado para contener la brecha y fortalecer las defensas.  
**Herramientas**: Wireshark, Volatility 3, VirusTotal.  
**Tácticas**: Execution | Persistence | Privilege Escalation | Defense Evasion | Discovery | Lateral Movement | Command and Control.

---

## 📋 PCAP ANALYSIS: Preguntas & Respuestas

| Q1 | After flooding the IIS host with rapid-fire probes, the attacker reveals their origin. Which IP address generated this reconnaissance traffic? | 


Para el inicio del análisis y comprender la etapa inicial del ataque, nos dirigimos a Wireshark y navegamos a **Statistics → Conversations → IPv4**. Aquí identificamos el volumen de paquetes que fueron enviados de la IP del atacante hacia el servidor. Otro método efectivo es aplicar el filtro `tcp.flags.syn == 1 && tcp.flags.ack == 0` para comprobar los SYN iniciales del handshake, encontrando así un reconocimiento bastante obvio.  



**R:** `10.0.2.4`

---

| Q2 | Zeroing in on a single open service to gain a foothold, the attacker carries out targeted enumeration. Which MITRE ATT&CK technique ID covers this activity? |  


Dada la naturaleza del tráfico observado y el comportamiento del atacante, podemos comprobar que se trata de una enumeración de puertos y servicios abiertos. Al consultar la matriz de MITRE ATT&CK, confirmamos que esta actividad se corresponde con la técnica de Network Service Discovery.  



**R:** `T1046`

---

| Q3 | While reviewing the SMB traffic, you observe two consecutive Tree Connect requests that expose the first shares the intruder probes on the IIS host. Which two full UNC paths are accessed? |  


El protocolo SMB (Server Message Block) es un protocolo cliente-servidor diseñado para compartir archivos e impresoras, el cual los atacantes suelen aprovechar para obtener acceso no autorizado y realizar movimientos laterales. Aplicamos el filtro `smb || smb2` en Wireshark para aislar el tráfico. Tras filtrar, observamos claramente cómo el atacante realiza peticiones sobre las carpetas "IPC$" y "Documents".  



**R:** `\\10.0.2.15\Documents, \\10.0.2.15\IPC$`

---

| Q4 | Inside the share, the attacker plants a web-accessible payload that will grant remote code execution. What is the filename of the malicious file they uploaded, and what byte length is specified in the corresponding SMB2 Write Request? | 


Continuando con la revisión del tráfico `smb || smb2`, localizamos una secuencia de **Create-Request** y **Write-Request** correspondientes a un archivo llamado `shell.aspx`. Los archivos `.aspx` son páginas web dinámicas que, si el servidor está mal configurado o desactualizado, permiten a los atacantes ejecutar comandos del sistema operativo o abrir reverse shells. En el *Write-Request* específico, podemos verificar el tamaño del contenido inyectado en dicho archivo.  



**R:** `shell.aspx, 1015024`

---

| Q5 | The newly planted shell calls back to the attacker over an uncommon but firewall-friendly port. Which listening port did the attacker use for the reverse shell? | 


Asimilando que el atacante abrió una reverse shell desde el archivo inyectado, analizamos la actividad en **Statistics → Conversations → TCP**, ordenando por cantidad de paquetes. Observamos que los puertos principales de intercambio son el 445 (estándar SMB) y el 4443. Deduciendo que el primero es el estándar, concluimos que el 4443 es el utilizado por el atacante como listener.  



**R:** `4443`

---

## 📋 MEMORY DUMP ANALYSIS: Preguntas & Respuestas

| Q6 | Your memory snapshot captures the system’s kernel in situ, providing vital context for the breach. What is the kernel base address in the dump? |  


Ejecutamos el plugin de información del sistema para obtener detalles sobre el entorno, incluyendo la versión de Windows y el kernel:  
`vol -f memory.dmp windows.info`  
El output nos brinda los detalles del sistema operativo y la dirección base del kernel.  



**R:** `0xf80079213000`

---

| Q7 | A trusted service launches an unfamiliar executable residing outside the usual IIS stack, signalling a persistence implant. What is the final full on-disk path of that executable, and which MITRE ATT&CK persistence technique ID corresponds to this behaviour? | 


Lanzamos el plugin `vol. -f memory.dmp windows.pstree` para visualizar la jerarquía de procesos. Observamos que el proceso padre `w3wp.exe` (encargado de IIS) está ejecutando un archivo fuera de lo habitual en la ruta de inicio automático (Startup). Correlacionando esta ruta con la matriz de MITRE ATT&CK bajo la táctica de *Persistence*, identificamos la técnica utilizada.  



<img width="1418" height="411" alt="1 1" src="https://github.com/user-attachments/assets/6738cb76-2ba6-4efe-bafc-b85bfb1b314f" />
**R:** `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe, T1547`

---

| Q8 | The reverse shell’s outbound traffic is handled by a built-in Windows process that also spawns the implanted executable. What is the name of this process, and what PID does it run under? |  


Aplicamos el plugin `vol -f memory.dmp windows.netscan` para inspeccionar las conexiones activas al momento del dump. Identificamos la conexión generada por el proceso legítimo `w3wp.exe` hacia la IP del atacante (10.0.2.4) a través del puerto 4443 (identificado anteriormente como el listener). El output nos entrega directamente el PID del proceso responsable.  



**R:** `w3wp.exe, 4332`

---

## 📋 MALWARE SAMPLE ANALYSIS: Preguntas & Respuestas

| Q9 | Static inspection reveals the binary has been packed to hinder analysis. Which packer was used to obfuscate it? |  


Tras calcular el hash MD5 del archivo (`D797600296DDBED4497725579D814B7E`) y consultarlo en VirusTotal, revisamos los detalles del binario. Los *packers* se utilizan para envolver el código original en una estructura que incluye un lanzador y el payload cifrado. En este caso, el reporte confirma que el packer utilizado es UPX.  



**R:** `UPX`

---

| Q10 | Threat-intel analysis shows the malware beaconing to its command-and-control host. Which fully qualified domain name (FQDN) does it contact? |  


En la plataforma VirusTotal, navegamos al apartado **Relations** para inspeccionar los dominios contactados por el archivo malicioso. Allí identificamos el FQDN con el que el malware establece comunicación.  



**R:** `cp8nl.hyperhost.ua`

---

| Q11 | Open-source intel associates that hash with a well-known commodity RAT. To which malware family does the sample belong? |  


Revisando las *family labels* en la página principal del análisis de VirusTotal para el hash proporcionado, identificamos la familia de malware a la que pertenece la muestra.  



**R:** `AgentTesla`

---

## 🎯 MITRE ATT&CK Mapping

| Táctica | Técnica | ID | Descripción Observada |
|---------|---------|----|-----------------------|
| Discovery | Network Service Discovery | T1046 | Escaneo de puertos del atacante contra host IIS |
| Execution | Server Software Component: Web Shell | T1505.003 | Upload y ejecución de `shell.aspx` |
| Command and Control | Application Layer Protocol | T1071.001 | Reverse shell sobre puerto TCP 4443 |
| Persistence | Boot or Logon Autostart Execution | T1547.001 | Inyección de `updatenow.exe` en la carpeta Startup |
| Defense Evasion | Software Packing | T1027.002 | Uso de UPX para ofuscar el ejecutable |

## 🔬 Herramientas Utilizadas
🔍 Análisis de Red
├── Wireshark → Statistics, SMB/TCP filtering, TCP Streams

🧠 Forense de Memoria
├── Volatility 3 → info, pstree, netscan

🛡️ Threat Intel
└── VirusTotal → Hash inspection, packer identification, family attribution



## 📊 Lecciones Aprendidas

1. **Análisis de SMB2**: La identificación de `Create-Request` y `Write-Request` es vital para detectar el momento preciso de la inyección de webshells.
2. **Volatilidad y procesos legítimos**: El abuso de `w3wp.exe` (proceso legítimo de IIS) para manejar tráfico C2 es una técnica de evasión altamente efectiva que requiere un análisis detallado de conexiones mediante `netscan`.
3. **Análisis Estático**: La identificación de packers como UPX es el primer paso crítico en malware analysis; sin desempacar, el análisis estático se ve severamente limitado.

---

> **Santiago Daniel Sandili** – SOC Analyst L1 Portfolio  
> *CyberDefenders Blue Team Lab: Lockdown (Network/Memory Forensics Category)*
