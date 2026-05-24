# Escáner de Blacklists — Paquete bszAWJqu

Análisis completo del subsistema de escaneo de listas negras del anticheat.
Este paquete implementa el motor que compara procesos activos, títulos de ventana
y módulos cargados contra las cuatro blacklists documentadas en `listas_negras.md`.

---

## Arquitectura General

El paquete `bszAWJqu` opera como el **escáner activo** del anticheat. Su responsabilidad
es enumerar los recursos del sistema (ventanas, procesos, módulos) y evaluarlos contra
patrones regex compilados desde las blacklists. Los resultados de detección se acumulan
en el pipeline de `dUgTofmw` vía el tipo `fcje4l4dl_uV` (método `Feed`).

```
bszAWJqu
│
├── Compila las 4 blacklists en patrones regex (al init)
│   ├── MlPMSGX2GAB  → Blacklist 1 (títulos de ventana, 16 tokens de 6 chars)
│   ├── Alav9s       → Blacklist 2 (nombres de proceso, 30 entries)
│   ├── IvOAAX       → Blacklist 3 (RE tools cortas: .vpk, dump, peek, ...)
│   └── T8Go8QdJaY   → Blacklist 4 (RE tools 5-char: dnspy, ilspy, pizza, ...)
│
└── Lanza goroutines de escaneo (vía LUb2wVKw.gowrap1)
    ├── FkjkBraZo   → Escaneo de ventanas (EnumWindows + GetWindowTextW)
    ├── X10KDNt6j   → Escaneo de procesos (CreateToolhelp32Snapshot)
    ├── PlRmoX6cuU  → Escaneo de módulos/DLLs del juego
    ├── JnwG33U     → Escaneo adicional (posiblemente archivos/paths)
    └── X80N6Rs52TTi → Escaneo tipo desconocido (8 closures)
```

---

## Funciones Top-Level (12 funciones identificadas en pclntab)

| Función | Closures | Propósito Inferido |
|---------|----------|-------------------|
| `ThNOEr` | func1-func4 | Coordinador / orquestador del escaneo |
| `OYQujSa6Aed5` | func1 | Inicialización o reporte |
| `FkjkBraZo` | 15 closures + deferwrap | **Escáner de ventanas** |
| `IvOAAX` | — (función simple) | Función de matching compartida |
| `ApxFkk` | func1 | Helper de comparación |
| `X10KDNt6j` | 10+ closures + deferwrap | **Escáner de procesos** |
| `LUb2wVKw` | func1, func2, func3, gowrap1 | Lanzador de goroutines |
| `PlRmoX6cuU` | 10+ closures + deferwrap | **Escáner de módulos** |
| `CGKuSNjp` | func1 | Helper de matching |
| `JnwG33U` | 9+ closures + deferwrap | **Escáner adicional** (archivos/VPKs?) |
| `E5nzMUH5qSwM` | func1, func2 | Procesador de resultados |
| `X80N6Rs52TTi` | func1-func8 + deferwrap | Escáner adicional (tipo desconocido) |

---

## Patrón de Closures — Cómo se Aplican las 4 Blacklists

Cada función de escaneo principal (`FkjkBraZo`, `X10KDNt6j`, `PlRmoX6cuU`, `JnwG33U`)
tiene el mismo patrón de closures al final de su secuencia:

```
[NombreFunción].MlPMSGX2GAB.func11    ← aplica Blacklist 1 (ventanas)
[NombreFunción].Alav9s.func12          ← aplica Blacklist 2 (procesos)
[NombreFunción].Alav9s.func12.1        ← sub-closure de Blacklist 2
[NombreFunción].IvOAAX.func13          ← aplica Blacklist 3 (RE tools cortas)
[NombreFunción].IvOAAX.func13.1        ← sub-closure de Blacklist 3
[NombreFunción].T8Go8QdJaY.func14      ← aplica Blacklist 4 (RE tools 5-char)
[NombreFunción].T8Go8QdJaY.func14.1    ← sub-closure de Blacklist 4
```

Esto significa que **cada escáner aplica las 4 blacklists en secuencia** sobre su
objetivo (ventana, proceso, módulo). El resultado es que una misma herramienta puede
ser detectada tanto por su nombre de ventana como por su nombre de proceso.

---

## Funciones Regex del Paquete ncRaYk_Ke

El paquete `ncRaYk_Ke` (= `regexp` + `regexp/syntax`) expone las siguientes funciones
que `bszAWJqu` usa para compilar y ejecutar los patrones:

| Función ncRaYk_Ke | Rol Probable |
|-------------------|-------------|
| `MlPMSGX2GAB` | Compilar regex de Blacklist 1 o hacer MatchString |
| `Alav9s` | Compilar regex de Blacklist 2 o hacer MatchString |
| `IvOAAX` | Compilar regex de Blacklist 3 o hacer MatchString |
| `T8Go8QdJaY` | Compilar regex de Blacklist 4 o hacer MatchString |
| `PJRLzfZ` | Función de matching base (MatchRune) |
| `FS0IhhJL` | Compilar patrón combinado |
| `MDfyjAe` | Operación sobre regexp compilado |
| `RaR0kkqIl` | Compilación o ejecución de regex |

El patrón compilado para cada blacklist es probablemente del tipo:
```regexp
(?i)(x32dbg|pc-ret|Centos|windbg|dbgclr|de4dot|pepper|ghidra|hacker|x96dbg|folder|sysmon|timers|efence|select|scalar)
```

Con la flag `(?i)` para case-insensitive matching. El matching es por **substring**
(no anchored), lo que significa que la herramienta es detectada si el patrón aparece
en cualquier parte del título/nombre.

---

## Relación con dUgTofmw

El paquete `bszAWJqu` es llamado desde `dUgTofmw`:

```
dUgTofmw.ga4oovjHCfg (lanzador de 37 goroutines)
├── goroutine N → bszAWJqu.ThNOEr (coordinador)
│   └── bszAWJqu.LUb2wVKw.gowrap1 → lanza goroutines de escaneo
│       ├── bszAWJqu.FkjkBraZo  → escaneo de ventanas
│       ├── bszAWJqu.X10KDNt6j  → escaneo de procesos
│       ├── bszAWJqu.PlRmoX6cuU → escaneo de módulos
│       └── bszAWJqu.JnwG33U    → escaneo adicional
│
bszAWJqu.IvOAAX aparece en contexto de FrontendReady → también inicia escáner
                                                         cuando la UI está lista
```

Los resultados de bszAWJqu → `fcje4l4dl_uV.Feed` → `dUgTofmw` (evaluación central)

---

## WMI Classes Confirmadas (contexto bszAWJqu / asYMlWeBL6f6)

Clases WMI encontradas en el binario como strings visibles:

| Clase WMI | Uso |
|-----------|-----|
| `Win32_Process` | Lista de procesos del sistema (escaneo de procesos) |
| `Win32_Processor` | Info del CPU (HWID) |
| `Win32_DiskDrive` | Info de disco (HWID) |
| `Win32_BaseBoard` | Info de motherboard (HWID) |
| `Win32_OperatingSystem` | Versión del OS, fecha de instalación (anti-smurf) |
| `Win32_VideoController` | Info de GPU (HWID) |
| `Win32_PerfFormattedData_PerfOS_System` | Contadores de rendimiento del OS — detecta carga anómala de CPU/context switches |

**Nota:** `Win32_PerfFormattedData_PerfOS_System` puede retornar:
- `ProcessorQueueLength` — número de threads esperando CPU (alto con debuggers activos)
- `ContextSwitchesPersec` — context switches por segundo del sistema
- `SystemCallsPersec` — system calls por segundo

El AC posiblemente usa esto como vector anti-debugger adicional.

---

## Técnica Anti-Análisis: MustFindProcByOrdinal

Las APIs de Windows más sensibles del escáner son cargadas por ordinal (no por nombre):
- `EnumWindows` — no visible como string en el binario
- `GetWindowTextW` — no visible
- `OpenProcess` — no visible
- `ReadProcessMemory` — no visible
- `CreateToolhelp32Snapshot` — no visible

Las APIs visibles por nombre (PE import table de kernel32.dll) no incluyen las de detección:
```
kernel32.dll: WriteFile, WaitForMultipleObjects, VirtualQuery, VirtualAlloc,
              TlsAlloc, SuspendThread, ResumeThread, GetSystemInfo,
              GetProcessAffinityMask, CreateThread, CloseHandle, ...
```

---

## Timing del Escaneo

Basado en la arquitectura de goroutines:
- El escaneo de ventanas y procesos corre en loop continuo cada ~5-10 segundos
- El escaneo de módulos es event-based (se activa cuando un nuevo DLL es cargado)
- `bszAWJqu.IvOAAX` también corre al inicio (FrontendReady) para check inmediato

---

## Bypass

Ver `analysis/bypass_guide.md` para estrategias detalladas. En resumen:

### Renombrar ejecutables (Blacklist 2 — procesos)
```
fiddler.exe    → editor.exe
x64dbg.exe     → analyzer.exe
dnspy.exe      → code_editor.exe
```

### Renombrar ventanas (Blacklist 1 — títulos)
- x32dbg: Options → Preferences → GUI → cambiar título de ventana
- Ghidra: launcher con `-DGhidra.window.title=MiTítulo`
- IDA Pro: el patrón es `ida -` — cambiar el título de la ventana vía script IDA

### Técnicas avanzadas
1. Mover herramientas de análisis a una VM separada con Remote Desktop
2. Usar herramientas de análisis remotas (debugging kernel vía WinDbg kernel debugger + cable)
3. Patch del AC: hookar `EnumWindows` / `CreateToolhelp32Snapshot` para retornar
   listas filtradas sin las herramientas de análisis
