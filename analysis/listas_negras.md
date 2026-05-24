# Listas Negras — Procesos y Títulos de Ventana

El AC mantiene dos blacklists separadas para detectar herramientas de análisis, debugging y cheating.

---

## Blacklist 1 — Títulos de Ventana (EnumWindows)

String combinada encontrada en el binario:

```
commonaddonsx32dbgpc-retCentoswindbgdbgclrde4dotpepperghidrahackerx96dbgfoldersysmontimersefenceselectscalar
```

Desglosada en tokens individuales:

| Token | Herramienta |
|-------|-------------|
| `commonaddons` | Referencia a addons comunes (uso incierto) |
| `x32dbg` | x32dbg — debugger para procesos 32-bit |
| `pc-ret` | PC Ret (herramienta de análisis) |
| `Centos` | CentOS (posible detección de VM Linux) |
| `windbg` | WinDbg — debugger de Microsoft |
| `dbgclr` | DbgCLR — debugger de runtime .NET |
| `de4dot` | de4dot — deobfuscador de .NET |
| `pepper` | Pepper Flash / Chromium component |
| `ghidra` | Ghidra — framework de RE de la NSA |
| `hacker` | Ventanas con "hacker" en el título |
| `x96dbg` | Variante de x64dbg/x96dbg |
| `folder` | Posiblemente "SteamHacker" u otras tools |
| `sysmon` | Sysinternals SysMon |
| `timers` | Herramienta de timers/profiling |
| `efence` | Electric Fence (memory debugger) |
| `selectscalar` | Posiblemente herramienta de análisis de memoria |

El AC usa `EnumWindows` + `GetWindowTextW` para iterar todas las ventanas del sistema y compara el título contra estos tokens (probablemente con `strings.Contains` o similar, case-insensitive).

---

## Blacklist 2 — Nombres de Procesos (Process List)

Esta lista se chequea contra los procesos en ejecución (via `Win32_Process` WMI o `CreateToolhelp32Snapshot`):

```
gdb.exe         reverse         processharp     od
fiddler         x64_dbg         petools         monitor
ollydbg         wpe pro         PhantOm         x32_dbg
phantom         WPE PRO         charles         checker
harmony         PETools         sniffer         MDBCrew
DIEmW           inMonitor       Discord         Opera
cmd.exe         fdm.exe         zen.exe         Arc.exe
```

### Categorías de Procesos Bloqueados

**Debuggers:**
- `gdb.exe` — GNU Debugger
- `ollydbg` — OllyDbg (popular RE tool)
- `x64_dbg` / `x32_dbg` — x64dbg/x32dbg

**Proxies y captura de red:**
- `fiddler` — Fiddler (HTTP proxy)
- `wpe pro` / `WPE PRO` — Winsock Packet Editor
- `charles` — Charles proxy
- `sniffer` — genérico

**Herramientas PE/RE:**
- `petools` / `PETools` — PE Tools
- `processharp` — ProcessSharp
- `DIEmW` — Detect-It-Easy (herramienta de identificación de binarios)

**Monitores:**
- `monitor` / `inMonitor` — herramientas de monitoreo
- `checker` — genérico

**Navegadores/Apps bloqueadas:**
- `Discord` — bloqueado por riesgo de overlay (puede superponer sobre el juego)
- `Opera` — bloqueado por riesgo de overlay
- `Arc.exe` — Arc Browser (overlay)
- `zen.exe` — Zen Browser
- `fdm.exe` — Free Download Manager

**Herramientas del sistema:**
- `cmd.exe` — Command Prompt (bloqueado mientras el AC está activo)
- `od` — od (octal dump, Unix tool)
- `reverse` — genérico para herramientas de reversing

---

## Módulo WatchList — _6di6zc0se2v

El paquete `_6di6zc0se2v` implementa el mecanismo de vigilancia:

```go
// Funciones identificadas:
_6di6zc0se2v.(*_o3YyW).WatchList    // función principal de la lista de vigilancia
_6di6zc0se2v.(*_o3YyW).Add          // agregar item a la lista
_6di6zc0se2v.(*_o3YyW).AddWith      // agregar con condición
_6di6zc0se2v.(*_o3YyW).Remove       // eliminar de la lista
_6di6zc0se2v.(*_o3YyW).Close        // cerrar el watcher
_6di6zc0se2v.(*Rs0QF1nUdhgw).Has    // verificar si un item está en la lista
_6di6zc0se2v.(*UWzc59).Has          // verificar (tipo alternativo)
```

El struct `Rs0QF1nUdhgw` y `UWzc59` son los tipos de "item vigilado" — tienen un método `Has` que devuelve bool, indicando si el proceso/ventana detectado está en la blacklist.

Hay 5 funciones `init.func` (func1 a func5) — posiblemente inicialización de 5 categorías diferentes de items vigilados.

---

## Bypass de las Blacklists

### Para títulos de ventana
El chequeo es por substring (los tokens son substrings a buscar en el título). Renombrar la ventana de la herramienta es suficiente en muchos casos. Si el debugger permite configurar el título de ventana, cambiarlo resuelve el problema.

Para `x32dbg` → cambiar el título de la ventana a cualquier cosa que no contenga "x32dbg".

### Para procesos
El nombre del proceso es lo que se compara. Renombrar el ejecutable evita la detección:
- `fiddler.exe` → `editor.exe`
- `cmd.exe` → bloqueado por nombre exacto, usar PowerShell (no está en lista)
- Discord: usar la versión web en vez del cliente de escritorio

### Nota sobre cmd.exe
`cmd.exe` está bloqueado. PowerShell (`powershell.exe`) no aparece en la lista — funciona como alternativa.

### Nota sobre Discord
Discord está bloqueado probablemente por el overlay de Discord Game SDK, que inyecta código en el proceso del juego. Deshabilitar el overlay de Discord y ejecutarlo debería funcionar en versiones más nuevas del AC (sin embargo la lista chequea el nombre del proceso).
