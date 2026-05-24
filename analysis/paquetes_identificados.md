# Paquetes Go Identificados — Mapa Completo

Este documento mapea los nombres garble-ofuscados del binario a sus librerías reales.
La identificación se realizó por análisis de métodos, campos y comportamiento.

---

## Paquetes del AC (código propio)

| Paquete Garbled | Descripción | Evidencia |
|-----------------|-------------|-----------|
| `main.(*HpP4qwz)` | Struct principal de la app Wails | Métodos: Startup, FrontendReady, StartL4D2, ConnectL4D2Server, BeforeClose |
| `dUgTofmw` | **Motor de evaluación central** — el paquete más grande del AC | **308 funciones**. Tipos: `(*fcje4l4dl_uV)` (Feed pipeline), `(*ZhEEW_uzlNn)` (evaluador con `h7MNIMO` de 8 etapas), `(*gZDPQlzEDoPS)` (error). Funciones críticas: `ga4oovjHCfg` (33 closures = scanner principal), `ZXugk2GxaotX` (14 closures), `KT300841` (17 closures), `CGRKkahhRkP` (12 closures = handler gRPC). 13 init funcs = 13 tipos de evento registrados. Ver `evaluador_central.md` |
| `HWARRxN` | **`google.golang.org/protobuf/internal/impl`** — runtime de mensajes protobuf | 550+ funciones. `(*KbBU6bdOa)` = `impl.MessageInfo` (Build, thSKtfSax3, _h76SfBr); `(*SLltHXdFZqty)` = FileDescriptor (ParentFile); `(*Yye7gKrP5)` = ServiceDescriptor; `(*WPaeyWXKC3et)` = MethodDescriptor (Input, Output, IsStreamingClient, IsStreamingServer); `(*Cv4Csm)` = MessageDescriptor (Enums, ExtensionRanges, de7YgkRNu); `(*As98DipdeM)` = FieldDescriptor (ByJSONName, ByName, ByNumber, ByTextName, Format, Get, Len, ProtoInternal); `(*G4Hq6JI)` = EnumDescriptor. **NOTA:** `KbBU6bdOa.Build` NO es HWID builder — es `MessageInfo.Build()` del runtime protobuf. El HWID collector real está en `asYMlWeBL6f6` |
| `bszAWJqu` | **Scanner de blacklists** — 5 scanners paralelos de ventanas/procesos/DLLs/archivos/red | **120 funciones**. 5 scanners: `FkjkBraZo` (ventanas/EnumWindows), `JnwG33U` (procesos/Toolhelp32), `PlRmoX6cuU` (módulos DLL/Module32Next), `X10KDNt6j` (archivos), `X80N6Rs52TTi` (red). Cada scanner aplica los 5 regexes de `ncRaYk_Ke` (Alav9s, IvOAAX, MlPMSGX2GAB, PJRLzfZ, T8Go8QdJaY). Auxiliares: `ThNOEr`, `OYQujSa6Aed5`, `LUb2wVKw`, `ApxFkk`, `CGKuSNjp`, `E5nzMUH5qSwM`, `IvOAAX`. Ver `scanner_blacklist.md` |
| `P4mAKk` | **Paquete de mensajes Protobuf del AC** | 16 tipos de mensaje con getters completos. Ver tabla completa en sección `Structs Protobuf` y `protocolo_protobuf.md`. Principales: `BEt_icchsrxy` (auth request), `ENOV9d` (HWID), `Qi1Z8I` (ban response), `H1oahxz1l3iY` (server response con CheatSigs), `W99qYP` (cheat signature con Pattern de bytes) |
| `x1JPPahi` | Contenedor de tipos de detección | Contiene ENOV9d, BiK8wj, W99qYP, Qi1Z8I, etc. |
| `_6di6zc0se2v` | WatchList — vigilancia continua (= `sNAkh4` en type table) | 81 entradas en pclntab; tipos: `(*_o3YyW)` (manager), `(*GrRzvM2)` (nodo), `(*Rs0QF1nUdhgw)`, `(*UWzc59)` (colecciones Has/String); 5 init funcs = 5 categorías (ventanas, procesos, red, módulos, archivos); integra gopacket (`cd1Utgxh0`, `wqT1oO`, `encTsS0_`). Ver `watchlist.md` |
| `DiU0jWi` | Detección de versión de Windows | `IsWindowsVersionAtLeast`, `RtlGetVersion` |
| `asYMlWeBL6f6` | Wrapper WMI (COM/DCOM) | 31 funciones init (func1-func24 con sub-closures), `WbemeDk0` = IWbemServices, `VDjfdjV` = enumerador WMI, `Aego3YEweIaU` = ExecQuery |
| `A39Z4i` | Ejecutor de queries WMI (librería Go WMI) | `(*DdY6Bn3b).Query` (con 4 deferwrap = 4 COM releases), `(*GNMRXRX).Query` = enumerador de resultados, `(*GUxA0QN3).Error` = tipo error WMI, `iFCNfGrVm5L` = función interna COM (16 closures), `wn2lrXVC` = setup de conexión |
| `sOAbtRgFLa6_` | Subsistema de detección adicional | 2 funciones principales: `GXErxuFMY9Z` (func1/2/3) y `UxvaZNFzHI` (func1-func5 + sub-closures) — patrón similar a bszAWJqu pero con menos complejidad |
| `xzFpaM3Mq3` | **`google.golang.org/grpc`** — cliente gRPC | 219 entradas. Strings confirmatorias en binario: `ClientStream`, `DialContext`, `Invoke`. Función principal `iwhm6fWm4v` (23 closures = bidirectional stream), `qd_m0I_QYkH7` (42 closures = conexión HTTP/2), `rKGI40jJGm` (42 closures = TLS handshake), `nL6NzEwvxPE` (27 closures = frame processor). Endpoint: `/auth` en `l4d2center.com:443` |

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
| `u26yruH2Ly_u` | **`net` (stdlib)** — paquete de red de Go | **1030 funciones**. `(*a6stst3y)` = `net.TCPConn` (Close, CloseRead, CloseWrite, File, LocalAddr, MultipathTCP, Read, ReadFrom, RemoteAddr, SetDeadline, SetKeepAlive, SetKeepAliveConfig, SetKeepAlivePeriod, SetLinger, SetNoDelay, SetReadBuffer, SetReadDeadline, SetWriteBuffer, SetWriteDeadline, SyscallConn, Write). El paquete más grande de stdlib en el binary |
| `dzpA6s` | **`crypto/cipher`** — modos de cifrado de bloque | `NewCTR`, `NewGCM`, `NewCBCEncrypter`, `NewCBCDecrypter` encontrados en tipo tabla. El AC usa AES en múltiples modos: GCM (autenticado, principal), CBC (secundario), CTR (streaming). Aparece adyacente a `PcTWfu` (AES) en la tabla de tipos |
| `AFdc2Qd` | **`crypto/elliptic` + `math/big`** — curvas elípticas y aritmética big-int | **225 funciones**. `(*AhD79vsT_)` = curva elíptica (Add, Bytes, BytesCompressed, BytesX, Double, ScalarBaseMult, ScalarMult, Select, Set, SetBytes, SetGenerator). Usadas por TLS/ECDH durante handshake gRPC. `(*BYSujzXozZ0).Add` = otro tipo de curva (P256/P384?) |
| `mPmrATx3LG` | **`google.golang.org/protobuf/internal/impl`** — runtime de mensajes protobuf | **1482 funciones** — la biblioteca más grande del binario. `(*_c4YMU8F)` implementa `protoreflect.Message` completa: Clear, Descriptor, Get, GetUnknown, Has, Interface, IsValid, LoadMessageInfo, Mutable, New, NewField, ProtoMessageInfo, ProtoMethods, Range, Set, SetUnknown, StoreMessageInfo, Type, WhichOneof. Múltiples tipos de mensaje protobuf generados por protoc |

---

## Paquetes de Red (identificados)

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `i1aqCskEISkX` | `net/http` + `golang.org/x/net/http2` | HTTP/2 framer (WriteContinuation, WriteData, WriteHeaders, WriteSettings), HTTP transport (RoundTrip, CloseIdleConnections), Authenticate, BasicAuth, SetBasicAuth |
| `gG7Vmb7` | **`github.com/google/gopacket/pcap`** — captura de paquetes en vivo | **452 funciones**. Confirmado por `WritePacketData` (método exclusivo de pcap) y cross-referencias a `q4ajG0RM.Aa8DB2bIK66` (gopacket type). El AC captura tráfico de red en vivo desde la interfaz de red del sistema para monitorear las conexiones del juego |
| `hh58lo` | Structs de respuesta A2S (campos JSON) | Tipos: `*hh58lo.Phr6ow`, `*hh58lo.GV2qnv`, `*hh58lo.Ea9gTx`, `*hh58lo.BT1E5g`, `*hh58lo.FnICOY`; campos json:"VAC", json:"SteamID", json:"Name", json:"Game", json:"AppID" |
| `HBJGtL` | **Cliente A2S Protocol** (implementación del protocolo Source Engine server query) | `(*Ns0npSNFWWqQ).QueryInfo` = A2S_INFO, `(*Ns0npSNFWWqQ).QueryPlayer` = A2S_PLAYER, `(*Ns0npSNFWWqQ).Close`, `(*JG00Xm6lG8)` = lector binario (ReadUint8/16/32/64/String/Float32), `(*XxIkaC9_)` = escritor binario (WriteBytes/WriteCString), 11 funciones init |
| `eUmM3IpzIrV` | **Capa de conectividad de red** (manager de conexiones + DTLS) | **328 funciones**. Referencia a `qTnr3tOf4N.A0lMBv6KYzU_` (DTLS type) y `i1aqCskEISkX` (http2). Tipos complejos: `c4yDowusz` con `func(*c4yDowusz)` — struct con callback de conexión. Probablemente `github.com/pion/transport` o el connectivity layer de gRPC. **CORRECCIÓN:** Identificación previa como "A2S_INFO struct" era incorrecta — el package real de A2S structs es `hh58lo` |
| `MyTTk7_I` | `crypto/tls` | `(*ExZca80).DecryptTicket/EncryptTicket/ResumptionState` = métodos de tls.SessionState |
| `ExZca80` | `crypto/tls.SessionState` | DecryptTicket, EncryptTicket, ResumptionState |
| `uL6SGHSpk` | **`crypto/x509/pkix`** — no x509 directamente | `(*JjZ1WNQA).FillFromRDNSequence/ToRDNSequence/String` = `pkix.Name`, `(*IBvuTjv).HasExpired` = `pkix.CertificateList.HasExpired` (verificación de CRL), `(*Ap8xj8SrB).String` = `pkix.AlgorithmIdentifier` |
| `LYUqBZOV7RHi` | **`crypto/x509`** — manejo de certificados X.509 | `(*XdDiONjbzcM)` = `x509.CertPool` (AddCert, Clone, AppendCertsFromPEM, Equal), `(*ZIbJOWC4).CheckSignature` = `x509.Certificate`, 31 funciones init (registro de algoritmos RSA/ECDSA/DSA) |
| `OwbnEHL7gtm` | `gopacket` / librería 802.11 | Métodos Bandwidth, ShortGI, GuardInterval, MCSIndex, FECType, ReportZerolen, IsZerolen, LastKnown, DelimCRCErr |
| `qTnr3tOf4N` | **DTLS / TLS-over-UDP** — implementación TLS para canal UDP seguro | 163 entradas; tipos: `(*ExZca80)` = SessionState, `(*Gy_XwjZ)` = Conn (conexión TLS), `(*vjSRqd6ov)` = record/paquete, `(*iTeUYgu)` = handshake config, `(*_ZyRwtuuaU)` = estado cliente, `(*vE6jf82QV)` = tipo alert/record, `(*JSj6sp1QG)` = cert/key, `(*QmRsQS5)` = factory (retorna `*ExZca80` o `*Gy_XwjZ`); referencias cruzadas a `l8F_pdQP` (x509.Certificate = `ZIbJOWC4`) y `eUmM3IpzIrV` |
| `l8F_pdQP` | **`crypto/x509`** (nombre corto garbled) | = mismo paquete que `LYUqBZOV7RHi` (path garbled); `ZIbJOWC4` = x509.Certificate; `decFunc` = string literals garbled; diferencia: `l8F_pdQP` = package NAME en type table, `LYUqBZOV7RHi` = package PATH en pclntab |

---

---

## Paquetes de Terceros Adicionales (identificados)

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `dvgs9C` | **`github.com/gin-gonic/gin`** — servidor HTTP web | Confirmado por `AbortWithStatus`, `SetHTMLTemplate`, `DefaultPostForm`, `NegotiateFormat`, `ShouldBindQuery`, `StatusCodeColor`, `SecureJsonPrefix`, `GetPostFormArray`, `SaveUpload...` en tabla de tipos. El AC ejecuta un **servidor HTTP local (Gin)** — probablemente para comunicación entre componentes o para la UI de Wails. Las rutas HTTP están cifradas por garble -literals, no visibles en análisis estático |
| `cXI2MODy` | **Sub-paquete de Gin** (`gin/render` o `gin/binding`) | Tiene `AbortWithStatus` y otros métodos Gin. Posiblemente `gin/render` para rendering de respuestas JSON/HTML |
| `Ve0nLKBMzmC` | **`github.com/go-playground/validator/v10`** — validación de datos | `(*adPnywIlKf75)`: Field, GetTag, ExtractType, Param, Parent, Top, ReportError, ReportValidationErrors, Validator. Valida los datos del protocolo gRPC y posiblemente los campos protobuf recibidos del servidor |
| `vfFlma` | Paquete de encoding/protocolo (171 funciones) | Tipos: `BAbJq9d`, `CCwyMQn` con interfaz `FindExtension` (protobuf). Referencias cruzadas a `oJ5pXwZm7R` y `t5W7d4Vh7E`. Probablemente sub-paquete del codec protobuf |

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
| `sNAkh4` | **Paquete de monitoreo de red propio del AC** — usa gopacket (`q4ajG0RM`) como dependencia | Tipos adyacentes a gopacket: `GrRzvM2` implementa `SetNetworkLayer`/`DecodeFromBytes`; `eeWJva_vkG` (arrays de 8 elementos, procesamiento por lotes), `cd1Utgxh0`, `gvBDt2BpBt7`, `wqT1oO`, `vaf3j0Z`, `ySbqKa9f0p`, `encTsS0_`. **CORRECCIÓN:** previamente identificado como gopsutil — incorrecto. El paquete gopsutil/v3/process real es `WvPUk5UlmHG` |
| `WvPUk5UlmHG` | **`github.com/shirou/gopsutil/v3/process`** | Confirmado por `CmdlineWithContext`, `CmdlineSliceWithContext`, `NameWithContext`, `CreateTimeWithContext`, `CmdlineSlice`, `Name`. Struct principal: `(*USEy__R7Bor9)`. Métodos en tabla de tipos: `CPUPercent`, `CreateTime`, `Foreground`, `IOCounters`, `MemoryInfo`, `MemoryMaps`, `NumThreads`, `PageFaults`, `SendSignal`. Llamado directamente desde `dUgTofmw.rpfbMOh` para leer cmdline del proceso del juego |
| `q4ajG0RM` | **`github.com/google/gopacket`** (biblioteca principal de captura de paquetes) | Métodos: `SetNetworkLayer`, `DecodeFromBytes`, `SetTransportLayer`, `SetApplicationLayer` — todas interfaces estándar de gopacket. Tipos: `Gt6tsVi`, `TXYGsDR`, `QOxXU6`, `Aa8DB2bIK66`, `UXODB2hYq`, `JorhuVUPZ`, `KAwWPEYU`, `DQcWKXdd`. Adyacente a `sNAkh4` en la tabla de tipos |
| `aFzxJpzp2` | **`crypto/hmac`** — HMAC implementation | `(*cESPyjL5y).ConstantTimeSum` (método exclusivo de `crypto/hmac`), `BlockSize`, `Size`, `Sum`, `Write`, `Reset`, `MarshalBinary`, `UnmarshalBinary` |
| `PcTWfu` | **`crypto/aes`** — cifrado AES con AES-NI | `(*pAWn49)` = AES block cipher; `(*myqAnYT)` = AES-GCM; `(*pRzjXqs)`, `(*gU1WIgp)` = modos CBC/CFB; funciones ensamblador: `_expand_key_128/192a/192b/256a/256b` |
| `pa9q8q` | Cifrado de bloque adicional (posiblemente `crypto/des` o `golang.org/x/crypto`) | `(*pXyC3Ai).BlockSize/Encrypt/Decrypt`, `(*lPQ6ELOmD).BlockSize/Encrypt/Decrypt` — estructura similar a crypto/des o chacha20 |
| `AFdc2Qd` | **`math/big`** — aritmética de grandes números | Usado por RSA key operations en x509/TLS |
| `aENMwam6VEd` | **`compress/gzip`** — compresión gzip | `(*HIpkYL9Ol).Multistream` = método exclusivo de `gzip.Reader`; métodos `Reset`, `Read`, `Close`; `(*HCy6tMrG_b).init` = writer init; `gKYJipeQ87s` = constructor; `AYHHDMBSV5y` = función auxiliar |
| `nc95MRB` | Codificación genérica numérica (wire format protobuf) | `GnYb1tW3Fa[go.shape.int64/uint64/float64/string/int/uint16/uint8/uintptr]` = AppendVarint/AppendFixed64 genérico; `UlarnT3w16bk[go.shape.int/string/int32]` = codificador genérico; `l9Rx6xXOt[go.shape.float64]` = float encoder |
| `m7wGi4U_E` | **`golang.org/x/sys/windows`** — DLL/syscall Windows | `(*JjnfzgPFB6W)` = LazyDLL (Load, Handle, NewProc, `jUWNUrK`); `(*BB6DDazDVN)` = DLL (FindProc, Release); `(*FfVzMmR)` = LazyProc (Addr, Call); `(*R3Cdh1AajU)` = Proc (Addr, Call, Find); `(*HK8M1WRH)` = Errno (Errno, Error); `(*B00rsmz36).Nanoseconds` = duración; 873 entradas totales, 34+ funciones init registrando todos los lazy procs de Windows |
| `aOD4q1Y` | Protobuf registry/descriptor | Registro de tipos proto |
| `Ve0nLKBMzmC` | `github.com/go-playground/validator/v10` | `(*adPnywIlKf75)`: Field, GetTag, ExtractType, Param, Parent, Top, ReportError, ReportValidationErrors, Validator |
| `ncRaYk_Ke` | **`regexp` + `regexp/syntax`** (stdlib Go) | `MatchRune` = `regexp/syntax.Inst.MatchRune`; Funciones compiladoras: `MlPMSGX2GAB`, `Alav9s`, `T8Go8QdJaY`, `PJRLzfZ`, `IvOAAX`, `FS0IhhJL`, `MDfyjAe`, `RaR0kkqIl`. Usadas por `bszAWJqu` para compilar y ejecutar las 4 blacklists contra ventanas/procesos |
| `bmV09O0` | Protobuf interno | Contexto de serialización protobuf; posiblemente `google.golang.org/protobuf/internal` |
| `fcje4l4dl_uV` | Pipeline productor-consumidor de dUgTofmw | Método `Feed` — recibe datos de goroutines productoras para procesamiento centralizado |
| `d8a0oX50i` | Utilidad crypto / encoding del AC | 9 entradas; funciones: `ZhDr99uMy`, `QqEcxnsa` (func1), `FLPyhrKfoTu`; genéricos: `QC1Nctw0QbWB[go.shape.string]`, `FJYZBfxOBhA[go.shape.int32]`, `KrxI5Eoc[go.shape.int]`; los genéricos con múltiples tipos sugieren una función de encoding que acepta int/string — posiblemente para serializar datos del HWID o generar tokens de sesión |
| `Plcr2cx` | Paquete crypto adyacente a d8a0oX50i | Funciones: `CgWgrWQ`, `rHQg4df6V`, `hEQiyR8`. Posiblemente: manejo de claves públicas/privadas o derivación de claves |
| `i7OyDUpCDA3q` | Helper de comparación/matching | Función `D5UnAj`; aparece entre funciones de `ncRaYk_Ke` sugiriendo helper de matching de strings o bytes |
| `Pyp7aGakY` | Tipo `OrderedSet` (lista con deduplicación) | `(*DSNcSHSTN)`: Add, AddUnique, Contains, AddSlice, AsSlice, AddSlicer, Filter, Each, Length, Clear, Deduplicate, Join, Sort — implementación de conjunto ordenado usada para acumular resultados de detección |
| `HBJGtL` | **Cliente A2S protocol** (Query Source Engine) | `(*Ns0npSNFWWqQ)` = struct del cliente; `QueryInfo`, `QueryPlayer`, `Close`; `(*JG00Xm6lG8)` = lector binario; `(*XxIkaC9_)` = escritor binario; 11 inits |
| `D9eGQ5PG` | Biblioteca UDP / transporte de red | Struct genérico `{uint16; uint8}` = packet header; `(*P61aMY8x9u_r).Error` = error type; funciones genéricas `ge3tkaLtE.*` |
| `Lvy2smJECjF` | **`math/rand`** | `(*J3sgRye).ExpFloat64`, `(*J3sgRye).Uint32` = generación de números pseudoaleatorios |
| `sa96sWj2YdO` | **`crypto/rand`** o `math/rand v2` | `(*j2se_tz1Yd).Uint64`, `(*V_IwguYI).Uint64` — generación de uint64 aleatorio |
| `Lvy2smJECjF` | **`math/rand`** | `(*J3sgRye).ExpFloat64`, `(*J3sgRye).Uint32` = generación de números pseudoaleatorios |
| `J4pbxzDN6T` | Paquete helper pequeño | Funciones: `JxJlCi2H` (func1), `psK3gqUeJD` (func1, func2) — muy pequeño, probablemente wrapper o utility |
| `i7OyDUpCDA3q` | Helper de comparación/matching | Función `D5UnAj`; aparece entre funciones de `ncRaYk_Ke` |
| `lAEmxoi9bxt` | **`bytes` + `strings`** stdlib | `(*DJwHDX)` = `bytes.Buffer` (Cap, Grow, Len, Reset, String, Write, WriteByte, `q3aLUvCl`); `(*Ag5Lnmg_WHv)` = `bytes.Reader` (Read, ReadAt, ReadByte, ReadRune, Seek, Size, UnreadByte, WriteTo); `(*aBUwSdEWXG)` = `strings.Builder` (Write, WriteString); `(*AiEta36hdu)` = `strings.Replacer` (Replace, WriteString) |
| `ux88b3Kaxj2` | **`io/fs`** — filesystem abstraction | `L_MN3FaO2` = `fs.FileMode` (String, IsDir, IsRegular, Perm, Type); `(*DayNID)` = `fs.PathError` (Error, Unwrap, Timeout); `HqQopxeAAae` = `fs.FileInfo`; `BMjI1OjH7G` = `fs.DirEntry` (slice type en ge3tkaLtE) |
| `ge3tkaLtE` | **`path/filepath`** o `io/fs.WalkDir` | 209 entradas con genéricos sobre `fs.DirEntry` interface; `(*anzMQ02OY).Next` = iterador de directorio; `c6N7a8lO` = función de traversal; usado por el AC para escanear directorios del juego en búsqueda de VPKs modificados |
| `BhCuafOD` | **Hash de 64 bits** (`hash/fnv` o `xxhash`) | `(*BDap2x).Write` + `(*BDap2x).Sum64` = hash.Hash64; usado para verificación de integridad de archivos del juego (comparar hash de VPK contra valor esperado del servidor) |
| `xzFpaM3Mq3` | **`google.golang.org/grpc`** — cliente gRPC | 219 entradas; función principal `iwhm6fWm4v` (23 closures = bidirectional stream handler), `qd_m0I_QYkH7` (42 closures = conexión gRPC), `rKGI40jJGm` (42 closures = TLS handshake gRPC), `nL6NzEwvxPE` (27 closures = frame processor); referencias `DialContext`, `ClientStream`, `Invoke` confirman gRPC |
| `keNa_4m` | Paquete de sesión/token del AC | `(*pvusDPtY5vCp)` con métodos `evojx0Jc`, `_GoIAD`, `h7OdHS4`; funciones: `AAs7V4vmMZA`, `TAZv8E`, `DPCCiaoS`, `IUJ55okf`, `uE66JFzHSqr`, `Lnr6NFAWAA` — posiblemente manejo de session tokens o key derivation |

---

## Paquetes Visibles Solo en Type Table (Sin Entradas pclntab)

Estos nombres de paquete aparecen en la tabla de tipos del runtime de Go
(separados por `\x01\x08` length-prefixed), pero NO tienen entradas en pclntab.
Posiblemente son packages cuyo código está completamente inlined por el compilador,
o son packages definidos solo por tipos (sin funciones exportadas separadas).

| Nombre en Type Table | Offset | Observación |
|----------------------|--------|-------------|
| `LDIRxH45` | 26239119 | Listado consecutivo a `dUgTofmw` y `bszAWJqu` |
| `GqkPuR7I` | 26239129 | — |
| `JSUhXt8Q` | 26239139 | — |
| `DRAH6Q_N` | 26239149 | — |
| `S3wyoPja` | 26239159 | — |
| `HSQkFeBl` | 26239169 | — |
| `S8NFupzZ` | 26239179 | Último de la secuencia |

Estos 7 paquetes aparecen juntos inmediatamente después de `dUgTofmw` y `bszAWJqu`
en la string table de tipos, sugiriendo que son paquetes del propio AC cuyos
nombres fueron garbled pero cuyas funciones están inlined en los packages principales.

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

## Paquetes del Main — Mapa Completo

El paquete `main` tiene **169 funciones** distribuidas en los siguientes tipos y funciones:

### Tipo Principal `(*HpP4qwz)` — Struct de la App Wails

| Método | Closures | Descripción |
|--------|----------|-------------|
| `Startup` | 60 directas + `ZrMEw4hJW.func61-75` (15 adicionales) | Inicio completo del sistema. `ZrMEw4hJW` = goroutine de monitoreo continuo con 15 closures |
| `FrontendReady` | 13+ closures | UI lista. Sub-goroutine: `IvOAAX.1` y `IvOAAX.1.1` |
| `ConnectL4D2Server` | 4 closures | Conexión al servidor: auth, loop, handler, reportador |
| `StartL4D2` | 4 closures + `func4.1` | Inicio del juego: lanzar, monitor módulos, screenshots, cleanup |
| `BeforeClose` | 4 closures + `deferwrap1/2` | Cleanup al cerrar el AC |

### Tipos Auxiliares en main

| Tipo | Métodos | Descripción |
|------|---------|-------------|
| `(*fyBBzjy)` | `exsr9Q9`, `Replace` | Template replacer para UI — strings de la interfaz |
| `(*ip4dz1)` | `s3WIwGIayRx` | Helper interno de inicialización |

### Funciones Free en main

| Función | Closures | Descripción |
|---------|----------|-------------|
| `QSUMsCa` | func1-6, `MDfyjAe.func7`, `RaR0kkqIl.func8`, gowrap1-4 | Scanner VPK/archivos con 4 goroutines (gowrap1-4) y 2 regex especiales |
| `c5l5shUHY6XB` | func1 + gowrap1 + deferwrap1 | Probable función `main()` o lanzador principal |
| `ZrMEw4hJW` | func61-75 (15 closures) | Goroutine de monitoreo continuo lanzada desde Startup |
| `decFunc` | — | Descifrador de string literals de garble-literals |
| `main` | — | Punto de entrada principal |
| `jBzxLnQ2u` | — | Helper de inicialización |
| `FLqUmcop` | func1 | Helper auxiliar |
| `cmzj5vg7g` | — | Utilidad |

### `ZrMEw4hJW` — Goroutine de Monitoreo Continuo

La función `ZrMEw4hJW` aparece tanto como función standalone como subroutine de `Startup`:
```
main.(*HpP4qwz).Startup.ZrMEw4hJW.func61  ← goroutine 1
main.(*HpP4qwz).Startup.ZrMEw4hJW.func62  ← goroutine 2
...
main.(*HpP4qwz).Startup.ZrMEw4hJW.func75  ← goroutine 15
```
Con 15 closures, esta goroutine probablemente maneja el **loop de heartbeat + checks periódicos** desde dentro de Startup.

### `QSUMsCa` — Scanner VPK Detallado

```
QSUMsCa (función principal)
├── func1, func1.1 — loop de archivos
├── func2, func3   — análisis de VPK
├── func4, func4.1 — hash y reporte
├── func5, func6   — cleanup
├── MDfyjAe.func7  — regex especial (blacklist VPK 1)
├── RaR0kkqIl.func8 — regex especial (blacklist VPK 2)
├── gowrap1        — goroutine 1 (scanner paralelo)
├── gowrap2        — goroutine 2
├── gowrap3        — goroutine 3
└── gowrap4        — goroutine 4 (4 goroutines de escaneo paralelo)
```

---

## Goroutines — Resumen Actualizado

| Función | Goroutines | Propósito |
|---------|------------|-----------|
| `main.(*HpP4qwz).Startup` | 60 directas + 15 (ZrMEw4hJW) = **75 total** | Inicialización completa del sistema |
| `main.(*HpP4qwz).ConnectL4D2Server` | 4 goroutines | Auth, Config, Loop de detección, Handler de reporte |
| `main.(*HpP4qwz).StartL4D2` | 4 closures + func4.1 | Lanzar juego, monitor DLLs, screenshots, cleanup |
| `main.(*HpP4qwz).FrontendReady` | 13+ closures | UI lista, eventos de WebView, IvOAAX goroutine |
| `main.(*HpP4qwz).BeforeClose` | 4 closures | Cleanup al cerrar |
| `main.QSUMsCa` | 4 goroutines (gowrap1-4) | Scanner VPK paralelo |
| `dUgTofmw.ga4oovjHCfg` | 33 closures | Motor de detección — scanner principal |
| `dUgTofmw.KT300841` | gowrap1 | Goroutine de evaluación |
| `dUgTofmw.JcCuAB7zH` | gowrap1 | Goroutine de reporte |
| `bszAWJqu.FkjkBraZo` | 10 closures | Scanner ventanas |
| `bszAWJqu.JnwG33U` | 10 closures | Scanner procesos |
| `bszAWJqu.PlRmoX6cuU` | 10 closures | Scanner módulos DLL |
| `bszAWJqu.X10KDNt6j` | 10 closures | Scanner archivos |
| `bszAWJqu.X80N6Rs52TTi` | 9 closures | Scanner red |
| `bszAWJqu.LUb2wVKw` | gowrap1 | Goroutine auxiliar scanner |

**Total estimado de goroutines en ejecución:** **200+ goroutines concurrentes** durante el monitoreo activo.

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
