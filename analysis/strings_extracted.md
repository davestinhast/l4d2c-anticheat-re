# Key Strings — Extracted from Binary

182,482 total strings extracted (min length 6). Key findings below.

---

## Identity / Branding

```
L4D2Center Anticheat
com.wails.L4D2Center Anticheat
L4D2Center
https://l4d2center.com/0
<title>L4D2Center Anticheat</title>
"version": "2.0.0"
```

## Wails Frontend (JavaScript bridge)

```javascript
import { StartL4D2, ConnectL4D2Server, FrontendReady } from "./wailsjs/go/main/App.js";

export function ConnectL4D2Server(arg1: string): Promise<void>;
export function FrontendReady(): Promise<void>;
export function StartL4D2(): Promise<void>;
```

## Recovered Go Functions (garble-obfuscated names survive partially)

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
P4mAKk.(*Ok4xyD92c8).GetVersion
P4mAKk.(*BEt_icchsrxy).GetVersion
P4mAKk.(*Qi1Z8I).GetSteamID
P4mAKk.(*BEt_icchsrxy).GetSmurf
P4mAKk.(*BEt_icchsrxy).GetAddons
P4mAKk.(*BEt_icchsrxy).GetAddonsProvided
VKiZI7.(*BchsYIa).ProcessFailed
VKiZI7.(*IdR0cr).AddProcessFailed
VKiZI7.(*RQ5Z40).GetProcessFailedKind
VKiZI7.(*IdR0cr).ExecuteScript
H5_Lib8AHNQN.(*SzXwEOvmo).ExecJS
oFJMvWezJ9O.(*M2mVCO8zapb9).GetWindowDPI
oFJMvWezJ9O.(*XVcwgpKUXH).GetWindowDPI
syscall.(*XCD7BytET).LookupAccount
HWARRxN.(*KbBU6bdOa).Build
aOD4q1Y.CCS4azCb.Build
MyTTk7_I.(*ExZca80).BuildNameToCertificate
DiU0jWi.(*PXdkqC).IsWindowsVersionAtLeast
```

## Debugger / RE Tool Blacklist (Raw String)

```
commonaddonsx32dbgpc-retCentoswindbgdbgclrde4dotpepperghidrahackerx96dbgfoldersysmontimersefenceselectscalar
```

## Process / Tool Name Blacklist (Separate List)

```
gdb.exe       reverse       processharp   od
fiddler       x64_dbg       petools       monitor
ollydbg       wpe pro       PhantOm       x32_dbg
phantom       WPE PRO       charles       checker
harmony       PETools       sniffer       MDBCrew
DIEmW         inMonitor     Discord       Opera
cmd.exe       fdm.exe       zen.exe       Arc.exe
```

## WMI Classes (Hardware Fingerprinting)

```
Win32_Process
Win32_Processor
Win32_DiskDrive
Win32_BaseBoard
Win32_ProcessorWithoutLoadPct
Win32_ProcessorConntrackStat
MaxClockSpeed
NumberOfCores
```

## Protobuf Field Names (JSON tags)

```
AppID          AppKey         Addons         AddonsProvided
Auth           Banned         Bots           buildType
CheatSigs      CheatSigsRequested            Code
CPU            Deaths         Disk           Duration
EDF            Error          ExtendedServerInfo  ExtraInfo
Folder         Game           GameID         GPU
HWData         Index          InstallDate    Map
MaxPlayers     Mode           Model          Money
MotherB        Name           Nickname       OS
Path           Pattern        Players        PnpID
Port           Protocol       Score          Serial
Server         ServerIP       ServerOS       ServerType
Session        Session2       Size           Smurf
SourceTV       SteamID        Success        TheShip
Type           VAC            Version        Visibility
Witnesses
```

## Crypto Algorithms (from strings)

```
SHA-224    SHA-256    SHA-384    SHA-512
MD5        CRC32      RC4
AES        (no explicit string — may be in obfuscated code)
```

## Screenshot / Screen Capture

```
BitBlt
PatBlt
GetDC
```

## Network / HTTP2 / gRPC Related

```
ConnectL4D2Server
ConnectEx
ReportError
ReportZombies
ReportZerolen
ReportValidationErrors
GetClientConn
NewClientConn
GetServer
GetServerIP
IsStreamingClient
IsStreamingServer
EncodeToken
DecryptTicket
EncryptTicket
```

## TLS / Certificate

```
TLSConfig
TLSClientConfig
TLSHandshakeStart
TLSNextProto
DialTLSContext
BuildNameToCertificate    ← Note: deprecated API, not cert pinning
```

> No hardcoded cert hashes or fingerprints found — no pinning.

## File Types Scanned (Addons)

```
.vpk    ← Valve Pak files (L4D2 addon format)
*.vpk   ← glob pattern for VPK enumeration
```

## Windows API Functions (Loaded Dynamically)

From strings that match known Win32 API names:
```
EnumWindows (inferred from behavior)
GetWindowTextW (inferred)
OpenProcess
ReadProcessMemory
WriteProcessMemory (in static imports only as concept)
VirtualQuery (in static imports)
CreateToolhelp32Snapshot
Module32First / Module32Next
Process32First / Process32Next
CoInitializeEx / CoCreateInstance (WMI via COM)
```

## Cheat Engine / Aimware Reference

```
aimware
```
(Found as a string — likely in cheat name database or config)

## cheat .vpk suffix check

```
.vpk
cef.pak   ← Chromium Embedded Framework pak (legitimate)
```

## Suspicious App Check (Non-gaming)

```
Opera       ← Opera browser (blocked — overlay risk?)
Discord     ← Discord (overlay risk)
fdm.exe     ← Free Download Manager
zen.exe     ← Zen browser
Arc.exe     ← Arc browser
cmd.exe     ← Command Prompt (blocked)
```

## Stack Size

```
SizeOfStackReserve: 0x200000  (2 MB — larger than default, Go uses goroutine stacks)
SizeOfStackCommit:  0x1000    (4 KB initial commit)
```
