# Bypass / Spoofer de HWID

El HWID del L4D2Center AC combina cinco fuentes de hardware recolectadas vía WMI. Este documento explica cómo interceptar y falsificar cada componente.

---

## Los Cinco Componentes del HWID

| Componente | Clase WMI | Campo clave | Dificultad |
|------------|-----------|------------|------------|
| CPU | Win32_Processor | ProcessorId | Media |
| Disco | Win32_DiskDrive | SerialNumber | Media-Baja |
| Motherboard | Win32_BaseBoard | SerialNumber | Alta |
| GPU | Win32_VideoController | PNPDeviceID | Media |
| OS | Win32_OperatingSystem | BuildNumber/Version | Baja |

---

## Vector de Intercepción — WMI COM

El AC llama a la API WMI a través de COM (`CoInitializeEx` + `CoCreateInstance`). Hay dos puntos de intercepción:

**Opción A — COM Object Hook:** Hookear la creación del objeto WMI (`IWbemServices::ExecQuery`) para retornar datos falsos.

**Opción B — WMI Provider:** Registrar un proveedor WMI que tome precedencia. Complejo pero limpio.

**Opción C — Driver de filtro:** El método más robusto — interceptar en el driver de la clase WMI.

---

## Implementación — COM Hook (Opción A)

```cpp
#include <windows.h>
#include <comdef.h>
#include <wbemidl.h>

// Tabla original de ExecQuery
typedef HRESULT (STDMETHODCALLTYPE *ExecQuery_t)(
    IWbemServices* self,
    const BSTR strQueryLanguage,
    const BSTR strQuery,
    long lFlags,
    IWbemContext* pCtx,
    IWbemObjectSink** ppResponseHandler
);

ExecQuery_t ExecQuery_original = nullptr;

// Hook de ExecQuery para interceptar consultas WMI
HRESULT STDMETHODCALLTYPE ExecQuery_hook(
    IWbemServices* self,
    const BSTR strQueryLanguage,
    const BSTR strQuery,
    long lFlags,
    IWbemContext* pCtx,
    IWbemObjectSink** ppResponseHandler
) {
    std::wstring query(strQuery);
    
    // Interceptar consultas de ProcessorId
    if (query.find(L"Win32_Processor") != std::wstring::npos) {
        // Redirigir a implementación que devuelve ProcessorId falso
        return ejecutar_con_processorid_falso(self, strQueryLanguage, 
            strQuery, lFlags, pCtx, ppResponseHandler);
    }
    
    // Interceptar consultas de serial de disco
    if (query.find(L"Win32_DiskDrive") != std::wstring::npos) {
        return ejecutar_con_serial_disco_falso(self, strQueryLanguage,
            strQuery, lFlags, pCtx, ppResponseHandler);
    }
    
    // Pasar todo lo demás sin modificar
    return ExecQuery_original(self, strQueryLanguage, strQuery, 
        lFlags, pCtx, ppResponseHandler);
}

// Instalación del hook en la vtable de IWbemServices
void instalar_wmi_hook() {
    // Obtener IWbemServices
    IWbemLocator* pLocator = nullptr;
    CoCreateInstance(CLSID_WbemLocator, nullptr, CLSCTX_INPROC_SERVER,
        IID_IWbemLocator, (LPVOID*)&pLocator);
    
    IWbemServices* pServices = nullptr;
    pLocator->ConnectServer(L"ROOT\\CIMV2", nullptr, nullptr, nullptr,
        0, nullptr, nullptr, &pServices);
    
    // La vtable de IWbemServices tiene ExecQuery en el índice 20
    void** vtable = *(void***)pServices;
    
    DWORD proteccion_anterior;
    VirtualProtect(&vtable[20], sizeof(void*), PAGE_READWRITE, &proteccion_anterior);
    
    ExecQuery_original = (ExecQuery_t)vtable[20];
    vtable[20] = (void*)ExecQuery_hook;
    
    VirtualProtect(&vtable[20], sizeof(void*), proteccion_anterior, &proteccion_anterior);
}
```

---

## Spoof de ProcessorId (CPU Serial)

El ProcessorId viene de la instrucción CPUID en el CPU. Puede ser falseado con:

```cpp
// Método 1: Patchear la respuesta WMI (ver hook arriba)
// El valor devuelto es un string hexadecimal como "BFEBFBFF000906ED"

std::wstring generar_processorid_falso(const std::wstring& id_real) {
    // Modificar los últimos 8 chars del ProcessorId real
    // para mantener coherencia con la familia de CPU
    std::wstring id_falso = id_real;
    id_falso[8]  = L'A';  // modificar bits menos significativos
    id_falso[9]  = L'B';
    id_falso[10] = L'C';
    id_falso[11] = L'D';
    return id_falso;
}

// Método 2: Patchear CPUID con un driver
// El AC podría llamar CPUID directamente si no confía en WMI
// Un hypervisor o driver puede interceptar instrucciones CPUID
```

---

## Spoof de Serial de Disco

```cpp
// Interceptar Win32_DiskDrive.SerialNumber en la respuesta WMI
// El serial viene del firmware del disco via DeviceIoControl/IOCTL_STORAGE_QUERY_PROPERTY

// Método A: Hook WMI (más simple, suficiente si el AC solo usa WMI)
// Método B: Hook DeviceIoControl para interceptar STORAGE_DEVICE_DESCRIPTOR

typedef BOOL (WINAPI *DeviceIoControl_t)(
    HANDLE, DWORD, LPVOID, DWORD, LPVOID, DWORD, LPDWORD, LPOVERLAPPED
);

DeviceIoControl_t DeviceIoControl_original;

BOOL WINAPI DeviceIoControl_hook(
    HANDLE hDevice,
    DWORD dwIoControlCode,
    LPVOID lpInBuffer, DWORD nInBufferSize,
    LPVOID lpOutBuffer, DWORD nOutBufferSize,
    LPDWORD lpBytesReturned,
    LPOVERLAPPED lpOverlapped
) {
    BOOL resultado = DeviceIoControl_original(hDevice, dwIoControlCode,
        lpInBuffer, nInBufferSize, lpOutBuffer, nOutBufferSize,
        lpBytesReturned, lpOverlapped);
    
    if (dwIoControlCode == IOCTL_STORAGE_QUERY_PROPERTY && resultado) {
        STORAGE_DEVICE_DESCRIPTOR* desc = (STORAGE_DEVICE_DESCRIPTOR*)lpOutBuffer;
        if (desc->SerialNumberOffset > 0) {
            // Reemplazar el serial con uno falso
            char* serial = (char*)desc + desc->SerialNumberOffset;
            strcpy_s(serial, 20, "FAKESN12345678");
        }
    }
    
    return resultado;
}
```

---

## Spoof de Motherboard Serial

El serial de motherboard viene de SMBIOS. Tres enfoques:

```cpp
// Método 1: Hook WMI Win32_BaseBoard
// Más simple, pero el AC podría verificar directamente vía SMBIOS

// Método 2: Hook del registro SMBIOS
// SMBIOS data está disponible en:
// HKLM\SYSTEM\CurrentControlSet\Services\mssmbios\Data\SMBiosData

// Método 3: Con una VM, configurar el serial en la config del hypervisor:
// VMware: smbios.serial = "FAKEMB123456"
// VirtualBox: VBoxManage setextradata [VM] "VBoxInternal/Devices/pcbios/0/Config/DmiSystemSerial" "FAKEMB123456"
// Hyper-V: requiere modificación de UEFI firmware
```

---

## Spoof de GPU PNPDeviceID

```cpp
// PNPDeviceID tiene formato: PCI\VEN_XXXX&DEV_XXXX&SUBSYS_XXXXXXXX&REV_XX\XXXX...
// El AC extrae el PnpID de Win32_VideoController

// Hook WMI Win32_VideoController:
// El campo PNPDeviceID tiene el subsystem ID único — modificar esa parte

std::wstring spoof_pnp_id(const std::wstring& pnp_real) {
    // Formato: PCI\VEN_10DE&DEV_2684&SUBSYS_412219DA&REV_A1\4&XXXXX&0&0008
    // SUBSYS_XXXXXXXX contiene el SubVendorID+SubDeviceID — más identificatorio
    // Modificar el segmento tras &SUBSYS_
    size_t pos = pnp_real.find(L"SUBSYS_");
    if (pos != std::wstring::npos) {
        std::wstring pnp_falso = pnp_real;
        // Reemplazar los 8 chars del subsystem ID
        for (int i = 0; i < 8 && pos + 7 + i < pnp_falso.size(); i++)
            pnp_falso[pos + 7 + i] = L"0123456789ABCDEF"[rand() % 16];
        return pnp_falso;
    }
    return pnp_real;
}
```

---

## Implementación Completa como DLL

Estructura recomendada para el spoofer:

```cpp
// hwid_spoofer.cpp — DLL inyectada antes del AC
#include <windows.h>
#include <string>

void instalar_todos_los_hooks() {
    instalar_wmi_hook();          // intercepta todas las consultas WMI
    instalar_deviceiocontrol_hook();  // intercepta IOCTL de disco
    // GPU y OS se manejan dentro del WMI hook
}

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    if (fdwReason == DLL_PROCESS_ATTACH) {
        DisableThreadLibraryCalls(hinstDLL);
        
        // Crear hilo para no bloquear el DLL_PROCESS_ATTACH
        CreateThread(nullptr, 0, [](LPVOID) -> DWORD {
            Sleep(500);  // esperar a que el proceso inicialice COM
            instalar_todos_los_hooks();
            return 0;
        }, nullptr, 0, nullptr);
    }
    return TRUE;
}
```

---

## Estrategia de Inyección

El spoofer debe estar activo ANTES de que el AC haga las consultas WMI (que ocurren en ConnectL4D2Server.func1).

Opciones:
1. **Inyectar con SetWindowsHookEx** antes de que el AC inicie WMI
2. **AppInit_DLLs** (si DEP lo permite) — se carga en todo proceso
3. **Modificar el AC directamente** (patchear el binario para no verificar WMI)
4. **Ejecutar desde dentro de una VM** con hardware falso configurado en el hypervisor — el método más limpio y difícil de detectar

---

## Nota sobre Date de Instalación (Anti-Smurf)

El AC también lee la fecha de instalación de Steam. Esto probablemente viene del registro de Windows:

```
HKLM\SOFTWARE\WOW6432Node\Valve\Steam  InstallDate (DWORD — timestamp UNIX)
```

o via WMI `Win32_Product` o directamente del filesystem (fecha de creación de la carpeta de Steam).

Para una cuenta nueva, este valor indicará instalación reciente → posible flag de smurf. Modificar este valor en el registro requiere permisos de administrador pero es trivial.
