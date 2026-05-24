# Binary Info — Detailed PE Analysis

## File Metadata

```
Filename:       l4d2c_anticheat.exe
SHA256:         (compute locally: certutil -hashfile l4d2c_anticheat.exe SHA256)
Size:           44,877,952 bytes (42.80 MB)
Magic:          MZ (DOS stub present)
PE Signature:   PE\0\0 at offset 0x80
```

## PE Optional Header (PE32+)

```
Architecture:           x86-64 (Machine: 0x8664)
Subsystem:              2 (Windows GUI — no console window)
Linker version:         3.0  ← Go toolchain linker signature
Optional header size:   0x70 (112 bytes, standard PE32+)
Entry point RVA:        0x78EC0  (VA: 0x0000000000478EC0)
ImageBase:              0x0000000000400000
SizeOfCode:             0x18FC200  (25.1 MB — massive code section)
SizeOfInitData:         0x000BEC00  (756 KB)
SizeOfUninitData:       0x00000000
SizeOfImage:            0x0326A000  (52.9 MB virtual)
SizeOfHeaders:          0x600
CheckSum:               0x02AD9EC3
OS version:             6.1 (Windows 7 minimum)
Image version:          1.0
Subsystem version:      6.1
Win32Version:           0
LoaderFlags:            0
```

## DLL Characteristics

```
0x8160:
  0x0020  HIGH_ENTROPY_VA      ← 64-bit ASLR with high entropy
  0x0040  DYNAMIC_BASE         ← ASLR enabled
  0x0100  NX_COMPAT            ← DEP/NX enabled
  0x8000  TERMINAL_SERVER_AWARE
```

## PE Sections

| # | Name | VAddr | VSize | RawSize | Perms | Description |
|---|------|-------|-------|---------|-------|-------------|
| 0 | `.text` | 0x1000 | 25.1 MB | 25.1 MB | RX | All Go code |
| 1 | `.rdata` | 0x18FE000 | 16.1 MB | 16.1 MB | R | Strings, type tables, constants |
| 2 | `.data` | 0x292F000 | 8.3 MB virt | 756 KB raw | RW | BSS + global vars |
| 3 | `.pdata` | 0x3189000 | 475 KB | 475 KB | R | Exception handlers (function table) |
| 4 | `.xdata` | 0x31FF000 | 192 B | 512 B | R | Unwind info |
| 5 | `.idata` | 0x3200000 | 1342 B | 1536 B | RW | Import table (kernel32.dll ONLY) |
| 6 | `.reloc` | 0x3201000 | 417 KB | 417 KB | R | Base relocations |
| 7 | `.symtab` | 0x3267000 | 4 B | 512 B | R | Symbol table (unusual) |
| 8 | `.rsrc` | 0x3268000 | 6.4 KB | 6.7 KB | R | Resources (manifest) |

## Data Directory

```
Entry 0: Export Directory         → 0x0 (no exports)
Entry 1: Import Directory         → 0x3200000 (kernel32.dll only)
Entry 2: Resource Directory       → 0x3268000 (.rsrc)
Entry 3: Exception Directory      → 0x3189000 (.pdata — function table)
Entry 4: Security Directory       → 0x2ACA400 (Authenticode signature)
Entry 5: Base Relocation          → 0x3201000 (.reloc)
Entry 6: Debug Directory          → 0x0 (NO DEBUG INFO — stripped)
Entry 9: TLS Directory            → 0x0 (no TLS callbacks)
Entry A: Load Config              → 0x0 (no load config)
Entry C: IAT                      → 0x292F040 (cached import addresses)
```

## Authenticode Signature

The binary is signed (Security Directory exists at 0x2ACA400, size 0x2480).
This means:
- It has a code signing certificate
- Windows will show a "Verified Publisher" in UAC prompts
- Certificate details: inspect with `sigcheck.exe` or `CFF Explorer`

## PE Timestamp: ZEROED

```
TimeDateStamp: 0x00000000 = Wed Dec 31 19:00:00 1969 (epoch 0)
```

**Deliberate anti-analysis measure.** The real build time was zeroed to prevent:
- Identifying the build date
- Correlating with code activity

## Go Runtime Indicators

Confirmed Go binary via:
1. Linker version 3.0 (Go's internal linker uses this)
2. Entry point leads to Go runtime init (`runtime.rt0_go`)
3. `.pdata` section contains Go's function exception table format
4. Strings: `goroutine`, `goexit`, `GOROOT`, `runtime.`, `go1.`
5. No C++ exception handlers (no RTTI, no vtables with C++ signatures)
6. Characteristic Go stack growth code pattern in .text

## Garble Obfuscation Indicators

Evidence of `garble` (Go obfuscation tool):
1. All package names are random strings (`qIE3Yzc`, `igWumSidaPL3`, `HpP4qwz`)
2. All type names are obfuscated (`Hydbgpt6`, `AQ2s03MYHxdb`)
3. PE timestamp zeroed (garble does this by default)
4. String literals in source partially obfuscated
5. Normal Go debug info stripped

Example recovered symbols (garbled → deduced):
```
main.(*HpP4qwz).StartL4D2             → main app struct, StartL4D2 method
P4mAKk.(*Qi1Z8I).GetSteamID          → some package, GetSteamID
P4mAKk.(*BEt_icchsrxy).GetSmurf      → same package, GetSmurf
VKiZI7.(*BchsYIa).ProcessFailed      → process failure handler
oFJMvWezJ9O.(*M2mVCO8zapb9).GetWindowDPI → window DPI (Wails UI)
```

## Resource Section (.rsrc)

Contains Windows manifest:
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
