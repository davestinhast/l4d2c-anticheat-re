# Ingeniería Inversa — L4D2Center Anticheat

Análisis estático completo del binario `l4d2c_anticheat.exe`, el sistema anti-cheat del servidor comunitario L4D2Center. Este repositorio documenta absolutamente todo lo que hace el programa, cómo lo hace, y los vectores de bypass identificados.

**Restricción:** Todo el análisis fue realizado de forma estática. El binario no fue ejecutado.

---

## Identificación del Binario

```
Archivo:        l4d2c_anticheat.exe
Tamaño:         44,877,952 bytes (42.80 MB)
Arquitectura:   x86-64 (PE32+)
Lenguaje:       Go (confirmado por múltiples indicadores del runtime)
Ofuscador:      garble (obfuscación de nombres, tipos, strings y timestamp PE)
Framework UI:   Wails v2 (Go + WebView2/Chromium embebido)
Servidor:       https://l4d2center.com/0
Protocolo:      HTTP/2 + TLS 1.2/1.3 + Protocol Buffers v3 (gRPC)
Firma digital:  Authenticode — "Editor verificado: L4D2Center" en UAC
Certificados:   Auto-firmados (Root CA 10 años, hoja ~2.3 años)
Certificate Pinning: NO (usa el store de certificados de Windows)
```

---

## Lo que hace el AC — Resumen Técnico

### Al iniciar (Startup — 87 goroutines)
- Carga la interfaz WebView2 (Chromium embebido) con UI de consola oscura
- Inicializa blacklists de ventanas, procesos y herramientas RE
- Carga el sistema WMI (COM/DCOM) para el HWID
- Configura el cliente HTTP/2 hacia `l4d2center.com`
- Configura el sistema de tokens de autenticación
- Expone tres métodos al frontend JS: `StartL4D2` (ID=2), `ConnectL4D2Server` (ID=0), `FrontendReady` (ID=1)

### Al conectar al servidor (ConnectL4D2Server — 4 goroutines)

**Goroutine 1 — Autenticación**
- Lee el SteamID del usuario desde el cliente de Steam local
- Ejecuta consultas WMI para recolectar HWID completo:
  - `Win32_Processor.ProcessorId` → CPU ID
  - `Win32_DiskDrive.SerialNumber` / `Win32_DiskDrive.Model` → Disco
  - `Win32_BaseBoard.SerialNumber` / `Win32_BaseBoard.Model` → Placa madre
  - `Win32_VideoController.PNPDeviceID` / `.Name` → GPU
  - `Win32_OperatingSystem.BuildNumber` / `.Version` → OS
- Lee fecha de instalación de Steam (anti-smurf) → `AAUUUh.InstallDate`
- Solicita token de auth al servidor l4d2center.com
- Verifica si la cuenta es smurf (`GetSmurf`)
- Verifica si la cuenta está baneada (`GetBanned` → `Qi1Z8I.Banned`)

**Goroutine 2 — Configuración de detección**
- Descarga `CheatSigs` (W99qYP[]) desde el servidor — patrones de bytes en memoria
- Enumera todos los archivos `.vpk` del directorio de addons del juego
- Busca VPKs con patrón `%s\%sk1e1y*.vpk` (cheats de L4D2 conocidos)
- Inicializa el WatchList (`_6di6zc0se2v`)

**Goroutine 3 — Loop de detección (ticker cada ~5-60s)**
- Scan de títulos de ventanas: `EnumWindows` + `GetWindowTextW` contra blacklist
- Scan de procesos: `CreateToolhelp32Snapshot` + `Process32First/Next` contra blacklist
- Scan de memoria del juego: `ReadProcessMemory` contra CheatSigs descargados
- Monitoreo de módulos DLL: `Module32First/Next` buscando inyecciones
- Heartbeat al servidor cada ~30 segundos
- Capturas de pantalla con `BitBlt` cada ~120 segundos (formato PNG NRGBA)

**Goroutine 4 — Handler de resultados**
- Al detectar algo: captura screenshot → reporta al servidor → desconecta/banea
- Mensajes de consola en rojo (`#FF0000`) = ban/error

### Al lanzar el juego (StartL4D2 — 5 goroutines)
- `os/exec.Cmd.Start()` → lanza `left4dead2.exe`
- `exec.Cmd.Wait()` → espera cierre del juego
- Vigilancia de módulos DLL cargados en el proceso del juego
- Capturas periódicas de pantalla como evidencia

### Motor de Detección Principal (dUgTofmw.ga4oovjHCfg — 37 closures)
- 33 goroutines de detección numeradas (func1–func33) + 4 sub-goroutines anidadas (func9.1, func12.1, func21.1, func28.1)
- Cada goroutine maneja un vector de detección independiente
- Patrón productor-consumidor: `(*fcje4l4dl_uV).Feed` alimenta datos entre goroutines
- Paralelización: scan de ventanas, procesos, memoria, módulos, network

---

## Estructura del Repositorio

```
analysis/
  binario.md                 Análisis PE: secciones, imports, metadatos
  goroutines.md              Arquitectura de goroutines y timing
  protobuf_esquema.md        Esquema protobuf completo reconstruido
  hwid.md                    HWID: queries WMI, structs, bypass detallado
  pipeline_hwid.md           Pipeline HWID completo: A39Z4i → FXWqsvy_ → dUgTofmw
  listas_negras.md           Blacklists completas de procesos y ventanas
  logica_deteccion.md        Lógica interna de detección (20 vectores documentados)
  protocolo_red.md           Protocolo de red, autenticación, MITM
  motor_deteccion.md         Motor de detección principal
  pdFrspK_G.md               Motor hlavBkMcO: 644 verificaciones activas de detección
  scanner_source_engine.md   Scanner de ConVars Source Engine (sOAbtRgFLa6_)
  monitoreo_proceso.md       gopsutil y monitoreo de proceso
  main_package.md            Paquete main: ciclo de vida Wails, 75 closures Startup, QSUMsCa
  paquetes_identificados.md  Mapa completo de paquetes garble (183 paquetes identificados)
  frontend_ui.md             Análisis de la UI: HTML, JavaScript, WebSocket IPC
  bypass_guide.md            Guía de bypass para todos los vectores (20 vectores)
  ui_logo.png                Logo extraído del binario (150,356 bytes)

proto/
  esquema.proto              Definición protobuf reconstruida (16 mensajes)
```

---

## Herramientas Utilizadas

- Búsqueda de strings en PowerShell (`[System.Text.Encoding]::GetEncoding(1252)`)
- Análisis de type reflection metadata de Go (nombres de tipos en `.rdata`)
- Análisis de anotaciones protobuf (`protobuf:"..."` / `json:"..."`) en `.rdata`
- Análisis de nombres de funciones Go (parcialmente visibles por el binding Wails)

---

## Hallazgos Clave

### Seguridad del Protocolo
- **Sin certificate pinning** — MITM con Burp/Fiddler funciona sin modificaciones
- El stack TLS está embebido en Go, no usa WinSSL/SChannel ni OpenSSL
- Tres métodos de MITM documentados: Burp proxy, hook TLS in-process, DNS redirect

### HWID
- **5 componentes:** CPU (`ProcessorId`), Disco (`SerialNumber`), GPU (`PNPDeviceID`), Motherboard (`SerialNumber`), OS (`BuildNumber`)
- Todas las queries van vía WMI COM: `IWbemServices::ExecQuery` en vtable índice 20
- Bypass documentado con hook de vtable WMI

### Detección
- **CheatSigs dinámicos** — patrones de bytes descargados del servidor en cada sesión
- **778 firmas de cheat hardcoded** en 5 paquetes de firmas estáticas: `ra_94HIlnc6`(299) + `EyjsrRr`(162) + `WGfDxX0zz2M`(117) + `YCJ5PUz_M`(116) + `kNpc1A53`(84). Supera en firmas estáticas a EAC y VAC
- **`hlavBkMcO` — motor de detección principal** con **644 sub-closures** en el paquete `pdFrspK_G`. Cada closure = una verificación activa de estado del juego. Distinto de las firmas estáticas: estas se EJECUTAN directamente, no se registran en tablas
- **Blacklist de ~62 entradas en 4 listas distintas** — BL1 (16 títulos de ventana, 6 chars fijos): x32dbg, windbg, ghidra, de4dot, dbgclr, hacker, sysmon, efence... | BL2 (~30 procesos/substrings): aimware, fiddler, ollydbg, x64_dbg, PhantOm, wpe pro, harmony, cef.pak, Discord-, Operacmd.exe, Arc.exe... | BL3 (6 herramientas RE cortas): dump, peek, kgdb, mdbg | BL4 (10 herramientas .NET + cheats): dnspy, ilspy, pizza (cheat L4D2), ida -, crack
- **Endpoint gRPC `/auth`** confirmado — encontrado en string table inmediatamente después de la blacklist
- **Búsqueda de VPKs** con patrón glob `k1e1y*.vpk` para cheats conocidos de L4D2
- **Screenshot automático** cada ~120s como evidencia de ban (PNG NRGBA)
- **`os/exec`** presente — el AC puede spawnar procesos del sistema (wmic, bcdedit, sc, powershell) para detección y recolección de datos adicionales

### Anti-Análisis
- **garble** — todos los nombres de paquetes, tipos y funciones son strings aleatorios
- **Timestamp PE = 0** — impide correlacionar con fecha de build
- **APIs por ordinal** — ReadProcessMemory, OpenProcess, CreateToolhelp32Snapshot no aparecen como strings
- **ObfuscatedCall** — mecanismo Wails personalizado con IDs numéricos en lugar de nombres de función

### Correcciones de Análisis Previo
- `reportZombies` = runtime de Go (GC), NO función del AC
- `ReportZerolen`/`IsZerolen` = librería gopacket/802.11, NO función del AC
- `ReportValidationErrors` = `go-playground/validator`, NO función del AC
- `mTkHARFkg8` = paquete `sync` de Go (WaitGroup, Mutex)
- `BEwVDQOh5` = `encoding/xml` de Go (no sistema de tokens)

### Goroutines (150+ en total durante monitoreo activo)
- Startup: 87 goroutines (incluyendo Wails + runtime) — `Startup` tiene 75 closures totales: `func1-60` directas + `ZrMEw4hJW.func61-75` (función inner nombrada) + `gowrap2`
- ConnectL4D2Server: 4 goroutines principales (`func1-4`)
- QSUMsCa (orquestador de detección): 4 gowraps + 2 inner goroutines (`MDfyjAe.func7`, `RaR0kkqIl.func8`) = 6 goroutines
- Motor de detección (`pdFrspK_G.hlavBkMcO`): 644 verificaciones activas paralelas
- StartL4D2: 4 closures + goroutine de capturas (`func4.1`)
- BhCuafOD (xxhash) confirma: el HWID final es un hash xxHash64 de los 5 componentes de hardware

---

## Mapa de Vectores de Bypass

| Vector | Dificultad | Método |
|--------|-----------|--------|
| Blacklist ventanas | Baja | Renombrar ventana del debugger |
| Blacklist procesos | Baja | Renombrar ejecutable |
| HWID pipeline (`FXWqsvy_`) | Media | Hook Z6ey1EJD.func1-11 con frida |
| CheatSigs memoria | Alta | No inyectar en proceso / hook VirtualQuery |
| 778 firmas estáticas | Alta | Polimorfismo / código sin firmas conocidas |
| `hlavBkMcO` 644 checks | Muy Alta | Análisis dinámico completo + hook Feed() |
| DLL modules | Alta | Manual mapping (sin LoadLibrary) |
| os/exec procesos | Media | Hook CreateProcess para output limpio |
| Screenshots | N/A | Solo evidencia, no prevención directa |
| Heartbeat | N/A | No requiere bypass |
| SteamID | Media | Requiere cuenta legítima |
| InstallDate | Media | Hook de lectura de registro |
| gopsutil | Baja | No usar flags sospechosos en procesos |
| Source Engine ConVars | Media | Hook ReadProcessMemory |
| Captura de paquetes (pcap) | Muy Alta | VM con NIC virtual |
| Protocolo red | Baja | Burp Suite (sin certificate pinning) |
