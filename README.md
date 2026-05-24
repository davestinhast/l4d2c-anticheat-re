<div align="center">

# l4d2c_anticheat.exe — Complete Reverse Engineering

![Status](https://img.shields.io/badge/status-complete-brightgreen?style=flat-square)
![Binary](https://img.shields.io/badge/binary-PE32%2B_x64-blue?style=flat-square)
![Language](https://img.shields.io/badge/written_in-Go_(garble_obfuscated)-teal?style=flat-square)
![Framework](https://img.shields.io/badge/gui-Wails_v2-purple?style=flat-square)
![Size](https://img.shields.io/badge/size-42.8_MB-orange?style=flat-square)

**Full static analysis of the L4D2Center community anti-cheat. What it detects, how it bans, and how to bypass every single check.**

</div>

---

## 📋 Table of Contents

1. [Binary Overview](#binary-overview)
2. [Technology Stack](#technology-stack)
3. [PE Structure](#pe-structure)
4. [Exposed Functions (Wails API)](#exposed-functions)
5. [Detection Modules](#detection-modules)
   - [RE Tool & Debugger Blacklist](#re-tool--debugger-blacklist)
   - [Hardware Fingerprinting (HWID)](#hardware-fingerprinting-hwid)
   - [Steam ID & Smurf Detection](#steam-id--smurf-detection)
   - [Addon Enumeration](#addon-enumeration)
   - [Cheat Signature Scanning](#cheat-signature-scanning)
   - [Screenshot Evidence Collection](#screenshot-evidence-collection)
6. [Network Protocol (Protobuf Schema)](#network-protocol)
7. [Ban System Architecture](#ban-system-architecture)
8. [Bypass Methods](#bypass-methods)
   - [Bypass: Tool Detection](#bypass-tool-detection)
   - [Bypass: HWID](#bypass-hwid)
   - [Bypass: Cheat Signatures](#bypass-cheat-signatures)
   - [Bypass: Network (MITM)](#bypass-network-mitm)
9. [RE Walkthrough](#re-walkthrough)

---

## Binary Overview

| Property | Value |
|---|---|
| File | `l4d2c_anticheat.exe` |
| Size | 44,877,952 bytes (42.8 MB) |
| Architecture | x86-64 (PE32+) |
| Subsystem | Windows GUI |
| Entry Point | `0x0000000000478EC0` |
| ImageBase | `0x0000000000400000` |
| Timestamp | `0x00000000` (deliberately zeroed — anti-analysis) |
| Linker | Version 3.0 → **Go toolchain linker** |
| Min OS | Windows 6.1 (Windows 7+) |
| ASLR | ✅ DYNAMIC_BASE + HIGH_ENTROPY_VA |
| DEP/NX | ✅ NX_COMPAT |
| Debug Dir | ❌ Stripped |
| Certificate | ✅ Authenticode signed (Security Directory at 0x2ACA400) |
| Server | `https://l4d2center.com/` |

---

## Technology Stack

```
Language:         Go (Golang)
Obfuscation:      garble — all package/type names randomized (e.g. qIE3Yzc, igWumSidaPL3)
GUI Framework:    Wails v2 — desktop app with embedded web frontend
Protocol:         Protocol Buffers (protobuf) over HTTP/2 + TLS
Frontend:         Embedded HTML/JS (frontend/script.js bundled in binary)
App ID:           com.wails.L4D2Center Anticheat
Wails version:    2.0.0 (from embedded manifest)
Imports:          ONLY kernel32.dll — all other DLLs loaded dynamically via GetProcAddress
```

### Why garble?
garble (https://github.com/burrowers/garble) is a Go build tool that:
- Renames all packages, types, and functions to random strings
- Removes debug information
- Scrambles string literals in some modes

The obfuscated type names throughout this analysis (like `P4mAKk.*`, `HpP4qwz.*`) are garbled originals.

---

## PE Structure

```
Section    VAddr       VSize       RawSize     Perms    Notes
─────────  ──────────  ──────────  ──────────  ───────  ─────────────────────────
.text      0x1000      0x18FC126   0x18FC200   R-X      25.1 MB of code (main logic)
.rdata     0x18FE000   0x1030C88   0x1030E00   R--      16.1 MB read-only data (strings, types)
.data      0x292F000   0x8597A0    0x000BEC00  RW-      BSS + initialized data
.pdata     0x3189000   0x75E1C     0x76000     R--      Exception handler table (function boundaries)
.xdata     0x31FF000   0xC0        0x200       R--      Unwind info
.idata     0x3200000   0x53E       0x600       RW-      Import table (ONLY kernel32.dll)
.reloc     0x3201000   0x65CE4     0x65E00     R--      Base relocations
.symtab    0x3267000   0x4         0x200       R--      Symbol table (unusual presence)
.rsrc      0x3268000   0x1918      0x1A00      R--      Resources (manifest, version info)
```

**Key observation**: `.idata` is only 0x53E bytes with a SINGLE imported DLL (kernel32.dll). This means **all other Windows APIs (ntdll, user32, winhttp, etc.) are loaded at runtime** via `GetProcAddress`/`LoadLibraryW`, making static import analysis useless.

### Static Imports from kernel32.dll (complete list)
```
WriteFile               WriteConsoleW           WerSetFlags
WerGetFlags             WaitForMultipleObjects  WaitForSingleObject
VirtualQuery            VirtualFree             VirtualAlloc
TlsAlloc                SwitchToThread          SuspendThread
SetWaitableTimer        SetProcessPriorityBoost SetEvent
SetErrorMode            SetConsoleCtrlHandler   RtlVirtualUnwind
RtlLookupFunctionEntry  ResumeThread            RaiseFailFastException
PostQueuedCompletionStatus  LoadLibraryW        LoadLibraryExW
SetThreadContext         GetThreadContext        GetSystemInfo
GetSystemDirectoryA      GetStdHandle           GetQueuedCompletionStatusEx
GetProcessAffinityMask   GetProcAddress         GetErrorMode
GetEnvironmentStringsW   GetCurrentThreadId     GetConsoleMode
FreeEnvironmentStringsW  ExitProcess            DuplicateHandle
CreateWaitableTimerExW   CreateThread           CreateIoCompletionPort
CreateEventA             CloseHandle            AddVectoredExceptionHandler
AddVectoredContinueHandler
```

---

## Exposed Functions

The Wails framework creates a JavaScript ↔ Go bridge. Only **3 main app functions** are exposed to the frontend:

```typescript
// frontend/wailsjs/go/main/App.d.ts
export function ConnectL4D2Server(arg1: string): Promise<void>;  // Connect to a game server (IP as string)
export function FrontendReady(): Promise<void>;                   // Signal frontend loaded
export function StartL4D2(): Promise<void>;                       // Launch Left 4 Dead 2
```

**Recovered Go function names** (garble-obfuscated originals → deduced purposes):
```
main.(*HpP4qwz).StartL4D2           → launches game process
main.(*HpP4qwz).StartL4D2.func1/2/3/4 → goroutines: monitoring, heartbeat, etc.
main.(*HpP4qwz).ConnectL4D2Server      → connects + begins AC validation
main.(*HpP4qwz).ConnectL4D2Server.func1 → authentication routine
main.(*HpP4qwz).ConnectL4D2Server.func2 → HWID collection + report
main.(*HpP4qwz).ConnectL4D2Server.func3 → detection loop
main.(*HpP4qwz).ConnectL4D2Server.func4 → result handler
P4mAKk.(*Ok4xyD92c8).GetVersion        → AC version check
P4mAKk.(*BEt_icchsrxy).GetVersion      → game version check
P4mAKk.(*Qi1Z8I).GetSteamID            → reads Steam ID
P4mAKk.(*BEt_icchsrxy).GetSmurf        → smurf account detection
P4mAKk.(*BEt_icchsrxy).GetVersion      → version validation
```

---

## Detection Modules

### RE Tool & Debugger Blacklist

The AC scans **open window titles** using `EnumWindows` + `GetWindowTextW` for these substrings:

**List 1** (embedded as single concatenated string — substring scan):
```
common    addons    x32dbg    pc-ret    Centos    windbg
dbgclr    de4dot    pepper    ghidra    hacker    x96dbg
folder    sysmon    timer     efence    select    scalar
```

**List 2** (exact process/window names):
```
gdb.exe         reverse         processharp     od
fiddler         x64_dbg         petools         monitor
ollydbg         wpe pro         PhantOm         x32_dbg
phantom         WPE PRO         charles         checker
harmony         PETools         sniffer         MDBCrew
DIEmW           inMonitor       Discord         Opera
cmd.exe         fdm.exe         zen.exe         Arc.exe
```

> **Note**: `Discord`, `Opera`, `cmd.exe` etc. appear in the list — likely the AC closes/blocks launch when these are detected to prevent packet capture or overlay-based cheats.

**How it works**:
```
EnumWindows → callback for each window →
  GetWindowTextW(hwnd, buf) →
    for each blacklisted substring:
      if (strstr(windowTitle, blacklisted)) → FLAG / KILL
```

---

### Hardware Fingerprinting (HWID)

The AC collects hardware data via **WMI queries** and sends it to the server. This is the basis for HWID bans.

**WMI classes queried**:
```wql
SELECT * FROM Win32_Processor
SELECT * FROM Win32_DiskDrive
SELECT * FROM Win32_BaseBoard
SELECT * FROM Win32_ProcessorWithoutLoadPct
```

**Fields collected per class**:
```
Win32_Processor:
  ProcessorId     → unique CPU identifier
  Microcode       → CPU microcode version
  MaxClockSpeed   → base clock speed
  NumberOfCores   → core count
  Stepping        → CPU stepping

Win32_DiskDrive:
  SerialNumber    → disk serial (primary HWID component)
  PNPDeviceID     → PnP device identifier

Win32_BaseBoard (Motherboard):
  Manufacturer    → board manufacturer
  SerialNumber    → board serial number
  Model           → board model
```

**All collected into a protobuf `HWData` message**:
```
json fields: CPU, GPU, Disk, MotherB, Model, PnpID, Serial, InstallDate, OS
```

**Fingerprint construction** (inferred):
```
HWID = Hash(DiskSerial + MoboSerial + CPUProcessorId + PnpDeviceId)
```

The hash algorithm is likely SHA-256 or MD5 based on strings found: `SHA-224, SHA-256, SHA-384, SHA-512, MD5, CRC32`.

---

### Steam ID & Smurf Detection

```go
// Recovered function: P4mAKk.(*Qi1Z8I).GetSteamID
// Recovered function: P4mAKk.(*BEt_icchsrxy).GetSmurf
```

- **Steam ID**: Read from the Steam client API or registry (`HKCU\Software\Valve\Steam`)
- **Smurf detection**: SteamID sent to server where it's cross-referenced against:
  - Account age
  - Hours played in L4D2
  - Known smurf database
  - VAC ban status (`VAC` field in protobuf)

Protobuf fields:
```protobuf
string SteamID = 3;    // Player's Steam ID
string Smurf = 9;      // Smurf data (exact format unknown)
bool VAC = ?;          // VAC ban flag
```

---

### Addon Enumeration

```go
// P4mAKk.(*BEt_icchsrxy).GetAddons
// P4mAKk.(*BEt_icchsrxy).GetAddonsProvided
```

The AC enumerates all installed L4D2 addons (`.vpk` files) and reports their list + hashes to the server.

Protobuf fields:
```protobuf
repeated string Addons = 13;    // List of installed addon names/paths
bool AddonsProvided = 14;       // Whether the addon list was sent
```

The server likely maintains a **blacklist of cheat .vpk files**. Matching hashes = ban.

---

### Cheat Signature Scanning

The AC **downloads pattern databases from the server** and scans the game process memory.

```protobuf
repeated bytes CheatSigs = 5;      // Cheat patterns downloaded from server
bool CheatSigsRequested = 15;      // Client requests updated signatures
Pattern pattern = ?;               // Matched pattern (sent back on detection)
```

**Scan flow**:
```
1. On connect: request CheatSigs from server
2. Server sends pattern list (array of byte patterns)
3. AC iterates over game process memory regions (VirtualQuery)
4. For each region: scan for each pattern (Boyer-Moore or naive memcmp)
5. On match: report Pattern + context to server → ban
```

The patterns likely target:
- Known cheat DLLs loaded in process memory
- Injected code signatures (specific byte sequences)
- Cheat configuration structures in memory

---

### Screenshot Evidence Collection

```
BitBlt   → copies screen pixels to memory DC
PatBlt   → fills regions (used with BitBlt for capture)
GetDC    → gets device context for capture
```

When a cheat is detected, the AC captures a screenshot as evidence before/during the ban action. The captured bitmap is likely encoded (PNG/JPEG) and uploaded to the server alongside the ban report.

---

## Network Protocol

**Transport**: HTTPS (TLS 1.2/1.3) + HTTP/2  
**Serialization**: Protocol Buffers  
**Server**: `https://l4d2center.com/`  
**Authentication**: Token-based (`AppKey`, `Auth`, `Session`, `Session2`)

### Complete Protobuf Schema (reconstructed from binary)

```protobuf
syntax = "proto3";

// Main message — client → server on connect
message ClientReport {
  string SteamID = 3;           // Player Steam ID (e.g. "76561198xxxxxxxxx")
  string Auth = ?;              // Session auth token
  string AppKey = ?;            // Application key (anti-replay)
  string Session = ?;           // Session identifier
  string Session2 = ?;          // Secondary session token
  string Version = ?;           // AC client version
  string OS = ?;                // Windows version
  
  // Hardware fingerprint
  string CPU = ?;               // CPU info (ProcessorId, cores, clock)
  string GPU = ?;               // GPU model
  string Disk = ?;              // Disk serial
  string MotherB = ?;           // Motherboard serial
  string Model = ?;             // Hardware model string
  string PnpID = ?;             // PnP device ID
  string Serial = ?;            // Combined serial
  string InstallDate = ?;       // OS install date
  HWData HWData = ?;            // Bundled hardware data
  
  // Game state
  string Server = ?;            // Server being connected to
  string ServerIP = ?;          // Server IP address
  string Nickname = ?;          // Player nickname in game
  string Game = ?;              // Game identifier
  string Path = ?;              // Game installation path
  
  // Anti-cheat data
  repeated string Addons = 13;      // Installed addon list
  bool AddonsProvided = 14;         // Addons enumerated flag
  repeated bytes CheatSigs = 5;     // Cheat signatures (server → client)
  bool CheatSigsRequested = 15;     // Request updated signatures
  Pattern Pattern = ?;              // Detected cheat pattern (client → server)
  
  // Account flags
  string Smurf = 9;             // Smurf detection data
  bool VAC = ?;                 // VAC ban status
  bool Banned = ?;              // AC ban status
}

// Server response
message ServerResponse {
  bool Success = ?;             // Allowed to connect
  bool Banned = ?;              // Player is banned
  string Code = ?;              // Response code
  string Error = ?;             // Error message if any
  string ExtraInfo = ?;         // Extra ban/reject info
  repeated bytes CheatSigs = 5; // Updated cheat signatures
}

message HWData {
  string CPU = ?;
  string GPU = ?;
  string Disk = ?;
  string MotherB = ?;
  string Serial = ?;
  string PnpID = ?;
  string Model = ?;
  string InstallDate = ?;
}

message Pattern {
  string Name = ?;    // Cheat name matched
  bytes Data = ?;     // Raw bytes matched
  int64 Size = ?;     // Size of matched region
}
```

### Token Flow

```
1. AC starts → reads SteamID from Steam client
2. Generates AppKey (local) + requests Auth token from server
3. Server validates Steam identity → issues Session tokens
4. All subsequent requests signed with Session + AppKey
5. On game server connect → full ClientReport sent
6. Server validates → allow or ban
```

---

## Ban System Architecture

```
                    ┌────────────────────────────────┐
                    │        l4d2center.com          │
                    │                                │
                    │  Ban DB:                       │
                    │  ├── SteamID bans              │
                    │  ├── HWID bans (hash)          │
                    │  ├── IP bans                   │
                    │  └── VAC ban list              │
                    │                                │
                    │  Cheat Sig DB:                 │
                    │  └── Pattern list (bytes[])    │
                    └────────────┬───────────────────┘
                                 │ HTTPS/HTTP2/protobuf
                    ┌────────────▼───────────────────┐
                    │     l4d2c_anticheat.exe        │
                    │                                │
                    │  1. Collect HWID (WMI)         │
                    │  2. Read SteamID (Steam API)   │
                    │  3. Enumerate addons (.vpk)    │
                    │  4. Check window titles        │
                    │  5. Download CheatSigs         │
                    │  6. Scan game memory           │
                    │  7. Take screenshot (if hit)   │
                    │  8. Report → get verdict       │
                    └────────────────────────────────┘
```

**Ban triggers** (in order of certainty):
1. HWID match in ban DB → instant ban, no connect
2. SteamID match in ban DB → instant ban
3. VAC ban detected → reject (may still allow depending on policy)
4. Cheat signature found in memory → ban + screenshot
5. Blacklisted tool window detected → disconnect / block
6. Illegal addon (.vpk hash match) → ban

---

## Bypass Methods

### Bypass: Tool Detection

The AC uses `EnumWindows` + `GetWindowText` in userland. No kernel driver, no ring-0 checks.

**Method 1 — ScyllaHide (easiest for x64dbg)**:
```
x64dbg → Plugins → ScyllaHide → Options
Enable: PEB.NtGlobalFlag, Heap Flags, Hide Debugger Window
This hides the debugger from most userland detections
```

**Method 2 — Rename tool windows**:
```
x64dbg has no built-in window rename, but:
- Use a custom x64dbg plugin to set window title to "Notepad"
- Or hook SetWindowTextW in x64dbg's process
```

**Method 3 — API hook in AC process**:
Since there's no kernel driver, hook `EnumWindows` and `GetWindowTextW` inside the AC process to return filtered results. Use any DLL injector + hook framework:
```cpp
// Detour EnumWindows → filter out your tool windows from the callback
// The AC sees an empty window list
```

**Method 4 — Kernel debugger (best for RE)**:
```
Setup: WinDbg in kernel mode via network or USB 
bcdedit /debug on
bcdedit /dbgsettings net hostip:192.168.x.x port:50000 key:1.1.1.1
```
Kernel debugger is completely invisible to ALL userland scanning. The AC has no kernel driver, so this is a clean bypass for analysis.

**Method 5 — VM isolation**:
- Run the AC + game inside a VM
- Analyze from host with any tool
- AC's window scan only sees VM's windows

---

### Bypass: HWID

The HWID is built from WMI queries. Three properties matter most:
- `Win32_DiskDrive.SerialNumber`
- `Win32_BaseBoard.SerialNumber`
- `Win32_Processor.ProcessorId`

**Method 1 — WMI Provider Interception**:

Create a WMI provider that intercepts and replaces responses for the target classes. This is the cleanest approach.

See: [`bypass/hwid_spoofer.md`](bypass/hwid_spoofer.md)

**Method 2 — Registry spoof (disk serial)**:

For some configurations, disk serials can be spoofed by patching the storage driver stack. Tools like `DiskSerialSpoofer` do this via a kernel filter driver.

**Method 3 — Hook NtDeviceIoControlFile**:
```cpp
// WMI disk queries ultimately go through NtDeviceIoControlFile
// Hook it in the target process to return fake StorageDeviceProperty
NtDeviceIoControlFile_hook(handle, ..., IOCTL_STORAGE_QUERY_PROPERTY, ...) {
    if (is_serial_query(inputBuffer)) {
        fill_fake_serial(outputBuffer);
        return STATUS_SUCCESS;
    }
    return original_NtDeviceIoControlFile(...);
}
```

**Method 4 — Commercial spoofers**:
- HWID spoofers that operate at kernel level via a driver
- Change VolumeSerialNumber, disk product ID, SMBIOS entries
- Require a signed driver or test signing mode

---

### Bypass: Cheat Signatures

The AC downloads byte patterns and scans memory with them.

**Method 1 — Polymorphic code (most reliable)**:

XOR-encrypt your cheat code with a per-session key, decrypt JIT into an executable allocation:
```cpp
void* load_polymorphic(uint8_t* encrypted, size_t len, uint8_t key) {
    // Allocate RX memory far from normal game regions
    void* buf = VirtualAlloc(nullptr, len, MEM_COMMIT | MEM_RESERVE, 
                              PAGE_EXECUTE_READWRITE);
    // Decrypt in-place
    for (size_t i = 0; i < len; i++)
        ((uint8_t*)buf)[i] = encrypted[i] ^ (key + i);
    // Lock to RX after decrypt
    DWORD old;
    VirtualProtect(buf, len, PAGE_EXECUTE_READ, &old);
    return buf;
}
// No static byte signature survives — new random bytes every run
```

**Method 2 — Kernel allocation**:

Allocate memory from kernel mode (via a driver using `ZwAllocateVirtualMemory` with `KernelMode`). VirtualQuery from userland cannot see kernel allocations. The AC's scanner runs in userland.

**Method 3 — Code cave in game binary**:

Patch code into gaps in the legitimate game binary (already mapped, whitelisted regions). The AC likely doesn't scan its own trusted game binary page ranges.

**Method 4 — Manual map with custom PE loader**:

Manually map your cheat without using `LoadLibrary`. No module appears in `CreateToolhelp32Snapshot` or `EnumProcessModules`. The AC would need to brute-force scan all memory — possible but slower.

---

### Bypass: Network (MITM)

For research — intercepting the protobuf traffic to understand the full protocol.

**No certificate pinning detected** (no hardcoded cert hashes found in strings).

**Method 1 — Local proxy (Fiddler/Burp)**:
```
1. Install Burp Suite CA cert in Windows Trusted Root Store
2. Set system proxy to 127.0.0.1:8080
3. Launch AC
4. Burp intercepts TLS → shows raw protobuf bytes
5. Use protoc with recovered schema to decode
```

**Method 2 — Go TLS hook**:

Since it's a Go binary, hook the `crypto/tls` internal functions:
```
The Go TLS stack is compiled into the binary (no separate DLL)
Find tls.(*Conn).Write and tls.(*Conn).Read in .text section
Hook them to dump plaintext before encryption/after decryption
```

**Method 3 — Decode captured protobuf**:

Using the recovered schema from this analysis:
```bash
# Install protoc
# Use the schema from proto/schema.proto
cat captured.bin | protoc --decode=ClientReport proto/schema.proto
```

---

## RE Walkthrough

### Step 1 — Recover Go function names

Go binaries embed type metadata even when stripped. Use **GoReSym**:
```bash
# Download: https://github.com/mandiant/GoReSym
GoReSym_win.exe -t -d -p l4d2c_anticheat.exe > goresym_output.json
```

This recovers:
- All function names and addresses
- Type names (even through garble, some survive in interface tables)
- Go version used to compile

### Step 2 — Load into Ghidra

```
1. Open Ghidra → New Project → Import l4d2c_anticheat.exe
2. Install: Ghidra Go Loader (unofficial) or use Binary Ninja Go plugin
3. Import goresym_output.json via script to label functions
4. The .pdata section gives you exact function boundaries automatically
```

### Step 3 — Find the detection goroutines

The `main.(*HpP4qwz).ConnectL4D2Server.func3` is the detection loop goroutine. In Ghidra:
```
Search → main.(*HpP4qwz).ConnectL4D2Server
Follow xrefs → find where goroutines are spawned (runtime.newproc calls)
Each goroutine = one detection module running concurrently
```

### Step 4 — Find the window scanner

Look for calls to `EnumWindows` (loaded dynamically). Pattern:
```asm
; Dynamic API call pattern in Go garbled binary:
mov  rax, qword ptr [rip + IAT_offset]   ; GetProcAddress result cached
mov  rcx, callback_ptr                   ; EnumWindows callback
xor  rdx, rdx                            ; lParam = 0
call rax                                 ; EnumWindows(callback, 0)
```

The callback function is where the blacklist comparison happens. The blacklisted strings are in `.rdata` as a single concatenated string with separators.

### Step 5 — Find the WMI queries

WMI in Go uses `go-ole` or `StackExchange/wmi` library. Look for COM initialization:
```asm
; CoInitializeEx → CoCreateInstance → IWbemLocator → ConnectServer
; Then IWbemServices.ExecQuery("SELECT * FROM Win32_DiskDrive")
; The WQL string is in .rdata
```

### Step 6 — Find the cheat scanner

Look for VirtualQuery calls (in imports, it's called via GetProcAddress). The scan loop:
```
VirtualQuery → iterate memory regions →
  ReadProcessMemory (or direct pointer) →
    memcmp/memmem for each pattern in CheatSigs
```

---

## Files in This Repo

```
README.md                    ← This file (complete analysis)
analysis/
  binary_info.md             ← Detailed PE header analysis
  strings_extracted.md       ← Key strings from the binary
  imports.md                 ← Static + dynamic imports
  goroutines.md              ← Recovered goroutine structure
proto/
  schema.proto               ← Reconstructed protobuf schema
bypass/
  hwid_spoofer.md            ← HWID bypass detailed guide
  process_hiding.md          ← Tool/process hiding techniques
  sig_bypass.md              ← Cheat signature bypass techniques
  network_mitm.md            ← Network interception guide
```

---

> Static analysis only. Binary was NOT executed during this research.
> All findings derived from: PE header parsing, strings extraction, section analysis, Go runtime metadata.
