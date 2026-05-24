# Goroutine Architecture — Concurrent Detection Design

L4D2Center Anticheat is a Go program. Go uses goroutines (lightweight threads) for concurrency.
Each detection module runs as a separate goroutine. This is the inferred architecture.

---

## Main Goroutine Flow

```
main() 
  └── Wails runtime init
        └── app.(*HpP4qwz) struct created
              ├── FrontendReady() registered
              ├── StartL4D2() registered
              └── ConnectL4D2Server() registered
```

## ConnectL4D2Server() Goroutines

When the player clicks "Connect", `ConnectL4D2Server(serverIP)` spawns multiple goroutines:

```
ConnectL4D2Server(ip string)
  ├── func1: Authentication goroutine
  │     → Read SteamID (GetSteamID)
  │     → Collect HWID (WMI queries)
  │     → Get auth token from server
  │     → Validate account (GetSmurf)
  │
  ├── func2: Detection setup goroutine
  │     → Request CheatSigs from server
  │     → Enumerate addons (GetAddons)
  │     → Start scanner loop
  │
  ├── func3: Main detection loop goroutine (ticker)
  │     → Every N seconds:
  │         ├── Scan window titles (EnumWindows blacklist)
  │         ├── Scan process memory (CheatSigs patterns)
  │         └── Heartbeat to server
  │
  └── func4: Result handler goroutine
        → On detection: Screenshot → Report → Disconnect/Ban
```

## StartL4D2() Goroutines

```
StartL4D2()
  ├── func1: Game process launcher
  │     → CreateProcess("left4dead2.exe")
  │     → Monitor process health
  │
  ├── func2: Game monitor goroutine
  │     → Detect game crash / exit
  │     → Re-launch if configured
  │
  ├── func3: Process injection watcher
  │     → Monitor game's module list
  │     → Flag unexpected DLLs
  │
  ├── func4: Screenshot timer goroutine
  │     → Periodic BitBlt captures
  │     → Store for ban evidence
  │
  └── func4.1: Screenshot sub-handler
```

---

## Goroutine Timing (Estimated)

| Goroutine | Estimated Interval |
|---|---|
| Heartbeat to server | ~30 seconds |
| Window title scan | ~5-10 seconds |
| Memory signature scan | ~60 seconds (expensive) |
| Screenshot capture | ~120 seconds (random?) |
| HWID heartbeat | Once on connect |

These are estimates based on typical AC behavior — not confirmed without dynamic analysis.

---

## Why Goroutines Matter for Bypass

1. **Timing windows**: Each goroutine has a sleep interval between checks. Executing cheat code during the sleep window avoids detection.

2. **Goroutine scheduling**: Go's runtime scheduler (GOMAXPROCS-limited) means all goroutines run on the same OS threads. A single long blocking operation can delay ALL goroutines (including detection) by one scheduler quantum.

3. **Channel communication**: Detection goroutines communicate results via channels. If you can delay/block the channel, detection reporting is delayed.

4. **Panic recovery**: Go goroutines recover from panics independently. If you can cause the detection goroutine to panic (controlled crash), it may not restart immediately.

---

## Finding Goroutine Boundaries in Ghidra

After loading with GoReSym labels, goroutine spawns are `runtime.newproc` calls:

```asm
; Go goroutine spawn pattern:
lea  rax, <function_pointer>      ; address of goroutine function
mov  qword ptr [rsp+8h], rax
mov  ecx, <stack_size>            ; usually 0 (use default)
call runtime.newproc               ; spawn goroutine
```

Search for `runtime.newproc` calls in:
- `main.(*HpP4qwz).ConnectL4D2Server` → this is where detection goroutines spawn
- `main.(*HpP4qwz).StartL4D2` → game monitoring goroutines

Each call spawns one goroutine with the function pointer as its entry point.

---

## Timer Implementation

Go timers use `runtime.newTimer` internally. The detection interval is likely:

```go
// Source code equivalent (garbled in binary):
ticker := time.NewTicker(10 * time.Second)
for {
    select {
    case <-ticker.C:
        scanWindows()
        scanMemory()
    case <-done:
        return
    }
}
```

In Ghidra: look for `time.NewTicker` or `time.After` calls near the scanning functions.
The interval value (in nanoseconds) will be a constant argument.
