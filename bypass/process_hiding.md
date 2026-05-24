# Process / Tool Hiding — Bypass Guide

The AC scans open window titles and process names looking for RE tools and debuggers.
It runs in USERLAND ONLY — no kernel driver. That's the exploit.

---

## Complete Blacklist (from binary)

### Substring scan (window titles):
```
common     addons     x32dbg     pc-ret
Centos     windbg     dbgclr     de4dot
pepper     ghidra     hacker     x96dbg
folder     sysmon     timer      efence
select     scalar
```

### Exact process/window names:
```
gdb.exe         reverse         processharp     od
fiddler         x64_dbg         petools         monitor
ollydbg         wpe pro         PhantOm         x32_dbg
phantom         WPE PRO         charles         checker
harmony         PETools         sniffer         MDBCrew
DIEmW           inMonitor       Discord         Opera
cmd.exe         fdm.exe         zen.exe         Arc.exe
```

---

## Method 1 — ScyllaHide (Fastest for x64dbg)

ScyllaHide is an x64dbg plugin that hides from anti-debug/anti-cheat techniques:

```
Download: https://github.com/x64dbg/ScyllaHide
Install:  Drop into x64dbg/plugins/ directory
Config:   Plugins → ScyllaHide → Options
          ✅ PEB.NtGlobalFlag
          ✅ PEB.HeapFlags  
          ✅ Hide from PEB.NtGlobalFlag
          ✅ DRx Breakpoints (hardware BP hiding)
          ✅ GetLocalTime / GetSystemTime
```

ScyllaHide also patches the window title. If not enough, use Method 2.

---

## Method 2 — Rename x64dbg Window

The AC scans for `x64dbg`, `x32dbg`, `x96dbg` substrings. x64dbg's window title is:
```
"x64dbg - [ProcessName]"
```

**Option A**: Edit x64dbg source → change window class name and title prefix → recompile.

**Option B**: Inject a DLL that hooks `SetWindowTextW` in x64dbg.exe to force a neutral title:
```cpp
// Force x64dbg title to something innocent
BOOL WINAPI Hooked_SetWindowTextW(HWND hwnd, LPCWSTR lpString) {
    if (GetCurrentProcessId() == x64dbg_pid && 
        wcsstr(lpString, L"x64dbg")) {
        return original_SetWindowTextW(hwnd, L"Notepad");
    }
    return original_SetWindowTextW(hwnd, lpString);
}
```

---

## Method 3 — Hook EnumWindows in AC Process

Since the AC is a Go binary with no protection on its own imports (only kernel32.dll imported statically, but `EnumWindows` is loaded via `GetProcAddress`), we can:

1. Inject a DLL into `l4d2c_anticheat.exe`
2. Hook `EnumWindows` at the IAT entry the Go runtime cached

```cpp
// After GetProcAddress returns the address of EnumWindows,
// the Go code caches it in a global variable in .data section
// We patch that pointer to point to our hook

BOOL WINAPI Hooked_EnumWindows(WNDENUMPROC lpEnumFunc, LPARAM lParam) {
    // Replace the callback with our filtered version
    return Real_EnumWindows([](HWND hwnd, LPARAM lp) -> BOOL {
        wchar_t title[512] = {};
        GetWindowTextW(hwnd, title, 512);
        
        // Block blacklisted titles from being seen
        const wchar_t* blacklist[] = {
            L"x64dbg", L"ghidra", L"ollydbg", L"fiddler", nullptr
        };
        for (int i = 0; blacklist[i]; i++) {
            if (wcsstr(title, blacklist[i])) return TRUE; // skip, continue enum
        }
        
        // Forward to original AC callback for safe windows
        WNDENUMPROC orig_cb = reinterpret_cast<WNDENUMPROC>(lp);
        return orig_cb(hwnd, 0);
    }, (LPARAM)lpEnumFunc);
}
```

---

## Method 4 — Kernel Debugger (Best for Analysis)

The AC has **no kernel driver**. It cannot detect kernel-mode debuggers.

### Setup WinDbg Kernel Debug (VM approach):

**VM Setup**:
```
1. Create Hyper-V or VMware VM with Windows
2. Install L4D2 + AC in VM
3. Enable kernel debug in VM:
   bcdedit /debug on
   bcdedit /dbgsettings net hostip:<HOST_IP> port:50000 key:1.1.1.1
4. Restart VM
5. On host: WinDbg → Attach to kernel → Network → fill in IP/port/key
```

**Result**: Full kernel debugging visibility. You can:
- Read all of AC's memory
- Set breakpoints on any function
- Trace AC execution
- The AC's EnumWindows scan will NEVER detect WinDbg on the host

---

## Method 5 — Separate Machine Analysis

The cleanest approach for static analysis (which we're doing):

```
Machine A (analysis):  Run Ghidra, IDA, any tool — AC not running here
Machine B (target):    Run AC + L4D2 normally — no tools visible
```

For dynamic analysis: dump the AC's memory from Machine B via kernel debug.

---

## Quick Reference — Safe Tools

Tools that are NOT in the blacklist:
```
✅ Ghidra (if window title doesn't have "ghidra"... but it does)
   → Rename Ghidra window or run on separate machine
✅ Binary Ninja   (not in blacklist)
✅ CFF Explorer   (not in blacklist)
✅ PE-Bear        (not in blacklist)
✅ HxD            (not in blacklist explicitly)
✅ Wireshark      (not in blacklist... but traffic analysis is obvious)
✅ x64dbg with ScyllaHide + renamed window
```

Tools that ARE blacklisted:
```
❌ x64dbg / x32dbg (title contains "x64dbg")
❌ OllyDbg
❌ WinDbg (userland instance — "windbg" substring)
❌ Ghidra (window title "ghidra")
❌ Fiddler
❌ Cheat Engine  
❌ Process Hacker ("hacker" substring)
❌ Sysmon ("sysmon")
❌ de4dot
❌ Charles proxy
❌ WPE Pro (packet editor)
❌ Cmd.exe (yes, Command Prompt is blocked)
❌ Discord (likely overlay detection)
```
