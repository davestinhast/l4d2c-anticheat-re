# Lógica de Detección Interna

Documentación completa de todos los mecanismos de detección identificados en el binario.

---

## Vector 1 — Escaneo de Firmas en Memoria (CheatSigs)

### Cómo funciona

```
1. Al conectar: cliente envía CheatSigsRequested=true
2. Servidor responde con H1oahxz1l3iY.CheatSigs ([]bytes — array de patrones)
3. Cada firma es un mensaje W99qYP { Name: "NombreCheat", Pattern: []byte{...} }
4. El AC itera la memoria del juego con VirtualQuery
5. Para cada región MEM_COMMIT y legible:
   memmem(región, tamaño, patrón, tamaño_patrón)
6. Si hay match: envía reporte al servidor con el nombre del cheat y la región
```

### Tipos de regiones escaneadas

```
Escaneadas (objetivo principal):
  PAGE_EXECUTE_READ         código ejecutable estándar
  PAGE_EXECUTE_READWRITE    código inyectado o JIT
  PAGE_EXECUTE_WRITECOPY    copy-on-write ejecutable

Probablemente omitidas:
  PAGE_NOACCESS             acceso directo causaría excepción
  MEM_FREE                  no asignado
  Rangos conocidos de DLLs del sistema (ntdll, kernel32, etc.)
  Rango propio del juego (left4dead2.exe — sería autodetección)
```

### Lectura de memoria

El AC es un proceso separado (no inyectado en el juego). Usa `ReadProcessMemory` con un handle al proceso `left4dead2.exe` obtenido con `OpenProcess`.

### Lo que se signa tipicamente

```
1. Cabeceras PE del cheat DLL (MZ + secciones específicas)
2. Prólogos de funciones conocidas de cheats
3. String literals del cheat (nombres de menú, configs)
4. Magic bytes / headers de frameworks de cheats
5. Secuencias de import características
6. Archivos de config cargados en memoria
```

---

## Vector 2 — Títulos de Ventana (EnumWindows)

El AC enumera todas las ventanas del sistema con `EnumWindows` + `GetWindowTextW` y verifica cada título contra la blacklist.

**Método de matching: REGEX** — confirmado por la presencia del paquete `ncRaYk_Ke` = `regexp/regexp/syntax` con el método `MatchRune`. Las strings de la blacklist son probablemente compiladas en una expresión regular combinada del tipo `(?i)(x32dbg|pc-ret|ghidra|...)` para matching eficiente.

Ver `analysis/listas_negras.md` para la lista completa.

Goroutine 3 ejecuta este check cada ~5-10 segundos.

**Implicación para el bypass:** Renombrar la ventana del debugger puede evadir este check SOLO si el regex usa anchors o patterns exactos. Si usa `(?i)x32dbg` como substring, cualquier nombre que NO contenga `x32dbg` evade el check.

---

## Vector 3 — Lista de Procesos

Enumeración de procesos del sistema via WMI (`Win32_Process`) o `CreateToolhelp32Snapshot`.

El package `_6di6zc0se2v` (WatchList) gestiona la vigilancia continua de procesos.

**Método de matching: REGEX** — igual que Vector 2, las strings de la blacklist de procesos se compilan en un patrón regex. El paquete `bszAWJqu` (posiblemente el escáner de procesos/ventanas) usa `ncRaYk_Ke` (regexp) para matching.

---

## Vector 4 — Módulos Cargados en el Juego

El goroutine func3 de `StartL4D2` vigila la lista de módulos DLL cargados en el proceso del juego:

```
main.(*HpP4qwz).StartL4D2.func3  — Process injection watcher
```

Usa `Module32First`/`Module32Next` vía `CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid)` para obtener todos los DLLs cargados. Cualquier DLL inesperado que aparezca después del inicio del juego es sospechoso.

---

## Vector 5 — Capturas de Pantalla (BitBlt)

```
main.(*HpP4qwz).StartL4D2.func4      — goroutine de screenshots
main.(*HpP4qwz).StartL4D2.func4.1    — sub-handler
```

Captura la pantalla periodicamente usando BitBlt/PatBlt + GetDC. Las capturas se almacenan y se envían al servidor como evidencia al momento del ban.

Estimado: cada ~120 segundos (interval aleatorio para dificultar la predicción).

---

## Vector 6 — Validación de Estado del Juego (Source Engine)

String encontrada en binario (offset 29623296): `i4localSurvivorGunFire`

Este string es el nombre de una variable interna del Source Engine. El AC lo usa probablemente como **anchor para localizar la ConVar table en memoria** — busca el string `i4localSurvivorGunFire` en el espacio de memoria del proceso del juego con `ReadProcessMemory`, luego navega la estructura de datos del Source Engine para leer valores de estado del juego.

Detecta:
- Valores imposibles (ej: salud de survivor fuera de rango)
- Velocidad de disparo anormal
- Flags de godmode/noclip activados vía ConVar
- Estado de cheat commands (`sv_cheats`, etc.)

Esto sugiere que el AC realiza **validación de estado de juego** además de escaneo de firmas — método que detecta cheats que modifican directamente los valores del juego en memoria sin inyectar código.

**Bypass:** No práctico sin entender exactamente qué valores compara. Requeire análisis dinámico para ver qué offsets se calculan desde el anchor.

---

## Vector 7 — Validación de Addons VPK

```
P4mAKk.(*BEt_icchsrxy).GetAddons           — lista de VPKs
P4mAKk.(*BEt_icchsrxy).GetAddonsProvided   — flag de envío
```

El AC enumera todos los archivos `.vpk` en el directorio de addons del juego y envía la lista al servidor. El servidor puede rechazar la conexión si se detecta un VPK con firma conocida de cheat.

También existe `GetAddonsProvided` que es un booleano — posiblemente el servidor puede solicitar la lista de addons bajo demanda (cuando hay sospecha).

---

## Vector 8 — Heartbeat y Validación Continua

Cada ~30 segundos el AC envía un heartbeat al servidor. El servidor puede:
- Responder con nuevas CheatSigs (actualización en caliente)
- Emitir un ban inmediato
- Solicitar un screenshot
- Terminar la sesión

Si el heartbeat no llega en el tiempo esperado, el servidor puede asumir que el AC fue terminado y aplicar sanciones.

---

## Vector 9 — Verificación de Versión

```
P4mAKk.(*Ok4xyD92c8).GetVersion    — versión del AC
P4mAKk.(*BEt_icchsrxy).GetVersion  — versión del juego
```

El servidor verifica que la versión del AC es la última. Versiones antiguas son rechazadas, forzando la actualización. Esto previene el uso de versiones hookeadas del AC.

---

## Vector 10 — Sistema Anti-Smurf

```
P4mAKk.(*BEt_icchsrxy).GetSmurf    — detección de smurf
P4mAKk.(*AAUUUh).GetInstallDate     — fecha de instalación
DiU0jWi.(*PXdkqC).IsWindowsVersionAtLeast — versión de Windows
```

Factores del sistema anti-smurf:
- Antigüedad de la cuenta Steam
- Fecha de instalación de Steam (desde registro/WMI)
- HWID — si el mismo hardware tiene múltiples cuentas baneadas
- Historial de la IP del servidor
- Campo `Witnesses` — posiblemente jugadores que reportaron al sospechoso

---

## Vector 11 — Monitoreo de Proceso vía gopsutil

El AC usa la biblioteca `gopsutil` (paquete `sNAkh4`) para monitorear el proceso del juego
con un nivel de detalle mucho más fino que los ACs convencionales.

### Datos recolectados del proceso left4dead2.exe

```
CmdlineWithContext   — línea de comandos completa al lanzar el juego
EnvironWithContext   — variables de entorno del proceso del juego
PercentWithContext   — uso de CPU del proceso
ThreadsWithContext   — lista de threads del proceso
MemoryInfo           — RSS, VMS, swap, hwm, stack
MemoryMaps           — mapa completo de regiones de memoria
IOCounters           — bytes leídos/escritos por el proceso
NumThreads           — número de threads
NumCtxSwitches       — context switches (voluntary + involuntary)
PageFaults           — page faults del proceso
Connections          — conexiones de red del proceso del juego
CPUAffinity          — afinidad de CPU
Foreground           — si el proceso está en foreground
```

Esto permite al AC detectar:
- Flags de debug en la línea de comandos (`-condebug`, `-insecure`, `+sv_cheats 1`)
- Variables de entorno sospechosas (`WINEDLLOVERRIDES`, `LD_PRELOAD`)
- Proxies locales en las conexiones del proceso (`127.0.0.1:8080`)
- Memoria anómala por inyección de código

Ver `analysis/monitoreo_proceso.md` para documentación completa.

---

## Vector 12 — A2S Query (Consulta al Servidor Source Engine)

El AC implementa el protocolo A2S (Source Engine server query) para consultar directamente
al servidor de juego e independientemente verificar los datos.

### Paquetes identificados

```
hh58lo           — cliente A2S query (40+ funciones)
eUmM3IpzIrV      — struct de respuesta A2S_INFO
gG7Vmb7          — biblioteca de transporte (300+ funciones)
```

### Campos A2S_INFO recolectados

La struct `eUmM3IpzIrV` contiene los campos estándar de A2S_INFO:

```json
{
  "Name": "nombre del servidor",
  "Map": "c1m1_hotel",
  "Folder": "left4dead2",
  "Game": "Left 4 Dead 2",
  "AppID": 550,
  "GameID": 550,
  "Players": 4,
  "MaxPlayers": 4,
  "Bots": 0,
  "VAC": true,
  "Visibility": 0,
  "Port": 27015,
  "Protocol": 17,
  "ServerOS": "W",
  "ServerType": "d",
  "EDF": 177,
  "Mode": "coop",
  "Deaths": 0,
  "Score": 0,
  "Duration": 1234,
  "SourceTV": false
}
```

El AC compara estos datos con lo que el jugador reporta para detectar inconsistencias
(ej: el jugador dice conectarse al servidor X pero A2S_INFO muestra datos diferentes).

---

## Paquete VKiZI7 — Aclaración

**VKiZI7 NO es detección de procesos.** Es el paquete de integración con WebView2 (Chromium embebido para la UI de Wails). Los métodos como `ProcessFailed`, `AddRef`, `Release` son parte de la interfaz COM de WebView2.

`VKiZI7.(*BchsYIa).ProcessFailed` = el renderer de WebView2 crasheó, no el juego.

---

## Tiempos Estimados de los Checks

| Vector de detección | Intervalo estimado |
|--------------------|-------------------|
| Heartbeat al servidor | ~30 segundos |
| Scan de títulos de ventana | ~5-10 segundos |
| Scan de memoria (CheatSigs) | ~60 segundos (costoso) |
| Screenshot | ~120 segundos (posiblemente aleatorio) |
| Módulos del juego | Continuo (event-based) |
| HWID (initial) | Una sola vez al conectar |

Estos son estimados — no confirmados sin análisis dinámico.

---

## Estructura del Reporte de Ban

Cuando se detecta algo, el handler (ConnectL4D2Server.func4) envía:

```protobuf
// Datos del reporte enviados al servidor:
SteamID:     "76561198xxxxxxxxx"
Pattern:     { Name: "nombre_cheat", Pattern: bytes_del_match }
HWData:      { ... hardware completo ... }
Witnesses:   [ SteamIDs de otros jugadores en el servidor ]
Score:       puntaje actual
Deaths:      muertes
Bots:        cantidad de bots
Screenshot:  datos de imagen (posiblemente campo Data o ExtraInfo)
```
