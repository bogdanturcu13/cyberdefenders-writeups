Análisis Forense de Red de Dana Bot
Introducción
En este laboratorio, profundizamos en una investigación forense de red para analizar un ciberataque que involucra el malware Dana Bot. El equipo del SOC (Centro de Operaciones de Seguridad) ha identificado actividad sospechosa dentro del tráfico de red, que revela que una máquina en la red ha sido comprometida. Esta brecha ha llevado a la exfiltración de datos sensibles de la empresa. Como analista de ciberseguridad, tu objetivo es identificar el incidente utilizando un archivo PCAP (Captura de Paquetes) e inteligencia de amenazas asociada para descubrir cómo ocurrió el compromiso e identificar detalles clave sobre el ataque.
El laboratorio se centra en diseccionar las tácticas, técnicas y procedimientos (TTP) utilizados por el atacante, incluyendo reconocimiento, acceso inicial, ejecución y persistencia. Utilizarás Wireshark para extraer y analizar artefactos de red críticos, como archivos maliciosos y comunicación con servidores externos. Al desofuscar código JavaScript y examinar artefactos relacionados, descubrirás cómo el malware se afianzó en la red, ejecutó cargas útiles adicionales y mantuvo su presencia. Este laboratorio brinda una oportunidad integral para practicar habilidades forenses del mundo real y obtener una visión más profunda sobre la detección y respuesta a ataques de malware sofisticados.

Análisis
P1. ¿Qué dirección IP utilizó el atacante durante el acceso inicial?
Respuesta:Analizando el archivo PCAP, podemos ver una solicitud DNS realizada a portfolio.serveirc.com.
La respuesta a la consulta devuelve la dirección IP 62.173.142.148. Cuando buscas este dominio en VirusTotal, se identifica como malicioso.

P2. ¿Cuál es el nombre del archivo malicioso utilizado para el acceso inicial?
Respuesta:Para comenzar a investigar el incidente relacionado con el malware Dana Bot, empezamos analizando el archivo de captura de red usando Wireshark. El objetivo principal es identificar el archivo malicioso utilizado para el acceso inicial. Primero comenzamos examinando los archivos en la función de exportación de objetos HTTP de Wireshark. Esto es accesible navegando a Archivo -> Exportar Objetos -> HTTP.
Al revisar los objetos HTTP exportados, un archivo llamado login.php destaca debido a que su tipo de contenido es application/octet-stream, lo que indica que podría no ser un simple archivo PHP, sino que podría contener datos codificados u ofuscados. El tipo MIME application/octet-stream es un tipo de archivo binario genérico que se utiliza para representar datos brutos que no entran en categorías específicas como texto, imágenes o formatos específicos de aplicación. Se usa a menudo cuando la naturaleza del contenido es desconocida o no puede ser interpretada directamente por el navegador web. Este tipo es común en escenarios donde se transfieren archivos, como ejecutables, scripts ofuscados o datos codificados, a través de HTTP. En el contexto de actividad maliciosa, los archivos etiquetados como application/octet-stream suelen ser sospechosos, ya que pueden contener cargas útiles como malware o scripts destinados a la explotación.
Para validar la sospecha sobre login.php, descarga el archivo del flujo HTTP y analízalo más a fondo en un entorno forense. Ejecutar el comando file en Linux sobre login.php revela que es un archivo JavaScript de gran tamaño (5558 bytes) y no tiene terminadores de línea, una característica a menudo asociada con JavaScript ofuscado o minificado.
Para entender este JavaScript, necesitamos desofuscarlo. Los scripts ofuscados se usan comúnmente en malware para ocultar la verdadera intención del código. Usar herramientas de desofuscación en línea ayuda a revelar su funcionalidad.
El script desofuscado en este caso muestra que aprovecha objetos ActiveX para realizar actividades maliciosas. Específicamente, el script incluye comandos para descargar otro archivo, llamado resources.dll, desde un dominio (soundata.top) y luego ejecutarlo usando rundll32.exe, una herramienta legítima del sistema a menudo abusada para la ejecución de malware. El script también se elimina a sí mismo después de la ejecución, una técnica anti-forense diseñada para dificultar el análisis.
Para obtener el recurso solicitado desde el servidor del atacante, inspeccionamos el flujo HTTP que sirvió el archivo login.php. El análisis del flujo HTTP correspondiente muestra:

La solicitud HTTP GET se realiza al recurso /login.php alojado en el dominio portfolio.serverirc.com.
La respuesta del servidor tiene un Content-Type de application/octet-stream, lo que confirma que el servidor está entregando datos binarios o codificados.
Además, el encabezado Content-Disposition especifica el nombre del archivo como allegato_708.js, indicando que se trata de un archivo JavaScript que se sirve como adjunto.
En el cuerpo de la respuesta, el código JavaScript está altamente ofuscado, caracterizado por nombres de variables largos y caracteres aleatorios. Esto confirma que el actor malicioso está intentando ocultar la funcionalidad del script.


P3. ¿Cuál es el hash SHA-256 del archivo malicioso utilizado para el acceso inicial?
Respuesta:El hash SHA-256 del archivo malicioso utilizado para el acceso inicial se puede calcular usando el siguiente comando de bash:
sha256sum login.php

Dando como resultado el valor de hash: 847b4ad90b1bda2d9117ae05776f3f902dda593fb15222895938acf476c4268.
Esto identifica de forma única el archivo login.php, que se ha confirmado que contiene JavaScript malicioso ofuscado. El hashing es un proceso criptográfico que toma una entrada (en este caso, el archivo login.php) y produce una cadena de caracteres de longitud fija, que es única para el contenido específico de la entrada. SHA-256 (Secure Hash Algorithm 256-bit) es ampliamente utilizado debido a su resistencia a colisiones. En seguridad, los hashes se utilizan para verificar la integridad de los archivos, identificar muestras de malware y comparar con bases de datos de inteligencia de amenazas.

P4. ¿Qué proceso se utilizó para ejecutar el archivo malicioso?
Respuesta:El proceso utilizado para ejecutar el código JavaScript malicioso es WScript.exe.
Basado en el JavaScript desofuscado analizado anteriormente, el script está diseñado para ser ejecutado usando el Windows Script Host (WSH), específicamente a través de WScript.exe, que es el intérprete predeterminado para archivos .js en sistemas Windows. Esto es evidente por la dependencia del script en objetos WScript como WScript.CreateObject, que son fundamentales para la funcionalidad de WSH.
Cuando se ejecuta el archivo JavaScript, WScript.exe procesa el código, iniciando los comandos incrustados en él. Por ejemplo, el script desofuscado usa WScript.CreateObject para invocar utilidades del sistema, como crear un objeto HTTP para descargar el archivo resources.dll, y posteriormente ejecutarlo a través de rundll32.exe.

P5. ¿Cuál es la extensión del segundo archivo malicioso utilizado por el atacante?
Respuesta:Como se puede ver en el análisis anterior, el segundo archivo que fue descargado por el script ofuscado es resources.dll y tiene una extensión .dll.

P6. ¿Cuál es el hash MD5 del segundo archivo malicioso?
Respuesta:Extrae el archivo del flujo de objetos HTTP de Wireshark como hicimos en la pregunta 1 con el archivo login.php, luego calcula el hash del DLL usando el siguiente comando:
md5sum resources.dll

El hash MD5 calculado del DLL es e758e07113016aca55d9eda2b0ffeebe.
