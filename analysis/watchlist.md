# WatchList — Sistema de Vigilancia Continua

Análisis del paquete `_6di6zc0se2v` (pclntab) / `sNAkh4` (type table),
que implementa el framework de vigilancia en tiempo real del anticheat.

---

## Identificación del Paquete

| Atributo | Valor |
|----------|-------|
| Nombre en pclntab | `_6di6zc0se2v` |
| Nombre en type table | `sNAkh4` |
| Tipo principal | `(*_o3YyW)` — WatchList manager |
| Init functions | 5 (una por categoría de vigilancia) |
| Total entradas pclntab | ~81 |

**Relación con sNAkh4:** En Go, el nombre de paquete en pclntab usa el path
garbled (`_6di6zc0se2v`) mientras el type table usa el package name garbled
(`sNAkh4`). Son el mismo paquete.

---

## Arquitectura General

```
_6di6zc0se2v
│
├── (*_o3YyW)  — WatchList manager (tipo principal)
│   ├── Add(item)                  — agregar ítem a vigilar
│   ├── AddWith(item, condition)   — agregar con condición
│   ├── Remove(item)               — eliminar ítem de vigilancia
│   ├── Close()                    — cerrar el watcher
│   ├── WatchList()                — retornar lista actual
│   │
│   ├── Scan loops (goroutines internas):
│   │   ├── nmdo2n          — loop principal de detección (func1, func3, func3.1, func4)
│   │   ├── ndrA44yQD0P     — monitor de estado (func1, func2, func3)
│   │   ├── hZlzX3U7eol     — filtro/comparador (func1, func2)
│   │   ├── rUMKWF          — handler de resultados (func1, func2)
│   │   ├── qFouZjKyDKnV    — callback de detección (func1)
│   │   ├── zVVbDT          — sub-handler (func1)
│   │   └── dN6aH2PMMfF     — goroutine auxiliar (func1)
│   │
│   └── Métodos internos:
│       ├── _UzXx3OV        — método privado (deferwrap1)
│       ├── b_P3_VePi7cp    — callback/comparador
│       ├── bnqvQJcHVKc     — validador interno
│       ├── iLYWoih3UX      — helper de estado
│       ├── qiIoSb8WahJF    — handler interno
│       ├── wUhHB8          — cleanup/flush
│       ├── xtjCTplB        — serializer
│       └── zPSJUAu0        — estado/config
│
├── (*GrRzvM2)  — Tipo de nodo del WatchList
│   ├── Add(item)
│   └── Close()
│
├── (*Rs0QF1nUdhgw)  — Colección de patrones (tipo 1)
│   ├── Has(item) bool    — verificar si ítem está en la lista
│   └── String() string   — representación como string
│
├── (*UWzc59)           — Colección de patrones (tipo 2)
│   ├── Has(item) bool
│   └── String() string
│
└── Funciones de nivel superior:
    ├── l3ff5L9s97X5      — lanzador de goroutines (gowrap1)
    ├── cd1Utgxh0         — inicialización del monitor de red
    ├── encTsS0_          — serialización/codificación de datos de red
    ├── wqT1oO            — loop principal de captura de red
    │   ├── wqT1oO.b9dzb6EdryQ
    │   └── wqT1oO.jm773o2w
    ├── _CIWVBWa          — helper interno de inicialización
    ├── OjvNRj_ak         — función de análisis
    ├── f8y6H5            — función auxiliar
    └── xwIBKPDC2ao4      — función auxiliar
```

---

## Las 5 Categorías de Vigilancia (init.func1 — init.func5)

El paquete tiene exactamente 5 funciones de inicialización, cada una registrando
una categoría de vigilancia diferente:

| Init | Categoría | Descripción |
|------|-----------|-------------|
| `init.func1` | **Ventanas** | Monitoreo de títulos de ventana vía EnumWindows |
| `init.func2` | **Procesos** | Lista de procesos activos vía CreateToolhelp32Snapshot |
| `init.func3` + `func3.1` | **Red** | Captura de paquetes vía gopacket (cd1Utgxh0, wqT1oO) |
| `init.func4` + `func4.1` | **Módulos/DLLs** | DLLs cargadas en el proceso del juego |
| `init.func5` + `func5.1` | **Archivos/VPK** | Archivos del juego (VPK integrity check) |

---

## Colecciones de Patrones

### `(*Rs0QF1nUdhgw)` — Set de strings para matching

```
Rs0QF1nUdhgw.Has(item string) bool
Rs0QF1nUdhgw.String() string
  └── String.func1    — iterador interno
  └── String.func1.1  — sub-closure
  └── String.func2    — formateador
```

Probablemente contiene los tokens de las blacklists compiladas como un set
de alta eficiencia (O(1) lookup) en lugar de regex matching.

### `(*UWzc59)` — Segunda colección de patrones

```
UWzc59.Has(item string) bool
UWzc59.String() string
  └── String.func1
  └── String.func2
  └── String.func2.1
  └── String.func3
```

---

## Subsistema de Red (gopacket)

La categoría de red (`init.func3`) usa el paquete `q4ajG0RM` (gopacket)
para captura y análisis de paquetes de red:

```
cd1Utgxh0     — inicializar captura de red (abre interfaz de red)
wqT1oO        — loop principal de captura (lee paquetes continuamente)
  ├── wqT1oO.b9dzb6EdryQ  — procesar paquete individual
  └── wqT1oO.jm773o2w     — handler de error/timeout
encTsS0_      — codificar/serializar datos de paquete capturado
```

**Qué monitorea la red:**
- Conexiones del proceso `left4dead2.exe` a IPs no esperadas
- Presencia de proxies locales (127.0.0.1:puerto) en las conexiones del juego
- Tráfico de herramientas de análisis de red (Wireshark, Fiddler)
- Posiblemente: verificar que el juego se conecta al servidor correcto

---

## Flujo de Detección

```
WatchList._o3YyW (goroutine permanente)
│
├── nmdo2n (loop principal, cada ~5-10s)
│   │
│   ├── [Para ventanas] EnumWindows → GetWindowTextW
│   │   └── Rs0QF1nUdhgw.Has(título) → ¿match con blacklist?
│   │
│   ├── [Para procesos] CreateToolhelp32Snapshot
│   │   └── Rs0QF1nUdhgw.Has(nombre_proceso) → ¿match?
│   │
│   ├── [Para módulos] Module32First/Next en left4dead2.exe
│   │   └── Rs0QF1nUdhgw.Has(nombre_dll) → ¿DLL sospechosa?
│   │
│   └── Si match → qFouZjKyDKnV() → Feed(fcje4l4dl_uV)
│                                    └──> dUgTofmw (evaluación)
│
└── wqT1oO (loop de red, continuo)
    └── Captura paquetes → análisis → Feed si sospechoso
```

---

## Interfaz con dUgTofmw

Los resultados de detección fluyen a través de `fcje4l4dl_uV.Feed`:

```
_6di6zc0se2v.(*_o3YyW).qFouZjKyDKnV
    └──> fcje4l4dl_uV.Feed(detección)
         └──> dUgTofmw (motor de evaluación central)
              └──> Qi1Z8I (ban response via gRPC)
```

---

## APIs de Windows Usadas (Cargadas por Ordinal)

| API | Propósito |
|-----|-----------|
| `EnumWindows` | Enumerar todas las ventanas del sistema |
| `GetWindowTextW` | Leer título de cada ventana |
| `OpenProcess` | Obtener handle del proceso del juego |
| `ReadProcessMemory` | Leer memoria del proceso |
| `CreateToolhelp32Snapshot` | Snapshot de procesos/módulos/threads |
| `Module32First` / `Module32Next` | Iterar módulos cargados en el proceso |
| `Process32First` / `Process32Next` | Iterar procesos del sistema |

Todas estas APIs se cargan por ordinal (NO por nombre), haciendo que no
aparezcan como strings visibles en el binario. La función encargada es
`MustFindProcByOrdinal` del paquete `m7wGi4U_E` (`golang.org/x/sys/windows`).

---

## Bypass

### Renombrar herramientas (blacklists de proceso/ventana)
Ver `analysis/bypass_guide.md` para la lista completa.

### Ocultar DLLs inyectadas
Si se inyecta una DLL en el proceso del juego, el `Module32Next` del WatchList
la verá. Alternativas:
1. **Manual mapping**: Cargar la DLL sin usar `LoadLibrary` (no aparece en lista de módulos)
2. **Reflective DLL injection**: La DLL no registra su nombre en la lista de módulos

### Ocultar de EnumWindows
El WatchList llama `EnumWindows` para ver todas las ventanas. Opciones:
1. Crear la ventana de la herramienta sin título (título vacío)
2. Cambiar el título dinámicamente antes de que el scan ocurra
3. Hook de `EnumWindows` en el proceso del AC (muy difícil sin inyección en el AC)

### Aislar el entorno de análisis
La técnica más limpia: VM separada con acceso remoto (RDP/VNC).
El AC no puede ver las ventanas/procesos de otra máquina.
