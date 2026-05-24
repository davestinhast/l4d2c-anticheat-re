# Scanner de Blacklist — Paquete bszAWJqu

Documentación completa del paquete `bszAWJqu` — el scanner de blacklists que
monitorea ventanas, procesos, módulos DLL, archivos y red contra las listas negras.

---

## Identificación

| Atributo | Valor |
|----------|-------|
| Nombre garbled | `bszAWJqu` |
| Función | Scanner de blacklists multi-categoría |
| Total funciones | 120 |
| Integra con | `ncRaYk_Ke` (regexp) para matching |

---

## Arquitectura

El paquete tiene **5 funciones scanner principales**, cada una implementando
el monitoreo de una categoría de datos:

```
bszAWJqu
│
├── FkjkBraZo   — Scanner de ventanas (EnumWindows)
├── JnwG33U     — Scanner de procesos (Toolhelp32Snapshot)
├── PlRmoX6cuU  — Scanner de módulos DLL (Module32First/Next)
├── X10KDNt6j   — Scanner de archivos/VPK
└── X80N6Rs52TTi — Scanner de red (conexiones de proceso)
```

---

## Patrones Regex Compilados

`ncRaYk_Ke` (= `regexp` stdlib garbled) contiene las siguientes funciones de
matching, cada una es un `*regexp.Regexp` compilado con una blacklist diferente:

| Nombre garbled | Descripción probable |
|----------------|---------------------|
| `Alav9s` | Blacklist 1 — herramientas de debugging (x32dbg, windbg, etc.) |
| `IvOAAX` | Blacklist 2 — herramientas de análisis (Ghidra, Fiddler, etc.) |
| `MDfyjAe` | Blacklist especial — usada por QSUMsCa (VPK scanner) |
| `MlPMSGX2GAB` | Blacklist 3 — strings de dump/peek |
| `PJRLzfZ` | Blacklist 4 — strings de crack/hack |
| `RaR0kkqIl` | Blacklist especial — usada por QSUMsCa (VPK scanner) |
| `T8Go8QdJaY` | Blacklist 5 — herramientas RE (dnspy, ilspy, ida, etc.) |
| `FS0IhhJL` | Función utilitaria del regexp package |

**Cada scanner aplica los 5 regexes** contra su tipo de dato:

```go
// Patrón conceptual de cada scanner:
func FkjkBraZo(ctx context.Context, feed *Feed) {
    for {
        EnumWindows(func(hwnd) {
            title := GetWindowTextW(hwnd)
            if Alav9s.MatchString(title) || IvOAAX.MatchString(title) ||
               MlPMSGX2GAB.MatchString(title) || PJRLzfZ.MatchString(title) ||
               T8Go8QdJaY.MatchString(title) {
                feed.Feed(DetectionEvent{Title: title, Type: Window})
            }
        })
        time.Sleep(5-10 * time.Second)
    }
}
```

---

## Detalle de Cada Scanner

### `FkjkBraZo` — Scanner de Ventanas (10 closures)

```
FkjkBraZo
├── deferwrap1              — cleanup en panic
├── func1 – func10          — loop principal + sub-etapas
│   ├── Alav9s.func12       — aplicar regex 1 (herramientas debug)
│   │   └── Alav9s.func12.1 — sub-closure del regex 1
│   ├── IvOAAX.func13       — aplicar regex 2 (análisis)
│   │   └── IvOAAX.func13.1
│   ├── MlPMSGX2GAB.func11  — aplicar regex 3 (dump/peek)
│   ├── PJRLzfZ.func15      — aplicar regex 4 (crack/hack)
│   └── T8Go8QdJaY.func14   — aplicar regex 5 (RE tools)
│       └── T8Go8QdJaY.func14.1
└── [→] dUgTofmw.Feed()     — reportar detección
```

**API Windows usada:** `EnumWindows` + `GetWindowTextW` (cargadas por ordinal)

---

### `JnwG33U` — Scanner de Procesos (10 closures)

```
JnwG33U
├── deferwrap1
├── func1 – func10
│   ├── Alav9s.func12, func12.1
│   ├── IvOAAX.func13, func13.1
│   ├── MlPMSGX2GAB.func11
│   ├── PJRLzfZ.func15
│   └── T8Go8QdJaY.func14, func14.1
└── [→] dUgTofmw.Feed()
```

**API Windows usada:** `CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS)` + `Process32Next`

---

### `PlRmoX6cuU` — Scanner de Módulos DLL (10 closures)

```
PlRmoX6cuU
├── deferwrap1
├── func1 – func10
│   ├── func1.1              — sub-closure
│   ├── Alav9s.func12, func12.1
│   ├── IvOAAX.func13, func13.1
│   ├── MlPMSGX2GAB.func11
│   ├── PJRLzfZ.func15
│   └── T8Go8QdJaY.func14, func14.1
└── [→] dUgTofmw.Feed()
```

**API Windows usada:** `CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid_juego)` + `Module32Next`

---

### `X10KDNt6j` — Scanner de Archivos (10 closures)

```
X10KDNt6j
├── deferwrap1
├── func1 – func10
│   ├── func2.1              — sub-closure
│   ├── Alav9s.func12, func12.1
│   ├── IvOAAX.func13, func13.1
│   ├── MlPMSGX2GAB.func11
│   ├── PJRLzfZ.func15
│   └── T8Go8QdJaY.func14, func14.1
└── [→] dUgTofmw.Feed()
```

**Escanea:** Archivos `.vpk` y posiblemente archivos de configuración del juego.

---

### `X80N6Rs52TTi` — Scanner de Red (9 closures)

```
X80N6Rs52TTi
├── deferwrap1
├── func1 – func8
│   └── IvOAAX.func9        — solo regex 2 (análisis de red)
│       └── IvOAAX.func9.1
└── [→] dUgTofmw.Feed()
```

**Diferencia notable:** Solo usa `IvOAAX` (1 regex) en lugar de los 5 regexes.
El scanner de red busca patrones específicos en las conexiones del proceso del juego.

**Funcionalidad probable:** Detectar proxies (`127.0.0.1:8080`), conexiones a IPs
de análisis/debug, o presencia de herramientas de captura de red activas.

---

## Funciones Auxiliares

```
ApxFkk          — inicializador / launcher (func1)
CGKuSNjp        — handler de detección (func1)
E5nzMUH5qSwM    — función de config (func1, func2)
IvOAAX          — regex 2 como función standalone
LUb2wVKw        — función de monitoreo adicional
  ├── func1, func2, func3  — etapas
  ├── func3.deferwrap1     — cleanup
  ├── func3.K10VxSXXTC.1   — goroutine interna nombrada
  └── gowrap1              — goroutine wrapper
OYQujSa6Aed5    — función auxiliar (func1)
ThNOEr          — función de control
  ├── func1, func2, func3, func4
  └── func2.1
X80N6Rs52TTi    — ver arriba
```

---

## Relación con la Blacklist de Strings

Ver `analysis/listas_negras.md` para las strings exactas de cada blacklist.

Mapeo tentativo (basado en el orden de aparición en el binario):

| Regex | Blacklist ID | Strings contenidas |
|-------|-------------|---------------------|
| `Alav9s` | Blacklist 1 | `x32dbg`, `pc-ret`, `centos`, `windbg`, `dbgclr`, `de4dot`, `pepper`, `ghidra`, `hacker` |
| `IvOAAX` | Blacklist 2 | `fiddler`, `ollydbg`, `discord`, `opera`, `steam`, `chrome`... (procesos comunes) |
| `MlPMSGX2GAB` | Blacklist 3 | `.vpk`, `dump`, `peek`, `kgdb`, `mdbg`, `xdeb` |
| `PJRLzfZ` | Blacklist 4 | `start`, `dnspy`, `ilspy`, `ILSpy`, `pizza`, `crack`, `ida`, `-brute`, `james`, `Debug` |
| `T8Go8QdJaY` | Blacklist 5 | No identificados completamente |

---

## Flujo de Detección Completo

```
bszAWJqu (cada scanner en goroutine separada, loop ~5-10s)
│
├── FkjkBraZo  ──→ EnumWindows → match regex → Feed()
├── JnwG33U    ──→ Process32Next → match regex → Feed()
├── PlRmoX6cuU ──→ Module32Next → match regex → Feed()
├── X10KDNt6j  ──→ filepath.Walk → match regex → Feed()
└── X80N6Rs52TTi ─→ net.Connections → match IvOAAX → Feed()
       │
       ▼
dUgTofmw.(*fcje4l4dl_uV).Feed(DetectionEvent)
       │
       ▼
dUgTofmw.(*ZhEEW_uzlNn).h7MNIMO (evaluación)
       │
       ▼
CGRKkahhRkP → gRPC → servidor → ban
```

---

## Notas para Bypass

Los 5 scanners corren **en paralelo** en goroutines separadas.
Cada uno aplica 5 regexes independientes.

- **Ventanas y procesos**: Renombrar las herramientas para que no hagan match
  con ninguno de los 5 regexes (ver bypass_guide.md)
- **Módulos DLL**: No inyectar DLLs con nombres conocidos. Usar manual mapping.
- **Archivos**: No tener VPKs de cheats en el directorio de addons
- **Red**: El scanner de red solo usa 1 regex — principalmente para detectar proxies
