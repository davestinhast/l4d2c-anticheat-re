# Listas Negras — Procesos, Ventanas y Herramientas RE

El AC mantiene cuatro blacklists separadas para detectar herramientas de análisis, debugging
y cheating. Tres de las cuatro son strings cortas que strings.exe (min-length 4+) no captura —
fueron encontradas mediante búsqueda cruda del binario en PowerShell.

---

## Blacklist 1 — Títulos de Ventana (EnumWindows + GetWindowTextW)

String combinada encontrada en el binario:

```
commonaddonsx32dbgpc-retCentoswindbgdbgclrde4dotpepperghidrahackerx96dbgfoldersysmontimersefenceselectscalar
```

Desglosada en tokens individuales:

| Token | Herramienta / Descripción |
|-------|--------------------------|
| `commonaddons` | Referencia a addons conocidos (uso incierto) |
| `x32dbg` | x32dbg — debugger para procesos 32-bit |
| `pc-ret` | PC Ret — herramienta de análisis de PC |
| `Centos` | CentOS — detección posible de VM Linux |
| `windbg` | WinDbg — debugger de Microsoft |
| `dbgclr` | DbgCLR — debugger de runtime .NET |
| `de4dot` | de4dot — deobfuscador de assemblies .NET |
| `pepper` | Pepper Flash / componente Chromium |
| `ghidra` | Ghidra — framework de RE de la NSA |
| `hacker` | Ventanas con "hacker" en el título |
| `x96dbg` | Variante de x64dbg / x96dbg |
| `folder` | Posiblemente "SteamHacker" u otras tools |
| `sysmon` | Sysinternals SysMon (process monitor) |
| `timers` | Herramientas de timers/profiling |
| `efence` | Electric Fence — memory debugger |
| `selectscalar` | Posiblemente herramienta de análisis de memoria |

El AC usa `EnumWindows` + `GetWindowTextW` para iterar todas las ventanas del sistema
y compara el título contra estos tokens (case-sensitive, substring match probable).

---

## Blacklist 2 — Nombres de Procesos del Sistema

Strings encontradas en el binario para comparar contra la lista de procesos:

```
gdb.exe         reverse         processharp     od
fiddler         x64_dbg         petools         monitor
ollydbg         wpe pro         PhantOm         x32_dbg
phantom         WPE PRO         charles         checker
harmony         PETools         sniffer         MDBCrew
DIEmW           inMonitor       Discord         Opera
cmd.exe         fdm.exe         zen.exe         Arc.exe
```

### Categorías

**Debuggers:**
- `gdb.exe` — GNU Debugger
- `ollydbg` — OllyDbg (debugger clásico de RE)
- `x64_dbg` / `x32_dbg` — x64dbg y x32dbg

**Proxies y captura de red:**
- `fiddler` — Fiddler HTTP proxy
- `wpe pro` / `WPE PRO` — Winsock Packet Editor (captura de paquetes)
- `charles` — Charles proxy

**Herramientas PE / RE:**
- `petools` / `PETools` — PE Tools
- `processharp` — ProcessSharp
- `DIEmW` — Detect-It-Easy (detección de packers/protectors)
- `reverse` — genérico para herramientas de reversing

**Monitores del sistema:**
- `monitor` / `inMonitor` — herramientas de monitoreo de procesos
- `checker` — genérico
- `sniffer` — sniffers de red

**Browsers y apps con overlay:**
- `Discord` — bloqueado por el overlay de Discord Game SDK
- `Opera` — bloqueado por riesgo de overlay
- `Arc.exe` — Arc Browser con overlay
- `zen.exe` — Zen Browser

**Herramientas del sistema:**
- `cmd.exe` — Command Prompt (bloqueado mientras el AC está activo)
- `fdm.exe` — Free Download Manager
- `od` — octal dump (herramienta Unix)

---

## Blacklist 3 — Herramientas RE (strings cortas, raw binary)

Encontrada mediante búsqueda cruda del binario. String concatenada:

```
.vpkdumppeekkgdbmdbgxdeb
```

Contexto en el binario: `eEpPPOST.vpkdumppeekkgdbmdbgxdeb%s%sallgall`

Tokens desglosados:

| Token | Herramienta |
|-------|-------------|
| `.vpk` | Extensión VPK — también glob pattern `*.vpk` para archivos |
| `dump` | Herramientas de memory dump (ProcDump, etc.) |
| `peek` | Herramientas de memory peek |
| `kgdb` | Kernel GDB — debugger a nivel de kernel |
| `mdbg` | Microsoft Debugger (mdbg.exe) |
| `xdeb` | xdeb — X Window debugger |

Nota: esta blacklist aparece junto con el string HTTP `POST` y formatos `%s%s`, sugiriendo
que está en la misma región de datos que las strings de formato de requests.

---

## Blacklist 4 — Herramientas .NET y RE (strings cortas, raw binary)

Encontrada justo después del glob pattern `*.vpk`. String concatenada:

```
startdnspyilspyILSpypizzacrackida -brutejamesDebug
```

Contexto en el binario:
```
k1e1y*.vpkstartdnspyilspyILSpypizzacrackida -brutejamesDebug/auth
```

Tokens desglosados:

| Token | Herramienta / Descripción |
|-------|--------------------------|
| `start` | Posiblemente "startdebug" — procesos que inician con "start" |
| `dnspy` | dnSpy — decompilador y debugger de .NET / C# |
| `ilspy` | ILSpy — decompilador de .NET / C# |
| `ILSpy` | ILSpy (case-sensitive, variante) |
| `pizza` | Pizza (herramienta de cheating conocida para Source games) |
| `crack` | Genérico — ventanas/procesos con "crack" en el nombre |
| `ida` | IDA Pro — el disassembler/debugger más popular |
| `-brute` | Parámetro de fuerza bruta — detecta si se lanza con este argumento |
| `james` | James (herramienta o cheat conocido) |
| `Debug` | Genérico — ventanas/procesos con "Debug" en el nombre |

Nota: el token `ida` detecta IDA Pro (tanto `ida.exe` como `ida64.exe`).
El `pizza` es un cheat conocido para Left 4 Dead 2.

---

## Glob Pattern de Archivos VPK

```
*.vpk
```

Encontrado directamente en el binario en la región de datos. El AC enumera todos los archivos
`.vpk` en el directorio de addons del juego y los reporta al servidor vía el mensaje `BiK8wj`
(campos Path y Size).

---

## Módulo WatchList — _6di6zc0se2v

El paquete `_6di6zc0se2v` implementa el mecanismo de vigilancia continua:

```go
// Funciones identificadas:
_6di6zc0se2v.(*_o3YyW).WatchList    // función principal de la lista de vigilancia
_6di6zc0se2v.(*_o3YyW).Add          // agregar item a la lista de vigilancia
_6di6zc0se2v.(*_o3YyW).AddWith      // agregar item con condición
_6di6zc0se2v.(*_o3YyW).Remove       // eliminar de la lista
_6di6zc0se2v.(*_o3YyW).Close        // cerrar el watcher
_6di6zc0se2v.(*Rs0QF1nUdhgw).Has    // verificar si un item está en la lista
_6di6zc0se2v.(*UWzc59).Has          // verificar (tipo alternativo)
```

5 funciones `init.func` (func1-func5) — posiblemente inicialización de 5 categorías distintas
de items vigilados (ventanas, procesos, módulos, etc.).

---

## Técnica Anti-Análisis Adicional: MustFindProcByOrdinal

El binario usa `MustFindProcByOrdinal` para cargar algunas APIs de Windows por número de
ordinal en lugar de nombre. Esto impide identificar qué APIs se llaman simplemente buscando
sus nombres como strings en el binario.

APIs cargadas por nombre (aún visibles): `VirtualQuery`, `BitBlt`, `PatBlt`, `GetDC`, `GetSystemInfo`
APIs cargadas por ordinal (sin strings visibles): `ReadProcessMemory`, `OpenProcess`,
`CreateToolhelp32Snapshot`, `EnumWindows`, `GetWindowTextW`, etc.

---

## Bypass de las Blacklists

### Para títulos de ventana (Blacklists 1, 4)
El chequeo es por substring en el título. Renombrar la ventana o usar la opción
de configuración del título en el debugger resuelve el problema en la mayoría de casos:
- `x32dbg`: Options → Preferences → GUI → cambiar título de ventana
- `ghidra`: renombrar en el launcher o modificar via agente Java
- `IDA Pro`: usar IDA con la ventana minimizada o renombrada

### Para procesos (Blacklistas 2, 3, 4)
El nombre del ejecutable es lo que se compara. Renombrar el binario evita la detección:
- `fiddler.exe` → `editor.exe`
- `cmd.exe` → bloqueado por nombre. PowerShell (`powershell.exe`) no está en la lista.
- `dnspy.exe` → `code_editor.exe`
- `ida64.exe` → `analyzer.exe`
- Discord: usar la versión web (discord.com) en vez del cliente de escritorio

### Nota sobre `pizza`
La detección de `pizza` por substring en ventanas/procesos sugiere que el AC
está específicamente adaptado para detectar cheats populares de la comunidad L4D2.
