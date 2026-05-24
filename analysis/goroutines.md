# Arquitectura de Goroutines — Diseño Concurrente

L4D2Center Anticheat es un binario Go. Go usa goroutines (hilos ligeros) para concurrencia. Cada módulo de detección corre como una goroutine separada.

---

## Flujo del Goroutine Principal

```
main()
  └── Wails runtime init
        └── app.(*HpP4qwz) struct creado
              ├── Startup()             — 75+ goroutines de inicialización
              ├── FrontendReady()       — UI lista (WebView2)
              ├── StartL4D2()           — lanzar el juego
              ├── ConnectL4D2Server()   — conectar a servidor
              └── BeforeClose()         — cleanup al cerrar
```

---

## Función Startup — 75+ Goroutines de Inicialización

Esta es la función más compleja del AC. Con 75+ goroutines lanzadas durante el startup,
es la que inicializa todo el sistema de detección antes de que el usuario haga nada.

```
main.(*HpP4qwz).Startup              — inicializador principal
main.(*HpP4qwz).Startup.deferwrap1   — cleanup garantizado al salir
main.(*HpP4qwz).Startup.gowrap2      — goroutine wrapper
main.(*HpP4qwz).Startup.func1        — goroutine 1
main.(*HpP4qwz).Startup.func2        — goroutine 2
...
main.(*HpP4qwz).Startup.func60       — goroutine 60
(func9.1, func11.1, func19.1, func22.1, func26.1, func27.1, func35.1,
 func40.1, func43.1, func47.1, func49.1, func53.1 — sub-goroutines)

main.(*HpP4qwz).Startup.ZrMEw4hJW.func61  — goroutine 61 (sub-función)
main.(*HpP4qwz).Startup.ZrMEw4hJW.func62  — goroutine 62
...
main.(*HpP4qwz).Startup.ZrMEw4hJW.func75  — goroutine 75
```

Las 75 goroutines de Startup son probablemente:
- Inicialización de blacklists (ventanas, procesos, herramientas RE)
- Setup de hooks de Windows APIs
- Inicialización de WMI (COM objects)
- Setup del cliente HTTP/2 hacia l4d2center.com
- Inicialización de gopsutil process monitor
- Configuración del escáner de memoria
- Registro de event handlers de Wails (UI)
- Setup del sistema de tokens (BEwVDQOh5)

El `ZrMEw4hJW` es una closure interna que agrupa las últimas 15 goroutines —
posiblemente el motor de detección principal que se inicializa como unidad separada.

---

## Goroutines de ConnectL4D2Server()

Cuando el usuario hace clic en "Conectar", `ConnectL4D2Server(serverIP)` lanza múltiples goroutines:

```
ConnectL4D2Server(ip string)
  ├── func1: Goroutine de autenticación
  │     → Lee SteamID (GetSteamID desde Steam local)
  │     → Obtiene fecha de instalación Steam (GetInstallDate)
  │     → Recolecta HWID vía WMI (HWARRxN.Build)
  │     → Solicita auth token al servidor (Authenticate)
  │     → Verifica smurf (GetSmurf)
  │     → Verifica si está baneado (GetBanned / Qi1Z8I)
  │
  ├── func2: Goroutine de configuración de detección
  │     → Solicita CheatSigs del servidor (H1oahxz1l3iY)
  │     → Enumera addons VPK (GetAddons)
  │     → Inicializa el WatchList (_6di6zc0se2v)
  │     → Inicia loop de escaneo
  │
  ├── func3: Goroutine del loop de detección principal (ticker)
  │     → Cada N segundos:
  │         ├── Scan de títulos de ventana (EnumWindows + blacklist)
  │         ├── Scan de memoria del proceso (ReadProcessMemory + CheatSigs)
  │         ├── Validación de estado del juego (i4localSurvivorGunFire)
  │         └── Heartbeat al servidor
  │
  └── func4: Goroutine handler de resultados
        → Al detección: Screenshot (BitBlt) → Reporte → Desconexión/Ban
```

---

## Goroutines de StartL4D2()

```
StartL4D2()
  ├── func1: Lanzador del proceso del juego
  │     → CreateProcess("left4dead2.exe")
  │     → Monitorea salud del proceso
  │
  ├── func2: Monitor del juego
  │     → Detecta crash / salida del juego
  │     → Relanzar si configurado
  │
  ├── func3: Vigilante de inyección de procesos
  │     → Monitorea lista de módulos DLL del juego
  │     → Module32First / Module32Next en loop
  │     → Marca DLLs inesperados
  │
  ├── func4: Goroutine de screenshots periódicos
  │     → Capturas periódicas con BitBlt
  │     → Almacena para evidencia de ban
  │
  └── func4.1: Sub-handler de screenshots
```

---

## Paquetes Clave y Su Rol

### Paquetes de Detección

| Paquete (garbled) | Rol identificado |
|-------------------|-----------------|
| `dUgTofmw` | Motor de detección principal (30+ funciones, 12 init) |
| `_6di6zc0se2v` | WatchList — vigilancia continua de procesos y ventanas |
| `HWARRxN` | Constructor de HWID (`KbBU6bdOa.Build`) |
| `sNAkh4` | Monitoreo de proceso via gopsutil (CPU, RAM, cmdline, env) |
| `x1JPPahi` | Contenedor de tipos de datos de detección (HWID, files, etc.) |

### Paquetes de Protocolo

| Paquete (garbled) | Rol identificado |
|-------------------|-----------------|
| `P4mAKk` | Mensajes Protobuf (17 tipos documentados) |
| `BEwVDQOh5` | Sistema de tokens (Token, RawToken, EncodeToken — 8 goroutines) |
| `i1aqCskEISkX` | Cliente HTTP/2 (Authenticate, BasicAuth) |
| `gG7Vmb7` | Gran biblioteca de cliente HTTP o A2S query (300+ funciones) |
| `hh58lo` | A2S server query — consultas al servidor Source Engine (40+ funciones) |
| `eUmM3IpzIrV` | Struct de respuesta A2S (info del servidor: Map, VAC, Players, etc.) |

### Paquetes de Infraestructura

| Paquete (garbled) | Rol identificado |
|-------------------|-----------------|
| `main.(*HpP4qwz)` | Struct principal de la app Wails |
| `DiU0jWi` | Detección de versión de Windows |
| `aFzxJpzp2` | Implementación de hash (BlockSize, Sum, Write, Reset) |
| `AFdc2Qd` | Operaciones de big number / crypto |
| `aOD4q1Y` | Protobuf registry / descriptor |
| `HWARRxN.MQI4Jn6aY2` | Protobuf file descriptor (definiciones de mensajes) |

### Paquetes de UI (no relacionados con detección)

| Paquete (garbled) | Rol identificado |
|-------------------|-----------------|
| `VKiZI7` | Integración WebView2 (interfaz UI — NO detección) |
| `eiW4NTKLkx` | HTML template engine (UI de Wails) |
| `H5_Lib8AHNQN` | Runtime de Wails (ExecJS, WebView) |
| `oFJMvWezJ9O` | Utilitarios de ventana (GetWindowDPI) |

---

## Timing Estimado de Goroutines

| Goroutine | Intervalo estimado |
|-----------|-------------------|
| Heartbeat al servidor | ~30 segundos |
| Scan de títulos de ventana | ~5-10 segundos |
| Scan de memoria (CheatSigs) | ~60 segundos |
| Screenshot | ~120 segundos (posiblemente aleatorio) |
| HWID (initial) | Una sola vez al conectar |
| Vigilancia de módulos | Continua (event-based) |

Estimados basados en comportamiento típico de ACs — no confirmados sin análisis dinámico.

---

## Implementación de Timers

Los timers de Go usan `time.NewTicker` internamente. El ticker de detección probablemente sigue este patrón:

```go
// Equivalente en código fuente (garblado en binario):
ticker := time.NewTicker(10 * time.Second)
for {
    select {
    case <-ticker.C:
        scanWindows()
        scanMemory()
        sendHeartbeat()
    case <-done:
        return
    }
}
```

En Ghidra: buscar `time.NewTicker` o `time.After` cerca de las funciones de escaneo. El valor del intervalo (en nanosegundos) es un argumento constante.

---

## Por Qué las Goroutines Importan para el Bypass

### Ventanas de tiempo
Cada goroutine tiene un sleep entre checks. Ejecutar código cheat durante el sleep evita la detección del scan de ese ciclo.

### Scheduling de Go
El scheduler de Go (limitado por GOMAXPROCS) hace que todas las goroutines corran en los mismos OS threads. Una operación bloqueante larga puede retrasar TODAS las goroutines (incluyendo las de detección) por un quantum del scheduler.

### Comunicación por canales
Las goroutines de detección reportan resultados vía channels. Si se puede retrasar/bloquear el channel, el reporte de detección se retrasa.

### Recuperación de panics
Las goroutines de Go se recuperan de panics independientemente. Si se puede hacer crashear la goroutine de detección de forma controlada, puede no reiniciarse inmediatamente.

---

## Encontrar Boundaries de Goroutines en Ghidra

Los spawns de goroutines son llamadas a `runtime.newproc`:

```asm
; Patrón de spawn de goroutine en Go:
lea  rax, <function_pointer>      ; dirección de la función goroutine
mov  qword ptr [rsp+8h], rax
mov  ecx, <stack_size>            ; usualmente 0 (usar default)
call runtime.newproc               ; lanzar goroutine
```

Buscar llamadas a `runtime.newproc` en:
- `main.(*HpP4qwz).ConnectL4D2Server` → goroutines de detección
- `main.(*HpP4qwz).StartL4D2` → goroutines de monitoreo del juego
