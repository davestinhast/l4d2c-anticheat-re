# Pipeline de Recolección de HWID — Análisis Completo

Documentación del flujo completo de recolección de hardware fingerprint (HWID)
del anticheat L4D2Center. Este pipeline es el responsable de generar el
identificador único de la máquina que se envía al servidor para ban por HWID.

---

## Arquitectura del Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                     PIPELINE DE HWID                                │
│                                                                     │
│  asYMlWeBL6f6          A39Z4i              FXWqsvy_                │
│  (WMI Wrapper)  ──►   (WMI Executor)  ──►  (HWID Collector)       │
│  IWbemServices         GUxA0QN3             Z6ey1EJD               │
│  ExecQuery             aSy1SLV              11 closures             │
│                                                                     │
│                              ▼                                      │
│                         BhCuafOD                                   │
│                        (Aggregator)                                 │
│                        15 funciones                                 │
│                                                                     │
│                              ▼                                      │
│                        dUgTofmw                                    │
│                    (Evaluador Central)                              │
│                       .Feed(evento)                                 │
│                                                                     │
│                              ▼                                      │
│                        xzFpaM3Mq3                                  │
│                       (gRPC Client)                                 │
│                   BEt_icchsrxy.HWData                              │
│                                                                     │
│                              ▼                                      │
│                    l4d2center.com:443                               │
│                    (Servidor AC)                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Paquete 1: `asYMlWeBL6f6` — Wrapper WMI (COM/DCOM)

| Atributo | Valor |
|----------|-------|
| Funciones | 31+ (init func1-func24 con sub-closures) |
| Rol | Inicializar COM/DCOM, abrir conexión WMI |
| Tipo principal | `WbemeDk0` = IWbemServices |
| Enumerador | `VDjfdjV` = IEnumWbemClassObject |
| Función query | `Aego3YEweIaU` = ExecQuery |

El paquete `asYMlWeBL6f6` es el wrapper de alto nivel para WMI.
Inicializa COM con `CoInitializeEx`, conecta a `root\cimv2` con
`ConnectServer`, y expone la interfaz `IWbemServices` al resto del AC.

---

## Paquete 2: `A39Z4i` — Ejecutor de Queries WMI

| Atributo | Valor |
|----------|-------|
| Funciones | 8 |
| Rol | Ejecutar queries WQL y recuperar resultados |
| Tipo query | `GUxA0QN3` = tipo de resultado WMI |
| Función principal | `aSy1SLV` (con deferwrap1 = COM release) |
| Helpers | `iZg8eg6m`, `Jk1qZQ04` |
| Init closures | 2 (`init.func1`, `init.func2`) |

`A39Z4i` recibe un query WQL (`SELECT * FROM Win32_DiskDrive`, etc.),
llama a `IWbemServices.ExecQuery`, itera los resultados con
`IEnumWbemClassObject.Next`, y devuelve los valores al caller.

El 4 COM releases (deferwrap1 en `aSy1SLV`) corresponde al patrón estándar:
1. `Release()` en IEnumWbemClassObject al terminar
2. `Release()` en IWbemClassObject por cada item
3. `Release()` en IWbemServices al cerrar
4. `CoUninitialize()` al salir

---

## Paquete 3: `FXWqsvy_` — Pipeline de Recolección HWID

| Atributo | Valor |
|----------|-------|
| Funciones | 38 |
| Función clave | `Z6ey1EJD` — 11 sub-closures |
| Tipos | `IwUTVxn`, `YRNNn8` (ambos comparable = map keys) |
| Rol | Orquestar los 11 colectores de hardware |

### La Función `Z6ey1EJD` — 11 Colectores de Hardware

Cada `Z6ey1EJD.funcN` recolecta un componente de hardware diferente:

```
Z6ey1EJD
│
├── func1        — Recolector #1  (probablemente CPU serial/ID)
├── func2        — Recolector #2  (probablemente Disk serial)
│   └── func2.1  — Sub-closure de disk
├── func3        — Recolector #3  (probablemente GPU)
│   └── func3.1  — Sub-closure de GPU
├── func4        — Recolector #4  (probablemente MotherBoard)
├── func5        — Recolector #5  (probablemente OS InstallDate)
├── func6        — Recolector #6  (probablemente NIC MAC address)
├── func7        — Recolector #7  (probablemente Windows product key hash)
├── func8        — Recolector #8  (probablemente registry MachineGuid)
├── func9        — Recolector #9  (probablemente BIOS serial)
├── func10       — Recolector #10 (probablemente Volume serial number)
└── func11       — Recolector #11 (probablemente Steam/SteamID hash)
```

Correspondencia con los campos del protobuf `ENOV9d`:
```protobuf
message ENOV9d {
    bool Success = 1;
    string OS = 2;       // Z6ey1EJD.func5 (InstallDate) o func8 (MachineGuid)
    string CPU = 3;      // Z6ey1EJD.func1
    string Disk = 4;     // Z6ey1EJD.func2
    string MotherB = 5;  // Z6ey1EJD.func4
    string GPU = 6;      // Z6ey1EJD.func3
    // + 5 campos adicionales no documentados en protobuf = func6-func11
}
```

### Otros Componentes de `FXWqsvy_`

| Función | Rol Probable |
|---------|-------------|
| `ZJZaSq7xmKez` | Wrapper principal con COM release (`deferwrap1`) |
| `MSffIimik` (4 closures) | Procesamiento multi-etapa de resultados WMI |
| `Cp0zUb6UdSs` (2 closures) | Combinar/normalizar valores de hardware |
| `A6hQnfaeFT2` (2 closures) | Post-procesamiento o encoding del HWID |
| `NkpHEP` (1 closure) | Helper de validación |
| `IwUTVxn`, `YRNNn8` | Structs de resultado (usados como map keys) |
| `AaPVLqQV6w`, `xxU6vhj6` | Funciones auxiliares de formato |
| `KKyCdJjT` | Helper interno |

---

## Paquete 4: `BhCuafOD` — Agregador de Datos

| Atributo | Valor |
|----------|-------|
| Funciones | 15 |
| Cross-refs a dUgTofmw | **28 veces** |
| Rol | Agregar datos HWID y alimentar el evaluador central |
| Funciones | `AkMM0bTM`, `aQsiH5dnHwxr`, `bB_ngmD5lmc`, `Cw5JcgNZgcS`, `dDCmrLax3`, `gKzfRwK`, `gphmRM`, `iOcatHKW`, `jHhAM0WBlR`, `jm8y1XbzXdJn`, `kmLjphab`, `kRBZ25`, `oagZnvcfRd`, `p23w3bgd`, `xVtZ7aRbM` |

`BhCuafOD` aparece entre `FXWqsvy_` y `AnNE8Dn7`/`Plcr2cx` en pclntab,
y tiene 28 referencias directas a `dUgTofmw` — el número más alto de
cualquier paquete externo excepto el propio código del AC.

Su rol probable es:
1. Recibir los componentes HWID recolectados por `FXWqsvy_`
2. Combinarlos en el struct `ENOV9d` del protobuf
3. Serializar y alimentar al evaluador central `dUgTofmw`

---

## Paquete Destino: `dUgTofmw` — Evaluador Central

El evaluador central recibe el HWID procesado a través de su método
`(*fcje4l4dl_uV).Feed(evento)` y lo incorpora al pipeline de evaluación
central para:
1. Comparar contra lista de HWID baneados del servidor
2. Incluir en el mensaje `BEt_icchsrxy.HWData` enviado al servidor
3. Detectar discrepancias HWID vs sesión anterior

---

## Queries WMI Ejecutadas (Inferidas)

Basado en los campos del protobuf `ENOV9d` y prácticas estándar de AC:

```sql
-- CPU
SELECT * FROM Win32_Processor WHERE DeviceID='CPU0'
-- Fields: ProcessorId, Name

-- Disk (cada unidad física)
SELECT * FROM Win32_DiskDrive
-- Fields: SerialNumber, Model, PNPDeviceID

-- GPU
SELECT * FROM Win32_VideoController
-- Fields: PNPDeviceID, Name, AdapterRAM

-- Motherboard
SELECT * FROM Win32_BaseBoard
-- Fields: SerialNumber, Manufacturer, Product

-- OS (para InstallDate / anti-smurf)
SELECT * FROM Win32_OperatingSystem
-- Fields: InstallDate, SerialNumber, OSArchitecture

-- BIOS
SELECT * FROM Win32_BIOS
-- Fields: SerialNumber, Manufacturer, Version

-- Network (MAC address)
SELECT * FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled=True
-- Fields: MACAddress
```

Estas queries son ejecutadas por `A39Z4i.aSy1SLV` a través del
`IWbemServices` inicializado por `asYMlWeBL6f6`.

---

## Detección Anti-Smurf

El campo `AAUUUh` (mensaje protobuf) tiene campos:
- `Success` (bool)
- `Type` (string) — tipo de fecha: registry, WMI, archivo
- `InstallDate` (string) — fecha de instalación del OS

Este dato se recolecta para detectar **smurfing** (crear cuentas nuevas
para evitar bans). Si el `InstallDate` es muy reciente comparado con el
historial de juego, el servidor puede flagear la cuenta.

---

## Flujo de Datos Completo

```
1. AC inicia → main.QSUMsCa goroutines spawneadas

2. asYMlWeBL6f6 init:
   CoInitializeEx(NULL, COINIT_MULTITHREADED)
   CoInitializeSecurity(...)
   CoCreateInstance(CLSID_WbemLocator)
   IWbemLocator.ConnectServer("root\\cimv2")
   → WbemeDk0 (IWbemServices) listo

3. FXWqsvy_.Z6ey1EJD llamado:
   Para cada func1..func11:
     A39Z4i.aSy1SLV(query_WQL)
     → GUxA0QN3 (resultado WMI)
     → extraer campo (SerialNumber, Model, etc.)
     → almacenar en IwUTVxn o YRNNn8

4. BhCuafOD.gKzfRwK (o similar):
   Agregar todos los campos en ENOV9d protobuf
   dUgTofmw.(*fcje4l4dl_uV).Feed(evento_HWID)

5. dUgTofmw.KT300841 o CGRKkahhRkP:
   Serializar BEt_icchsrxy con HWData = ENOV9d
   Enviar via xzFpaM3Mq3 a l4d2center.com:443

6. Servidor responde:
   H1oahxz1l3iY { Success, Error, Banned, CheatSigs }
   Si Banned=true → P4mAKk.Qi1Z8I con SteamID + mensaje
```

---

## Bypass del Pipeline HWID

| Componente | Método de Bypass | Dificultad |
|------------|-----------------|-----------|
| `A39Z4i.aSy1SLV` — queries WMI | Hook `IWbemServices.ExecQuery` via COM vtable patching para devolver seriales falsos | Alta |
| `FXWqsvy_.Z6ey1EJD.funcN` — colectores | Hook cada función individual en memoria del proceso AC | Alta |
| `asYMlWeBL6f6.Aego3YEweIaU` — ExecQuery | Redirect ExecQuery en la DLL wbemprox.dll del sistema | Alta |
| WMI global en el sistema | WMI spoofing: modificar respuestas de WMI a nivel del provider | Media |
| **Enfoque recomendado** | Análisis dinámico con frida: `Interceptor.attach(addr_Z6ey1EJD_func1, { onLeave: (retval) => { retval.replace(spoofed) } })` para cada colector | Media |

### Pasos para Bypass Efectivo

```
1. Obtener dirección de memoria de cada Z6ey1EJD.funcN en runtime
   (frida: Module.findExportByName o escaneo de pclntab en memoria)

2. Para cada func1..func11:
   - Interceptar en el punto de retorno (onLeave)
   - Reemplazar el valor de retorno con el serial falso hardcodeado

3. Alternativamente: hook completo de A39Z4i.aSy1SLV
   - Reemplazar los resultados WMI antes de que FXWqsvy_ los procese
   - Más simple pero menos granular

4. Verificar que la respuesta del servidor sea Success=true
   (indica que el HWID spoofed no está baneado)
```

**Nota crítica:** El AC también puede usar `GTShob4MX` (registry) para
leer `HKLM\SOFTWARE\Microsoft\Cryptography\MachineGuid` como componente
adicional de HWID. Este valor debe ser también spoofed consistentemente.

---

## Referencias Cruzadas

```
pipeline_hwid.md ← paquetes_identificados.md  (definiciones de paquetes)
pipeline_hwid.md ← protocolo_protobuf.md       (ENOV9d, AAUUUh structs)
pipeline_hwid.md → bypass_guide.md             (sección bypass HWID)
pipeline_hwid.md → logica_deteccion.md         (Vectores 7-9 WMI/HWID)
```
