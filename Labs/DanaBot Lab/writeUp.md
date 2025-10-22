# DanaBot Lab

## Escenario
En este laboratorio realizamos una investigación de **forense de red** para analizar un ataque que involucra al malware **Dana Bot**. El **SOC** ha detectado actividad sospechosa dentro del tráfico de red, revelando que una máquina de la red ha sido comprometida. Esta brecha ha permitido la **exfiltración de datos sensibles de la empresa**. Como analista de ciberseguridad, el objetivo es investigar el incidente utilizando un archivo **PCAP** y fuentes de **Threat Intelligence** para descubrir cómo ocurrió el compromiso y extraer detalles clave del ataque. Se utilizará **Wireshark** para extraer y analizar artefactos críticos de red, como archivos maliciosos y comunicaciones con servidores externos.

> **Advertencia:** No ejecutar ninguno de los ficheros en sistemas de producción. Analizar en un entorno forense o sandbox controlado.

---

## Preguntas y Respuestas

### 1. Dirección IP utilizada por el atacante durante el acceso inicial
> Analizando el archivo PCAP, se identifica una petición DNS hacia `portfolio.serveirc.com`.

**Respuesta:**  
La respuesta de la consulta DNS devuelve la IP: **`62.173.142.148`**.  
Al buscar este dominio en fuentes de inteligencia (por ejemplo VirusTotal) se identifica como **malicioso**.

![Petición DNS en Wireshark](https://github.com/bogdanturcu13/cyberdefenders-writeups/tree/main/Labs/DanaBot%20Lab/img/dns.png)

---

### 2. Nombre del archivo malicioso usado para el acceso inicial
> Para ver los archivos que se han tratado durante el intercambio de paquetes, se analiza el archivo PCAP en Wireshark y se exportan los objetos HTTP (`File → Export Objects → HTTP`).  

**Respuesta:**  
Existen dos posibilidades: que el malware no se encuentre todavía en los sistemas víctimas y deba entrar (POST) o que sí se encuentre en los sistemas y adquiera datos/archivos necesarios para llevar a cabo el ataque (POST). Por tanto podemos filtrar en Wireshark por http.request.method == 'GET' or http.request.method == 'POST'. Vemos que no hay ningún POST, por lo que analizamos los GET. El primer GET que se realiza es hacia /login.php del dominio portfolio.serveirc.com y el contenido que transporta es **`allegato_708.js`**.

![Exportación de objetos HTTP en Wireshark](https://github.com/bogdanturcu13/cyberdefenders-writeups/tree/main/Labs/DanaBot%20Lab/img/allegato_708.png)

---

### 3. SHA-256 del archivo malicioso inicial
> Se puede calcular usando la herramienta `sha256sum` en Linux:

`sha256sum login.php`

**Respuesta:**
847b4ad90b1bda2d9117ae05776f3f902dda593fb15222895938acf476c4268

---

### 4. ¿Qué proceso ha cargado el archivo malicioso?
> Para estudiar el funcionamiento del malware podemos buscarlo en ANY.RUN y ver su ejecución. O también podemos hacer un `strings allegato_708.js` y desofuscar el resultado con alguna herramienta online como deobfuscate.io. De cualquier manera vemos que el JS invoca un proceso llamado wscript.exe.

**Respuesta:**  
wscript.exe

![Procesos de DanaBot en ANY.RUN](https://github.com/bogdanturcu13/cyberdefenders-writeups/tree/main/Labs/DanaBot%20Lab/img/anyrun.png)

---

### 5. ¿Cuál es el segundo archivo malicioso que usa el atacante?
> Seguimos la misma mecánica, buscar archivos en HTTP (`File → Export Objects → HTTP`).

**Respuesta:**  
resources.dll desde el dominio soundata.top.

![Exportación de objetos HTTP en Wireshark](https://github.com/bogdanturcu13/cyberdefenders-writeups/tree/main/Labs/DanaBot%20Lab/img/objects.png)

---

### 6. Hash MD5 del segundo archivo malicioso
> Se puede calcular usando la herramienta `md5sum` en Linux:

**Respuesta:**  
e758e07113016aca55d9eda2b0ffeebe



