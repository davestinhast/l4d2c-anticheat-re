# Motor de Detección Principal — Paquete dUgTofmw

El paquete `dUgTofmw` (nombre garblado) es el motor de detección central del AC.
Contiene la mayor concentración de lógica de análisis del sistema, con más de 50 funciones
identificadas y 12 goroutines de inicialización.

---

## Funciones Principales

### ga4oovjHCfg — Lanzador de Goroutines de Detección (37 closures totales)

La función más grande del paquete. Lanza **33 goroutines paralelas** (`func1` a `func33`)
más **4 sub-goroutines anidadas** — confirmado por extracción directa del `pclntab` del binario.

**Conteo exacto (confirmado por pclntab):**
- `func1` a `func33` → 33 goroutines de detección paralela
- `func9.1`, `func12.1`, `func21.1`, `func28.1` → 4 sub-goroutines anidadas
- **Total: 37 closures**

```
dUgTofmw.ga4oovjHCfg          — función principal (dispatcher)
dUgTofmw.ga4oovjHCfg.func1    — goroutine 1
dUgTofmw.ga4oovjHCfg.func2    — goroutine 2
...
dUgTofmw.ga4oovjHCfg.func9    — goroutine 9
dUgTofmw.ga4oovjHCfg.func9.1  — sub-goroutine de func9 (reporte/acción)
dUgTofmw.ga4oovjHCfg.func10   — goroutine 10
dUgTofmw.ga4oovjHCfg.func11   — goroutine 11
dUgTofmw.ga4oovjHCfg.func12   — goroutine 12
dUgTofmw.ga4oovjHCfg.func12.1 — sub-goroutine de func12 (reporte/acción)
...
dUgTofmw.ga4oovjHCfg.func21   — goroutine 21
dUgTofmw.ga4oovjHCfg.func21.1 — sub-goroutine de func21 (reporte/acción)
...
dUgTofmw.ga4oovjHCfg.func28   — goroutine 28
dUgTofmw.ga4oovjHCfg.func28.1 — sub-goroutine de func28 (reporte/acción)
dUgTofmw.ga4oovjHCfg.func29   — goroutine 29
...
dUgTofmw.ga4oovjHCfg.func33   — goroutine 33
```

Las 4 sub-goroutines anidadas (`.1`) están en goroutines específicas (9, 12, 21, 28),
lo que sugiere que esas 4 goroutines de detección son las que tienen lógica de dos fases:
la goroutine principal detecta, y la sub-goroutine ejecuta la acción resultante
(reporte al servidor, screenshot, desconexión).

### Patrón Productor-Consumidor — (*fcje4l4dl_uV).Feed

El motor de detección usa un patrón productor-consumidor identificado por el método `Feed`:

```
dUgTofmw.(*fcje4l4dl_uV).Feed   — método que alimenta datos al motor de detección
```

`fcje4l4dl_uV` es el tipo del motor/pipeline. El método `Feed` recibe datos de las
goroutines productoras (escaneo de ventanas, procesos, memoria) y los pasa a las
goroutines consumidoras (análisis, reporte). Este patrón explica por qué algunas
goroutines tienen sub-goroutines anidadas — la principal produce, la anidada consume.

### CGRKkahhRkP — Handler de Reportes / Heartbeat (12 sub-funciones + deferwrap)

```
dUgTofmw.CGRKkahhRkP             — función de reporte principal
dUgTofmw.CGRKkahhRkP.deferwrap1  — cleanup garantizado (defer)
dUgTofmw.CGRKkahhRkP.func1       — sub-handler 1
...
dUgTofmw.CGRKkahhRkP.func11.2.1  — sub-sub-sub-handler (lógica profunda)
dUgTofmw.CGRKkahhRkP.func12      — sub-handler 12
```

La presencia de `deferwrap1` indica que esta función usa `defer` para cleanup — apropiado
para una función que maneja conexiones de red y debe limpiar recursos al salir.

### KT300841 — Detección con Goroutine (17 sub-funciones + gowrap1)

```
dUgTofmw.KT300841          — función de detección
dUgTofmw.KT300841.gowrap1  — wrapper de goroutine
dUgTofmw.KT300841.func1    — ...
...
dUgTofmw.KT300841.func17   — ...
```

`gowrap1` es generado por garble para goroutines lanzadas con `go func()`. Su presencia
junto a 17 sub-funciones indica que esta función lanza una goroutine con lógica compleja.

### JcCuAB7zH — Detección Adicional (9 sub-funciones + gowrap1)

```
dUgTofmw.JcCuAB7zH          — función de detección
dUgTofmw.JcCuAB7zH.gowrap1  — wrapper de goroutine
dUgTofmw.JcCuAB7zH.func1    — ...
...
dUgTofmw.JcCuAB7zH.func8.1  — sub-goroutine de func8
dUgTofmw.JcCuAB7zH.func9    — ...
```

### ZXugk2GxaotX — Escaneo de Memoria (14 sub-funciones)

```
dUgTofmw.ZXugk2GxaotX          — escáner principal
dUgTofmw.ZXugk2GxaotX.func1    — ...
...
dUgTofmw.ZXugk2GxaotX.func8.1  — sub-goroutine (posiblemente el ReadProcessMemory loop)
...
dUgTofmw.ZXugk2GxaotX.func14   — ...
```

La sub-goroutine `.func8.1` sugiere que hay un loop interno de lectura de memoria
que corre como goroutine separada.

### MOKJ2g5p — Análisis Profundo (anidamiento extremo)

```
dUgTofmw.MOKJ2g5p.func3.1.1              — triple anidamiento
dUgTofmw.MOKJ2g5p.func3.1.1.deferwrap1  — defer dentro del triple anidamiento
dUgTofmw.MOKJ2g5p.func3.1.2
dUgTofmw.MOKJ2g5p.func3.1.2.1           — cuádruple anidamiento
```

El anidamiento cuádruple (`func3.1.2.1`) y el `deferwrap1` en el tercer nivel
indican que esta función hace algo que requiere cleanup garantizado en el contexto
más interno — probablemente apertura y cierre de handles de proceso (`OpenProcess` /
`CloseHandle`) con `defer CloseHandle(handle)`.

### RB_nRb — Análisis de Procesos (lógica profunda, 6 sub-funciones)

```
dUgTofmw.RB_nRb.func1.1      — sub-goroutine
dUgTofmw.RB_nRb.func1.2
dUgTofmw.RB_nRb.func1.2.1   — sub-sub
dUgTofmw.RB_nRb.func3.1
dUgTofmw.RB_nRb.func3.2.1   — sub-sub
dUgTofmw.RB_nRb.func3.4
```

### dfzoI5X7aj — Función de File Scanning (5 sub-funciones + deferwrap)

```
dUgTofmw.dfzoI5X7aj.deferwrap1  — cleanup garantizado
dUgTofmw.dfzoI5X7aj.func1       — enumeración de archivos
...
dUgTofmw.dfzoI5X7aj.func5
```

Candidata principal para la función que enumera los archivos `.vpk` del juego
(glob pattern `*.vpk`) y los reporta como `BiK8wj` (Path + Size) al servidor.

### JclT3ZyNnu — Procesamiento de Addons (2 sub-goroutines + deferwrap)

```
dUgTofmw.JclT3ZyNnu.deferwrap1
dUgTofmw.JclT3ZyNnu.func1
dUgTofmw.JclT3ZyNnu.func1.1   — sub-goroutine
dUgTofmw.JclT3ZyNnu.func2
dUgTofmw.JclT3ZyNnu.func2.1   — sub-goroutine
```

---

## Funciones de Soporte

```
dUgTofmw.gZDPQlzEDoPS.Error   — tipo de error personalizado del paquete
dUgTofmw.bXwPuh7bgI0           — función utilitaria interna
dUgTofmw.C7rmMxBQ              — función de comunicación
dUgTofmw.cafEKmBsI7W           — función de análisis
dUgTofmw.CaVnmKHua             — función de análisis
dUgTofmw.cK12Fi                — función utilitaria
dUgTofmw.Dll9_E_he             — función relacionada con DLL (análisis de DLLs?)
dUgTofmw.DMrazc6               — función de datos
dUgTofmw.dWiyVdNx2Fv           — función de análisis
dUgTofmw.EEnO9BVP              — función con deferwrap (manejo de recursos)
dUgTofmw.egmY80_               — función utilitaria
dUgTofmw.EM4NEpmhW             — función con 4 sub-goroutines
dUgTofmw.eOZqci                — función con 1 sub-goroutine
dUgTofmw.ETdFm1LN              — función de análisis
dUgTofmw.etEBMu                — función de análisis
dUgTofmw.Fa5wQ7bfv             — función con sub-goroutines anidadas (análisis profundo)
dUgTofmw.FmOyTPW               — función con 1 sub-goroutine
dUgTofmw.g8Qnj_3_S             — función utilitaria
dUgTofmw.GaGfiYt               — función de análisis
dUgTofmw.GnAAjxzf              — función de análisis
dUgTofmw.HWH9TFuU              — función con 5 sub-goroutines + deferwrap (HWID?)
dUgTofmw.ix23fWF1BKm           — función utilitaria
dUgTofmw.J7mNmNwGVIbG          — función con sub-goroutines anidadas
dUgTofmw.JBCw5jG               — función con deferwrap (manejo de recursos)
dUgTofmw.Jm9lCZuTayH           — función de análisis
dUgTofmw.jslJ644O              — función utilitaria
dUgTofmw.JsO_uXo               — función de análisis
dUgTofmw.JYekK0C               — función de análisis
dUgTofmw.Ku3NIva               — función con sub-goroutines (lógica de bXwPuh7bgI0)
dUgTofmw.lgKnH05Wp             — función utilitaria
dUgTofmw.Lxx8Q7N               — función con deferwrap
dUgTofmw.LzxyCO                — función con 2 sub-goroutines
dUgTofmw.MUbxHiHbA             — función de análisis
dUgTofmw.NVUoLGCRivS           — función con 4 sub-goroutines (análisis de procesos?)
dUgTofmw.P6t_tYOk5O8           — función con 4 sub-goroutines + deferwrap
dUgTofmw.pV5JbQn9WF            — función con sub-goroutines anidadas
dUgTofmw.pZV38F2F5p6H          — función de análisis
dUgTofmw.Q100bRnHXe            — función de análisis
dUgTofmw.rpfbMOh               — función con 4 sub-goroutines [CONFIRMADO: llama a WvPUk5UlmHG.(*USEy__R7Bor9).CmdlineSlice — lee la línea de comandos del proceso left4dead2.exe]
dUgTofmw.t5rfyIM               — función con 2 sub-goroutines
dUgTofmw.t6BMqE0               — función con 1 sub-goroutine
dUgTofmw.T7nZWPA_QXN           — función con 1 sub-goroutine
dUgTofmw.TmDL6w                — función con 2 sub-goroutines
dUgTofmw.tN6ha6DAaubp          — función de análisis
dUgTofmw.UbS8cVmNACG2          — función con 1 sub-goroutine
dUgTofmw.VA0jJhHuwb0l          — función con 3 sub-goroutines anidadas
dUgTofmw.wLSYYZ6n              — función con 1 sub-goroutine
dUgTofmw.wmP3Hc                — función utilitaria
dUgTofmw.WSRrqZ_Tt             — función de análisis
dUgTofmw.wUPsw0yucxFy          — función con 1 sub-goroutine
dUgTofmw.XKhwdFnJ              — función de análisis
dUgTofmw.Xl9KPzkO1h            — función de análisis
dUgTofmw.xT_i_Uv               — función de análisis
dUgTofmw.yjl0SSj               — función con 3 sub-goroutines anidadas
dUgTofmw.YjouPlOpm             — función con 2 sub-goroutines
dUgTofmw.Z2RUaMq               — función de análisis
```

---

## Inicialización (dUgTofmw.init)

El paquete tiene 12 funciones de inicialización:

```
dUgTofmw.init          — inicializador principal
dUgTofmw.init.func1    — ...
dUgTofmw.init.func2    — ...
...
dUgTofmw.init.func12   — ...
dUgTofmw.init.func12.1 — sub-init anidada
```

12 funciones de init sugieren que el paquete registra 12 componentes de detección
en el sistema de inicialización global de Go. Esto podría corresponder a:
- Registro de blacklists
- Inicialización de conexiones WMI
- Setup de hooks de Windows APIs
- Registro de goroutines de detección

---

## Técnica Anti-Análisis: MustFindProcByOrdinal

Encontrado en el binario: `MustFindProcByOrdinal`

El AC carga algunas APIs de Windows por número de ordinal en vez de nombre. Esto hace
que los nombres de API (ReadProcessMemory, OpenProcess, etc.) no aparezcan como strings
en el binario, dificultando el análisis estático.

APIs que no se encontraron como strings (probablemente cargadas por ordinal):
```
ReadProcessMemory
OpenProcess
CreateToolhelp32Snapshot
Process32First / Process32Next
Module32First / Module32Next
EnumWindows
GetWindowTextW
DeviceIoControl
```

---

## Construcción del HWID — HWARRxN.KbBU6bdOa.Build

El HWID se construye en el paquete `HWARRxN` con la función `KbBU6bdOa.Build`.

Las consultas WMI para cada componente usan `IWbemServices::ExecQuery` (COM).
El hook de intercepción para el bypass debe instalarse en el método `ExecQuery` de la
vtable de `IWbemServices` (índice 20 en la vtable).

```cpp
// Offset de ExecQuery en la vtable de IWbemServices
void** vtable = *(void***)pServices;
ExecQuery_original = (ExecQuery_t)vtable[20];
vtable[20] = (void*)ExecQuery_hook;
```

Las clases WMI consultadas:
```
Win32_Processor       — ProcessorId (CPU serial)
Win32_DiskDrive       — SerialNumber, Model
Win32_BaseBoard       — SerialNumber, Model (motherboard)
Win32_VideoController — PNPDeviceID, Name (GPU)
Win32_OperatingSystem — BuildNumber, Version
Win32_Product         — InstallDate de Steam (anti-smurf)
```

---

## Búsqueda en Ghidra

Para analizar `dUgTofmw` en Ghidra (después de aplicar GoReSym):

1. Buscar la función `dUgTofmw.ga4oovjHCfg` — es la más grande y el punto de entrada
2. Identificar los 33 llamados a `runtime.newproc` — cada uno es una goroutine de detección
3. Para cada goroutine: ver qué APIs de Windows llama
4. Las funciones con `deferwrap1` son buenas candidatas para el análisis de manejo de handles

```
# En Ghidra: Symbol Table → filtrar por "dUgTofmw"
# Ordenar por tamaño descendente para ver las funciones más complejas primero
```
