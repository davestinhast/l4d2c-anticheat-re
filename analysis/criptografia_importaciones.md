# Criptografía e Importaciones del Binario

Documentación de toda la infraestructura criptográfica del anticheat
y las DLLs de Windows que importa (tanto estáticamente como en forma dinámica).

---

## Stack Criptográfico Completo

El AC usa múltiples capas de criptografía para proteger su comunicación con el servidor
y verificar la integridad de sus propios datos.

### AES — Cifrado Simétrico

**Paquete:** `PcTWfu` = `crypto/aes`

```
(*pAWn49)       — bloque AES base (BlockSize, Encrypt, Decrypt, NewGCM)
(*myqAnYT)      — modo AES-GCM (AEAD: autenticado y cifrado)
  - NonceSize() — tamaño del nonce (12 bytes para GCM)
  - Overhead()  — overhead del tag de autenticación (16 bytes)
  - Seal()      — cifrar + autenticar
  - Open()      — descifrar + verificar
(*pRzjXqs)      — otro modo (CBC o CTR)
  - BlockSize, Encrypt, Decrypt
(*gU1WIgp)      — otro modo (CFB o OFB)
  - BlockSize, Encrypt, Decrypt

Funciones AES-NI (hardware acceleration):
  _expand_key_128    — expansión de clave AES-128
  _expand_key_192a   — expansión de clave AES-192 (parte 1)
  _expand_key_192b   — expansión de clave AES-192 (parte 2)
  _expand_key_256a   — expansión de clave AES-256 (parte 1)
  _expand_key_256b   — expansión de clave AES-256 (parte 2)
```

**Implicación:** El AC cifra datos con AES-GCM (modo autenticado). Los datos enviados
al servidor y los datos del HWID están cifrados con claves que el AC tiene embebidas
de forma cifrada (garble -literals las cifra también).

---

### HMAC — Autenticación de Mensajes

**Paquete:** `aFzxJpzp2` = `crypto/hmac`

```
(*cESPyjL5y)            — wrapper HMAC
  - BlockSize()         — tamaño del bloque del hash subyacente
  - Size()              — tamaño del tag HMAC
  - Write(data []byte)  — alimentar datos
  - Sum(b []byte)       — obtener tag HMAC
  - Reset()             — reiniciar el estado
  - ConstantTimeSum()   — obtener tag en tiempo constante (anti timing attack)
  - MarshalBinary()     — serializar estado
  - UnmarshalBinary()   — deserializar estado
```

**`ConstantTimeSum` es el método distinctive** — solo `crypto/hmac` lo implementa en Go.
Usado para verificar integridad de mensajes sin vulnerabilidad de timing attack.

---

### Cifrado Adicional

**Paquete:** `pa9q8q` (identidad exacta pendiente — posiblemente `crypto/des` o `golang.org/x/crypto/chacha20`)

```
(*pXyC3Ai)      — cipher 1
(*lPQ6ELOmD)    — cipher 2 (estructura más compleja: func1-func3 + func3.1)
```

---

### X.509 y PKI

**Paquete:** `LYUqBZOV7RHi` = `crypto/x509`

```
(*XdDiONjbzcM)                    — x509.CertPool
  - AddCert(*Certificate)          — agregar certificado al pool
  - Clone()                        — clonar el pool
  - AppendCertsFromPEM(pem)        — agregar certificados desde PEM
  - Subjects()                     — lista de sujetos (deprecated en Go 1.19+)
  - Equal()                        — comparar pools
  - AddCertWithConstraint()        — agregar con restricciones personalizadas

(*ZIbJOWC4)                       — x509.Certificate
  - CheckSignature()               — verificar firma del certificado

(*n8Zy11Z).NewPublicKey            — crear nueva clave pública

31 funciones init                  — registra RSA, ECDSA, DSA, Ed25519, etc.
```

**Paquete:** `uL6SGHSpk` = `crypto/x509/pkix`

```
(*JjZ1WNQA)                       — pkix.Name
  - FillFromRDNSequence()          — llenar desde ASN.1 RDN sequence
  - ToRDNSequence()                — convertir a ASN.1 RDN sequence
  - String()                       — representación string (CN=..., O=...)

(*IBvuTjv).HasExpired             — pkix.CertificateList.HasExpired()
                                    verifica si una CRL (Certificate Revocation List) ha expirado

(*Ap8xj8SrB).String              — pkix.AlgorithmIdentifier string representation
```

**Uso:** El AC verifica que el certificado del servidor `l4d2center.com` no esté revocado
usando CRL checking (`HasExpired`). Esto es una capa de seguridad adicional sobre TLS
estándar para detectar si el servidor fue comprometido.

---

### TLS — Capa de Transporte

**Paquete:** `MyTTk7_I` = `crypto/tls`

```
(*ExZca80)                        — tls.SessionState
  - DecryptTicket()               — descifrar session ticket
  - EncryptTicket()               — cifrar session ticket
  - ResumptionState()             — estado para reanudación de sesión
```

**TLS Certificate en el binario:**
```
Subject: https://l4d2center.com/0
Issuer: DigiCert Trusted G4 RSA4096 SHA256 2024 CA1
Offset en binario: 44213115 (0x2A3E4B3)
```

El AC hace **certificate pinning** — verifica que el certificado del servidor sea
exactamente este, sin importar si el usuario tiene otros certificados instalados.
Esto previene ataques MITM con certificados de otras CAs.

---

### Números Aleatorios

**Paquete:** `Lvy2smJECjF` = `math/rand`
```
(*J3sgRye).ExpFloat64  — distribución exponencial (para delays aleatorios)
(*J3sgRye).Uint32      — uint32 pseudoaleatorio
```

**Paquete:** `sa96sWj2YdO` = posiblemente `crypto/rand` o `math/rand/v2`
```
(*j2se_tz1Yd).Uint64  — uint64 criptográficamente aleatorio
(*V_IwguYI).Uint64    — alternativa
```

**Uso:** Los delays entre checks usan distribución exponencial para hacer impredecible
el timing (no se puede predecir cuándo exactamente el AC hará un scan).

---

### Big Numbers

**Paquete:** `AFdc2Qd` = `math/big`

Usado por las operaciones RSA (generación/verificación de firmas, intercambio de claves).

---

## Importaciones del PE (DLLs)

### DLL Importada por Nombre (PE Import Table)

Solo **kernel32.dll** aparece en la tabla de importación estática:

```
WriteFile                    CloseHandle
WriteConsoleW                AddVectoredExceptionHandler
WerSetFlags                  AddVectoredContinueHandler
WerGetFlags                  CreateEventA
WaitForMultipleObjects       CreateIoCompletionPort
WaitForSingleObject          CreateThread
VirtualQuery                 CreateWaitableTimerExW
VirtualFree                  DuplicateHandle
VirtualAlloc                 ExitProcess
TlsAlloc                     FreeEnvironmentStringsW
SwitchToThread               GetConsoleMode
SuspendThread                GetCurrentThreadId
SetWaitableTimer             GetEnvironmentStringsW
SetProcessPriorityBoost      GetErrorMode
SetEvent                     GetProcAddress
SetErrorMode                 GetProcessAffinityMask
SetConsoleCtrlHandler        GetQueuedCompletionStatusEx
RtlVirtualUnwind             GetStdHandle
RtlLookupFunctionEntry       GetSystemDirectoryA
ResumeThread                 GetSystemInfo
RaiseFailFastException       SetThreadContext
PostQueuedCompletionStatus   GetThreadContext
LoadLibraryW
LoadLibraryExW
```

**Nota sobre SuspendThread/ResumeThread:** El AC puede suspender y reanudar threads.
Esto podría usarse para pausar el proceso del juego durante un scan de memoria crítico.

**Nota sobre SetProcessPriorityBoost:** El AC ajusta la prioridad de boost del proceso
— posiblemente se asigna alta prioridad para garantizar que sus goroutines corran
incluso bajo carga alta del sistema.

---

### DLLs Cargadas Dinámicamente (via LoadLibraryW + GetProcAddress)

Encontradas como strings en el binario o inferidas por los APIs que usa:

| DLL | APIs Probables | Propósito |
|-----|---------------|-----------|
| `pdh.dll` | `PdhOpenQuery`, `PdhAddCounter`, `PdhCollectQueryData` | Performance Data Helper — monitoreo de `Win32_PerfFormattedData_PerfOS_System` |
| `user32.dll` | `EnumWindows`, `GetWindowTextW`, `GetWindowLong`, `IsWindow`, `GetForegroundWindow` | Enumeración de ventanas para blacklist |
| `gdi32.dll` | `BitBlt`, `PatBlt`, `GetDC`, `CreateCompatibleBitmap` | Screenshots periódicos |
| `ntdll.dll` | `NtQueryInformationProcess`, `RtlGetVersion` | Información avanzada de procesos, versión de Windows |
| `winmm.dll` | Timers multimedia | Timers de alta resolución para intervalos de detección |
| `bcryptprimitives.dll` | `BCryptGenRandom` | Generación criptográfica de números aleatorios |

---

### APIs Cargadas por Ordinal (MustFindProcByOrdinal)

Las APIs más sensibles NO aparecen como strings en el binario. Son cargadas por número
de ordinal, no por nombre:

```
ReadProcessMemory            — leer memoria del proceso left4dead2.exe
OpenProcess                  — obtener handle del proceso del juego
CreateToolhelp32Snapshot     — snapshot de procesos/módulos/threads
EnumWindows                  — enumerar todas las ventanas del sistema
GetWindowTextW               — leer título de una ventana
Module32First / Module32Next — iterar módulos cargados en un proceso
Process32First / Next        — iterar procesos del sistema
Thread32First / Next         — iterar threads
```

**Implicación para bypass:** Hookar `ReadProcessMemory` u `OpenProcess` en el propio
proceso del AC (si se puede inyectar código en él) para filtrar los datos que recibe.

---

## Flujo de Cifrado del Protocolo gRPC

```
1. AC construye mensaje BEt_icchsrxy (protobuf)
   ├── HMAC-SHA256 de los datos del HWID (via aFzxJpzp2)
   ├── AES-GCM del payload (via PcTWfu)
   └── Serialización protobuf del mensaje cifrado

2. gRPC encapsula el mensaje en HTTP/2 DATA frame
   └── Todo el canal ya está cifrado por TLS 1.3 (MyTTk7_I)
       con certificate pinning contra l4d2center.com

3. El servidor responde con:
   ├── CheatSigs actualizadas (nuevas firmas de cheats)
   ├── Ban response (Qi1Z8I)
   └── Session renewal (P8ePybs)
```

---

## WMI Classes Usadas

Identificadas como strings visibles en el binario (no cifradas por garble -literals):

| Clase WMI | Datos Extraídos | Propósito |
|-----------|----------------|-----------|
| `Win32_Processor` | Serial, Model, MaxClockSpeed | HWID — identificador de CPU |
| `Win32_DiskDrive` | Serial, Model | HWID — identificador de disco |
| `Win32_BaseBoard` | Serial, Model | HWID — identificador de motherboard |
| `Win32_VideoController` | Model, PnpID | HWID — identificador de GPU |
| `Win32_OperatingSystem` | InstallDate, Version | Anti-smurf — fecha instalación SO |
| `Win32_Process` | Name, CommandLine | Escaneo de procesos |
| `Win32_PerfFormattedData_PerfOS_System` | ProcessorQueueLength, ContextSwitchesPersec | Anti-debugger — carga anómala |

La clase `Win32_PerfFormattedData_PerfOS_System` es inusual en un anticheat — detecta
debuggers por el overhead de CPU/context switches que generan cuando están activos.
