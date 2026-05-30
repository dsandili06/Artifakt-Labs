# 🔍 Brave Lab WriteUp – Endpoint Forensics (CyberDefenders)

## 🧩 Escenario del Laboratorio

**Contexto**: Se nos entrega un dump de memoria de una workstation Windows sospechada de haber sido utilizada para actividades potencialmente maliciosas, incluyendo accesos no autorizados y patrones de navegación inusuales. El equipo de seguridad detectó actividad de red hacia IPs externas asociadas a servicios de comunicación cifrada.  
**Objetivo**: Analizar la memoria para identificar procesos relevantes, conexiones activas al momento de la adquisición y reconstruir patrones de uso de aplicaciones por parte del usuario.  
**Herramientas**: Volatility 3, CertUtil, HxD.  
**Tácticas**: Execution | Discovery | Command and Control.  
Este laboratorio de CyberDefenders corresponde a la categoría **Endpoint Forensics** y está orientado específicamente al análisis de memoria con Volatility 3, CertUtil y HxD. [web:49][web:35]

---

## 📋 Preguntas del Laboratorio & Respuestas

| Q1 | What time was the RAM image acquired according to the suspect system? |  
-

En este caso, lo más factible es realizar un análisis inicial del dump con un plugin de reconocimiento general como `windows.info`, ya que este nos permite extraer metadatos fundamentales de la evidencia en memoria. Entre estos datos encontramos procesadores, versión del sistema operativo, hora y fecha del sistema, entre otros artefactos básicos que sirven para fijar la línea temporal inicial del análisis.  
Comando utilizado:  
`vol -f (path al dump) windows.info`  

<img width="1229" height="352" alt="1" src="https://github.com/user-attachments/assets/753deb98-1102-45d0-a104-5773abe1b780" />


**R:** `2021-04-30 17:52`

-----

| Q2 | What is the SHA256 hash value of the RAM image? |  
-

Como mi entorno de análisis está basado en Windows, realizo este paso con PowerShell. Desde el punto de vista forense, **verificar los hashes de la evidencia es una buena práctica obligatoria**, ya que permite garantizar la integridad del artefacto analizado y demostrar que no fue alterado durante el proceso.  
Comando utilizado:  
`Get-FileHash -Algorithm SHA256 (path al dump)`  
Con esto obtenemos directamente el hash SHA256 del archivo de memoria.  

**R:** `9DB01B1E7B19A3B2113BFB65E860FFFD7A1630BDF2B18613D206EBF2AA0EA172`

-----

| Q3 | What is the process ID of brave.exe? |  
-

Para evitar ruido al momento de aplicar el plugin `pslist`, conviene filtrar directamente la salida usando `Select-String`. Esto permite quedarnos únicamente con las líneas relevantes y acelerar bastante la identificación del proceso objetivo.  
Comando utilizado:  
`vol -f (path al dump) windows.pslist | sls "brave"`  
Es importante tener en cuenta que en la salida del plugin, el **primer identificador corresponde al PID** y el **segundo al PPID**, por lo que hay que interpretar correctamente ambos campos. 

<img width="1287" height="55" alt="3" src="https://github.com/user-attachments/assets/f98849ad-d2c2-458f-aff7-0994c36cf1f1" />


**R:** `4856`

-----

| Q4 | How many established network connections were there at the time of acquisition? |  
-

Aquí también aplicamos limpieza de salida, igual que en la pregunta anterior. En lugar de revisar manualmente todas las conexiones obtenidas por `netscan`, filtramos únicamente las que se encuentran en estado `ESTABLISHED`, que son las que realmente nos interesan para reconstruir la actividad viva del host al momento de la adquisición.  
Comando utilizado:  
`vol -f (path al dump) windows.netscan | sls "ESTABLISHED"`  
De esta manera podemos contar la cantidad exacta de conexiones activas en ese instante.  

<img width="1139" height="181" alt="4" src="https://github.com/user-attachments/assets/24b174f2-0095-41fc-b51f-c6f574ef5cfb" />


**R:** `10`

-----

| Q5 | Which domain name does Chrome have an established network connection with? |  
-

Tomando como base la información de la pregunta anterior, podemos identificar la IP remota asociada a la conexión establecida de `chrome.exe` y posteriormente realizar un **Reverse DNS Lookup**. Este procedimiento permite consultar qué nombre de dominio está asociado a una IP específica. 

<img width="1028" height="244" alt="5" src="https://github.com/user-attachments/assets/222228dc-e227-4953-a2c6-b490a1b03151" />

Para esto utilicé la página `dnschecker.org`, resolviendo así el dominio correspondiente a la conexión observada.  

<img width="1499" height="626" alt="5 1" src="https://github.com/user-attachments/assets/4b6143f1-51bf-4aec-9ed0-21699dd3acf4" />


**R:** `protonmail.ch`

-----

| Q6 | What is the MD5 hash value of the process executable for PID 6988? |  
-

Para esta pregunta necesitamos extraer el proceso asociado al PID `6988`, ya que el objetivo es trabajar posteriormente sobre el ejecutable recuperado, ya sea para análisis con `Strings`, extracción de IOCs o cálculo de hashes del binario.  
Comando utilizado para el dump del proceso:  
`vol -f (path al dump) windows.pslist --pid 6988 --dump`  
Una vez extraído el proceso, calculamos el hash MD5 con PowerShell:  
`Get-FileHash -Algorithm MD5 (path al dump del proceso)`  

<img width="1258" height="209" alt="6" src="https://github.com/user-attachments/assets/54b5084f-327a-4026-a7d4-a52a7b96c67d" />


**R:** `0B493D8E26F03CCD2060E0BE85F430AF`

-----

| Q7 | Can you identify the word that begins at offset 0x45BE876 and is 6 bytes long? |  
-

Para identificar exactamente lo que solicita la pregunta, recurrimos a **HxD**, un editor hexadecimal muy útil para este tipo de validaciones puntuales. Navegamos directamente al offset `0x45BE876` y, una vez posicionados allí, leemos los 6 bytes indicados por la consigna.  

<img width="1919" height="679" alt="7" src="https://github.com/user-attachments/assets/81883e68-d1f0-4e28-83dd-541e61875879" />


**R:** `hacker`

-----

| Q8 | What is the creation date and time of the parent process of powershell.exe? |  
-

En este caso utilizamos el plugin `windows.pstree`, ya que es la forma más clara de visualizar la relación padre-hijo entre procesos dentro del dump. Al revisar el árbol de procesos y ubicar `powershell.exe`, podemos identificar qué proceso lo originó; en este caso, el parent process es `explorer.exe`. A partir de esa misma salida obtenemos la fecha y hora de creación del proceso padre.  
Comando utilizado:  
`vol -f (path al dump) windows.pstree`  

<img width="1902" height="357" alt="8" src="https://github.com/user-attachments/assets/b22a8267-012e-46e7-83d9-8d852f226f14" />


**R:** `2021-04-30 17:39`  

-----

| Q9 | What is the full path and name of the last file opened in notepad? |  
-

Aquí aplicamos el plugin `windows.cmdline`, que nos entrega los parámetros con los que fue ejecutado cada proceso. Para limpiar la salida y enfocarnos únicamente en `notepad.exe`, utilizamos nuevamente `Select-String`. De esta manera identificamos que desde la carpeta temporal de Windows (`\Temp\`) se abrió un archivo potencialmente sensible o sospechoso a través de Notepad.  
Comando utilizado:  
vol -f (path al dump) windows.cmdline | sls "notepad"`  

<img width="1053" height="86" alt="9" src="https://github.com/user-attachments/assets/ea188541-f0ab-403f-9668-77c624ffced7" />


**R:** `C:\Users\JOHNDO~1\AppData\Local\Temp\7zO4FB31F24\accountNum`

-----

| Q10 | How long did the suspect use Brave browser? (In Hours) | 
-

Aquí se nos consulta cuánto tiempo utilizó Brave el sospechoso. Para responder esto recurrimos a **UserAssist**, una clave del registro que almacena información sobre la ejecución de procesos con interfaz gráfica. Este artefacto es muy valioso porque permite identificar la ruta de ejecución del proceso, tiempo activo, cantidad de veces que fue ejecutado un programa y último momento de uso.  
Para extraer esta información desde memoria utilizamos el plugin específico de Volatility:  
`vol -f (path al dump) windows.registry.userassist | sls "brave"`  
De esta manera filtramos el ruido del output y nos quedamos únicamente con la evidencia asociada a Brave.  

<img width="1919" height="244" alt="10" src="https://github.com/user-attachments/assets/447472f8-644f-49e1-b290-a8a2011073ee" />


**R:** `4`  


-----

## 🛡️ Proceso de CTI (Cyber Threat Intelligence)

### 1. **Validación inicial de la evidencia**
```text
Get-FileHash -Algorithm SHA256 (path al dump)
```
Se calculó el hash SHA256 de la imagen de memoria para preservar la integridad de la evidencia y trabajar sobre una referencia verificable durante todo el análisis.

### 2. **Enumeración de procesos relevantes**
```text
vol -f (path al dump) windows.pslist | sls "brave"
vol -f (path al dump) windows.pstree
vol -f (path al dump) windows.cmdline | sls "notepad"
```
Con estos plugins se identificó el PID de `brave.exe`, la relación padre-hijo de `powershell.exe` y el último archivo abierto mediante Notepad.

### 3. **Enumeración de conexiones activas**
```text
vol -f (path al dump) windows.netscan | sls "ESTABLISHED"
```
Se localizaron las conexiones establecidas al momento de la adquisición, permitiendo luego resolver el dominio asociado a la conexión activa de Chrome mediante reverse DNS.

### 4. **Extracción y análisis de binario**
```text
vol -f (path al dump) windows.pslist --pid 6988 --dump
Get-FileHash -Algorithm MD5 (path al dump del proceso)
```
Se extrajo el ejecutable asociado al PID `6988` para posterior hashing y posible análisis estático orientado a IOCs y atribución.

### 5. **Artefactos hexadecimales y registro**
```text
HxD → offset 0x45BE876
vol -f (path al dump) windows.registry.userassist | sls "brave"
```
Con HxD se validó la cadena en el offset solicitado, mientras que UserAssist permitió reconstruir el tiempo de uso del navegador Brave.

## 🎯 MITRE ATT&CK Mapping

| Táctica | Técnica | ID | Descripción Observada |
|---------|---------|----|-----------------------|
| Discovery | Process Discovery | T1057 | Enumeración de procesos relevantes como `brave.exe`, `powershell.exe` y `notepad.exe` |
| Discovery | System Network Connections Discovery | T1049 | Identificación de conexiones activas en estado `ESTABLISHED` mediante `windows.netscan` |
| Command and Control | Application Layer Protocol | T1071.001 | Conexión activa de Chrome hacia `protonmail.ch`, servicio asociado a comunicaciones cifradas |
| Execution | PowerShell | T1059.001 | Presencia y análisis del árbol de ejecución relacionado con `powershell.exe` |
| Collection | Data from Local System | T1005 | Identificación del archivo abierto en Notepad dentro del directorio temporal del usuario |

## 🔬 Herramientas Utilizadas


Memoria
├── Volatility 3 → windows.info, windows.pslist, windows.netscan, windows.pstree, windows.cmdline, windows.registry.userassist

├── PowerShell → Get-FileHash para SHA256 y MD5

└── HxD → análisis manual de offset hexadecimal

Enriquecimiento

└── DNSChecker → Reverse DNS Lookup para resolución de FQDN


## 📊 Lecciones Aprendidas

1. **La validación de hashes de evidencia no es opcional**: es un paso básico pero crítico para garantizar integridad forense y trazabilidad del análisis.
2. **Volatility 3 permite reconstruir comportamiento con mucha precisión** si se combinan correctamente plugins como `pslist`, `pstree`, `netscan`, `cmdline` y `userassist`.
3. **La correlación entre procesos, conexiones activas y artefactos del registro** aporta contexto suficiente para reconstruir uso real del sistema y posibles acciones sospechosas del usuario.

---

> **Santiago Daniel Sandili** – SOC Analyst L1 Portfolio  
> *CyberDefenders Blue Team Lab: Brave (Endpoint Forensics Category)*
