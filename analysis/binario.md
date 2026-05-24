# Análisis del Binario PE

## Metadatos del Archivo

```
Nombre:             l4d2c_anticheat.exe
Tamaño:             44,877,952 bytes (42.80 MB)
Magic:              MZ (DOS stub presente)
Firma PE:           PE\0\0 en offset 0x80
SHA256:             calcular localmente: certutil -hashfile l4d2c_anticheat.exe SHA256
```

## Cabecera PE Optional (PE32+)

```
Arquitectura:           x86-64 (Machine: 0x8664)
Subsistema:             2 (Windows GUI — sin ventana de consola)
Versión del linker:     3.0  (firma del linker interno de Go)
Tamaño cabecera opt.:   0x70 (112 bytes, estándar PE32+)
Entry point RVA:        0x78EC0  (VA: 0x0000000000478EC0)
ImageBase:              0x0000000000400000
SizeOfCode:             0x18FC200  (25.1 MB — sección de código masiva)
SizeOfInitData:         0x000BEC00  (756 KB)
SizeOfUninitData:       0x00000000
SizeOfImage:            0x0326A000  (52.9 MB virtual)
SizeOfHeaders:          0x600
CheckSum:               0x02AD9EC3
Versión OS mínima:      6.1 (Windows 7)
```

## Características DLL

```
0x8160:
  0x0020  HIGH_ENTROPY_VA      ASLR de 64 bits con alta entropía
  0x0040  DYNAMIC_BASE         ASLR habilitado
  0x0100  NX_COMPAT            DEP/NX habilitado
  0x8000  TERMINAL_SERVER_AWARE
```

## Secciones PE

| # | Nombre | VAddr | VSize | RawSize | Permisos | Descripción |
|---|--------|-------|-------|---------|----------|-------------|
| 0 | `.text` | 0x1000 | 25.1 MB | 25.1 MB | RX | Todo el código Go |
| 1 | `.rdata` | 0x18FE000 | 16.1 MB | 16.1 MB | R | Strings, tablas de tipos, constantes, frontend embebido |
| 2 | `.data` | 0x292F000 | 8.3 MB virt | 756 KB raw | RW | BSS + variables globales |
| 3 | `.pdata` | 0x3189000 | 475 KB | 475 KB | R | Exception handlers (tabla de funciones de Go) |
| 4 | `.xdata` | 0x31FF000 | 192 B | 512 B | R | Unwind info |
| 5 | `.idata` | 0x3200000 | 1342 B | 1536 B | RW | Import table (SOLO kernel32.dll) |
| 6 | `.reloc` | 0x3201000 | 417 KB | 417 KB | R | Base relocations |
| 7 | `.symtab` | 0x3267000 | 4 B | 512 B | R | Symbol table (inusual) |
| 8 | `.rsrc` | 0x3268000 | 6.4 KB | 6.7 KB | R | Recursos (manifest Windows) |

## Data Directory

```
Entry 0: Export Directory         → 0x0 (sin exports)
Entry 1: Import Directory         → 0x3200000 (solo kernel32.dll)
Entry 2: Resource Directory       → 0x3268000 (.rsrc)
Entry 3: Exception Directory      → 0x3189000 (.pdata)
Entry 4: Security Directory       → 0x2ACA400 (firma Authenticode, tamaño 0x2480)
Entry 5: Base Relocation          → 0x3201000 (.reloc)
Entry 6: Debug Directory          → 0x0 (SIN info de debug — stripped)
Entry 9: TLS Directory            → 0x0
Entry C: IAT                      → 0x292F040
```

## Imports Estáticos (kernel32.dll — Completos)

Solo kernel32.dll es importado estáticamente. Todo lo demás se carga en runtime via `GetProcAddress`.

```
WriteFile               WriteConsoleW           WerSetFlags
WerGetFlags             WaitForMultipleObjects  WaitForSingleObject
VirtualQuery            VirtualFree             VirtualAlloc
TlsAlloc                SwitchToThread          SuspendThread
SetWaitableTimer        SetProcessPriorityBoost SetEvent
SetErrorMode            SetConsoleCtrlHandler   RtlVirtualUnwind
RtlLookupFunctionEntry  ResumeThread            RaiseFailFastException
PostQueuedCompletionStatus LoadLibraryW          LoadLibraryExW
SetThreadContext        GetThreadContext         GetSystemInfo
GetSystemDirectoryA     GetStdHandle            GetQueuedCompletionStatusEx
GetProcessAffinityMask  GetProcAddress          GetErrorMode
GetEnvironmentStringsW  GetCurrentThreadId      GetConsoleMode
FreeEnvironmentStringsW ExitProcess             DuplicateHandle
CreateWaitableTimerExW  CreateThread            CreateIoCompletionPort
CreateEventA            CloseHandle             AddVectoredExceptionHandler
AddVectoredContinueHandler
```

**Punto importante:** La ausencia de imports como `CreateToolhelp32Snapshot`, `OpenProcess`, `ReadProcessMemory`, `EnumWindows` en la tabla de imports estáticos indica que estas funciones se resuelven dinámicamente en tiempo de ejecución via `GetProcAddress`. Esto es estándar en binarios Go — el runtime de Go resuelve los syscalls que necesita.

## Firma Authenticode

El binario está firmado digitalmente (Security Directory en 0x2ACA400, tamaño 0x2480). Esto significa:
- Muestra "Editor verificado" en el diálogo UAC
- Windows no mostrará advertencias de SmartScreen
- Inspeccionar con `sigcheck.exe` o `CFF Explorer` para ver el certificado

## Timestamp PE — Deliberadamente en Cero

```
TimeDateStamp: 0x00000000 = epoch 0 (miércoles 31 dic 1969)
```

Medida anti-análisis deliberada. `garble` borra el timestamp de compilación por defecto para impedir:
- Identificar la fecha de build
- Correlacionar con actividad del repositorio

## Indicadores del Runtime Go

Confirmado binario Go por múltiples indicadores:
1. Versión de linker 3.0 (linker interno de Go usa esto)
2. Entry point lleva a `runtime.rt0_go`
3. `.pdata` tiene formato de tabla de funciones de Go
4. Strings: `goroutine`, `goexit`, `GOROOT`, `runtime.`, `go1.`
5. Sin exception handlers C++ (sin RTTI, sin vtables C++)
6. Patrón característico de stack growth de Go en .text

## Indicadores de Ofuscación garble

Evidencia de `garble` (herramienta de ofuscación para Go):
1. Todos los nombres de paquetes son strings aleatorios (`P4mAKk`, `VKiZI7`, `eiW4NTKLkx`, `HWARRxN`)
2. Todos los nombres de tipos son aleatorios (`HpP4qwz`, `BEt_icchsrxy`, `Qi1Z8I`)
3. PE timestamp en cero (garble hace esto por defecto)
4. String literals del código fuente parcialmente ofuscados
5. Debug info de Go eliminada

Los nombres de métodos que sobreviven parcialmente son aquellos expuestos vía la interfaz Wails (se bindan como string al frontend JS y no pueden ofuscarse sin romper la funcionalidad).

## API de Wails — Funciones Expuestas al Frontend

Los archivos JavaScript `App.js` y `App.d.ts` están embebidos en el binario como parte del
filesystem virtual de Wails. Fueron extraídos directamente:

### App.d.ts (TypeScript declarations)

```typescript
// Auto-generated. DO NOT EDIT.
export function ConnectL4D2Server(arg1:string):Promise<void>;
export function FrontendReady():Promise<void>;
export function StartL4D2():Promise<void>;
```

### App.js (JavaScript implementation)

```javascript
// Auto-generated. DO NOT EDIT.
export function ConnectL4D2Server(arg1) {
  return ObfuscatedCall(0, [arg1]);
}
export function FrontendReady() {
  return ObfuscatedCall(1, []);
}
export function StartL4D2() {
  return ObfuscatedCall(2, []);
}
```

**`ObfuscatedCall(id, args)`** es un mecanismo de seguridad personalizado de este AC.
Wails normalmente usa `window.go.main.App.FunctionName()` para llamar al backend Go,
lo que expone los nombres de funciones en el JavaScript. Esta implementación usa IDs
numéricos en vez de nombres para ocultar las funciones al análisis del frontend.

IDs de funciones:
- `0` → `ConnectL4D2Server(serverIP: string)` — conectar a servidor (arg1 = IP:Puerto)
- `1` → `FrontendReady()` — UI lista (Wails llama esto automáticamente)
- `2` → `StartL4D2()` — lanzar el juego Left 4 Dead 2

Los tres métodos implementados en el struct `HpP4qwz` coinciden exactamente con estas tres funciones.

## Manifest de Recursos (.rsrc)

```xml
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <assemblyIdentity 
    type="win32" 
    name="com.wails.L4D2Center Anticheat" 
    version="0.0.0.0" 
    processorArchitecture="*"/>
  <dependency>
    <dependentAssembly>
      <assemblyIdentity 
        type="win32" 
        name="Microsoft.Windows.Common-Controls" 
        version="6.0.0.0"
        publicKeyToken="6595b64144ccf1df"/>
    </dependentAssembly>
  </dependency>
</assembly>
```
