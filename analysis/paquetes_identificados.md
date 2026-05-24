# Paquetes Go Identificados — Mapa Completo

Este documento mapea los nombres garble-ofuscados del binario a sus librerías reales.
La identificación se realizó por análisis de métodos, campos y comportamiento.

---

## Paquetes del AC (código propio)

| Paquete Garbled | Descripción | Evidencia |
|-----------------|-------------|-----------|
| `main.(*HpP4qwz)` | Struct principal de la app Wails | Métodos: Startup, FrontendReady, StartL4D2, ConnectL4D2Server, BeforeClose |
| `dUgTofmw` | Motor de detección principal | 67 funciones, `ga4oovjHCfg` lanza 34 goroutines de detección |
| `HWARRxN` | Paquete principal del AC (protocolo + HWID) | `KbBU6bdOa.Build` = constructor HWID, `MQI4Jn6aY2` = descriptor protobuf, `WPaeyWXKC3et` = MethodDescriptor gRPC |
| `P4mAKk` | Mensajes Protobuf del AC | 16 structs con campos Get*/Reset/String/ProtoReflect |
| `x1JPPahi` | Contenedor de tipos de detección | Contiene ENOV9d, BiK8wj, W99qYP, Qi1Z8I, etc. |
| `_6di6zc0se2v` | WatchList — vigilancia continua | Métodos Add, AddWith, Remove, WatchList, Close, Has |
| `DiU0jWi` | Detección de versión de Windows | `IsWindowsVersionAtLeast`, `RtlGetVersion` |
| `asYMlWeBL6f6` | Wrapper WMI (COM/DCOM) | 24 funciones init, `WbemeDk0` = IWbemServices, `Aego3YEweIaU` = ExecQuery |

---

## Paquetes Stdlib de Go (identificados)

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `mTkHARFkg8` | `sync` | `(*HQHzZyeNDHQO).Add/Done/Wait` = WaitGroup; `(*ev96n7K).Lock/Unlock` = Mutex |
| `HQHzZyeNDHQO` | `sync.WaitGroup` | Métodos Add, Done, Wait, Wait.func1 |
| `ev96n7K` | `sync.Mutex` | Métodos Lock, Unlock |
| `OkOdcOG` | Primitiva sync (Pool/Map) | Métodos `aJDv...` relacionados con sync |
| `BEwVDQOh5` | `encoding/xml` | Métodos EncodeToken, RawToken, Decode, Flush, Indent = API de `encoding/xml` |
| `q7klgJ` | `encoding/json` | `decFunc`, `DayNID`, tipos HqQopxeAAae, GZl_OkHQ, SLiecRC85GxP |
| `O2WU0BYsD` | `os/exec` | `(*SGUe_Ecp).Start/Run/Wait/Output/CombinedOutput/Environ` = exec.Cmd |
| `SGUe_Ecp` | `exec.Cmd` | Todos los métodos de exec.Cmd presentes |

---

## Paquetes de Red (identificados)

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `i1aqCskEISkX` | `net/http` + `golang.org/x/net/http2` | HTTP/2 framer (WriteContinuation, WriteData, WriteHeaders, WriteSettings), HTTP transport (RoundTrip, CloseIdleConnections), Authenticate, BasicAuth, SetBasicAuth |
| `gG7Vmb7` | Biblioteca de red UDP (300+ funciones) | Probablemente `github.com/pion/udp` o similar |
| `hh58lo` | A2S Protocol (Source Engine server query) | 40+ funciones, campos json:"VAC", json:"SteamID", json:"Name", json:"Game", json:"AppID" |
| `eUmM3IpzIrV` | Struct de respuesta A2S | Contiene `eaDzRhPGHK7j` con campos de servidor Source Engine |
| `MyTTk7_I` | `crypto/tls` | `(*ExZca80).DecryptTicket/EncryptTicket/ResumptionState` = métodos de tls.SessionState |
| `ExZca80` | `crypto/tls.SessionState` | DecryptTicket, EncryptTicket, ResumptionState |
| `uL6SGHSpk` | `crypto/x509` | Manejo de certificados X.509/TLS |
| `OwbnEHL7gtm` | `gopacket` / librería 802.11 | Métodos Bandwidth, ShortGI, GuardInterval, MCSIndex, FECType, ReportZerolen, IsZerolen, LastKnown, DelimCRCErr |

---

## Paquetes de UI (Wails)

| Paquete Garbled | Descripción | Evidencia |
|-----------------|-------------|-----------|
| `VKiZI7` | Integración WebView2 | Solo UI, no detección |
| `eiW4NTKLkx` | HTML template engine de Wails | Wails UI |
| `H5_Lib8AHNQN` | Runtime de Wails (ExecJS, WebView) | Wails runtime |
| `hwvnZR_pbO` | WebSocket IPC de Wails | `LFkHvs5`: WebsocketIPC, DesktopIPC, RuntimeDesktopJS |
| `Moh1QXpPW` | Sistema de eventos de Wails | Emit, Notify, On, Off, OnMultiple, OffAll |
| `aarD4xK0csQ` | Generador de bindings de Wails | UpdateObfuscatedCallMap, ToJSON |
| `oFJMvWezJ9O` | Utilidades de ventana | GetWindowDPI |

---

## Paquetes de Infraestructura Go

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `sNAkh4` | `github.com/shirou/gopsutil` | CPUPercent, MemoryInfo, Cmdline, Environ, IOCounters, NumFDs, OpenFiles |
| `aFzxJpzp2` | Implementación de hash (SHA256/HMAC) | Métodos BlockSize, Sum, Write, Reset |
| `AFdc2Qd` | Operaciones de big number | `crypto/big` o similar |
| `aOD4q1Y` | Protobuf registry/descriptor | Registro de tipos proto |
| `Ve0nLKBMzmC` | `github.com/go-playground/validator/v10` | `(*adPnywIlKf75)`: Field, GetTag, ExtractType, Param, Parent, Top, ReportError, ReportValidationErrors, Validator |
| `ncRaYk_Ke` | **`regexp` o `regexp/syntax`** (stdlib Go) | `MatchRune` method identificado en tabla de tipos — `regexp/syntax.Inst.MatchRune` es el único método con ese nombre en stdlib. Funciones: `FS0IhhJL`, `MDfyjAe`, `RaR0kkqIl`, `MlPMSGX2GAB`, `Alav9s`, `T8Go8QdJaY`, `PJRLzfZ`. El AC compila regex para matching de blacklists |
| `bmV09O0` | Protobuf interno | Encontrado en contexto de serialización protobuf; posiblemente `google.golang.org/protobuf/internal` |
| `fcje4l4dl_uV` | Tipo del pipeline productor-consumidor de dUgTofmw | Método `Feed` identificado — recibe datos de goroutines productoras |
| `bszAWJqu` | Paquete AC desconocido — posiblemente sistema de eventos/detección | Funciones: `FkjkBraZo` (15 closures + deferwrap), `IvOAAX` (aparece en FrontendReady context), `OYQujSa6Aed5`, `ApxFkk`, `X10KDNt6j`. Usa regexp (`ncRaYk_Ke`). Podría ser el sistema de escaneo de procesos/ventanas |
| `d8a0oX50i` | Paquete con aritmética de big numbers | Aparece junto a `math/big.PhCJi5OjhM2` en pclntab. Funciones: `ZhDr99uMy`, `QqEcxnsa`, `FLPyhrKfoTu`. Probablemente crypto: generación de tokens o HMAC usando `math/big` |
| `Plcr2cx` | Paquete adyacente a d8a0oX50i | Funciones: `CgWgrWQ`, `rHQg4df6V`, `hEQiyR8`. Posiblemente manejo de claves o certificados |
| `i7OyDUpCDA3q` | Paquete inlined en main.QSUMsCa | Función `D5UnAj` identificada; aparece entre funciones de ncRaYk_Ke sugiriendo que es un helper de matching o comparación |

---

## Structs Protobuf del AC (P4mAKk) — Mapa Completo

| Struct Garbled | Propósito | Campos Principales |
|----------------|-----------|-------------------|
| `BEt_icchsrxy` | Mensaje principal cliente→servidor | Auth(1), Session(2), Session2(3), Version(4), Game(5), Server(6), Code(7), Data(8), Smurf(9), HWData(10), AppKey(11), ExtraInfo(12), Addons(13), AddonsProvided(14), CheatSigsRequested(15) |
| `ENOV9d` | HWID completo | Success(1), OS(2), CPU(3), Disk(4), MotherB(5), GPU(6) |
| `EetHP7O` | Disco individual | Success(1), Serial(2), Model(3) |
| `KLfm8u7` | Componente con serial (MB/CPU) | Success(1), Serial(2), Model(3) |
| `BlhNU1RKfjg` | GPU individual | Success(1), Model(2), PnpID(3) |
| `AAUUUh` | Anti-smurf (fecha instalación) | Success(1), Type(2), InstallDate(3) |
| `BiK8wj` | Archivo addon/VPK | Path(1), Size(2) |
| `ZwlL2R6Gvy` | Servidor del juego | Success(1), ID(2), Name(3) |
| `W99qYP` | Firma de cheat | Name(1), Pattern(2) |
| `Qi1Z8I` | Respuesta de ban | Success(1), Error(2), SteamID(3), Nickname(4), Banned(5) |
| `H1oahxz1l3iY` | Respuesta del servidor con CheatSigs | Success(1), Error(2), ServerIP(3), CheatSigs(5) |
| `Ok4xyD92c8` | Verificación de versión | Success(1), Version(2) |
| `P8ePybs` | Respuesta de autenticación | Success(1), Error(2), Session(3) |
| `QBb3vhiLW7n` | Request de auth/heartbeat | Auth(1), Session(2) |
| `N0Iwv9FQ` | Respuesta genérica | Success(1), Error(2) |
| `MdQhYNc` | Mensaje vacío (heartbeat ping) | Sin campos |

---

## Servicios gRPC — Descriptor

El AC usa gRPC con HTTP/2 + Protobuf sobre TLS. El descriptor del servicio está en:

```
HWARRxN.(*WPaeyWXKC3et) = protoreflect.MethodDescriptor
```

Métodos del descriptor:
- `FullName()` - nombre completo `package.Service.Method`
- `Input()` / `Output()` - tipos de mensaje de entrada/salida
- `IsStreamingClient()` / `IsStreamingServer()` - tipo de streaming

La presencia de ambos métodos de streaming sugiere al menos un método bidireccional
(posiblemente el loop de heartbeat/detección que mantiene la conexión abierta).

---

## Goroutines — Resumen

| Función | Goroutines | Propósito |
|---------|------------|-----------|
| `main.(*HpP4qwz).Startup` | 87 total (60 directas + 15 closure + 12 sub) | Inicialización completa del sistema |
| `main.(*HpP4qwz).ConnectL4D2Server` | 4 goroutines principales | Auth, Config, Loop de detección, Handler |
| `main.(*HpP4qwz).StartL4D2` | 5 goroutines | Lanzar juego, monitor, DLL watch, screenshots |
| `main.(*HpP4qwz).FrontendReady` | 13+ goroutines | Inicialización de la UI y eventos |
| `main.(*HpP4qwz).BeforeClose` | 4+ goroutines | Cleanup al cerrar |
| `dUgTofmw.ga4oovjHCfg` | 37 closures (33 goroutines + 4 sub-goroutines anidadas) | Motor de detección principal con 33 checks paralelos — confirmado por extracción de pclntab |
| `dUgTofmw.VA0jJhHuwb0l` | 3 goroutines | Sub-sistema de detección |

**Total estimado de goroutines en ejecución:** 150+ goroutines concurrentes durante el monitoreo activo.

---

## Strings de Detección Encontrados en el Binario

### Blacklist de Procesos/Ventanas
```
gdb.exe   reverse   process  sharpod  fiddler  x64_dbg  petools  monitor
ollydbg   wpe pro   PhantOm  x32_dbg  phantom  WPE PRO  charles  checker
harmony   PETools   sniffer  MDBCrew  DIEmW    WinMonitor  Discord  - Opera
cmd.exe   fdm.exe   zen.exe  Arc.exe  aimware  cef.pak
```

### Blacklist de Ventanas (keywords cortos)
```
x32dbg  pc-ret  Centos  windbg  dbgclr  de4dot  pepper  ghidra  hacker
x96dbg  folder  sysmon  timers  efence  select  scalar
```

### Blacklist de Herramientas RE
```
dnspy  ilspy  ILSpy  pizza  crack  ida  brute  james  Debug
.vpk   dump   peek   kgdb   mdbg  xdeb
```

### Glob Pattern de Addons
```
%s\%sk1e1y*.vpk     ← busca VPKs con "k1e1y" en el path (cheats conocidos)
```

### Colores de Consola del AC
```
#A9A9A9  (timestamps — gris)
#FF0000  (ban/error — rojo)
#FFFFFF  (mensajes normales — blanco)
#00FF00  (éxito — verde)
```
