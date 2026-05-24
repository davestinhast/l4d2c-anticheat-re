# HWID — Sistema de Identificación de Hardware

El AC recopila un fingerprint de hardware (HWID) completo usando WMI (Windows Management
Instrumentation) vía COM. El HWID se serializa en protobuf como `ENOV9d` y se envía al
servidor en el campo `HWData` (field 10) del mensaje principal `BEt_icchsrxy`.

---

## Arquitectura del HWID

```
ENOV9d (HWData)
├── Success (bool)       — si la recopilación fue exitosa
├── OS[]   (repeated)   → Win32_OperatingSystem
├── CPU[]  (repeated)   → Win32_Processor
├── Disk[] (repeated)   → Win32_DiskDrive
├── MotherB[] (repeated)→ Win32_BaseBoard
└── GPU[]  (repeated)   → Win32_VideoController
```

El diseño `repeated` para cada componente indica que el AC enumera **todos** los dispositivos
de cada categoría (no solo el primero). En un PC con múltiples discos, enviará todos los seriales.

---

## Componentes del HWID — Detalles

### CPU — Win32_Processor

Struct inferido del método `GetCPU` en `ENOV9d`.

```proto
// Contiene ProcessorId (string hexadecimal de 16 chars de la CPU)
// Probablemente usa KLfm8u7 o EetHP7O como sub-struct
message CPUInfo {
  bool  Success     = 1;
  bytes ProcessorId = 2;  // CPUID instruction result — único por familia de CPU
  bytes Model       = 3;  // Win32_Processor.Name ("Intel Core i9-13900K")
}
```

**Campo WMI:** `Win32_Processor.ProcessorId`

El `ProcessorId` es el resultado de la instrucción CPUID — no cambia entre reinicios y es
específico del modelo/stepping del procesador. No es spoofeble sin hook de WMI.

---

### Disk — Win32_DiskDrive → EetHP7O

```proto
// Field numbers confirmados en el binario
message EetHP7O {
  bool  Success = 1;
  bytes Serial  = 2;  // Win32_DiskDrive.SerialNumber — número de serie del fabricante
  bytes Model   = 3;  // Win32_DiskDrive.Model — modelo del disco ("Samsung SSD 870 EVO")
}
```

**Campo WMI:** `Win32_DiskDrive.SerialNumber`

El serial del disco es el identificador más robusto. No cambia ni con formateo.
Para NVMe: también puede venir de `Win32_DiskDrive` o de IOCTL directo.

---

### Motherboard — Win32_BaseBoard → KLfm8u7

```proto
// Field numbers confirmados en el binario
message KLfm8u7 {
  bool  Success = 1;
  bytes Serial  = 2;  // Win32_BaseBoard.SerialNumber — serial SMBIOS de la placa
  bytes Model   = 3;  // Win32_BaseBoard.Model — modelo de la placa
}
```

**Campo WMI:** `Win32_BaseBoard.SerialNumber`

El serial de la motherboard viene de la tabla SMBIOS (DMI). En muchas placas genéricas/chinas
el valor es `"Default string"` o `"To Be Filled By O.E.M."`. En placas de marca (ASUS, MSI,
Gigabyte) es único.

---

### GPU — Win32_VideoController → BlhNU1RKfjg

```proto
// Field numbers confirmados en el binario
message BlhNU1RKfjg {
  bool  Success = 1;
  bytes Model   = 2;  // Win32_VideoController.Name — nombre de la GPU
  bytes PnpID   = 3;  // Win32_VideoController.PNPDeviceID — "PCI\VEN_10DE&DEV_2684&..."
}
```

**Campo WMI:** `Win32_VideoController.PNPDeviceID`

El PNP Device ID incluye el Vendor ID (10DE=NVIDIA, 1002=AMD) y Device ID del chip.
No incluye número de serie individual — identifica el modelo de GPU, no el chip específico.

---

### OS — Win32_OperatingSystem

Formato exacto del sub-struct no confirmado en el binario (no tiene struct protobuf visible).
Probable contenido:
- `Win32_OperatingSystem.BuildNumber` — versión exacta de Windows (p.ej. "22631" para Win11 23H2)
- `Win32_OperatingSystem.Version` — "10.0.22631"
- `Win32_OperatingSystem.SerialNumber` — número de serie de la instalación de Windows

---

## Implementación WMI — Paquete asYMlWeBL6f6

El AC usa COM/WMI directamente vía `CoCreateInstance` + `IWbemServices::ExecQuery`.

```go
// Identificadores del paquete WMI (garble)
asYMlWeBL6f6.WbemeDk0     // IWbemServices o IWbemLocator
asYMlWeBL6f6.VDjfdjV      // tipo WMI adicional
asYMlWeBL6f6.Aego3YEweIaU // función principal de ExecQuery
```

El paquete tiene **24 funciones `init`** (`init.func1` a `init.func24`), lo que sugiere
inicialización de múltiples namespaces WMI o múltiples queries precompiladas.

### Queries WMI identificadas (por nombre de clase en el binario)

| Clase WMI | Campo extraído | Propósito |
|-----------|---------------|-----------|
| `Win32_Processor` | `ProcessorId` | ID único de CPU para HWID |
| `Win32_DiskDrive` | `SerialNumber`, `Model` | ID único de disco para HWID |
| `Win32_BaseBoard` | `SerialNumber`, `Model` | ID único de placa madre para HWID |
| `Win32_VideoController` | `PNPDeviceID`, `Name` | Identificador de GPU |
| `Win32_OperatingSystem` | `BuildNumber`, `Version` | Versión exacta del OS |
| `Win32_Process` | (procesos corriendo) | Blacklist de procesos |
| `Win32_ProcessorWithoutLoadPct` | (variante de CPU) | CPU sin métrica de carga |
| `Win32_PerfFormattedData_PerfOS_System` | (métricas) | Counters de rendimiento |

---

## APIs de Windows Cargadas por Ordinal (Anti-Análisis)

Las funciones WMI/COM se cargan via `GetProcAddress` con nombre normal, pero otras APIs
críticas se cargan **por ordinal** usando `MustFindProcByOrdinal` para ocultar su uso:

```
ReadProcessMemory          OpenProcess
CreateToolhelp32Snapshot   Process32First / Process32Next
Module32First / Module32Next
EnumWindows / GetWindowTextW
DeviceIoControl
BitBlt / PatBlt / GetDC
CreateCompatibleDC / CreateCompatibleBitmap
RegOpenKeyEx / RegQueryValueEx / RegCloseKey
CoInitializeEx / CoCreateInstance
```

**Implicación para el bypass:** `GetProcAddress` hookeado no intercepta estas APIs.
Hay que hookear directamente la función en memoria por su dirección.

---

## Flujo de Recopilación del HWID

```
ConnectL4D2Server(ip)
  └── goroutine func1 (autenticación)
        ├── GetSteamID()           — leer SteamID del usuario local
        ├── GetInstallDate()       — fecha de instalación de Steam (anti-smurf)
        ├── HWARRxN.Build()        — construir ENOV9d vía WMI
        │     ├── Query Win32_Processor
        │     ├── Query Win32_DiskDrive
        │     ├── Query Win32_BaseBoard
        │     ├── Query Win32_VideoController
        │     └── Query Win32_OperatingSystem
        ├── Authenticate(server)   — enviar HWID + auth al servidor
        ├── GetSmurf()             — verificar anti-smurf con InstallDate
        └── GetBanned()            — verificar si la cuenta está baneada
```

---

## Sistema Anti-Smurf — AAUUUh

El paquete `AAUUUh` contiene la fecha de instalación de Steam como medida anti-smurf:

```proto
message AAUUUh {
  bool  Success     = 1;
  bytes Type        = 2;  // tipo de clasificación (cuenta nueva, sospechosa, etc.)
  int64 InstallDate = 3;  // timestamp UNIX de instalación de Steam
}
```

**Field confirmado:** `protobuf:"varint,3,opt,name=InstallDate,proto3"`

La fecha de instalación proviene de:
- Registro de Windows: `HKCU\Software\Valve\Steam` o `HKLM\SOFTWARE\WOW6432Node\Valve\Steam`
- WMI: `Win32_Product` donde Name like 'Steam'

Una cuenta con fecha de instalación muy reciente se marca como posible smurf.

---

## Bypass del HWID

### Método 1 — Hook de VTable WMI (recomendado)

```cpp
// IWbemServices::ExecQuery está en el índice 20 de la vtable
void** vtable = *(void***)pServices;

typedef HRESULT (STDMETHODCALLTYPE *ExecQuery_t)(
    IWbemServices*, BSTR, BSTR, LONG, IWbemContext*, IEnumWbemClassObject**);

ExecQuery_t original_ExecQuery = (ExecQuery_t)vtable[20];

HRESULT STDMETHODCALLTYPE Hook_ExecQuery(
    IWbemServices* pThis, BSTR strQueryLanguage, BSTR strQuery,
    LONG lFlags, IWbemContext* pCtx, IEnumWbemClassObject** ppEnum)
{
    HRESULT hr = original_ExecQuery(pThis, strQueryLanguage, strQuery, lFlags, pCtx, ppEnum);
    if (SUCCEEDED(hr) && ppEnum) {
        std::wstring query(strQuery);
        // Detectar qué query es y reemplazar los valores de retorno
        if (query.find(L"Win32_DiskDrive") != std::wstring::npos) {
            *ppEnum = new FakeDiskEnumerator(*ppEnum); // serial falso
        } else if (query.find(L"Win32_BaseBoard") != std::wstring::npos) {
            *ppEnum = new FakeBoardEnumerator(*ppEnum);
        } else if (query.find(L"Win32_Processor") != std::wstring::npos) {
            *ppEnum = new FakeCPUEnumerator(*ppEnum);
        }
    }
    return hr;
}

void InstallWMIHook(IWbemServices* pServices) {
    void** vtable = *(void***)pServices;
    DWORD oldProtect;
    VirtualProtect(&vtable[20], sizeof(void*), PAGE_READWRITE, &oldProtect);
    vtable[20] = (void*)Hook_ExecQuery;
    VirtualProtect(&vtable[20], sizeof(void*), oldProtect, &oldProtect);
}
```

### Método 2 — Hook de CoCreateInstance

Interceptar `CoCreateInstance` antes de que se cree el objeto `IWbemServices`:

```cpp
// Hook CoCreateInstance para detectar cuando crea IWbemLocator
// Luego hook ExecQuery en la interfaz IWbemServices que retorna
```

### Método 3 — Servidor WMI Local (sin hook)

Modificar las respuestas WMI a nivel de sistema usando el proveedor WMI custom.
Requiere privilegios de administrador pero es más limpio que hooking en proceso.

---

## Datos que NO se Pueden Spoof Fácilmente

| Dato | Motivo |
|------|--------|
| IP real del jugador | El servidor ve la IP de red, no WMI |
| SteamID real | Viene de la sesión activa de Steam (steam_api64.dll o registro) |
| Fecha de instalación de Steam | Solo spoofe con hook del registro |
| UUID del sistema BIOS | SMBIOS directo, necesita hook a nivel kernel para spoof limpio |

Para un bypass completo se necesita también spoof del SteamID y la fecha de instalación
además del hardware.
