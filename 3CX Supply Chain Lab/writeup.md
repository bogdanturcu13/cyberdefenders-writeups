# 3CX Supply Chain Lab

## Escenario
Una gran corporación multinacional depende en gran medida del software **3CX** para la comunicación telefónica, convirtiéndolo en un componente crítico de sus operaciones comerciales.  

Después de una actualización reciente de la aplicación de escritorio 3CX, las alertas de antivirus marcan instancias esporádicas en las que el software se elimina de algunas estaciones de trabajo, mientras que otras permanecen intactas. Descartando esto como un falso positivo, el equipo de TI ignora las alertas, solo para notar:

- Rendimiento degradado
- Tráfico de red extraño hacia servidores desconocidos
- Empleados reportan problemas con la aplicación 3CX

El equipo de seguridad de TI identifica patrones de comunicación inusuales vinculados a **actualizaciones recientes del software**.  

Como analista de inteligencia de amenazas, tu responsabilidad es:

1. Examinar este posible ataque a la **cadena de suministro**.
2. Descubrir cómo los atacantes comprometieron la aplicación 3CX.
3. Identificar al posible **actor de amenazas** involucrado.
4. Evaluar la **extensión general del incidente**.

---

## Preguntas y Respuestas

### 1. Versiones maliciosas de 3CX en Windows
> Entender el alcance del ataque e identificar qué versiones exhiben comportamiento malicioso es crucial para tomar decisiones informadas si estas versiones comprometidas están presentes en la organización.

**Respuesta:**  
Las dos versiones de 3CX detectadas como vulnerables son:

- 18.12.407  
- 18.12.416  

**Número de versiones vulnerables:** 2

---

### 2. Hora de creación UTC del malware `.msi`
> Determinar la antigüedad del malware ayuda a evaluar la extensión del compromiso y rastrear la evolución de familias y variantes de malware.

**Respuesta:**  
`2023-03-13 6:33 UTC`

---

### 3. DLL maliciosas depositadas por el archivo `.msi`
> Analizar los archivos depositados por el instalador de software de Microsoft (.msi) es crucial para identificar archivos maliciosos e investigar su potencial completo.

**Respuesta:**  
Las DLL detectadas como maliciosas por VirusTotal son:

- `ffmpeg.dll`  
- `d3dcompiler_47.dll`

---

### 4. ID de la técnica MITRE usada por los `.msi` para cargar el DLL malicioso
> Reconocer las técnicas de persistencia usadas es esencial para estrategias de mitigación y mejoras de defensa futuras.

**Respuesta:**  
- Técnica: **DLL Side-Loading**  
- ID MITRE: `T1574`  
- Funciones observadas:
  - Persistencia en el sistema: hijack execution flow  
  - Escalada de privilegios: hijack execution flow  
  - Evasión de defensas: hijack execution flow  

---

### 5. Categoría de amenaza de los DLL maliciosos
> Reconocer el tipo de malware es esencial para la investigación.

**Respuesta:**  
- Categoría: **Trojan**

---

### 6. Técnicas de evasión de virtualización/sandbox
> Entender cómo el malware puede evadir la detección en entornos virtualizados o sistemas de análisis.

**Respuesta:**  
- ID MITRE: `T1497`

---

### 7. Hipervisor objetivo de las técnicas anti-análisis en `ffmpeg.dll`
> Evitar perder tiempo durante ingeniería inversa identificando técnicas anti-análisis.

**Respuesta:**  
- Hipervisor: **VMWare**  
- Descripción: `Reference anti-VM strings targeting VMWare`

---

### 8. Algoritmo de encriptación usado por `ffmpeg.dll`
> Identificar el método criptográfico usado en el malware es crucial para entender sus técnicas de evasión y funcionalidad.

**Respuesta:**  
- Algoritmo: **RC4 PRGA**

---

### 9. Grupo APT responsable del ataque
> Identificar al grupo APT responsable permite investigar TTPs usuales y descubrir otras actividades maliciosas potenciales.

**Respuesta:**  
- Grupo: **Lazarus** (Corea del Norte)  
- Fuente: [Dark Reading](https://www.darkreading.com/endpoint-security/automatic-officlal-updates-malicious-3cx-enterprises)
