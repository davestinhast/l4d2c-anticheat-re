# Fingerprinting de Hardware — HWID

El AC recolecta identificadores de hardware vía WMI (Windows Management Instrumentation) y los combina en un identificador único que se envía al servidor. Este HWID es el vector principal de ban permanente.

---

## Consultas WMI Identificadas

### Win32_Processor — CPU

```
Clase WMI:   Win32_Processor (y Win32_ProcessorWithoutLoadPct)
Campos:      MaxClockSpeed, NumberOfCores, ProcessorId
```

- `ProcessorId` — identificador único del procesador (64 bits, grabado en el chip)
- `MaxClockSpeed` — velocidad máxima en MHz
- `NumberOfCores` — número de núcleos físicos

El campo **ProcessorId** es el más importante — es un identificador grabado en el hardware del CPU que no puede cambiarse sin cambiar el físico.

### Win32_DiskDrive — Disco

```
Clase WMI:   Win32_DiskDrive
Campos:      Model, SerialNumber
```

- `SerialNumber` — número de serie del disco (grabado en el firmware)
- `Model` — modelo del disco (ej: "Samsung SSD 870 EVO 500GB")

### Win32_BaseBoard — Motherboard

```
Clase WMI:   Win32_BaseBoard
Campos:      Model, SerialNumber, Manufacturer
```

- `SerialNumber` — número de serie de la placa madre
- `Manufacturer` — fabricante (ej: "ASUSTeK COMPUTER INC.")

### Win32_VideoController — GPU

```
Clase WMI:   Win32_VideoController
Campos:      Name/Description, PNPDeviceID
```

- `PNPDeviceID` — ID de dispositivo Plug-and-Play (contiene VID/PID/serial único)
- `Name` — nombre del adaptador (ej: "NVIDIA GeForce RTX 4090")

Este campo aparece como `PnpID` en el esquema protobuf.

### Win32_OperatingSystem — Sistema Operativo

```
Clase WMI:   Win32_OperatingSystem
Campos:      Caption, Version, BuildNumber, OSArchitecture
```

Recolecta la versión exacta de Windows instalada.

### Win32_PerfFormattedData_PerfOS_System — Estadísticas del Sistema

```
Clase WMI:   Win32_PerfFormattedData_PerfOS_System
```

Datos de rendimiento del sistema operativo. Posiblemente usado para detectar máquinas virtuales (los contadores de rendimiento se comportan diferente en VMs).

### Win32_ProcessConntrackStat — Estadísticas de Red

```
Clase WMI:   Win32_ProcessConntrackStat
```

Estadísticas de conexiones de red activas por proceso. Posiblemente usado para detectar proxies/VPNs activos al momento de conectar.

### Win32_Process — Procesos

```
Clase WMI:   Win32_Process
```

Enumeración de procesos en ejecución. Combinada con la blacklist de procesos.

---

## Estructura del HWID Enviado (ENOV9d)

El HWID completo se envía como mensaje protobuf embebido en `HWData` (field 10 del mensaje principal):

```
ENOV9d {
  Success: true,
  OS: [Win32_OperatingSystem data],
  CPU: [
    { ProcessorId: "BFEBFBFF000906ED", MaxClockSpeed: 3600, Cores: 8 },
    ...
  ],
  Disk: [
    { Model: "Samsung SSD 870 EVO", Serial: "S59ANX0T123456" },
    { Model: "WDC WD20EZRZ-00Z5HB0", Serial: "WD-WCC4N1234567" },
    ...
  ],
  MotherB: [
    { Model: "ROG STRIX B550-F GAMING", Serial: "MB1234567890" },
  ],
  GPU: [
    { Model: "NVIDIA GeForce RTX 4090", PnpID: "PCI\\VEN_10DE&DEV_2684&..." },
    ...
  ]
}
```

---

## Función HWARRxN — Constructor del HWID

```
HWARRxN.(*KbBU6bdOa).Build     — construye el HWID combinado
aOD4q1Y.CCS4azCb.Build         — build alternativo (protobuf builder)
```

El package `HWARRxN` es el responsable de recolectar todos los datos WMI y construir el mensaje `ENOV9d`.

---

## Algoritmos de Hash Presentes

Los strings del binario incluyen: `SHA-224`, `SHA-256`, `SHA-384`, `SHA-512`, `MD5`, `CRC32`, `RC4`.

El HWID enviado al servidor probablemente es un hash de la combinación de campos, no los valores raw. Esto hace que el servidor solo compare hashes, nunca vea los valores reales.

---

## Anti-Smurf — GetInstallDate

```
P4mAKk.(*AAUUUh).GetInstallDate
```

El AC lee la **fecha de instalación de Steam** (probablemente desde el registro de Windows o via WMI) y la envía al servidor. Las cuentas con Steam instalado recientemente son marcadas como potenciales smurfs.

Campo protobuf: `InstallDate` (field 3, int64 — timestamp UNIX)

---

## Consideraciones para Bypass de HWID

Los cinco componentes que más importan para la identificación única:

| Componente | Dificultad de spoofear | Notas |
|------------|----------------------|-------|
| CPU ProcessorId | Media | Requiere patching de WMI respuestas o driver |
| Disco Serial | Media-Baja | `diskpart` puede cambiar el serial de algunos discos; VMs tienen seriales configurables |
| Motherboard Serial | Alta | Grabado en firmware SMBIOS; necesita modificar ACPI tables en VM o usar un driver |
| GPU PnpID | Baja | Puede spoofearse modificando el registro o con un driver de filtro |
| OS info | Baja | Win32_OperatingSystem es fácil de interceptar |

Ver `bypass/bypass_hwid.md` para implementación.
