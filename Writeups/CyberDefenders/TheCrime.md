# 🔍 The Crime Lab WriteUp – Mobile Forensics (CyberDefenders)

## 🧩 Escenario del Laboratorio

**Contexto**: Una víctima ha desaparecido bajo circunstancias sospechosas. Las autoridades han adquirido una imagen forense de su dispositivo Android y solicitan al equipo de DFIR un análisis exhaustivo para reconstruir su actividad reciente, identificar sus deudas, ubicación y planes de viaje que permitan determinar el móvil del crimen.  
**Objetivo**: Analizar la imagen forense del dispositivo Android con ALEAPP para extraer artefactos de aplicaciones instaladas, historial de llamadas, mensajes, caché de fotos y conversaciones de Discord que permitan reconstruir la línea temporal de la víctima.  
**Herramientas**: ALEAPP.  
**Tácticas**: Collection | Discovery.

El laboratorio provee una imagen forense de un dispositivo Android para análisis con ALEAPP.

---

## 📋 Preguntas del Laboratorio & Respuestas

| Q1 | Based on the accounts of the witnesses and individuals close to the victim, it has become clear that the victim was interested in trading. This has led him to invest all of his money and acquire debt. Can you identify the SHA256 of the trading application the victim primarily used on his phone?
-

En la sección **Installed Apps (GMS) for user 0** de ALEAPP encontramos el listado completo de aplicaciones instaladas en el dispositivo (5 entradas). Entre ellas, además de múltiples versiones de Discord y YouTube, se identifica una aplicación con Bundle ID `com.ticno.olymptrade`, correspondiente a la plataforma de trading **OlympTrade**. En la columna SHA-256 Hash podemos leer directamente el hash del archivo APK instalado.

<img width="1919" height="755" alt="1" src="https://github.com/user-attachments/assets/6c9a3577-c232-4679-99e2-4d02e2c3d19e" />

**R:** `4f168a772350f283a1c49e78c1548d7c2c6c05106d8b9feb825fdc3466e9df3c`

------

| Q2 | According to the testimony of the victim's best friend, he said, "While we were together, my friend got several calls he avoided. He said he owed the caller a lot of money but couldn't repay now". How much does the victim owe this person?
-

En la sección **SMS messages** de ALEAPP encontramos un único mensaje SMS recibido el **2023-09-20 20:09:49**. Su contenido es una amenaza extorsiva explícita del acreedor hacia la víctima: *"It's time for you to pay back the money you owe me, but you're not picking up my calls. You better think twice about not paying, because it won't end well for you. Prepare the sum of 250,000 EGP, and I'll expect your call within an hour at most."* El monto de la deuda queda explicitado directamente en el cuerpo del mensaje.

<img width="1919" height="753" alt="2" src="https://github.com/user-attachments/assets/af81daed-bee2-48c1-bf41-f885349e3076" />

<img width="1919" height="709" alt="2 1" src="https://github.com/user-attachments/assets/c5037582-b9eb-406a-8c43-a07a6fba806e" />

**R:** `250000`

------

| Q3 | What is the name of the person to whom the victim owes money?
-

Correlacionando el número remitente del SMS amenazante (`+201172137258`) con la sección **Contacts** de ALEAPP, identificamos que dicho número pertenece al contacto guardado en la agenda de la víctima como **Shady Wahab**. Esta correlación entre el SMS y la agenda del dispositivo establece la identidad del acreedor que amenaza a la víctima.

<img width="1919" height="565" alt="3" src="https://github.com/user-attachments/assets/cb770373-b49d-4ef4-ae82-40b890c85869" />


**R:** `Shady Wahab`

------

| Q4 | Based on the statement from the victim's family, they said that on September 20, 2023, he departed from his residence without informing anyone of his destination. Where was the victim located at that moment?
-

Al revisar la sección **Recent Activity** en ALEAPP, identificamos que la última aplicación con actividad registrada fue **com.google.android.apps.maps** con `Last_Time_Moved: 2023-09-20 23:50:29`. Al examinar el snapshot asociado a esa actividad reciente, podemos ver la pantalla de Google Maps del dispositivo mostrando la ubicación activa de la víctima: **The Nile Ritz-Carlton, Cairo**.

<img width="1919" height="528" alt="4" src="https://github.com/user-attachments/assets/f142f12d-7455-4553-bf8e-c80f1c9f7840" />

<img width="788" height="518" alt="4 1" src="https://github.com/user-attachments/assets/766d55ae-7f81-4f0f-b4f0-77730ce3b2ad" />


**R:** `The Nile Ritz-Carlton`

------

| Q5 | The detective continued his investigation by questioning the hotel lobby. She informed him that the victim had reserved the room for 10 days and had a flight scheduled thereafter. The investigator believes that the victim may have stored his ticket information on his phone. Look for where the victim intended to travel.
-

Navegando a la sección **Google Photos (gphotos-1) - Cache** en ALEAPP, encontramos en caché una imagen correspondiente a un boarding pass de **Egypt Airlines**. Este artefacto es especialmente valioso porque persiste en caché aunque el usuario haya eliminado la imagen original de su galería. El ticket muestra todos los detalles del viaje planeado por la víctima: vuelo 310, fecha 01/10/2023, horario 09:00 AM, Gate 08, Seat 20, Origen: Cairo → Destino: **Las Vegas**.


<img width="1924" height="594" alt="5" src="https://github.com/user-attachments/assets/a907211b-3101-47c9-86c4-c75782014af3" />


<img width="1919" height="624" alt="5 1" src="https://github.com/user-attachments/assets/26d5bbd2-33b0-4ba5-90fb-dd7f821a9c8e" />

**R:** `Las Vegas`

------

| Q6 | After examining the victim's Discord conversations, we discovered he had arranged to meet a friend at a specific location. Can you determine where this meeting was supposed to occur?
-

Al examinar la sección **Discord Chats** dentro de ALEAPP, encontramos una conversación entre el usuario `infern0_o` y `rob1ns0n`. En el segundo mensaje, `rob1ns0n` responde: *"What a wonderful news! We'll meet at **The Mob Museum**. I'll await your call when you arrive. Enjoy your flight bro ❤️"*. El punto de encuentro acordado queda confirmado explícitamente en el contenido del mensaje.

<img width="1274" height="688" alt="6" src="https://github.com/user-attachments/assets/801fa45a-1503-4a09-ab56-4e96010da5f7" />

**R:** `The Mob Museum`

------

## 🔬 Resumen del Proceso de Análisis

### 1. **Identificación de la Aplicación de Trading**

Installed Apps (GMS) → com.ticno.olymptrade

└── SHA-256: 4f168a772350f283a1c49e78c1548d7c2c6c05106d8b9feb825fdc3466e9df3c

→ OlympTrade: plataforma de trading responsable de las deudas de la víctima



### 2. **Reconstrucción de la Deuda y el Acreedor**

SMS messages → +201172137258 → Amenaza extorsiva: 250,000 EGP

Contacts → +201172137258 = Shady Wahab

→ Correlación directa: acreedor identificado por nombre y número



### 3. **Geolocalización de la Víctima**

Recent Activity → com.google.android.apps.maps

└── Last_Time_Moved: 2023-09-20 23:50:29

Snapshot Maps → The Nile Ritz-Carlton, Cairo

→ Última ubicación conocida de la víctima el día de su desaparición



### 4. **Plan de Viaje**

Google Photos Cache → Boarding pass Egypt Airlines

├── Vuelo: 310 | Fecha: 01/10/2023 | Hora: 09:00 AM

├── Origen: Cairo → Destino: Las Vegas

└── Contexto: reserva de 10 días en hotel antes de la partida



### 5. **Punto de Encuentro Acordado**

Discord Chats → infern0_o ↔ rob1ns0n

└── Meeting point confirmado: The Mob Museum (Las Vegas)

---


## 🔬 Herramientas Utilizadas

🔍 Análisis Forense Android

└── ALEAPP → Installed Apps, SMS Messages, Contacts, Recent Activity, Google Photos Cache, Discord Chats

---

## 📊 Lecciones Aprendidas

1. **Google Photos Cache como evidencia forense**: El directorio `disk_cache` de Google Photos almacena imágenes en caché aunque el usuario las haya eliminado de su galería. Es una fuente frecuentemente ignorada que puede contener evidencia crítica como boarding passes, documentos fotografiados y capturas de pantalla.

2. **Correlación de artefactos múltiples**: La reconstrucción completa del caso requirió correlacionar artefactos de 6 fuentes distintas (Installed Apps, SMS, Contacts, Recent Activity, Google Photos, Discord). Ningún artefacto individual era suficiente; la narrativa se construyó cruzando la información de todos ellos.

3. **SMS como evidencia de amenaza directa**: Los mensajes SMS almacenados en `mmssms.db` constituyen evidencia forense directa de extorsión. Su contenido, timestamp y número de remitente son determinantes para establecer móvil y autoría en una investigación criminal.

---

> **Santiago Daniel Sandili** – SOC Analyst L1 Portfolio  
> *CyberDefenders Blue Team Lab: The Crime (Mobile Forensics Category)*
