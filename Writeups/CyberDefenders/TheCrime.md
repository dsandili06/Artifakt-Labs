# 🔍 The Crime Lab WriteUp – Mobile Forensics (CyberDefenders)

## 🧩 Escenario del Laboratorio

**Contexto**: Un individuo sospechoso ha sido identificado como posible autor de un crimen. Las autoridades han adquirido una imagen forense de su dispositivo Android y solicitan al equipo de DFIR un análisis exhaustivo para reconstruir su actividad reciente, identificar su red de contactos, y correlacionar la evidencia digital con los hechos investigados.  
**Objetivo**: Analizar la imagen forense del dispositivo Android con ALEAPP para extraer artefactos de comunicación, actividad de aplicaciones, historial de llamadas, mensajes y metadatos de vuelo que permitan reconstruir la línea temporal del sospechoso.  
**Herramientas**: ALEAPP.  
**Tácticas**: Collection | Discovery.

El laboratorio provee una imagen forense de un dispositivo Android para análisis con ALEAPP.

---

## 📋 Preguntas del Laboratorio & Respuestas

| Q1 | Based on the accounts found on the device, what is the email used by the suspect?
-

En ALEAPP navegamos a la sección de **Partner Settings** dentro de Device Info, donde se listan las cuentas asociadas al dispositivo. Allí podemos identificar el email registrado por el sospechoso como cuenta principal del dispositivo Android.

<img width="1919" height="765" alt="1" src="" />

**R:** `mohamedshaahmed@gmail.com`

------

| Q2 | What is the name of the application used by the suspect to communicate with the criminal?
-

Al examinar la sección **Discord Chats** dentro de ALEAPP, encontramos conversaciones activas en el dispositivo. Los mensajes muestran una coordinación explícita entre el sospechoso (`infern0_o`) y un interlocutor (`rob1ns0n`), donde se discute un cambio de planes, se confirma un vuelo reservado para el **01/10 a las 9:00 AM** y se acuerda el punto de encuentro en **The Mob Museum**. Esta comunicación activa a través de Discord confirma que fue la aplicación utilizada para coordinar la actividad criminal.

<img width="1919" height="765" alt="2" src="" />

**R:** `Discord`

------

| Q3 | What is the name of the criminal that the suspect is trying to meet?
-

Continuando el análisis de los **Discord Chats**, en el segundo mensaje visible en la conversación podemos identificar el nombre de usuario del interlocutor al que el sospechoso está tratando de encontrarse.

<img width="1919" height="765" alt="2" src="" />

**R:** `rob1ns0n`

------

| Q4 | What is the destination city of the suspect?
-

Navegando a la sección **Google Photos (gphotos-1) - Cache** en ALEAPP, encontramos en caché una imagen correspondiente a un boarding pass de **Egypt Airlines**. El pasaje muestra claramente el trayecto del vuelo desde **El Cairo** hacia el destino final. Este artefacto de Google Photos es especialmente valioso porque persiste en caché aunque el usuario haya eliminado la imagen original de su galería.

<img width="1919" height="765" alt="3" src="" />

<img width="1919" height="765" alt="3 1" src="" />

**R:** `Las Vegas`

------

| Q5 | What is the name of the flight company in the ticket found on the phone?
-

En el mismo boarding pass recuperado del caché de Google Photos analizado en la pregunta anterior, podemos leer claramente la aerolínea impresa en el ticket de vuelo del sospechoso.

<img width="1919" height="765" alt="3 1" src="" />

**R:** `Egypt Airlines`

------

| Q6 | What is the city where the crime was committed?
-

Al revisar la sección **Recent Activity** en ALEAPP, identificamos que la última aplicación con actividad registrada fue **com.google.android.apps.maps** con `Last_Time_Moved: 2023-09-20 23:50:29`. El snapshot de la actividad reciente de Google Maps muestra la ubicación que el sospechoso estaba consultando al momento del análisis: **The Nile Ritz-Carlton, Cairo**, lo que indica que el crimen fue cometido en esa ciudad.

<img width="1919" height="765" alt="4" src="" />

<img width="1919" height="765" alt="4 1" src="" />

**R:** `Cairo`

------

| Q7 | What is the date on which the crime was committed?
-

Correlacionando los artefactos analizados: el boarding pass muestra que el vuelo del sospechoso estaba programado para el **01/10/2023**, mientras que la actividad de Google Maps fue registrada el **2023-09-20** y los Discord Chats corresponden al **20/09/2023**. El crimen fue cometido el día en que el sospechoso aún se encontraba en El Cairo, antes de su partida.

<img width="1919" height="765" alt="4" src="" />

**R:** `20/09/2023`

------

| Q8 | What is the name of the victim?
-

En la sección **Contacts** de ALEAPP encontramos la agenda del dispositivo con 6 contactos. Correlacionando con la sección **Call Logs**, identificamos que el número `+201172137258` fue el destinatario de múltiples llamadas perdidas y rechazadas el **20/09/2023** entre las 19:31 y 19:45. Este mismo número pertenece al contacto guardado como **Shady Wahab** en la agenda del sospechoso, identificándolo como la víctima.

<img width="1919" height="765" alt="5" src="" />

<img width="1919" height="765" alt="5 1" src="" />

**R:** `Shady Wahab`

------

| Q9 | What is the phone number of the criminal?
-

Revisando la sección **SMS messages** en ALEAPP, encontramos un único mensaje SMS recibido el **2023-09-20 20:09:49** desde el número `+201172137258`. El contenido del mensaje es una amenaza explícita de carácter extorsivo: *"It's time for you to pay back the money you owe me, but you're not picking up my calls. You better think twice about not paying, because it won't end well for you. Prepare the sum of 250,000 EGP, and I'll expect your call within an hour at most."* Este número corresponde al mismo contacto identificado en los call logs, confirmando que es el número del criminal.

<img width="1919" height="765" alt="6" src="" />

**R:** `+201172137258`

------

| Q10 | What is the name of the stock market application on the phone?
-

En la sección **Installed Apps (GMS) for user 0** de ALEAPP encontramos el listado completo de aplicaciones instaladas en el dispositivo (5 entradas). Entre ellas, además de múltiples versiones de Discord y YouTube, se identifica una aplicación con Bundle ID `com.ticno.olymptrade`, correspondiente a la plataforma de trading **OlympTrade**, siendo la única aplicación de mercado de valores presente en el dispositivo.

<img width="1919" height="765" alt="7" src="" />

**R:** `OlympTrade`

------

## 🔬 Resumen del Proceso de Análisis

### 1. **Identificación del Sospechoso**

Partner Settings → Email: mohamedshaahmed@gmail.com

Discord Chats → Usuario: infern0_o

→ Conversación activa con: rob1ns0n (criminal)



### 2. **Reconstrucción del Plan de Viaje**

Google Photos Cache → Boarding pass Egypt Airlines

├── Pasajero: Mohamed Ahmed

├── Vuelo: 310 | Fecha: 01/10/2023 | Hora: 09:00 AM

├── Origen: Cairo → Destino: Las Vegas

└── Punto de encuentro acordado por Discord: The Mob Museum



### 3. **Geolocalización del Crimen**

Recent Activity → com.google.android.apps.maps

└── Last_Time_Moved: 2023-09-20 23:50:29

Snapshot Maps → The Nile Ritz-Carlton, Cairo

→ Ciudad del crimen: Cairo | Fecha: 20/09/2023



### 4. **Identificación de la Víctima y el Criminal**

Call Logs → +201172137258 → 9 llamadas perdidas/rechazadas (19:31–19:45)

Contacts → +201172137258 = Shady Wahab (víctima)

SMS messages → +201172137258 → Amenaza extorsiva de 250,000 EGP



### 5. **Aplicaciones Relevantes**

Installed Apps → com.ticno.olymptrade → OlympTrade (stock market app)

---

## 🎯 MITRE ATT&CK Mapping

| Táctica | Técnica | ID | Descripción Observada |
|---------|---------|----|-----------------------|
| Collection | Data from Local System | T1005 | Artefactos de Discord, Google Photos, SMS y Contacts extraídos del dispositivo Android |
| Command and Control | Application Layer Protocol: Web Protocols | T1071.001 | Discord utilizado como canal de comunicación cifrado para coordinación criminal |
| Discovery | System Information Discovery | T1082 | Boarding pass, ubicación Google Maps y apps instaladas revelan actividad del sospechoso |
| Impact | Financial Theft | T1657 | Mensaje SMS exige 250,000 EGP bajo amenaza explícita a la víctima Shady Wahab |

---

## 🔬 Herramientas Utilizadas

🔍 Análisis Forense Android

└── ALEAPP → Partner Settings, Discord Chats, Google Photos Cache, Recent Activity, Call Logs, Contacts, SMS Messages, Installed Apps

---

## 📊 Lecciones Aprendidas

1. **Google Photos Cache como evidencia forense**: El directorio `disk_cache` de Google Photos almacena imágenes en caché aunque el usuario las haya eliminado de su galería. Es una fuente frecuentemente ignorada que puede contener evidencia crítica como boarding passes, documentos fotografiados y capturas de pantalla.

2. **Correlación de artefactos múltiples**: La reconstrucción completa del caso requirió correlacionar artefactos de 6 fuentes distintas (Discord, Google Photos, Maps, Call Logs, Contacts, SMS). Ningún artefacto individual era suficiente; la narrativa se construyó cruzando la información de todos ellos.

3. **SMS como evidencia de amenaza directa**: Los mensajes SMS almacenados en `mmssms.db` constituyen evidencia forense directa de extorsión. Su contenido, timestamp y número de remitente pueden ser determinantes para establecer móvil y autoría en una investigación criminal.

---

> **Santiago Daniel Sandili** – SOC Analyst L1 Portfolio  
> *CyberDefenders Blue Team Lab: The Crime (Mobile Forensics Category)*
