# Monitoreo del Proceso del Juego — gopsutil

El AC usa la biblioteca Go `gopsutil` (o una versión interna similar) para monitorear el
proceso `left4dead2.exe` con un nivel de detalle inusual para un anticheat de juego.

---

## Funciones gopsutil Identificadas en el Binario

Las siguientes funciones fueron encontradas directamente como strings en el binario,
confirmando el uso de gopsutil para monitoreo del proceso del juego:

```
CmdlineWithContext      — línea de comandos completa del proceso
EnvironWithContext      — variables de entorno del proceso
PercentWithContext      — porcentaje de CPU usado por el proceso
ThreadsWithContext      — lista de threads del proceso
MemoryInfo              — RSS, VMS, swap del proceso
MemoryMaps              — mapeo de memoria del proceso (regiones)
IOCounters              — bytes leídos/escritos por el proceso
NumThreads              — número de threads
PageFaults              — page faults del proceso
NumCtxSwitches          — context switches (voluntary + involuntary)
SendSignal              — enviar señal al proceso
Foreground              — si el proceso está en foreground
Connections             — conexiones de red del proceso
CPUAffinity             — afinidad de CPU del proceso
RlimitUsage             — límites de recursos del proceso
```

---

## Datos Recolectados

### Línea de Comandos (CmdlineWithContext)

El AC puede leer los argumentos con los que fue lanzado el juego. Esto permite detectar:

```
-condebug          — habilita consola de debug en L4D2
-textmessagedebug  — debug de mensajes de texto
-dev               — modo de desarrollo
-insecure          — modo inseguro (deshabilita VAC)
+sv_cheats 1       — consola con cheats habilitados
```

Si el juego fue lanzado con `-insecure` o `+sv_cheats`, el AC podría reportarlo.

### Variables de Entorno (EnvironWithContext)

Lee el entorno completo del proceso del juego. Puede detectar:

```
WINEDLLOVERRIDES    — indica que está corriendo bajo Wine/Proton con DLLs modificadas
WINEDEBUG           — modo de debug de Wine
LD_PRELOAD          — DLLs precargadas (Linux via Proton)
STEAM_COMPAT_DATA_PATH — ruta de compatibilidad de Steam (Proton)
```

Aunque L4D2 es un juego de Windows, algunos jugadores usan Proton. Las variables de
entorno de Proton podrían ser un flag en el análisis del servidor.

### Conexiones de Red (Connections)

Lee las conexiones de red del proceso del juego. Esto permite al AC verificar:

- Que el juego está realmente conectado al servidor correcto
- Si hay conexiones sospechosas a herramientas de análisis de red
- Si hay proxies o VPNs activos en el proceso

### Porcentaje de CPU (PercentWithContext)

Monitoreo de uso de CPU del proceso. Picos inusuales de CPU podrían indicar
actividad de herramientas de trampas que corren en el mismo proceso.

### Memoria (MemoryInfo)

```
rss    — Resident Set Size (memoria física usada)
vms    — Virtual Memory Size (memoria virtual total)
swap   — memoria en swap
hwm    — high water mark de memoria
stack  — tamaño de stack
```

Valores de memoria anómalos podrían indicar inyección de código.

### Context Switches (NumCtxSwitches)

```
voluntary    — context switches voluntarios
involuntary  — context switches involuntarios (preempted)
```

Un número inusualmente alto de context switches involuntarios puede indicar que hay
código externo interferiendo con el proceso del juego.

---

## Paquete Go — sNAkh4

El paquete `sNAkh4` (garbled) implementa el wrapper de gopsutil:

```
sNAkh4.cd1Utgxh0      — inicialización del monitor de proceso
sNAkh4.gvBDt2BpBt7    — goroutine de monitoreo continuo
sNAkh4.ySbqKa9f0p     — handler de resultados de monitoreo
sNAkh4.Rs0QF1nUdhgw   — tipo de resultado de monitoreo
sNAkh4.UWzc59         — tipo de estado del proceso
sNAkh4.encTsS0_       — codificación de resultados
sNAkh4.GrRzvM2        — función de inicio del monitor
sNAkh4.vaf3j0Z        — validación de proceso
sNAkh4.wqT1oO         — loop de monitoreo
```

---

## Datos Enviados al Servidor

Los campos JSON identificados para stats de proceso/sistema confirman qué datos
se envían al servidor en el reporte:

```json
{
  "pid": 1234,
  "cpu": "porcentaje uso CPU",
  "rss": 123456789,
  "vms": 987654321,
  "hwm": 500000000,
  "stack": 8388608,
  "user": 45.2,
  "system": 12.1,
  "idle": 42.7,
  "iowait": 0.1,
  "irq": 0.0,
  "softirq": 0.0,
  "steal": 0.0,
  "voluntary": 1234,
  "involuntary": 567,
  "pending_process": 0,
  "pending_thread": 0,
  "width": 1920,
  "height": 1080,
  "physicalSize": 27
}
```

---

## Por Qué Esto Es Relevante para el Bypass

### Variables de entorno
Si se tienen herramientas de análisis que se inyectan via variables de entorno
(como `LD_PRELOAD` o `WINEDLLOVERRIDES`), el AC las puede detectar leyendo el entorno
del proceso del juego.

### Línea de comandos
Lanzar el juego con flags de debug puede ser un indicador de actividad sospechosa.
En particular `-insecure` deshabilita VAC pero el AC de L4D2Center seguiría activo y
podría tratar este flag como sospechoso.

### Conexiones de red
Si hay un proxy local (`127.0.0.1:8080`) captando el tráfico del juego, las
conexiones del proceso del juego lo revelarían.

### Implicaciones para el bypass
Un bypass completo debe evitar dejar rastros en:
1. Variables de entorno del proceso del juego
2. Línea de comandos del juego
3. Conexiones de red del proceso del juego
4. Número de threads y context switches del proceso

El método más limpio es usar una VM separada para las herramientas de análisis,
aislando completamente el proceso del juego del entorno de análisis.
