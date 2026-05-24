# Listas Negras — Procesos, Ventanas y Herramientas RE

El AC mantiene múltiples blacklists para detectar herramientas de análisis, debugging
y cheating. Fueron extraídas mediante análisis crudo del binario en PowerShell
(ISO-8859-1, sin ejecución del binario).

**Nota técnica sobre el almacenamiento:** Todas las strings de blacklist están almacenadas
como substrings contiguas en el bloque `.rodata` del binario Go. No hay bytes nulos entre
entradas — los límites exactos entre strings adyacentes están definidos por los descriptores
`{ptr, len}` del slice de Go en otra región de memoria. Esto significa que la separación
entre algunas entradas es inferida, no confirmada al 100%.

---

## Bloque Cleartext Principal (offsets 29507905–29529100)

Extraído del dump crudo de bytes. Bloque completo confirmado (ISO-8859-1):

```
cef.pakaimware%w : %wgdb.exereverseprocesssharpodfiddlerx64_dbgpetoolsmonitor
ollydbgwpe proPhantOmx32_dbgphantomWPE PROcharlescheckerharmonyPEToolssniffer
MDBCrewDIEmWinMonitorDiscord- Operacmd.exefdm.exezen.exeArc.exe
```

El separador `%w : %w` es un format string de error de Go que quedó intercalado entre
las dos mitades de la blacklist por el layout del compilador — NO es parte de ninguna entrada.

---

## Blacklist 1 — Títulos de Ventana (EnumWindows + GetWindowTextW)

String confirmada por análisis del pclntab (offset 0x01C24134):

```
x32dbgpc-retCentoswindbgdbgclrde4dotpepperghidrahackerx96dbgfoldersysmontimersefenceselectscalar
```

**Prefijos que NO son blacklist:** `german`, `french`, `%s%s%s`, `common`, `addons`
— son strings de construcción de path del directorio de addons, almacenadas adyacentes.

Tokens de la blacklist de ventanas — **16 entradas, todas de exactamente 6 caracteres**:

| Token | Herramienta / Descripción |
|-------|--------------------------|
| `x32dbg` | x32dbg — debugger para procesos 32-bit |
| `pc-ret` | PC-Ret — herramienta de análisis de proceso / cheat no identificado |
| `Centos` | CentOS — posible detección de entorno VM/WSL Linux |
| `windbg` | WinDbg — debugger de kernel de Microsoft |
| `dbgclr` | DbgCLR — debugger de runtime .NET (Visual Studio CLR Debugger) |
| `de4dot` | de4dot — deobfuscador de assemblies .NET |
| `pepper` | PepperFlash / componente CEF / posible cheat |
| `ghidra` | Ghidra — framework de RE de la NSA |
| `hacker` | Substring — captura Process Hacker, "hacker" en nombre de ventana |
| `x96dbg` | Variante de x64dbg (x96dbg = versión alternativa) |
| `folder` | Substring — posiblemente Windows Explorer con carpeta sospechosa |
| `sysmon` | Sysinternals SysMon — monitor de eventos de sistema |
| `timers` | Timer Resolution / herramientas de timing (usadas por cheaters) |
| `efence` | Electric Fence — memory debugger (Linux port) |
| `select` | Herramienta con "select" en el título (propósito incierto) |
| `scalar` | Herramienta con "scalar" en el título (propósito incierto) |

El AC usa `EnumWindows` + `GetWindowTextW` para iterar todas las ventanas del sistema
y compara el título contra estos tokens. Método de matching: REGEX (paquete `ncRaYk_Ke` = `regexp`),
probablemente patrón combinado `(?i)(x32dbg|pc-ret|Centos|windbg|...)`.

---

## Blacklist 2 — Nombres de Procesos y Substrings

Extraída del bloque cleartext (offsets ~29507905–29529100). Las entradas están almacenadas
contiguas sin separadores; los límites entre entradas adyacentes se infieren por longitud
y contexto.

### Bloque A — Cheats y herramientas conocidas

| Entry | Long. | Tipo | Descripción |
|-------|-------|------|-------------|
| `cef.pak` | 7 | archivo | Chromium Embedded Framework — detecta Aimware específicamente (usa CEF para su GUI) |
| `aimware` | 7 | substring | Cheat aimware.net (CS:GO / TF2) |
| `wpe pro` | 7 | substring | WPE Pro — Winsock Packet Editor (captura de paquetes, con espacio) |
| `WPE PRO` | 7 | substring | WPE Pro en mayúsculas (case-variant para regex) |
| `PhantOm` | 7 | substring | Cheat PhantOm (capitalización exacta preservada) |
| `phantom` | 7 | substring | Cheat phantom (variante lowercase) |
| `harmony` | 7 | substring | Cheat harmony |
| `MDBCrew` | 7 | substring | Grupo de cheats MDBCrew |
| `checker` | 7 | substring | Genérico — "checker" en nombre de proceso/ventana |

### Bloque B — Debuggers y proxies

| Entry | Long. | Tipo | Descripción |
|-------|-------|------|-------------|
| `gdb.exe` | 7 | proceso | GNU Debugger |
| `x64_dbg` | 7 | substring | x64dbg — debugger 64-bit |
| `x32_dbg` | 7 | substring | x32dbg — debugger 32-bit (con guión bajo, distinto del token `x32dbg` de B1) |
| `ollydbg` | 7 | substring | OllyDbg — debugger clásico de RE |
| `fiddler` | 7 | substring | Fiddler — HTTP/S proxy |
| `charles` | 7 | substring | Charles — HTTP proxy |
| `sniffer` | 7 | substring | Genérico — sniffers de red |

### Bloque C — Herramientas PE y análisis

| Entry | Long. | Tipo | Descripción |
|-------|-------|------|-------------|
| `petools` | 7 | substring | PE Tools (lowercase) |
| `PETools` | 7 | substring | PE Tools (capitalización exacta) |
| `monitor` | 7 | substring | Genérico — Process Monitor, System Monitor, etc. |
| `reverse` | 7 | substring | Genérico — procesos/ventanas con "reverse" |
| `process` | 7 | substring | Genérico — Process Explorer, Process Hacker, etc. |
| `sharp` | 5 | substring | Substring — dnSpy (SharpSpy), SharpPcap, dotnet-trace |

### Bloque D — Substrings ambiguos (límites inferidos)

Zona de 14 bytes continuos `DIEmWinMonitor` — **dos interpretaciones posibles**:

| Interpretación A | Interpretación B |
|-----------------|-----------------|
| `DIEmW` (5) + `WinMonitor` (10) — dos entradas independientes | `DIEmWinMonitor` (14) — una sola entrada substring |
| `DIEmW` = variante del cheat DIE / "Detect It Easy" modificado | Matches cualquier proceso/ventana con "DIEmWinMonitor" exacto |
| `WinMonitor` = herramienta de monitoreo de Windows | Menos probable (nombre demasiado específico para ser substring) |

**Probabilidad:** Interpretación A es más consistente con el patrón del resto de la blacklist
(entradas cortas de 5-10 chars). La W inicial de "WinMonitor" se une visualmente a "DIEmW" por
el layout contiguo de Go.

Entrada `od` (2 bytes): presente en el blob contiguo entre `sharp` y `fiddler`
(`sharpodfiddler`). Podría ser: OllyDbg abreviado, `od` unix tool, o ruido de layout.
Con solo 2 chars, matching por substring generaría demasiados falsos positivos — **su
función exacta como entry standalone es incierta**.

### Bloque E — Procesos de browsers y apps con overlay

| Entry | Descripción |
|-------|-------------|
| `Discord-` | Discord — título de ventana "Discord - [canal]" (el guión es parte del patrón) |
| `Operacmd.exe` | Opera Browser — proceso específico del command-line de Opera |
| `fdm.exe` | Free Download Manager |
| `zen.exe` | Zen Browser |
| `Arc.exe` | Arc Browser (overlay integrado) |

**Nota sobre Discord-:** El byte siguiente al guión es `0x20` (espacio), lo que sugiere que
la entry completa podría ser `"Discord- "` (8 chars con espacio final). Discord usa el formato
de título `"Discord - #canal"`, por lo que el patrón `"Discord-"` con o sin espacio captura
ese formato.

**Nota sobre Opera:** La entry es `Operacmd.exe` — el ejecutable del componente command-line
de Opera. No es la aplicación principal `opera.exe` ni detección genérica de "Opera".

---

## Blacklist 3 — Herramientas RE Cortas (strings raw, zona VPK)

Encontrada en la región de datos adyacente al glob pattern de VPK.
String concatenada confirmada en el binario:

```
.vpkdumppeekkgdbmdbgxdeb
```

Contexto completo: `eEpPPOST.vpkdumppeekkgdbmdbgxdeb%s%sallgall`

| Token | Herramienta |
|-------|-------------|
| `.vpk` | Extensión VPK — glob `*.vpk` para archivos de addons |
| `dump` | Herramientas de memory dump (ProcDump, WinPMem, etc.) |
| `peek` | Memory peek tools |
| `kgdb` | Kernel GDB — debugger a nivel de kernel |
| `mdbg` | mdbg.exe — Microsoft Debugger para .NET |
| `xdeb` | xdeb — debugger X Window (menos probable en contexto Windows) |

---

## Blacklist 4 — Herramientas .NET y Cheats L4D2 (zona k1e1y*.vpk)

Encontrada inmediatamente después del glob `k1e1y*.vpk`. String concatenada:

```
startdnspyilspyILSpypizzacrackida -brutejamesDebug
```

Contexto en el binario:
```
k1e1y*.vpkstartdnspyilspyILSpypizzacrackida -brutejamesDebug/auth
```

Tokens confirmados (offset 0x01C1E1E8) — **todos de exactamente 5 caracteres**:

| Token | Longitud | Herramienta / Descripción |
|-------|----------|--------------------------|
| `start` | 5 | Procesos con "start" — propósito incierto |
| `dnspy` | 5 | dnSpy — decompilador y debugger de .NET/C# |
| `ilspy` | 5 | ILSpy — decompilador .NET/C# (lowercase) |
| `ILSpy` | 5 | ILSpy — decompilador .NET/C# (capitalización exacta) |
| `pizza` | 5 | **Cheat pizza para Left 4 Dead 2** (detección específica L4D2) |
| `crack` | 5 | Genérico — ventanas/procesos con "crack" |
| `ida -` | 5 | **IDA Pro** — patrón de título de ventana `"IDA - [archivo]"` |
| `brute` | 5 | Herramientas de fuerza bruta |
| `james` | 5 | James — herramienta o cheat (no identificado con certeza) |
| `Debug` | 5 | Genérico — ventanas/procesos con "Debug" en el nombre |

**CORRECCIÓN CRÍTICA sobre `ida -`:** El token es `ida -` (5 bytes: `69 64 61 20 2D`) — incluye
espacio y guión. Está diseñado para capturar el título de ventana de IDA Pro (`"IDA - archivo.idb"`).
Renombrar `ida64.exe` o `ida.exe` NO evita la detección — el título de la ventana también debe
evitar el patrón `"ida -"`.

---

## Glob Pattern de Archivos VPK

```
*.vpk        — todos los archivos VPK en el directorio de addons
k1e1y*.vpk   — patrón específico para cheats L4D2 conocidos
%s\%sk1e1y*.vpk — formato completo con path base y subdirectorio
```

El AC enumera todos los archivos `.vpk` del directorio de addons del juego y reporta
al servidor vía mensaje protobuf (campos `Path` y `Size`). El patrón `k1e1y` es un
identificador hardcoded de cheats VPK conocidos para L4D2.

---

## Módulo WatchList — `_6di6zc0se2v`

El paquete `_6di6zc0se2v` implementa el mecanismo de vigilancia continua:

```go
_6di6zc0se2v.(*_o3YyW).WatchList    // función principal de la lista de vigilancia
_6di6zc0se2v.(*_o3YyW).Add          // agregar item a la lista de vigilancia
_6di6zc0se2v.(*_o3YyW).AddWith      // agregar item con condición
_6di6zc0se2v.(*_o3YyW).Remove       // eliminar de la lista
_6di6zc0se2v.(*_o3YyW).Close        // cerrar el watcher
_6di6zc0se2v.(*Rs0QF1nUdhgw).Has    // verificar si un item está en la lista
_6di6zc0se2v.(*UWzc59).Has          // verificar (tipo alternativo)
```

5 funciones `init.func` (func1-func5) — inicialización de 5 categorías distintas de
items vigilados (ventanas, procesos, módulos, archivos VPK, otro).

---

## Técnica Anti-Análisis: MustFindProcByOrdinal

El binario carga algunas APIs de Windows por número de ordinal en lugar de nombre.
Esto impide identificar las APIs simplemente buscando sus nombres como strings.

APIs cargadas por **nombre** (visibles en binario): `VirtualQuery`, `BitBlt`, `PatBlt`,
`GetDC`, `GetSystemInfo`

APIs cargadas por **ordinal** (sin strings visibles): `ReadProcessMemory`, `OpenProcess`,
`CreateToolhelp32Snapshot`, `EnumWindows`, `GetWindowTextW`, y otras.

---

## Resumen: Total de Entradas por Blacklist

| Blacklist | Método de detección | Entradas conocidas |
|-----------|--------------------|--------------------|
| BL1 — Títulos de ventana | EnumWindows + GetWindowTextW + regex | 16 |
| BL2 — Procesos/substrings | CreateToolhelp32Snapshot + substring match | ~30 |
| BL3 — Herramientas RE cortas | Substring match (zona VPK) | 6 |
| BL4 — .NET + cheats L4D2 | Substring match (zona k1e1y) | 10 |
| **Total** | | **~62 entradas** |

---

## Bypass de las Blacklists

### Para títulos de ventana (BL1, BL4)

El chequeo es por substring REGEX en el título de ventana. Cambiar el título resuelve el problema:

- **x32dbg / x96dbg:** Options → Preferences → GUI → cambiar título de ventana
- **WinDbg:** `|0.writetitle "custom"` en la consola de WinDbg  
- **Ghidra:** modificar el property `application.name` en `support/launch.properties`
- **IDA Pro:** El patrón es `"ida -"` (con espacio-guión). Renombrar `ida64.exe` NO es suficiente.
  Hay que evitar que el título de ventana contenga `"ida -"`. Opciones:
  - Renombrar el archivo `.idb/.i64` a algo que no genere ese patrón
  - Script IDA: modificar el título de ventana vía API Win32 al abrir
  - Usar IDA en modo headless/batch (`-A -S script.idc`)

### Para procesos (BL2, BL3, BL4)

El nombre del ejecutable y/o el título de ventana es lo que se compara. Renombrar el binario evita:

- `fiddler.exe` → `proxy_tool.exe` (evita substring "fiddler")
- `dnspy.exe` → `code_view.exe` (evita "dnspy")
- `gdb.exe` → `analyze.exe`
- `Fiddler.exe` en título → cambiar a cualquier título sin "fiddler"

### Para Browsers con overlay

- **Discord:** usar discord.com en navegador web en lugar del cliente de escritorio
- **Arc/Zen browser:** usar otro navegador sin overlay mientras el AC está activo
- **Opera:** el proceso detectado es `Operacmd.exe` específicamente — el ejecutable principal
  `opera.exe` no está en la lista

### Entradas NO bypasseables solo con renombrar

- `process`, `monitor`, `reverse`, `sharp` — son substrings genéricos. Si el nombre de
  ventana del debugger contiene cualquiera de estas palabras, es detectado
- `cef.pak` — detecta la presencia del archivo CEF en el sistema/proceso, no solo el nombre
- `pizza` — detección de cheat VPK específico de L4D2

### Nota sobre `od` (2 chars)

Si es una entry real (incierto), cualquier proceso con "od" en el nombre sería detectado.
Poco probable que genere false positives reales dado que la mayoría de herramientas
comunes no contienen esa secuencia.
