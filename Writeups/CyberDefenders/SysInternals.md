# 🔍 SysInternals Lab WriteUp – Endpoint Forensics (CyberDefenders)

## 🧩 Escenario del Laboratorio

**Contexto**: Un usuario descargó un archivo ejecutable haciéndose pasar por una herramienta legítima de SysInternals. Las autoridades sospechan que dicho archivo es malware y solicitan al equipo de DFIR un análisis forense completo de la imagen de disco para determinar qué fue descargado, cuándo fue ejecutado, qué infraestructura contactó y qué acciones realizó en el sistema.  
**Objetivo**: Analizar una imagen forense de disco Windows con Autopsy, AppCompatCacheParser, AmcacheParser y VirusTotal para reconstruir la cadena de infección completa: desde la descarga del malware hasta su persistencia en el sistema.  
**Herramientas**: Autopsy, AppCompatCacheParser (Eric Zimmerman), AmcacheParser (Eric Zimmerman), Timeline Explorer (Eric Zimmerman), VirusTotal.  
**Tácticas**: Initial Access | Execution | Persistence | Defense Evasion | Command and Control.

Este laboratorio de dificultad **Medium** requirió investigar artefactos forenses de Windows que no son de conocimiento inmediato: el **Amcache**, el **AppCompatCache** y el **historial de comandos de PowerShell**. A continuación se detalla tanto el proceso de análisis como los conceptos aprendidos durante la resolución.

---

## 📚 Conceptos Clave Investigados

Antes de poder responder las preguntas del laboratorio, fue necesario investigar y comprender tres artefactos forenses de Windows:

**AppCompatCache (ShimCache)**  
Registro de Windows ubicado en `HKLM\\SYSTEM\\CurrentControlSet\\Control\\Session Manager\\AppCompatCache`. Registra los ejecutables que han sido **creados o modificados** en el sistema, incluyendo su ruta y timestamp de última modificación. Es importante aclarar que su presencia **no garantiza ejecución** — solo indica que el archivo existió en ese path. Se parsea con `AppCompatCacheParser.exe` de Eric Zimmerman.

**Amcache.hve**  
Hive de registro ubicado en `C:\\Windows\\AppCompat\\Programs\\Amcache.hve`. A diferencia del ShimCache, el Amcache **sí registra ejecución real** de programas, almacenando además el hash **SHA1** del ejecutable, su ruta completa y el timestamp de primera ejecución. Se parsea con `AmcacheParser.exe` de Eric Zimmerman.

**PSReadLine - ConsoleHost_history.txt**  
Archivo ubicado en `C:\\Users\\<USER>\\AppData\\Roaming\\Microsoft\\Windows\\PowerShell\\PSReadLine\\ConsoleHost_history.txt`. Almacena el **historial de comandos de PowerShell** ejecutados por el usuario, similar al `.bash_history` en Linux. Es un artefacto de alto valor forense porque puede revelar comandos maliciosos ejecutados manualmente o por scripts.

---

## 📋 Preguntas del Laboratorio & Respuestas

| Q1 | What was the malicious executable file name that the user downloaded?
-

En Autopsy, navegando a la ruta `Users/Public/Downloads`, encontramos el archivo **SysInternals.exe** marcado con una **X roja** indicando que fue eliminado del sistema de archivos (estado Unallocated). Su nombre imita deliberadamente la suite legítima de herramientas de Microsoft SysInternals, una técnica clásica de **masquerading** para evadir sospechas del usuario.

<img width="1919" height="720" alt="1" src="https://github.com/user-attachments/assets/a15ee811-9280-4d12-b489-71921196507d" />

**R:** `SysInternals.exe`

------

| Q2 | When was the last time the malicious executable file was modified? 12-hour format?
-

Para obtener el timestamp preciso de última modificación del ejecutable, utilizamos las herramientas de **Eric Zimmerman**: primero `AppCompatCacheParser.exe` para parsear el hive `SYSTEM` del registro. 

<img width="1919" height="191" alt="2" src="https://github.com/user-attachments/assets/61070feb-b6fb-4d19-823b-34713e5f89d3" />

El output '.csv' fue cargado en **Timeline Explorer** para su análisis. Filtrando por `sysinternals` en el AppCompatCache, encontramos la entrada con `Last Modified Time UTC: 2022-11-15 21:18:51`.

<img width="1919" height="220" alt="2 1" src="https://github.com/user-attachments/assets/863dad4b-d362-455f-88be-afd14c0d575e" />

**R:** `11/15/2022 6:18:51 PM`

------

| Q3 | What is the SHA1 hash value of the malware?
-

El **Amcache**, a diferencia del AppCompatCache, registra el hash SHA1 real del ejecutable al momento de su primera ejecución. 

<img width="1900" height="288" alt="3" src="https://github.com/user-attachments/assets/2710dc9a-9474-4378-81d5-70d69cab5b2f" />

Cargando el CSV generado por `AmcacheParser.exe` en Timeline Explorer y filtrando por `sysinternals`, encontramos la entrada correspondiente a `c:\\users\\public\\downloads\\sysinternals.exe` con su hash SHA1 completo.

<img width="1919" height="407" alt="3 1" src="https://github.com/user-attachments/assets/888d6ec3-e5d4-42d9-8a60-338c26a781d0" />


**R:** `fa1002b02fc5551e075ec44bb4ff9cc13d563dcf`

------

| Q4 | What is the malware's family?
-

Buscando el hash MD5 del ejecutable `72e6d1728a546c2f3ee32c063ed09fa6ba8c46ac33b0dd2e354087c1ad26ef48` en **VirusTotal**, el análisis muestra que **51 de 70** vendors lo clasifican como malicioso. En la sección **Detection** el vendor **Alibaba** lo detecta como `Downloader:Win32/Rozena.cadb0acb`.

<img width="1919" height="838" alt="4" src="https://github.com/user-attachments/assets/d941a351-1af6-430b-8691-73a7cdf0cbb0" />

**R:** `Rozena`

------

| Q5 | What is the first mapped domain's Fully Qualified Domain Name (FQDN)?
-

El análisis se realizó en dos niveles complementarios. Primero, en **VirusTotal** sección **Relations**, vemos que el malware contactó la URL `http://www.malware430.com/html/VMwareUpdate.exe`. 

<img width="1896" height="620" alt="5" src="https://github.com/user-attachments/assets/9b252670-24ba-4c2d-aa6f-29608e4c734d" />

**R:** `www.malware430.com`

------

| Q6 | The mapped domain is linked to an IP address. What is that IP address?
-
Desde el punto de vista forense del host, en **Autopsy** navegamos al archivo `ConsoleHost_history.txt` ubicado en `IEUser\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\`. Este historial de PowerShell contiene el comando: `Add-Content -Path $env:windir\\System32\\drivers\\etc\\hosts -Value "192.168.15.10 www.malware430.com" -Force`, revelando que el malware **modificó el archivo hosts** para mapear el dominio malicioso a una IP interna, técnica de DNS hijacking local para redirigir tráfico o evadir detección.

<img width="1919" height="813" alt="6" src="https://github.com/user-attachments/assets/82684056-3481-4042-ac34-e722ba27cb83" />


El mismo comando `Add-Content` que mapea el dominio en el archivo `hosts` especifica explícitamente la dirección IP asociada: `192.168.15.10`. Esta IP privada fue mapeada tanto a `www.malware430.com` como a `www.sysinternals.com`, lo que sugiere que el atacante buscaba interceptar conexiones legítimas a SysInternals para redirigirlas a su infraestructura.


**R:** `192.168.15.10`

------

| Q7 | What is the name of the executable dropped by the first-stage executable?
-

En **VirusTotal**, sección **Behavior** (Process Tree), observamos la cadena de ejecución completa del malware. El ejecutable `SysInternals.exe` lanza como proceso: `"%ComSpec%" /C %windir%\\vmtoolsIO.exe -install && net start VMwareIOHelperService && sc config VMwareIOHelperService start= auto`. Esto confirma que `SysInternals.exe` descarga y coloca `vmtoolsIO.exe` en `C:\Windows\`, simulando como si fuese una herramienta legítima de VMware.

<img width="1919" height="400" alt="7 y 8" src="https://github.com/user-attachments/assets/dc32d486-052f-4f41-9325-2f66b695beb6" />

**R:** `vmtoolsIO.exe`

------

| Q8 | What is the name of the service installed by 2nd stage executable?
-

Continuando el análisis del árbol de procesos de la captura anterior, el mismo comando ejecutado por el segundo stage instala un servicio de Windows mediante `sc config VMwareIOHelperService start= auto`, el nombre del servicio imita un componente legítimo de VMware Tools.


**R:** `VMwareIOHelperService`

------

## 🔬 Resumen del Proceso de Análisis

### 1. **Descarga e Identificación del Malware**

Autopsy → Users/Public/Downloads → SysInternals.exe (deleted/unallocated)

└── Técnica: Masquerading (T1036) — nombre imita suite legítima de Microsoft



### 2. **Timeline del Ejecutable**

AppCompatCacheParser → SYSTEM hive → Last Modified: 2022-11-15 21:18:51 UTC

AmcacheParser → Amcache.hve → SHA1: fa1002b02fc5551e075ec44bb4ff9cc13d563dcf

└── Amcache confirma ejecución real; AppCompatCache confirma presencia en disco



### 3. **Análisis de Reputación**

VirusTotal (MD5: 72e6d1728a546c2f3ee32c063ed09fa6ba8c46ac33b0dd2e354087c1ad26ef48)

├── 51/70 vendors: malicious

├── Family: deyma / jaik

└── Categoría: Trojan + Downloader



### 4. **Infraestructura C2 y DNS Hijacking**

PSReadLine history → ConsoleHost_history.txt

├── hosts modification: 192.168.15.10 → www.malware430.com

└── hosts modification: 192.168.15.10 → www.sysinternals.com



### 5. **Cadena de Infección Completa**

SysInternals.exe (1st stage)

└── Drops: C:\\Windows\\vmtoolsIO.exe (2nd stage)

└── Instala servicio: VMwareIOHelperService (start=auto → persistencia)

---

## 🎯 MITRE ATT&CK Mapping

| Táctica | Técnica | ID | Descripción Observada |
|---------|---------|----|-----------------------|
| Initial Access | User Execution: Malicious File | T1204.002 | Usuario descarga y ejecuta SysInternals.exe desde C:\\Users\\Public\\Downloads |
| Defense Evasion | Masquerading | T1036 | Ejecutable nombrado SysInternals.exe para imitar herramienta legítima de Microsoft |
| Defense Evasion | Modify Hosts File | T1565.001 | PowerShell modifica hosts para mapear www.malware430.com y www.sysinternals.com a 192.168.15.10 |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | Comandos maliciosos ejecutados via PowerShell, registrados en ConsoleHost_history.txt |
| Persistence | Create or Modify System Process: Windows Service | T1543.003 | vmtoolsIO.exe instala VMwareIOHelperService con start=auto para sobrevivir reinicios |
| Command and Control | Application Layer Protocol: Web Protocols | T1071.001 | Malware contacta http://www.malware430.com/html/VMwareUpdate.exe para descarga de 2nd stage |

---

## 🔬 Herramientas Utilizadas

  Análisis de Imagen Forense

├── Autopsy → File system browsing y PSReadLine history extraction

  Parseo de Artefactos de Registro (Eric Zimmerman Tools)

├── AppCompatCacheParser.exe → Parseo de ShimCache desde hive SYSTEM

├── AmcacheParser.exe → Parseo de Amcache.hve para SHA1 y evidencia de ejecución

└── Timeline Explorer → Visualización y filtrado de CSVs generados

  Análisis de Malware

└── VirusTotal → Reputación por hash, Process Tree, Contacted URLs, Family labels

---

## 📊 Lecciones Aprendidas

1. **Amcache vs AppCompatCache**: Son dos artefactos distintos y complementarios. El **AppCompatCache** (ShimCache) registra archivos que existieron en el disco pero **no garantiza ejecución**. El **Amcache** sí confirma ejecución real y además almacena el **SHA1** del binario, lo que lo convierte en fuente directa de inteligencia de amenazas sin necesidad de tener el archivo físico.

2. **PSReadLine como artefacto forense**: El archivo `ConsoleHost_history.txt` es el equivalente Windows del `.bash_history` de Linux. En entornos donde los logs de PowerShell están deshabilitados (ScriptBlock Logging, Module Logging), este archivo puede ser la **única evidencia** de comandos maliciosos ejecutados manualmente por el atacante o por scripts de primer stage.

3. **Masquerading con nombres de herramientas legítimas**: Nombrar malware como `SysInternals.exe` o `vmtoolsIO.exe` es una táctica deliberada para evadir sospechas visuales y potencialmente bypassear reglas de detección que excluyen nombres de herramientas de administración conocidas. La modificación del archivo `hosts` para apuntar `www.sysinternals.com` a una IP controlada por el atacante demuestra un entendimiento profundo de cómo los administradores interactúan con esas herramientas.

---

> **Santiago Daniel Sandili** – SOC Analyst L1 Portfolio  
> *CyberDefenders Blue Team Lab: SysInternals (Endpoint Forensics Category)*
