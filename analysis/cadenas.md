# Strings Clave — Extraídas del Binario

182,482 strings totales extraídas con `strings.exe` (longitud mínima 6). Hallazgos relevantes organizados por categoría.

---

## Identidad y Branding

```
L4D2Center Anticheat
com.wails.L4D2Center Anticheat
L4D2Center
https://l4d2center.com/0
<title>L4D2Center Anticheat</title>
"version": "2.0.0"
```

## Wails Frontend (JS Bridge Embebido)

```javascript
import { StartL4D2, ConnectL4D2Server, FrontendReady } from "./wailsjs/go/main/App.js";

export function ConnectL4D2Server(arg1: string): Promise<void>;
export function FrontendReady(): Promise<void>;
export function StartL4D2(): Promise<void>;
```

## Funciones Go Recuperadas (nombres ofuscados por garble)

```
main.(*HpP4qwz).StartL4D2
main.(*HpP4qwz).StartL4D2.func1
main.(*HpP4qwz).StartL4D2.func2
main.(*HpP4qwz).StartL4D2.func3
main.(*HpP4qwz).StartL4D2.func4
main.(*HpP4qwz).StartL4D2.func4.1
main.(*HpP4qwz).ConnectL4D2Server
main.(*HpP4qwz).ConnectL4D2Server.func1
main.(*HpP4qwz).ConnectL4D2Server.func2
main.(*HpP4qwz).ConnectL4D2Server.func3
main.(*HpP4qwz).ConnectL4D2Server.func4

P4mAKk.(*Ok4xyD92c8).GetVersion        → versión del AC
P4mAKk.(*BEt_icchsrxy).GetVersion      → versión del juego
P4mAKk.(*Qi1Z8I).GetSteamID            → Steam ID del jugador
P4mAKk.(*Qi1Z8I).GetBanned             → estado de ban
P4mAKk.(*Qi1Z8I).GetNickname           → nombre en Steam
P4mAKk.(*BEt_icchsrxy).GetSmurf        → detección de smurf
P4mAKk.(*BEt_icchsrxy).GetAddons       → addons VPK instalados
P4mAKk.(*BEt_icchsrxy).GetHWData       → hardware fingerprint
P4mAKk.(*AAUUUh).GetInstallDate        → fecha de instalación de Steam
P4mAKk.(*ENOV9d).GetCPU                → datos de CPU
P4mAKk.(*ENOV9d).GetDisk               → datos de disco
P4mAKk.(*ENOV9d).GetGPU                → datos de GPU
P4mAKk.(*ENOV9d).GetMotherB            → datos de motherboard
P4mAKk.(*ENOV9d).GetOS                 → datos del SO
P4mAKk.(*H1oahxz1l3iY).GetCheatSigs   → firmas de cheats del servidor
P4mAKk.(*H1oahxz1l3iY).GetServerIP    → IP del servidor de juego
P4mAKk.(*W99qYP).GetName               → nombre del cheat firmado
P4mAKk.(*W99qYP).GetPattern            → bytes de la firma

VKiZI7.(*BchsYIa).ProcessFailed        → crash del renderer WebView2 (UI)
VKiZI7.(*IdR0cr).ExecuteScript         → ejecutar JS en el WebView

HWARRxN.(*KbBU6bdOa).Build             → construir HWID combinado
BEwVDQOh5.(*HUBoNKO).Token             → token de auth activo
BEwVDQOh5.(*YDWWSd4K).EncodeToken      → codificar auth token

i1aqCskEISkX.(*eJ7sJnVS).Authenticate  → autenticación HTTP
_6di6zc0se2v.(*_o3YyW).WatchList       → lista de vigilancia de procesos

DiU0jWi.(*PXdkqC).IsWindowsVersionAtLeast → chequeo de versión de Windows
syscall.(*XCD7BytET).LookupAccount         → resolución de nombres de cuenta
```

## Blacklist de Herramientas RE (String Combinada)

```
commonaddonsx32dbgpc-retCentoswindbgdbgclrde4dotpepperghidrahackerx96dbgfoldersysmontimersefenceselectscalar
```

## Blacklist de Procesos

```
gdb.exe       reverse       processharp   od
fiddler       x64_dbg       petools       monitor
ollydbg       wpe pro       PhantOm       x32_dbg
phantom       WPE PRO       charles       checker
harmony       PETools       sniffer       MDBCrew
DIEmW         inMonitor     Discord       Opera
cmd.exe       fdm.exe       zen.exe       Arc.exe
```

## Clases WMI (Fingerprinting de Hardware)

```
Win32_Process
Win32_Processor
Win32_ProcessorWithoutLoadPct
Win32_ProcessConntrackStat
Win32_DiskDrive
Win32_BaseBoard
Win32_VideoController
Win32_OperatingSystem
Win32_PerfFormattedData_PerfOS_System

Campos consultados:
  MaxClockSpeed         NumberOfCores
  ProcessorId           SerialNumber
  Model                 Manufacturer
  PNPDeviceID           OSArchitecture
```

## Campos Protobuf (Tags JSON)

```
AppID          AppKey         Addons         AddonsProvided
Auth           Banned         Bots           BuildType
CheatSigs      CheatSigsRequested            Code
CPU            Deaths         Disk           Duration
EDF            Error          ExtraInfo      Folder
Game           GameID         GPU            HWData
Index          InstallDate    Map            MaxPlayers
Mode           Model          Money          MotherB
Name           Nickname       OS             Path
Pattern        Players        PnpID          Port
Protocol       Score          Serial         Server
ServerIP       ServerOS       ServerType     Session
Session2       Size           Smurf          SourceTV
SteamID        Success        TheShip        Type
VAC            Version        Visibility     Witnesses
```

## Algoritmos Criptográficos

```
SHA-224    SHA-256    SHA-384    SHA-512
MD5        CRC32      RC4
AES        (sin string explícita — probablemente en código ofuscado)
```

## Captura de Pantalla

```
BitBlt
PatBlt
GetDC
```

## Funciones de Red

```
ConnectL4D2Server    ConnectEx           ReportError
ReportZombies        ReportZerolen       ReportValidationErrors
GetClientConn        NewClientConn       GetServer
GetServerIP          IsStreamingClient   IsStreamingServer
EncodeToken          DecryptTicket       EncryptTicket
```

## TLS / Certificados

```
TLSConfig            TLSClientConfig     TLSHandshakeStart
TLSNextProto         DialTLSContext      BuildNameToCertificate
```

`BuildNameToCertificate` es una API deprecada de Go — indica ausencia de pinning.

## Tipos de Archivo Escaneados

```
.vpk     (archivos Valve Pak — formato de addon de L4D2)
*.vpk    (patrón glob para enumeración)
cef.pak  (Chromium Embedded Framework — legítimo, del WebView2)
```

## Win32 APIs (Cargadas Dinámicamente)

```
EnumWindows            GetWindowTextW         OpenProcess
ReadProcessMemory      VirtualQuery           CreateToolhelp32Snapshot
Module32First          Module32Next           Process32First
Process32Next          CoInitializeEx         CoCreateInstance
```

## Detección de Estado del Juego (Source Engine)

```
i4localSurvivorGunFire    (ConVar/netprop del Source Engine)
reportZombies             (función de reporte de estado)
```

## Apps Bloqueadas por Overlay

```
Opera     (riesgo de overlay)
Discord   (overlay en el juego)
fdm.exe   (Free Download Manager)
zen.exe   (Zen Browser)
Arc.exe   (Arc Browser)
cmd.exe   (Command Prompt)
```

## Stack Size del Binario

```
SizeOfStackReserve: 0x200000  (2 MB — mayor al default, Go usa goroutine stacks)
SizeOfStackCommit:  0x1000    (4 KB commit inicial)
```
