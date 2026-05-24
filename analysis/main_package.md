# Paquete `main` — Estructura y Análisis

Documentación completa del paquete `main` del AC, que contiene la lógica
de inicialización Wails, el orquestador de detección, y los 5 métodos de
ciclo de vida expuestos al runtime de Wails.

---

## Estructura General

```
main
│
├── main()                    ← Entry point (7 closures: func1-7)
│     └── wails.Run(app)       ← inicia Wails + WebView2
│
├── (*HpP4qwz)                ← Struct principal del AC
│   ├── Startup()              ← 75 closures + gowrap2
│   ├── FrontendReady()        ← 10 closures + IvOAAX.1/1.1
│   ├── ConnectL4D2Server()    ← 4 closures = 4 goroutines principales
│   ├── StartL4D2()            ← 4 closures
│   └── BeforeClose()          ← 4 closures + 2 deferwraps
│
├── QSUMsCa()                 ← Orquestador de detección
│   ├── func1-6 directas
│   ├── MDfyjAe.func7          ← inner function goroutine 7
│   ├── RaR0kkqIl.func8        ← inner function goroutine 8
│   └── gowrap1-4              ← 4 goroutines lanzadas
│
├── ZrMEw4hJW()               ← Función global (usada por Startup)
├── FLqUmcop()                ← Función libre (1 closure)
├── c5l5shUHY6XB()            ← Función libre (1 gowrap, deferwrap1)
├── fyBBzjy                   ← Struct helper
│   ├── (*fyBBzjy).Replace      ← string replacement
│   └── (*fyBBzjy).exsr9Q9      ← función garbled
├── ip4dz1                    ← Struct helper pequeño
│   └── (*ip4dz1).s3WIwGIayRx
├── jBzxLnQ2u()               ← Función libre
├── cmzj5vg7g()               ← Función libre
└── decFunc()                 ← PRESENTE — strings encriptados con garble -literals
```

---

## `decFunc` Presente en main

El paquete `main` tiene `decFunc` activo, lo que significa que todos los
string literals usados directamente en `main` están encriptados con garble
`-literals`.

Strings encriptados probables en main:
- Rutas de directorio de Left 4 Dead 2 / Steam
- Mensajes de estado mostrados en la UI
- Claves de registro de Windows leídas via `GTShob4MX`
- Endpoints de red (aunque estos también están en `xzFpaM3Mq3`)
- Nombres de archivos VPK de cheats conocidos

---

## `(*HpP4qwz).Startup` — 75 Closures

Función llamada por el runtime de Wails cuando la aplicación inicia.
Es la más compleja del paquete con **75 closures + 1 gowrap**.

```
Startup (75 closures totales)
├── func1 ... func60            ← 60 closures directas
│   ├── func9.1, func11.1, etc. ← closures anidadas
│   ├── func15.1, func17.1, etc.
│   ├── func19.1, func22.1, etc.
│   ├── func24-func60 directas
│   └── func35.1, func40.1, func43.1, etc.
├── ZrMEw4hJW (función inner)
│   ├── func61 ... func75       ← 15 closures más (registros?)
└── gowrap2                     ← 1 goroutine
```

### Rol de Startup.func1-60

Cada closure probable corresponde a uno de estos roles:
1. Inicialización de blacklists (ventanas, procesos, módulos)
2. Carga de firmas de detección del servidor (`CheatSigs`)
3. Setup del cliente gRPC (`xzFpaM3Mq3`)
4. Configuración WMI (`asYMlWeBL6f6`)
5. Inicialización del motor de detección (`pdFrspK_G.DaZXKAoyfrI`)
6. Setup del pipeline HWID (`FXWqsvy_`)
7. Configuración del scanner de paquetes de red (`sNAkh4`)
8. Registro de handlers de teclado global (keylogger del AC)
9. Verificación de kernel debug mode (via `GTShob4MX` registry)
10. Inicialización del sistema de screenshots (`BitBlt`)

### Rol de ZrMEw4hJW.func61-75

`ZrMEw4hJW` es una función nombrada (garbled) dentro de Startup.
Sus 15 closures (func61-75) probablemente registran:
- Handlers de eventos Wails (IPC)
- Callbacks de ventana (resize, close, focus)
- Goroutines de monitoreo secundarios

---

## `(*HpP4qwz).FrontendReady` — 10 Closures

Llamada por Wails cuando el WebView2 frontend termina de cargar.

```
FrontendReady
├── func1
│   └── IvOAAX.1           ← inner function (Wails IPC binding setup)
│       └── IvOAAX.1.1      ← closure anidada
├── func2
│   ├── func2.1-4          ← sub-closures
│   └── func2.3.1, func2.4.1
├── func3
│   ├── func3.1, func3.2
└── func4
```

La función `IvOAAX` dentro de `func1` corresponde al setup de bindings
Wails — registra los métodos expuestos al JavaScript del frontend:
- `ConnectL4D2Server` (ID=0)
- `FrontendReady` (ID=1)
- `StartL4D2` (ID=2)

---

## `(*HpP4qwz).ConnectL4D2Server` — 4 Goroutines

Triggered desde el frontend JS cuando el usuario hace clic en "Conectar".
Lanza 4 goroutines paralelas:

```
ConnectL4D2Server
├── func1 → Goroutine 1: Autenticación + HWID (FXWqsvy_ pipeline)
├── func2 → Goroutine 2: Descarga CheatSigs + setup blacklists
├── func3 → Goroutine 3: Loop de detección (QSUMsCa → pdFrspK_G.hlavBkMcO)
└── func4 → Goroutine 4: Handler de resultados + screenshot + report
```

---

## `(*HpP4qwz).StartL4D2` — 4 Closures

Lanza el juego y establece monitoreo DLL.

```
StartL4D2
├── func1 → os/exec.Start(left4dead2.exe)
├── func2 → exec.Cmd.Wait() (espera cierre)
├── func3 → Vigilancia de módulos DLL cargados en proceso del juego
└── func4 → Capturas periódicas de screenshot como evidencia
    └── func4.1 → goroutine de captura
```

---

## `QSUMsCa` — Orquestador de Detección

Función independiente del paquete main que actúa como orquestador del
motor de detección. Lanza 4 goroutines numeradas (`gowrap1-4`):

```
QSUMsCa
├── func1 ... func6           ← setup y configuración
├── MDfyjAe                   ← inner function para goroutine 7
│   └── MDfyjAe.func7         ← goroutine de detección adicional
├── RaR0kkqIl                 ← inner function para goroutine 8
│   └── RaR0kkqIl.func8       ← goroutine de detección adicional
└── gowrap1-4                 ← 4 goroutines de detección principales
```

El flujo completo de detección:
```
ConnectL4D2Server.func3
  → QSUMsCa (8 goroutines)
      → pdFrspK_G.DaZXKAoyfrI (setup scanner)
          → pdFrspK_G.hlavBkMcO (644 checks activos)
              → sOAbtRgFLa6_.UxvaZNFzHI (Source Engine ConVars)
              → bszAWJqu.FkjkBraZo (ventanas blacklist)
              → bszAWJqu.JnwG33U (procesos blacklist)
              → bszAWJqu.PlRmoX6cuU (módulos DLL)
              → ra_94HIlnc6 + EyjsrRr + WGfDxX0zz2M + YCJ5PUz_M + kNpc1A53 (778 firmas)
              → dUgTofmw.(*fcje4l4dl_uV).Feed() → report al servidor
```

---

## `(*HpP4qwz).BeforeClose` — 4 Closures + 2 Deferwraps

Llamada por Wails antes de cerrar la ventana.

```
BeforeClose
├── deferwrap1               ← cleanup primario (COM release?)
├── deferwrap2               ← cleanup secundario
├── func1                    ← cancel context de goroutines
├── func2                    ← flush datos pendientes al servidor
├── func3                    ← cierre de conexión gRPC
└── func4                    ← liberación de handles Win32
```

---

## Tipos y Funciones Auxiliares

### `fyBBzjy` — String Replacer

```go
type fyBBzjy struct { /* campos garbled */ }
func (*fyBBzjy) Replace(s string) string   // reemplaza strings (blacklist matching?)
func (*fyBBzjy) exsr9Q9(...)               // función auxiliar garbled
```

Probable uso: normalización de nombres de procesos/ventanas antes de
comparar contra la blacklist.

### `ip4dz1` — Helper pequeño

```go
type ip4dz1 struct { /* campos garbled */ }
func (*ip4dz1) s3WIwGIayRx(...)  // función garbled
```

### Funciones Libres

| Función | Rol Probable |
|---------|-------------|
| `ZrMEw4hJW` | Registro de handlers Wails (inner de Startup) |
| `FLqUmcop` | Función de inicialización con 1 closure |
| `c5l5shUHY6XB` | Lanzador de goroutine (1 gowrap + deferwrap1) |
| `jBzxLnQ2u` | Función libre garbled |
| `cmzj5vg7g` | Función libre garbled |
| `QSUMsCa` | Orquestador principal de detección (4 gowraps) |

---

## Flujo Completo de Ciclo de Vida

```
1. main() inicia Wails runtime
2. Wails carga WebView2 con HTML/JS del frontend
3. Wails llama Startup(ctx)  ← 75 goroutines/closures de setup
4. WebView2 carga frontend JS
5. Wails llama FrontendReady(ctx) ← bindings JS↔Go registrados
6. [Usuario] hace clic "Conectar"
7. JS llama ConnectL4D2Server() via Wails IPC (ObfuscatedCall ID=0)
   ├── Goroutine 1: SteamID + HWID → auth server → token
   ├── Goroutine 2: Descarga CheatSigs + setup blacklists
   ├── Goroutine 3: QSUMsCa → pdFrspK_G.hlavBkMcO (loop detección)
   └── Goroutine 4: screenshot + report handler
8. [Usuario] hace clic "Iniciar L4D2"
9. JS llama StartL4D2() (ObfuscatedCall ID=2)
   ├── func1: lanza left4dead2.exe
   ├── func2: wait() loop
   ├── func3: módulos DLL del proceso
   └── func4: screenshots periódicos
10. [Usuario] cierra ventana
11. Wails llama BeforeClose(ctx)
    ├── cancela goroutines de detección
    ├── flush datos al servidor
    └── libera recursos COM/Win32
```

---

## Estadísticas

| Métrica | Valor |
|---------|-------|
| Total funciones en main | 183 |
| Closures en Startup | 75 (func1-60 + ZrMEw4hJW.func61-75) |
| Goroutines lanzadas por QSUMsCa | 4 (gowrap1-4) + 2 inner (MDfyjAe.func7, RaR0kkqIl.func8) |
| Goroutines de ConnectL4D2Server | 4 (func1-4) |
| Goroutines de StartL4D2 | 4 (func1-4) |
| decFunc presente | SÍ — strings encriptados |
| Métodos expuestos al frontend JS | 3 (Startup=auto, ConnectL4D2Server=ID0, StartL4D2=ID2) |
| Struct principal | `HpP4qwz` |
