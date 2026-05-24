# Guía de Bypass — L4D2Center Anticheat

Documentación técnica de todas las técnicas de bypass identificadas para cada vector
de detección del AC. Basado 100% en análisis estático del binario.

**IMPORTANTE:** Este análisis es para uso educativo y de investigación.

---

## Resumen de Vectores y Dificultad de Bypass

| Vector | Técnica | Dificultad |
|--------|---------|------------|
| Blacklist de ventanas | Renombrar ventanas / ocultar | Fácil |
| Blacklist de procesos | Renombrar procesos / suspend | Fácil |
| Escaneo de memoria (CheatSigs) | No cargar en memoria del juego | Media |
| Módulos DLL del juego | No inyectar DLLs | Media |
| HWID fingerprinting | Hook WMI ExecQuery | Media |
| Heartbeat / sesión | No necesita bypass (pasivo) | N/A |
| gopsutil monitoring | Evitar flags sospechosos | Media |
| Capturas de pantalla | No necesita bypass (es evidencia) | Alta |
| A2S server query | Coincide con datos reales | N/A |
| Anti-smurf (InstallDate) | Dificil sin análisis dinámico | Alta |
| Blacklist de herramientas RE | Mover herramientas al VM | Fácil |
| Anti-debugger (WMI PerfOS) | Depuración remota / kernel en VM | Muy Alta |
| Integridad de archivos VPK | No modificar VPKs en addons | Fácil |
| Certificate pinning | Parchear offset 0x2A3E4B3 o hook `MyTTk7_I.(*A0lMBv6KYzU_).Read` (tls.Conn) | Alta |
| Captura de paquetes (gopacket/pcap) | VM con NIC virtualizada / deshabilitar NPCAP | Muy Alta |
| Scanner Source Engine (ConVars) | Hook `ReadProcessMemory` o mantener ConVars limpias | Media |

---

## Vector 1 — Bypass de Blacklist de Ventanas

**Cómo funciona la detección:**
El AC llama `EnumWindows` + `GetWindowTextW` cada ~5-10 segundos.
Las strings de la blacklist son compiladas en un patrón **regex** por `ncRaYk_Ke` (= `regexp` stdlib de Go) para matching eficiente. El patrón probablemente es del tipo:
```
(?i)(x32dbg|pc-ret|centos|windbg|dbgclr|de4dot|pepper|ghidra|hacker|...)
```

**Bypass:**
1. **Renombrar la ventana**: `SetWindowTextW(hwnd, L"")` — título vacío no hace match contra ningún regex
2. **Ocultar la ventana del tool**: `ShowWindow(hwnd, SW_HIDE)` — ventana oculta, no enumerable por `EnumWindows`
3. **Cambiar el título temporalmente**: El AC checkea cada 5-10 segundos. Un hook en
   `GetWindowTextW` puede retornar un título falso cuando el AC llama.
4. **Si el regex usa substrings**: basta con que el título NO contenga ninguno de los tokens. Un título como `"ANALYSIS_TOOL_01"` no haría match con ningún pattern conocido.

**Hook de GetWindowTextW:**
```cpp
// El AC llama GetWindowTextW por ordinal desde asYMlWeBL6f6 
// Hook vía VEH o via tabla de import del proceso anticheat
typedef int (WINAPI *GetWindowTextW_t)(HWND, LPWSTR, int);
GetWindowTextW_t original_GetWindowTextW;

int WINAPI Hook_GetWindowTextW(HWND hWnd, LPWSTR lpString, int nMaxCount) {
    int result = original_GetWindowTextW(hWnd, lpString, nMaxCount);
    // Si el título contiene una palabra sospechosa, reemplazarlo
    if (ContainsBlacklistedWord(lpString)) {
        lstrcpyW(lpString, L"Calculator");  // Título inocuo
    }
    return result;
}
```

---

## Vector 2 — Bypass de Blacklist de Procesos

**Cómo funciona la detección:**
El paquete `_6di6zc0se2v` (WatchList) vigila continuamente la lista de procesos.
Blacklist confirmada (parcial):
```
fiddler ollydbg discord opera ...
(ver analysis/listas_negras.md para lista completa)
```

**Bypass:**
1. **Renombrar el ejecutable**: `Fiddler.exe` → `MyApp.exe`
2. **Process hollowing en nombre inocuo**: Crear proceso inocuo, cargar tool en su espacio
3. **Hook de Process32Next**: El AC usa `CreateToolhelp32Snapshot` (por ordinal).
   Hookear la API para filtrar procesos del snapshot.

**Hook de Process32Next para ocultar procesos:**
```cpp
typedef BOOL (WINAPI *Process32Next_t)(HANDLE, LPPROCESSENTRY32W);
Process32Next_t original_Process32Next;

BOOL WINAPI Hook_Process32Next(HANDLE hSnapshot, LPPROCESSENTRY32W lppe) {
    while (original_Process32Next(hSnapshot, lppe)) {
        // Si el proceso está en la blacklist del AC, saltar
        if (IsBlacklistedProcess(lppe->szExeFile)) {
            continue;  // Buscar el siguiente proceso
        }
        return TRUE;
    }
    return FALSE;
}
```

---

## Vector 3 — Bypass de Escaneo de Firmas en Memoria

**Cómo funciona la detección:**
```
1. Servidor → AC: lista de CheatSigs (patrones de bytes)
2. AC: VirtualQuery sobre memoria del juego
3. AC: ReadProcessMemory para leer regiones
4. AC: memmem(región, patrón) por cada firma
5. Si hay match → reporte + ban
```

**Bypass — Opción A: No estar en la memoria del juego**
El método más efectivo: no inyectar el cheat como DLL dentro del proceso del juego.
Usar un proceso externo separado para leer/escribir la memoria.

**Bypass — Opción B: Hook de VirtualQuery**
```cpp
typedef SIZE_T (WINAPI *VirtualQuery_t)(LPCVOID, PMEMORY_BASIC_INFORMATION, SIZE_T);
VirtualQuery_t original_VirtualQuery;

SIZE_T WINAPI Hook_VirtualQuery(LPCVOID lpAddress, PMEMORY_BASIC_INFORMATION lpBuffer, SIZE_T dwLength) {
    SIZE_T result = original_VirtualQuery(lpAddress, lpBuffer, dwLength);
    // Si la región contiene nuestro código cheat, marcarla como no legible
    if (IsOurCheatRegion(lpAddress)) {
        lpBuffer->Protect = PAGE_NOACCESS;  // El AC saltará esta región
    }
    return result;
}
```

**Bypass — Opción C: Encriptación en memoria**
Mantener el código del cheat encriptado mientras no se usa. Desencriptar solo durante
la ejecución, re-encriptar inmediatamente. Las ventanas de escaneo del AC son ~60s.

**Bypass — Opción D: Modificar las firmas**
Las CheatSigs se descargan del servidor dinámicamente. Si se puede interceptar la
respuesta del servidor (ver bypass del heartbeat), se pueden modificar los patrones
para que no coincidan con el cheat.

---

## Vector 4 — Bypass de Detección de Módulos DLL

**Cómo funciona la detección:**
`StartL4D2.func3` monitorea los módulos del juego con `Module32First/Next`.
Cualquier DLL no conocida que aparezca después del inicio es sospechosa.

**Bypass — Opción A: No inyectar DLLs**
No usar inyección de DLL. En cambio, usar escritura externa de memoria desde otro proceso.

**Bypass — Opción B: Manual mapping**
No usar `LoadLibrary` (que registra la DLL en la lista de módulos).
Usar manual mapping: mapear manualmente el PE en memoria sin registrarlo en el PEB.

**Bypass — Opción C: Hook de Module32Next**
Similar al hook de Process32Next, ocultar la DLL inyectada de la enumeración.

---

## Vector 5 — Bypass de HWID Fingerprinting

Este es el vector más importante para evitar bans por hardware.

**Cómo funciona la detección:**
```
asYMlWeBL6f6.Aego3YEweIaU (WMI collector):
  WMI Win32_Processor     → ProcessorId, MaxClockSpeed
  WMI Win32_DiskDrive     → SerialNumber, Model
  WMI Win32_BaseBoard     → SerialNumber, Model
  WMI Win32_VideoController → PNPDeviceID, Name
  WMI Win32_OperatingSystem → InstallDate, BuildNumber, Version

Nota: HWARRxN.(*KbBU6bdOa).Build es el runtime de protobuf (MessageInfo.Build),
NO el HWID builder. El collector real es asYMlWeBL6f6.Aego3YEweIaU.
```

**Bypass — Hook del ExecQuery de IWbemServices:**

```cpp
#include <wbemidl.h>

typedef HRESULT (STDMETHODCALLTYPE *ExecQuery_t)(
    IWbemServices*, const BSTR, const BSTR, LONG, IWbemContext*, IEnumWbemClassObject**
);

ExecQuery_t original_ExecQuery = nullptr;

HRESULT STDMETHODCALLTYPE Hook_ExecQuery(
    IWbemServices* pThis,
    const BSTR strQueryLanguage,
    const BSTR strQuery,
    LONG lFlags,
    IWbemContext* pCtx,
    IEnumWbemClassObject** ppEnum
) {
    HRESULT hr = original_ExecQuery(pThis, strQueryLanguage, strQuery, lFlags, pCtx, ppEnum);
    if (SUCCEEDED(hr) && ppEnum) {
        // Envolver el enumerador para modificar los resultados
        *ppEnum = new FakeWbemEnumerator(*ppEnum, strQuery);
    }
    return hr;
}

// Instalación del hook en la vtable de IWbemServices
void InstallWMIHook(IWbemServices* pServices) {
    void** vtable = *(void***)pServices;
    // IWbemServices::ExecQuery está en el índice 20 de la vtable
    DWORD oldProtect;
    VirtualProtect(&vtable[20], sizeof(void*), PAGE_READWRITE, &oldProtect);
    original_ExecQuery = (ExecQuery_t)vtable[20];
    vtable[20] = (void*)Hook_ExecQuery;
    VirtualProtect(&vtable[20], sizeof(void*), oldProtect, &oldProtect);
}
```

**FakeWbemEnumerator** debe retornar valores falsos cuando el AC consulta:
- `Win32_Processor::ProcessorId` → CPU ID falso pero consistente
- `Win32_DiskDrive::SerialNumber` → Serial falso
- `Win32_BaseBoard::SerialNumber` → Serial de placa falsa
- `Win32_VideoController::PNPDeviceID` → PNP ID falso

**Clave:** Los valores falsos deben ser **consistentes entre ejecuciones** del AC.
Usar valores derivados de un seed fijo (ej: hash del nombre de usuario real).

---

## Vector 6 — Bypass de Heartbeat

**Cómo funciona:**
El AC envía un `MdQhYNc {}` (mensaje protobuf vacío) al servidor cada ~30 segundos.
El servidor puede responder con un ban inmediato o nuevas firmas.

**No necesita bypass activo** si no estás cheateando.
Si el servidor detecta un HWID baneado: el ban viene en la respuesta al heartbeat.

**Para análisis del protocolo:**
Capturar el heartbeat con Burp Suite/Fiddler (ver `protocolo_red.md`) permite
entender qué datos extra manda con cada heartbeat.

---

## Vector 7 — Bypass de Monitoreo gopsutil

**Cómo funciona:**
El paquete `sNAkh4` usa gopsutil para monitorear `left4dead2.exe`:
- `CmdlineWithContext` — detecta flags de debug
- `EnvironWithContext` — detecta variables de entorno sospechosas
- `Connections` — detecta proxies locales
- `MemoryMaps` — mapa de regiones de memoria

**Bypass:**

1. **No lanzar el juego con flags de debug:**
   - Evitar: `-condebug`, `-insecure`, `+sv_cheats 1`, `-dev`

2. **No usar variables de entorno sospechosas:**
   - Evitar: `WINEDLLOVERRIDES`, `LD_PRELOAD`

3. **No tener proxies locales activos durante el juego:**
   - Cerrar Fiddler/Charles antes de lanzar el AC

4. **VM para herramientas de análisis:**
   - El método más limpio: las herramientas de análisis corren en una VM separada
   - El proceso del juego en la máquina host no tiene nada sospechoso

---

## Vector 8 — Bypass de Blacklist de Herramientas RE

**Blacklist 3 (raw binary):**
```
.vpk dump peek kgdb mdbg xdeb
```

**Blacklist 4 (raw binary):**
```
start dnspy ilspy ILSpy pizza crack ida -brute james Debug
```

**Bypass:**
1. **Cambiar nombre del ejecutable**: `x64dbg.exe` → `explorer2.exe`
2. **Cambiar el título de ventana**: `x64dbg - [...]` → `" "` (espacio)
3. **No tener herramientas RE abiertas mientras se juega**: Análisis separado en VM
4. **El AC solo checkea cuando ConnectL4D2Server es llamado**: Herramientas cerradas
   para conectar, abiertas después si necesario (aunque el AC sigue corriendo)

---

## Bypass de Análisis de Red (Capturar el Protocolo)

Para capturar el tráfico entre el AC y el servidor sin que el AC lo detecte:

### Certificate Pinning — BLOQUEANTE

El AC implementa **certificate pinning completo**. El certificado de `l4d2center.com`
está hardcodeado en el binario:

```
Subject:  https://l4d2center.com/0
Issuer:   DigiCert Trusted G4 RSA4096 SHA256 2024 CA1
Offset:   0x2A3E4B3 (decimal 44213139)
```

El AC verifica que el cert del servidor sea exactamente ese, ignorando el store de
certificados de Windows. Burp Suite / Fiddler **NO funcionan** directamente.

### Método 1 — Parchear el Certificate Pinning en el Binario

```
1. Abrir el binario en Ghidra
2. Ir al offset 0x2A3E4B3 — localizar el certificado DER hardcodeado
3. Reemplazar el cert con el cert de la CA de Burp Suite (mismo tamaño o parchear referencias)
4. Ajustar la longitud del cert en la instrucción que pasa el slice a x509.AddCert
5. Ahora el AC aceptará el cert de Burp como válido
```

**Alternativa**: Hookear `crypto/tls.(*Conn).VerifyHostname` o la función que compara
el certificado para que siempre retorne `nil` (sin error).

### Método 2 — Hook Interno del TLS (Sin parchear el binario)

Para evitar que el AC detecte el proxy, hookear `crypto/tls` dentro del proceso:

```cpp
// En Ghidra + GoReSym: encontrar la dirección de
// crypto/tls.(*Conn).Write y crypto/tls.(*Conn).Read

// Hook crypto/tls.(*Conn).Write para capturar plaintext antes de encriptar
int64_t HOOK_tls_write(void* conn, struct GoSlice* slice, ...) {
    // Volcar el plaintext
    DumpToFile("tls_out.bin", slice->data, slice->len);
    return original_tls_write(conn, slice, ...);
}

// Hook crypto/tls.(*Conn).Read para capturar respuestas del servidor
```

**Ventaja**: Captura el plaintext ANTES del cifrado TLS — el certificate pinning
no importa porque el hook intercepta después de que TLS ya estableció la conexión.

### Método 3 — Redirigir DNS + Servidor Propio (Requiere bypass de pinning)

```
1. Editar C:\Windows\System32\drivers\etc\hosts:
   127.0.0.1  l4d2center.com

2. Parchear cert en binario (Método 1) O hookear la verificación
3. Levantar servidor gRPC local que implemente el protocolo
4. El AC conecta al servidor local → logear todo → reenviar al real (si aplica)
```

**ADVERTENCIA:** El AC monitorea las conexiones del proceso `left4dead2.exe`.
Un proxy en `127.0.0.1:8080` podría ser visible en las conexiones del juego.
Usar el proxy solo en el proceso del AC, no en el del juego.

### Método 2 — Redirigir DNS + Servidor Propio

```
1. Editar C:\Windows\System32\drivers\etc\hosts:
   127.0.0.1  l4d2center.com

2. Levantar servidor HTTPS local (nginx) con cert auto-firmado
3. Importar CA propia como confiable en Windows
4. El AC conecta al servidor local → logear todo → reenviar al real (si aplica)
```

### Método 3 — Hook Interno del TLS

Para evitar que el AC detecte el proxy, hookear `crypto/tls` dentro del proceso:

```cpp
// En Ghidra + GoReSym: encontrar la dirección de
// crypto/tls.(*Conn).Write y crypto/tls.(*Conn).Read

// Hook crypto/tls.(*Conn).Write para capturar plaintext antes de encriptar
int64_t HOOK_tls_write(void* conn, struct GoSlice* slice, ...) {
    // Volcar el plaintext
    DumpToFile("tls_out.bin", slice->data, slice->len);
    return original_tls_write(conn, slice, ...);
}

// Hook crypto/tls.(*Conn).Read para capturar respuestas del servidor
```

---

## Bypass de Detección de Cheat Externo (Proceso Separado)

Para un cheat que corre como proceso separado (externo al juego):

**Lo que el AC puede detectar de un proceso externo:**
1. Título de ventana → Renombrar o ocultar ventana
2. Nombre de proceso → Renombrar ejecutable
3. Conexiones de red del juego → El cheat externo no tiene conexión al juego

**Lo que el AC NO puede detectar de un proceso externo** (sin kernel driver):
- La lectura de memoria del juego desde un proceso externo con `OpenProcess` + `ReadProcessMemory`
- La escritura de memoria del juego con `WriteProcessMemory` desde proceso externo
- La inyección de threads remotos (pero esta técnica SI es detectable via `Module32Next`)

**El AC usa `ReadProcessMemory` para escanear el juego** desde su propio proceso.
Un cheat externo que usa la misma API no es más detectable que el AC mismo.

---

## Vector 13 — Bypass del Anti-Debugger (Win32_PerfFormattedData_PerfOS_System)

Este es el vector de anti-debug más difícil de evadir del AC. A diferencia de
`IsDebuggerPresent` o `CheckRemoteDebuggerPresent`, este método no mide la
**presencia** del debugger — mide su **efecto colateral** en el sistema.

**Cómo funciona la detección:**
```
WMI: SELECT ProcessorQueueLength, ContextSwitchesPersec
     FROM Win32_PerfFormattedData_PerfOS_System

Si ProcessorQueueLength > THRESHOLD:   → sospechoso (debugger activo causa stalls)
Si ContextSwitchesPersec > THRESHOLD:  → sospechoso (debugger genera context switches)
```

Valores normales vs. con debugger activo:
| Métrica | Sistema limpio | Con x64dbg + breakpoints |
|---------|---------------|--------------------------|
| ProcessorQueueLength | 0-2 | 5-20+ |
| ContextSwitchesPersec | 1000-5000 | 10000+ |

**Por qué `IsDebuggerPresent = false` no ayuda:**
El AC NO usa `IsDebuggerPresent`. Parchear esa API no tiene ningún efecto en este vector.

**Bypass — Opción A: Sistema con poca carga**
Si el sistema tiene muy pocos procesos corriendo y el debugger usa los breakpoints
con moderación, las métricas pueden mantenerse en rangos normales. Poco confiable.

**Bypass — Opción B: Depuración remota (RECOMENDADA)**

```
Setup:
  Máquina A (Target): Corre left4dead2.exe + l4d2c_anticheat.exe
                      El AC solo ve las métricas de la Máquina A — limpia
  
  Máquina B (Debugger): Corre x64dbg / WinDbg con conexión remota a Máquina A
                        Todo el overhead del debugger ocurre en Máquina B
                        
Configuración:
  1. En Máquina A: levantar dbgsrv.exe (Windows Debugging Server)
     C:\dbgsrv.exe -t tcp:port=1234
  2. En Máquina B: conectar WinDbg remotamente
     File → Connect to Remote Debugging Session: tcp:server=<IP_A>,port=1234
  3. El proceso del juego en Máquina A no tiene debugger LOCAL — las métricas WMI son normales
```

**Bypass — Opción C: Depuración Kernel via VM (MEJOR PARA ANÁLISIS PROFUNDO)**

```
Setup:
  Host (Debugger): WinDbg con extensión KDNET
  Guest VM (Target): Windows 11 + left4dead2.exe + AC
  
  Configuración:
  1. En VM: habilitar debugging de kernel
     bcdedit /debug on
     bcdedit /dbgsettings net hostip:<IP_HOST> port:50000
  2. En Host: WinDbg conecta a través de red
     File → Attach to kernel → Net: port=50000

Resultado: El kernel debugger opera a nivel de hipervisor.
La VM (donde corre el AC) no tiene overhead de debugger a nivel de OS.
Las métricas WMI del guest son 100% normales.
```

**Bypass — Opción D: Hookear la Query WMI**

```cpp
// El AC hace ExecQuery con Win32_PerfFormattedData_PerfOS_System
// Hookear IWbemServices::ExecQuery para interceptar esta query específica

HRESULT STDMETHODCALLTYPE Hook_ExecQuery(..., const BSTR strQuery, ...) {
    if (wcsstr(strQuery, L"Win32_PerfFormattedData_PerfOS_System")) {
        // Retornar valores falsos: ProcessorQueueLength=0, ContextSwitchesPersec=1500
        return ReturnFakePerfData(ppEnum);
    }
    return original_ExecQuery(pThis, strQueryLanguage, strQuery, lFlags, pCtx, ppEnum);
}
```

**Conclusión:** Para análisis del AC, usar **Opción B o C**. La depuración remota o
en VM elimina completamente el efecto colateral del debugger en la máquina target.

---

## Vector 14 — Bypass de Integridad de Archivos (Hash de VPKs)

**Cómo funciona la detección:**
El paquete `BhCuafOD` (hash 64-bit, probablemente FNV-1a o xxhash) calcula hashes
de los archivos `.vpk` en el directorio de addons del juego. El servidor compara
estos hashes contra una lista de hashes conocidos de cheats.

El struct `BiK8wj` del protocolo protobuf reporta:
```protobuf
message BiK8wj {
    string Path = ?;  // ruta del archivo VPK
    int64  Size = ?;  // tamaño en bytes
}
```

El AC escanea el sistema de archivos vía `ge3tkaLtE` (path/filepath) y `ux88b3Kaxj2` (io/fs).

**Bypass:**

1. **No tener VPKs de cheats durante el juego** — método más simple
2. **Renombrar el VPK** — el servidor probablemente busca por hash, no por nombre
3. **Modificar el VPK** — cambiar un byte del archivo cambia el hash FNV-1a/xxhash
   completamente (efecto avalancha). Si el servidor verifica por hash, un VPK levemente
   modificado no haría match con el hash conocido del cheat
4. **Hook del paquete de hash** — hookear `BhCuafOD.(*BDap2x).Sum64` para retornar
   hashes aleatorios, haciendo que el servidor no pueda identificar ningún VPK

**Nota:** El campo `Size` también se reporta — un VPK modificado puede cambiar de
tamaño si se añaden/eliminan bytes. Mantener el mismo tamaño si se modifica el contenido.

---

## Bypass del Sistema Anti-Smurf

**Datos usados por el anti-smurf:**
```
- SteamID de la cuenta
- InstallDate de Steam (fecha de instalación)
- HWID de la máquina
- IP del servidor
- Campo Witnesses (jugadores que reportaron)
```

**Bypass parcial:**
- HWID: hook del WMI (ver Vector 5)
- InstallDate: modificar el registro de Windows donde Steam guarda la fecha
  - Clave probable: `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Steam`
  - Campo: `InstallDate` (DWORD en formato YYYYMMDD)
- SteamID: esto NO se puede cambiar sin una cuenta nueva

**Nota:** El anti-smurf tiene el campo `Witnesses` — basado en reportes de otros jugadores.
Esto es independiente del HWID y SteamID, no bypasseable.

---

## Bypass del Análisis de Addons VPK

**Cómo funciona:**
El AC enumera los `.vpk` en el directorio de addons y los reporta al servidor.
El servidor puede rechazar conexiones con VPKs de cheats conocidos.

**Bypass:**
- No tener VPKs de cheats en el directorio de addons durante el juego
- Los VPKs de cheats deben estar fuera del directorio del juego

---

## Vector 15 — Bypass de Captura de Paquetes en Vivo (gopacket/pcap)

**Cómo funciona la detección:**
El AC usa `github.com/google/gopacket/pcap` (paquete `gG7Vmb7`, 452 funciones) para captura
de paquetes en tiempo real desde la interfaz de red. El paquete `uaIqvkWry1B` (gopacket core,
203 funciones) decodifica los paquetes capturados por capas (Link, Network, Transport).

Esto opera a nivel más bajo que socket-level monitoring — ve el raw traffic antes de que
llegue a los sockets de la aplicación.

**Qué puede detectar:**
- Proxies locales que interceptan tráfico del juego
- Modificación de paquetes UDP del Source Engine (network hacks)
- Presencia de adaptadores de red virtuales (VPN, TUN/TAP)
- Tráfico hacia IPs conocidas de proveedores de cheats

**Bypass — Opción A: VM con adaptador de red virtualizado**
```
Host: AC + left4dead2.exe corren en VM Guest
Guest: El pcap solo ve el adaptador virtual (e1000/virtio)
El tráfico real entre Guest y Host es manejado por el hipervisor

Resultado: El AC captura paquetes del adaptador virtual, no del real.
Si el cheat server está en el Host, el tráfico no es visible por pcap en el Guest.
```

**Bypass — Opción B: Kernel driver que oculta interfaz de red**
Interceptar las llamadas `pcap_open_live` / `DeviceIoControl` hacia `\Device\NPF_*`
(NPCAP driver) para retornar sin datos o con tráfico filtrado. Requiere driver propio.

**Bypass — Opción C: Deshabilitar NPCAP/WinPcap**
Sin la biblioteca NPCAP instalada, `pcap_open_live` falla. El AC probablemente maneja
este caso graciosamente (no crash), simplemente sin captura de paquetes.

**Nota:** Este es el vector más invasivo del AC. Opera debajo del nivel de aplicación.
Para bypass confiable, la VM es la única solución no invasiva.

---

## Bypass del TLS — Mapeo de Tipos Confirmado (crypto/tls)

Ahora que `MyTTk7_I` = `crypto/tls` está confirmado, las funciones exactas para hookear son:

```
MyTTk7_I.(*A0lMBv6KYzU_).Read    = crypto/tls.(*Conn).Read
MyTTk7_I.(*A0lMBv6KYzU_).Write   = crypto/tls.(*Conn).Write
MyTTk7_I.(*ExZca80).SetSessionTicketKeys = crypto/tls.(*Config).SetSessionTicketKeys
```

Para bypass del certificate pinning via hook interno:

```cpp
// Usar GoReSym o la pclntab para encontrar dirección de:
// MyTTk7_I.(*A0lMBv6KYzU_).Read  →  el Hook más útil para captura

// Con la dirección encontrada, instalar IAT/inline hook
// y volcar todos los datos leídos (plaintext post-TLS-decrypt)
```

La verificación de certificado probablemente ocurre dentro de `_ZyRwtuuaU.vdBoClcqZRtE`
(función con 12 closures en MyTTk7_I) — probablemente el TLS handshake completo.

---

## Plan de Acción para Bypass Completo

Para un bypass completo del sistema sin kernel drivers:

```
Paso 1: Proceso externo separado
  → Cheat corre como proceso externo (no DLL inyectada)
  → Nombre del proceso: algo inocuo (ej: "svchost_helper.exe")
  → Sin ventana visible (o ventana con título inocuo)

Paso 2: Hook WMI para HWID falso
  → Instalar hook en ExecQuery antes de que el AC llame WMI
  → Retornar seriales falsos pero consistentes

Paso 3: No usar flags de debug en el juego
  → Lanzar left4dead2.exe sin parámetros sospechosos

Paso 4: Herramientas de análisis en VM separada
  → x64dbg, Ghidra, dnSpy → VM aislada
  → La máquina host está limpia durante el juego

Paso 5: Proxying para análisis (SEPARADO del juego)
  → Solo para investigación del protocolo
  → Nunca mientras se conecta a un servidor del AC

Paso 6: Considerar captura de paquetes (Vector 15)
  → Si se analiza tráfico, usar VM para aislar el pcap del AC
  → NPCAP desinstalado en el sistema target reduce el riesgo
```

---

## Notas de Implementación

### El AC no usa Kernel Drivers
Sin análisis dinámico no podemos confirmar 100%, pero el análisis estático NO detectó:
- Creación de servicio de kernel driver
- Comunicación con `\\.\` device drivers
- `DeviceIoControl` con GUIDs conocidos de AC drivers

Esto sugiere que el bypass es posible **sin kernel drivers propios**.

### Las Goroutines de Detección Son Vulnerables a Race Conditions
```go
// Las goroutines duermen entre checks:
ticker := time.NewTicker(10 * time.Second)
for {
    select {
    case <-ticker.C:
        scanWindows()   // Aquí hace el check
        scanMemory()    // Y aquí
    }
}
```
Durante el sleep de 10 segundos, el cheat puede operar sin ser detectado.
Las acciones del cheat deben ser rápidas y limpiar rastros antes del próximo tick.

### El Scheduler de Go Puede Retrasar Goroutines de Detección
GOMAXPROCS limita los OS threads simultáneos. Una operación bloqueante larga en el
thread principal puede retrasar las goroutines de detección por un quantum del scheduler.
No es una técnica de bypass confiable pero es un dato interesante para timing.

---

## Vector 16 — Bypass de Firmas Estáticas Hardcoded (778 patrones)

Esta es la nueva superficie de ataque más grande descubierta en este análisis.

**Por qué es diferente a las CheatSigs del servidor:**
- Las 778 firmas están COMPILADAS en el binario (no se pueden cambiar sin un nuevo release)
- Se registran en el init() → disponibles ANTES de conectar al servidor
- Las strings de nombres de firmas están cifradas por garble -literals
- Sin análisis dinámico no se conocen los patrones exactos

**Cómo identificar qué patrones están registrados (análisis dinámico necesario):**

```python
# El punto de registro es en dUgTofmw o pdFrspK_G
# Al ejecutar en VM con frida/x64dbg, interceptar la función de registro:
# Buscar en pclntab la función llamada desde EyjsrRr.init.func1
# Hookar esa función → loggear todos los argumentos (nombre + patrón de bytes)

# Con frida (requiere ejecutar en VM):
import frida

session = frida.attach("l4d2c_anticheat.exe")
script = session.create_script("""
    // Encontrar dirección de la función de registro de firmas
    // desde pclntab, hookar y logear llamadas durante init()
    var registerSig = ptr("0xXXXXXX");  // dirección de la función de registro
    Interceptor.attach(registerSig, {
        onEnter: function(args) {
            console.log("Firma:", args[0].readUtf8String(), 
                        "Patron:", args[1].readByteArray(args[2].toInt32()));
        }
    });
""")
script.load()
""")
```

**Bypass:**

1. **No tener en memoria ningún patrón conocido:**
   - Las 778 firmas probablemente detectan: cabeceras PE de cheats conocidos, 
     strings hardcoded de menús de cheats, magic bytes de frameworks (ImGui cheats, 
     gmod cheats, etc.)
   - Un cheat "limpio" (sin signatures conocidas) no sería detectado

2. **Polimorfismo de código:**
   - Cifrar el payload del cheat en memoria
   - Descifrar solo al ejecutar, re-cifrar inmediatamente
   - Las ventanas de scan del AC no verán el patrón

3. **Reducir la huella de firmas conocidas:**
   - No usar frameworks de cheats populares (que tendrían firmas específicas)
   - Escribir código desde cero sin magic bytes reconocibles

4. **Hookear la función de registro durante init():**
   - Si se puede hookear la función de registro de firmas antes del init,
     se puede vaciar la lista o modificar los patrones para que no hagan match

**Nota crítica:** Las 778 firmas locales + las firmas descargadas del servidor suman
potencialmente 1000+ patrones en runtime. Esto supera a la mayoría de ACs comerciales
en número de firmas estáticas.

---

## Resumen Actualizado de Vectores y Bypass

| Vector | Descripción | Dificultad Bypass |
|--------|-------------|-------------------|
| 1 | CheatSigs del servidor (bytes patterns) | Media — cifrar memoria |
| 2 | Blacklist ventanas (EnumWindows + regex) | Fácil — renombrar/ocultar |
| 3 | Blacklist procesos (Toolhelp32) | Fácil — renombrar proceso |
| 4 | Módulos DLL del juego | Media — manual mapping |
| 5 | HWID fingerprinting — WMI pipeline (`FXWqsvy_`) | Media — hook cada colector en Z6ey1EJD |
| 6 | Heartbeat gRPC | N/A si no hay ban |
| 7 | gopsutil monitoring | Media — VM aislada |
| 8 | Screenshots (BitBlt) | Alta — evidencia irrefutable |
| 9 | A2S Query servidor | N/A — datos reales |
| 10 | Anti-smurf (InstallDate) | Alta — múltiples factores |
| 11 | Servidor HTTP local (Gin) | Media — análisis dinámico |
| 12 | A2S_INFO validation | N/A |
| 13 | Anti-debugger WMI PerfOS | Alta — depuración remota |
| 14 | Hash de VPKs | Fácil — no tener VPKs cheat |
| 15 | Captura de paquetes (gopacket/pcap) | Muy Alta — VM con NIC virtual |
| 16 | 778 firmas hardcoded | Alta — polimorfismo o código limpio |
| 17 | Toast notifications | N/A (informativo) |
| 18 | System tray | N/A (informativo) |
| 19 | **os/exec — procesos del sistema** | Media — hookear CreateProcess |
| 20 | **hlavBkMcO 644 verificaciones activas** | Muy Alta — análisis dinámico total |
| Source Engine ConVars | Scanner `sOAbtRgFLa6_` | Media — hook ReadProcessMemory |
| Certificate pinning | TLS cert hardcoded | Alta — hookear tls.Conn |
| Pipeline HWID completo | `asYMlWeBL6f6` → `A39Z4i` → `FXWqsvy_` → `BhCuafOD` → `dUgTofmw` | Media (múltiples hooks) |

---

## Bypass Vector 19 — os/exec (Procesos Externos)

El paquete `O2WU0BYsD` = `os/exec` permite al AC ejecutar comandos del sistema.

```
Usos probables:
  wmic cpu get ProcessorId     → fallback HWID si COM falla
  bcdedit /enum                → detectar modo debug del kernel
  sc query                     → detectar servicios de cheats
  powershell Get-WmiObject     → queries WMI alternativas

Bypass:
  1. Hook CreateProcessW en kernel32 para interceptar comandos
  2. Para "wmic" o "powershell": substituir stdout con output limpio
  3. Método más quirúrgico: hook O2WU0BYsD.(*SGUe_Ecp).Output
     directamente en el proceso AC con frida
```

---

## Bypass Vector 20 — hlavBkMcO 644 Verificaciones

### Plan de Análisis Dinámico Completo

```
Fase 1 — Preparación:
  VM limpia Windows 11, L4D2 vanilla, frida 16.x

Fase 2 — Baseline (estado limpio):
  Ejecutar AC + L4D2 sin mods
  Logear TODAS las llamadas a dUgTofmw.(*fcje4l4dl_uV).Feed
  Identificar qué closures de hlavBkMcO se activan normalmente
  Este es el "estado base" esperado — no se reporta nada

Fase 3 — Identificación de detecciones:
  Introducir cheat conocido (aimbot simple)
  Observar qué closures adicionales disparan Feed
  Esas closures = las que detectan ese cheat específico
  Repetir con diferentes tipos de cheats

Fase 4 — Construcción del bypass:
  Para cada closure que queremos silenciar:
    Hookear la closure para que no llame Feed
    O: filtrar eventos en Feed antes de que lleguen al evaluador
  Verificar que el AC no crashea
  Verificar que la detección está desactivada

Fase 5 — Bypass HWID:
  Hookar FXWqsvy_.Z6ey1EJD.func1..func11
  Reemplazar seriales con valores de máquina limpia
  Confirmar que servidor responde Success=true

Herramientas:
  frida >= 16.0    hooking dinámico
  x64dbg           análisis estático
  Ghidra+GoReSym   decompilación con nombres
  Process Monitor  syscalls
  Wireshark        tráfico gRPC post-TLS-bypass
```
