# Paquetes Go Identificados — Mapa Completo

Este documento mapea los nombres garble-ofuscados del binario a sus librerías reales.
La identificación se realizó por análisis de métodos, campos y comportamiento.

---

## Paquetes del AC (código propio)

| Paquete Garbled | Descripción | Evidencia |
|-----------------|-------------|-----------|
| `main.(*HpP4qwz)` | **Struct principal de la app Wails — orquestador máximo del AC** | `Startup` (75 closures: func1-60 directas + `ZrMEw4hJW.func61-75` = inner function, 1 gowrap). `FrontendReady` (10 closures + `IvOAAX.1/1.1` = inner function). `ConnectL4D2Server` (4 closures = 4 goroutines principales). `StartL4D2` (4 closures). `BeforeClose` (4 closures + 2 deferwraps). `decFunc` PRESENTE en main = strings encriptados. `QSUMsCa` = función de detección (4 gowraps + `MDfyjAe.func7` + `RaR0kkqIl.func8` = orquestador de detección). `ZrMEw4hJW` = función nombrada global en main. `fyBBzjy` = helper struct (`Replace` + `exsr9Q9`). Total main: 183 funciones. Ver `main_package.md` |
| `dUgTofmw` | **Motor de evaluación central** — el paquete más grande del AC | **308 funciones**. Tipos: `(*fcje4l4dl_uV)` (Feed pipeline), `(*ZhEEW_uzlNn)` (evaluador con `h7MNIMO` de 8 etapas), `(*gZDPQlzEDoPS)` (error). Funciones críticas: `ga4oovjHCfg` (33 closures = scanner principal), `ZXugk2GxaotX` (14 closures), `KT300841` (17 closures), `CGRKkahhRkP` (12 closures = handler gRPC). 13 init funcs = 13 tipos de evento registrados. Ver `evaluador_central.md` |
| `HWARRxN` | **`google.golang.org/protobuf/internal/impl`** — runtime de mensajes protobuf | 550+ funciones. `(*KbBU6bdOa)` = `impl.MessageInfo` (Build, thSKtfSax3, _h76SfBr); `(*SLltHXdFZqty)` = FileDescriptor (ParentFile); `(*Yye7gKrP5)` = ServiceDescriptor; `(*WPaeyWXKC3et)` = MethodDescriptor (Input, Output, IsStreamingClient, IsStreamingServer); `(*Cv4Csm)` = MessageDescriptor (Enums, ExtensionRanges, de7YgkRNu); `(*As98DipdeM)` = FieldDescriptor (ByJSONName, ByName, ByNumber, ByTextName, Format, Get, Len, ProtoInternal); `(*G4Hq6JI)` = EnumDescriptor. **NOTA:** `KbBU6bdOa.Build` NO es HWID builder — es `MessageInfo.Build()` del runtime protobuf. El HWID collector real está en `asYMlWeBL6f6` |
| `bszAWJqu` | **Scanner de blacklists** — 5 scanners paralelos de ventanas/procesos/DLLs/archivos/red | **120 funciones**. 5 scanners: `FkjkBraZo` (ventanas/EnumWindows), `JnwG33U` (procesos/Toolhelp32), `PlRmoX6cuU` (módulos DLL/Module32Next), `X10KDNt6j` (archivos), `X80N6Rs52TTi` (red). Cada scanner aplica los 5 regexes de `ncRaYk_Ke` (Alav9s, IvOAAX, MlPMSGX2GAB, PJRLzfZ, T8Go8QdJaY). Auxiliares: `ThNOEr`, `OYQujSa6Aed5`, `LUb2wVKw`, `ApxFkk`, `CGKuSNjp`, `E5nzMUH5qSwM`, `IvOAAX`. Ver `scanner_blacklist.md` |
| `P4mAKk` | **Paquete de mensajes Protobuf del AC** | 16 tipos de mensaje con getters completos. Ver tabla completa en sección `Structs Protobuf` y `protocolo_protobuf.md`. Principales: `BEt_icchsrxy` (auth request), `ENOV9d` (HWID), `Qi1Z8I` (ban response), `H1oahxz1l3iY` (server response con CheatSigs), `W99qYP` (cheat signature con Pattern de bytes) |
| `x1JPPahi` | Contenedor de tipos de detección | Contiene ENOV9d, BiK8wj, W99qYP, Qi1Z8I, etc. |
| `_6di6zc0se2v` | WatchList — vigilancia continua (= `sNAkh4` en type table) | 81 entradas en pclntab; tipos: `(*_o3YyW)` (manager), `(*GrRzvM2)` (nodo), `(*Rs0QF1nUdhgw)`, `(*UWzc59)` (colecciones Has/String); 5 init funcs = 5 categorías (ventanas, procesos, red, módulos, archivos); integra gopacket (`cd1Utgxh0`, `wqT1oO`, `encTsS0_`). Ver `watchlist.md` |
| `DiU0jWi` | Detección de versión de Windows | `IsWindowsVersionAtLeast`, `RtlGetVersion` |
| `asYMlWeBL6f6` | Wrapper WMI (COM/DCOM) | 31 funciones init (func1-func24 con sub-closures), `WbemeDk0` = IWbemServices, `VDjfdjV` = enumerador WMI, `Aego3YEweIaU` = ExecQuery |
| `A39Z4i` | Ejecutor de queries WMI (librería Go WMI) | `(*DdY6Bn3b).Query` (con 4 deferwrap = 4 COM releases), `(*GNMRXRX).Query` = enumerador de resultados, `(*GUxA0QN3).Error` = tipo error WMI, `iFCNfGrVm5L` = función interna COM (16 closures), `wn2lrXVC` = setup de conexión |
| `FXWqsvy_` | **Pipeline de recolección HWID** — procesa resultados WMI y alimenta el evaluador central | **38 funciones**. Función clave: `Z6ey1EJD` con **11 closures** (func1-func11) = 11 colectores de hardware (CPU, Disk, GPU, MotherB, OS, etc.). Tipo comparable `IwUTVxn` y `YRNNn8` (usados como map keys). Flujo: `A39Z4i` → `FXWqsvy_.Z6ey1EJD` → `BhCuafOD` → `dUgTofmw.Feed()`. Funciones adicionales: `ZJZaSq7xmKez` (deferwrap1 = COM release), `MSffIimik` (4 closures = procesamiento multi-etapa), `Cp0zUb6UdSs` (2 closures), `A6hQnfaeFT2` (2 closures), `NkpHEP` (1 closure), `AaPVLqQV6w`, `xxU6vhj6`, `YRNNn8`, `KKyCdJjT`, `IwUTVxn`. Ver `pipeline_hwid.md` |
| `pdFrspK_G` | **Paquete del motor principal de detección** — contiene la función masiva `hlavBkMcO` | **789 funciones totales**. Solo 5 funciones top-level (no-closure): `hlavBkMcO` (**644+ closures** = motor de detección central), `DaZXKAoyfrI` (función de setup), `i8xZKDcoOE` (helper), `PviJufUQhhN` (helper), `decFunc` (descifrador de string literals garble). `hlavBkMcO.func1` ... `hlavBkMcO.func644` = 644 sub-closures = 644 verificaciones de detección individuales. **Este es el corazón del anticheat**. Sin struct types (0 receivers) — paquete de funciones procedurales puras. 0 init closures (no registro de firmas, la función `hlavBkMcO` SE LLAMA directamente). Ver `pdFrspK_G.md` |
| `sOAbtRgFLa6_` | **Scanner de memoria / estado Source Engine** — Vector de detección 6 | **20 funciones**. Funciones: `GXErxuFMY9Z` (init, 5 closures), `UxvaZNFzHI` (scanner principal, 7 closures: func1-7 + 4 sub-closures), `BY2TsEzeKap` (llamada desde `dUgTofmw.KT300841`), `H1gN3xqm` (helper). Usa `f3SkGnYxZwK.NOZFARjOV` = `time.Now()` para timing del loop. Aplica regexp para matching. `UxvaZNFzHI` feeds eventos al pipeline `dUgTofmw`. Paquete adyacente a `uaIqvkWry1B` (gopacket) en pclntab. Detecta: flags de ConVar Source Engine (`i4localSurvivorGunFire` como anchor), velocidad de disparo, godmode/noclip, sv_cheats |
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
| `zD7bA8PweG` | **`flag`** stdlib — parser de argumentos de línea de comandos | **81 funciones**. `(*LqP61GcYptNV)` = `flag.FlagSet` (Bool, BoolVar, String, StringVar, Var, Parse, PrintDefaults, Output, VisitAll, eL6wdBalnAM). `(*_hvPOAf)` = `flag.Value` interfaz (Get, IsBoolFlag, Set, String). `(*n7i9ueiw)` = tipo bool para flags. `decFunc` activo (garble -literals) |
| `FPB_79` | **`regexp/syntax`** stdlib — AST y compilación de expresiones regulares | **177 funciones**. `(*JYJ9UJf38Cdo)` = `syntax.Regexp` (CapNames, Equal, MaxCap, Simplify, String). `(*L_7HQf5pUpJQ)` = `syntax.Inst` (MatchRune, MatchRunePos, MatchEmptyWidth, String). `(*LI1mdlchryb)` = `syntax.Prog` programa compilado (Prefix, StartCond, String). `(*LPPSSi)` = `syntax.Error`. `(*iqe7rrvB86E8)` = motor de matching interno (~30 métodos garbled). `(*eVZ5Bn)` = sort helper (Len, Less, Swap). Usada por `ncRaYk_Ke` (regexp) |
| `_GNApqU1drY` | **`encoding/asn1`** stdlib — codificación DER/BER | **198 funciones**. `(*T_BJGweU9Wk)` = `asn1.BitString` (At, RightAlign — métodos exclusivos de `asn1.BitString`). `(*EkQXmseS)` = `asn1.ObjectIdentifier` (Equal, String). Múltiples tipos internos con solo `Encode()/Len()` = encoders DER internos por tipo de dato (`ammnYaCNKcu`, `aTM_kp8Q`, `d5ZNTrI`, `fUCb2v`, `hmuFTi2KWT`, `hyY8s1v27x`, `l0RxhRv`, `piOct6EK3hK4`). Error types: `mbVbbz5_dunW`, `NG54ZqBW`, `PVuEatr9` = asn1.SyntaxError/StructuralError. Usada para parsing de certificados X.509 |
| `aGxe6x3` | **`html/template`** stdlib — template HTML con escape seguro | **263 funciones**. `(*IgRKTJSx)` = `template.Template` (AddParseTree, Clone, DefinedTemplates, Delims, ErrorContext, Execute, ExecuteTemplate, Funcs, Lookup, New, Option, Parse, ParseFiles, ParseFS). `(*DhEc_NXJxl9g)` = `template.Error` (Error, Unwrap). Usada por Wails para templates HTML |
| `GVADC0m` | **`text/template/parse`** stdlib — parser de AST de templates | **414 funciones**. `(*_6BLHNm7_4)` = `parse.Node` (Copy, Position, String, Type). `(*BiNaLMryAQ09)` = `parse.Tree` (ErrorContext). Incluye todos los tipos de nodo del AST: IfNode, RangeNode, WithNode, ActionNode, PipeNode, CommandNode, etc. — cada uno implementando Copy/Position/String/Type. Usada internamente por `aGxe6x3` (html/template) |
| `Lfl5pm7AWVtT` | **`encoding/base64`** stdlib — codificación base64 | **26 funciones**. `(*CrXpAK4)` = `base64.Encoding` (AppendEncode, Decode, DecodedLen, DecodeString, Encode, EncodedLen, EncodeToString, WithPadding). `(*IA_r2V19s)` = `base64.CorruptInputError`. Constantes garbled: StdEncoding, URLEncoding, RawStdEncoding, RawURLEncoding. Usada para encoding de datos HWID, tokens, payloads HTTP |
| `ldZb0s` | **`io`** stdlib — interfaces y primitivas de I/O | **139 funciones**. `(*HP5psmyMOb)` = `io.PipeWriter` (Close, CloseWithError, Write). `(*KIWBmeTUw95a)` = `io.PipeReader` (Close, CloseWithError, Read). `(*gQi4oLRCfY)` = multi-reader/closer (Close, Read, WriteTo). `(*b5p79K)` = reader wrapper. `(*k7bRkt2NdU)` = writer (ReadFrom, Write, WriteString). `(*iKlkYVcFymV)` = lector interno (Read, WriteTo). `(*ABggrM8)` = lector simple |
| `LbVoc9TjklSB` | **`net/netip`** stdlib — tipos IP modernos (Go 1.18+) | **262 funciones**. `(*G_V9mRiw)` = `netip.Addr` (AppendTo, As16, As4, AsSlice, BitLen, Compare, Is4, Is4In6, Is6, IsGlobalUnicast, IsInterfaceLocalMulticast, IsLinkLocalMulticast, IsLinkLocalUnicast, IsLoopback, IsMulticast, IsPrivate, IsUnspecified, IsValid, Less, MarshalBinary, MarshalText, Next, Prefix, Prev — coincidencia TOTAL con netip.Addr). `(*aoFmo4)` = `netip.AddrPortError`. `vv4rbCwWx2` = tipo Addr/AddrPort usado como parámetro genérico en `jsD5qQ4`. Usado para gestión de IPs de red en la capa de captura de paquetes |
| `NswwKak` | **`bufio`** stdlib — I/O con buffer | **46 funciones**. `(*VqjBxHx)` = `bufio.Writer` (Available, AvailableBuffer, Buffered, Flush, ReadFrom, Reset, Size, Write, WriteByte, WriteRune, WriteString). `(*fGvUU9anb)` = `bufio.Reader` o `bufio.Scanner` (eMxufh, ghw_9tSo, Read). `(*qN9ab4rq)` = writer wrapper (Close, Write) |
| `O2_engYw6e` | **`net/textproto`** stdlib — protocolo de texto de red (HTTP/SMTP/FTP headers) | **85 funciones**. `(*OGC3KRXB_q)` = `textproto.Reader` (DotReader, ReadCodeLine, ReadContinuedLine, ReadContinuedLineBytes, ReadDotBytes, ReadDotLines, ReadLine, ReadLineBytes, ReadMIMEHeader, ReadResponse). `(*NaANsequyfaS)` = `textproto.MIMEHeader` (Add, Del, Get, Set, Values). `(*GGrL0m_V)` = `textproto.Error`. `(*Tzta_XXUGJ)` = `textproto.ProtocolError`. Usada internamente por net/http para parsing de headers |
| `sMYs0N1` | **`os`** stdlib — operaciones del sistema operativo | **382 funciones**. `(*b4U1_yJd)` = `os.File` (Chdir, Chmod, Chown, Close, Fd, Name, Read, ReadAt, ReadDir, Readdirnames, ReadFrom, Seek, SetDeadline, SetReadDeadline, SetWriteDeadline, Stat, Sync, SyscallConn, Truncate, Write, WriteAt, WriteString). `(*dht6dL)` = `os.fileStat` = fs.FileInfo (Info, IsDir, Name). Funciones del paquete: Create, Open, OpenFile, Remove, Stat, MkdirAll, etc. |
| `S35i9Z5G` | **`crypto/sha256`** stdlib — hash SHA-256 y SHA-224 | **22 funciones**. `(*aFRxzGtg5YbJ)` = `sha256.digest` (BlockSize, MarshalBinary, Reset, Size, Sum, UnmarshalBinary, Write). `A3gy7MH8GCma`, `AQitwuMUp5F` = constructores New/New224. `buZ_nOgYC`, `F7QqpLXaYdHr`, `FgyDdOb_JFda` = funciones internas de compresión SHA-256. Usada con `aFzxJpzp2` (HMAC) para HMAC-SHA256 de autenticación |
| `TrQOw3q` | **`crypto/rsa`** stdlib — cifrado y firma RSA | **52 funciones**. `(*RHa1P5VPE)` = `rsa.PrivateKey` (Decrypt, Equal, Precompute, Public, Sign, Size, Validate). `(*BzLYb96v7E)` = `rsa.PublicKey` (Equal, Size). `(*AXORfhz)` = hash config type (HashFunc). Funciones: `A3hbai` = EncryptOAEP o VerifyPKCS1v15, `dt8ctOS7`/`eAfCGB` = OAEP/PKCS1 helpers. Usada en X.509/TLS para verificación de firma de certificados |
| `uBxEkCisahOX` | **`context`** stdlib — contextos de cancelación y timeouts | **183 funciones**. Múltiples tipos implementando `context.Context` (Deadline, Done, Err, Value): `cazjpzQNu9p` = cancelCtx (con String), `chd21WO` = timerCtx, `eRV0Jl` = valueCtx, `jK1v7rI` = emptyCtx (Background/TODO), `kjQT4S8F1I` = cancelCtx con String. Funciones: `WithCancel`, `WithDeadline`, `WithTimeout`, `WithValue`, `Background`, `TODO`. Usado extensivamente en gRPC, gopacket, net/http |
| `WM4c2TtmTWTc` | **`golang.org/x/crypto/chacha20poly1305`** o **`crypto/cipher.AEAD`** | **27 funciones**. `(*_HDIIzi)` implementa `cipher.AEAD` (NonceSize, Open, Overhead, Seal — métodos exactos de AEAD). Helpers: `bIa4nd5x4`, `dEJcfXHtr`, `EkVmeiMT`, `heU3ymNbFJ0`, `iFV3EJRodLm4`. La presencia de este wrapper AEAD adicional (junto a `dzpA6s` que tiene NewGCM) sugiere uso de múltiples esquemas de cifrado autenticado |
| `ZjyFEdOLj` | **`encoding/binary`** stdlib — lectura/escritura de integers en big/little endian | **17 funciones**. `aJ6HgUqCH` = `binary.LittleEndian` (PutUint16, PutUint32, PutUint64, Uint16, Uint32, Uint64). `hM0aAK` = `binary.BigEndian` (AppendUint16, PutUint16, PutUint32, PutUint64, Uint16, Uint32, Uint64). Usada en parsing de protobuf wire format, paquetes de red, datos binarios de HWID |
| `OibOJGGM67` | **`crypto/ecdh`** stdlib — intercambio de claves ECDH con curvas elípticas | **70 funciones**. `(*afnLhU[*AFdc2Qd.AhD79vsT_])` = P256 curve (GenerateKey, NewPrivateKey, NewPublicKey, String). `(*afnLhU[*AFdc2Qd.LAtTHo1_ow])` = P384 curve. `(*afnLhU[*AFdc2Qd.QKDrGlZ2DsTi])` = P521 curve. Genérico `[go.shape.*uint8]` = implementación base. Usada para ECDH key exchange durante TLS handshake |
| `E5D2QU` | **`internal/poll`** stdlib — polling I/O de bajo nivel (Windows IOCP) | Tipo principal `Yt_S_p_T` = `poll.FD` (file descriptor interno). Métodos identificados: `ConnectEx`, `WSAIoctl`, `SetsockoptIPv6Mreq`, `ReadMsgInet4`, `ReadMsgInet6`, `WriteMsgInet4`, `WriteMsgInet6`, `ReadDirentNames` — API exclusiva de Windows IOCP. Es el paquete interno que implementa las operaciones de socket async bajo `net` y `os`. No accesible directamente, importado por `u26yruH2Ly_u` (net stdlib) |
| `CNEPNkZW_` | **`mime/multipart`** stdlib — manejo de contenido multipart/form-data | Métodos de `Reader` (ReadForm, NextPart, NextRawPart) y `Writer` (CreateFormField, CreateFormFile, CreatePart, FormDataContentType, SetBoundary, Close). Usado por gin/binding para procesar uploads de archivos y form data multipart |
| `cQ_qBU` | **`hash/crc32`** stdlib — checksum CRC-32 | Implementación de hash CRC32 (IEEE, Castagnoli, Koopman). Usado internamente por `compress/gzip` y `compress/zlib` para checksums de integridad |
| `T5Zwd2` | **`compress/flate`** stdlib — algoritmo DEFLATE (base de gzip/zlib) | Dos tipos sort.Interface para árbol Huffman; Writer con `Close/Flush/Write`; Reader con `Close/Read`; dos tipos de error. `D9eGQ5PG` (lector de bits) es un paquete interno de flate. Usado por `aENMwam6VEd` (gzip) y `AVlBz4h1y9u` (zlib) |
| `AVlBz4h1y9u` | **`compress/zlib`** stdlib — compresión zlib (DEFLATE + checksum Adler-32) | `(*HIpkYL9Ol_z)` = `zlib.Reader` (Close, Read, Reset). `(*HCy6tMrG_b_z)` = `zlib.Writer` (Close, Flush, Reset, Write). Depende de `T5Zwd2` (flate) y `cQ_qBU` (crc32). Usado para descomprimir datos de red o archivos del juego |
| `aU_GUDmLQsSR` | **`bufio`** stdlib — I/O con buffer (versión completa) | **Versión completa**: `(*Reader)` con todos los métodos (ReadByte, ReadBytes, ReadLine, ReadRune, ReadSlice, ReadString, UnreadByte, UnreadRune, Buffered, Discard, Peek, Reset, Size, WriteTo), `(*Writer)` completo, `(*Scanner)` completo (Bytes, Err, Scan, Split, Text, Token), `(*ReadWriter)`. NOTA: `NswwKak` es una vista parcial del mismo paquete; `aU_GUDmLQsSR` es el nombre principal en pclntab |
| `efJn___` | **`image`** stdlib — tipos de imagen Go (Image, Rectangle, Point, RGBA, etc.) | Tipos principales: `image.Image` interfaz (Bounds, At, ColorModel), `image.RGBA`, `image.NRGBA`, `image.Gray`, `image.Paletted`, `image.Rectangle` (Inset, Intersect, Overlaps, Union). Usado probablemente por Wails para manejo de iconos/screenshots del juego |
| `FGCGZvVhePF` | **`crypto/elliptic`** stdlib — curvas elípticas P224/P256/P384/P521 | Implementaciones de curvas NIST sobre `math/big`. Métodos: Add, Double, ScalarBaseMult, ScalarMult, IsOnCurve, Params. Usado por `OibOJGGM67` (ecdh) y `TrQOw3q` (rsa) para operaciones de clave pública TLS. NOTA: `AFdc2Qd` combina `crypto/elliptic` y `math/big` en una sola entrada garbled |
| `eca9NwL` | **`log`** stdlib — logging estándar de Go | Logger con `Print/Printf/Println`, `Fatal/Fatalf/Fatalln`, `Panic/Panicf/Panicln`. Prefijo, flags de timestamp, output configurable. Usado por varios paquetes de infraestructura del AC para logging interno |
| `a6Mqf0P71i` | **`errors`** stdlib — creación y wrapping de errores | `errors.New`, `errors.Is`, `errors.As`, `errors.Unwrap`. El paquete más pequeño de stdlib pero el más utilizado. Todos los paquetes del AC crean tipos de error que implementan `error` interface |
| `a8KcF_z7WQ` | **`crypto`** stdlib — registro central de algoritmos criptográficos | Contiene `crypto.Hash` (SHA1, SHA256, SHA384, SHA512, MD5, etc.) como enum + registro de implementaciones. `Hash.Available()`, `Hash.HashFunc()`, `Hash.New()`. Hub que conecta todos los paquetes crypto del binario |
| `gWu1wy0t` | **`crypto/md5`** stdlib — hash MD5 | `(*md5.digest)` con BlockSize=64, Size=16, Sum, Write. Funciones: `New`, función de compresión interna. Usado para hashes legacy o checksums de compatibilidad (ej: identificadores de archivos VPK) |
| `jylXCgIg4c5` | **`fmt`** stdlib — formateo de strings e I/O | **50 funciones**. `wK7za1f` = `fmt.State` (Flag, Precision, Width, Write, WriteString — interfaz exacta de fmt.State). `zKrSboQvt4m` = `fmt.ScanState` (Read, ReadRune, SkipSpace, Token, UnreadRune, Width — interfaz exacta de fmt.ScanState). `j1xDvYu` = RuneReader interno. `mZ5C3gIpMW2u`/`xOt7zjRZ` = tipos de error wrap. Funciones: Print/Printf/Println/Sprintf/Fprintf/Sscanf/Errorf |
| `BfqSCVbjbFt6` | **`golang.org/x/crypto/chacha20`** o cipher mode pack — cifrado de stream + AEAD | **22 funciones**. `_CjLfdDvIq`/`r3IRx6HDaub`: BlockSize, CryptBlocks, `SetIV` — block mode CFB/OFB con SetIV. `awYZwE`: `XORKeyStream` — cifrado de stream. `DeD31aPe`: Read — reader cifrado. `gkgHtaFzr9e0`: NonceSize, Open, Overhead, Seal — AEAD completo (chacha20-poly1305 o AES-GCM wrapper). `SetIV` es característica de `golang.org/x/crypto` (no aparece en stdlib cipher). Complementa `WM4c2TtmTWTc` y `dzpA6s` |

---

## Paquetes de Red (identificados)

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `i1aqCskEISkX` | `net/http` + `golang.org/x/net/http2` | HTTP/2 framer (WriteContinuation, WriteData, WriteHeaders, WriteSettings), HTTP transport (RoundTrip, CloseIdleConnections), Authenticate, BasicAuth, SetBasicAuth |
| `gG7Vmb7` | **`github.com/google/gopacket/pcap`** — captura de paquetes en vivo | **452 funciones**. Confirmado por `WritePacketData` (método exclusivo de pcap) y cross-referencias a `q4ajG0RM.Aa8DB2bIK66` (gopacket type). El AC captura tráfico de red en vivo desde la interfaz de red del sistema para monitorear las conexiones del juego |
| `hh58lo` | Structs de respuesta A2S (campos JSON) | Tipos: `*hh58lo.Phr6ow`, `*hh58lo.GV2qnv`, `*hh58lo.Ea9gTx`, `*hh58lo.BT1E5g`, `*hh58lo.FnICOY`; campos json:"VAC", json:"SteamID", json:"Name", json:"Game", json:"AppID" |
| `HBJGtL` | **Cliente A2S Protocol** (implementación del protocolo Source Engine server query) | `(*Ns0npSNFWWqQ).QueryInfo` = A2S_INFO, `(*Ns0npSNFWWqQ).QueryPlayer` = A2S_PLAYER, `(*Ns0npSNFWWqQ).Close`, `(*JG00Xm6lG8)` = lector binario (ReadUint8/16/32/64/String/Float32), `(*XxIkaC9_)` = escritor binario (WriteBytes/WriteCString), 11 funciones init |
| `eUmM3IpzIrV` | **Capa de conectividad de red** (manager de conexiones + DTLS) | **328 funciones**. Referencia a `qTnr3tOf4N.A0lMBv6KYzU_` (DTLS type) y `i1aqCskEISkX` (http2). Tipos complejos: `c4yDowusz` con `func(*c4yDowusz)` — struct con callback de conexión. Probablemente `github.com/pion/transport` o el connectivity layer de gRPC. **CORRECCIÓN:** Identificación previa como "A2S_INFO struct" era incorrecta — el package real de A2S structs es `hh58lo` |
| `MyTTk7_I` | **`crypto/tls`** stdlib — implementación TLS 1.0-1.3 completa | **969 funciones** — el segundo paquete más grande del binario. `(*A0lMBv6KYzU_)` = `tls.Conn` (Close, CloseWrite, ConnectionState, Read, SetDeadline, SetReadDeadline, SetWriteDeadline, Write). `(*ExZca80)` = `tls.Config` (SetSessionTicketKeys — método exclusivo de tls.Config). `(*OSDeEYT7W)` = `tls.Dialer` (DialContext). `(*jgpXHkDaJV6m)`, `(*qFdp4v4JCan)` = implementaciones AEAD para cipher suites. Múltiples tipos Error (DohBZ1PKOE, Duh8raXX4QFb, k2RfUaxZ, lVtXLidN, XHJO_Bqq0O, ZlSuiKV). **NOTA:** `qTnr3tOf4N.ExZca80` es un tipo DIFERENTE (DTLS SessionState) que casualmente tiene el mismo nombre garbled en otro package |
| `uL6SGHSpk` | **`crypto/x509/pkix`** — no x509 directamente | `(*JjZ1WNQA).FillFromRDNSequence/ToRDNSequence/String` = `pkix.Name`, `(*IBvuTjv).HasExpired` = `pkix.CertificateList.HasExpired` (verificación de CRL), `(*Ap8xj8SrB).String` = `pkix.AlgorithmIdentifier` |
| `LYUqBZOV7RHi` | **`crypto/x509`** — manejo de certificados X.509 | `(*XdDiONjbzcM)` = `x509.CertPool` (AddCert, Clone, AppendCertsFromPEM, Equal), `(*ZIbJOWC4).CheckSignature` = `x509.Certificate`, 31 funciones init (registro de algoritmos RSA/ECDSA/DSA) |
| `OwbnEHL7gtm` | `gopacket` / librería 802.11 | Métodos Bandwidth, ShortGI, GuardInterval, MCSIndex, FECType, ReportZerolen, IsZerolen, LastKnown, DelimCRCErr |
| `qTnr3tOf4N` | **DTLS / TLS-over-UDP** — implementación TLS para canal UDP seguro | 163 entradas; tipos: `(*ExZca80)` = SessionState, `(*Gy_XwjZ)` = Conn (conexión TLS), `(*vjSRqd6ov)` = record/paquete, `(*iTeUYgu)` = handshake config, `(*_ZyRwtuuaU)` = estado cliente, `(*vE6jf82QV)` = tipo alert/record, `(*JSj6sp1QG)` = cert/key, `(*QmRsQS5)` = factory (retorna `*ExZca80` o `*Gy_XwjZ`); referencias cruzadas a `l8F_pdQP` (x509.Certificate = `ZIbJOWC4`) y `eUmM3IpzIrV` |
| `l8F_pdQP` | **`crypto/x509`** (nombre corto garbled) | = mismo paquete que `LYUqBZOV7RHi` (path garbled); `ZIbJOWC4` = x509.Certificate; `decFunc` = string literals garbled; diferencia: `l8F_pdQP` = package NAME en type table, `LYUqBZOV7RHi` = package PATH en pclntab |
| `WU2ixCjkiE` | **`golang.org/x/net/http2`** — implementación HTTP/2 completa | **865 funciones** — el paquete HTTP/2 más grande. `(*AKP7549SZE5)` = `http2.HeadersFrame` (HasPriority, Header, HeaderBlockFragment, HeadersEnded, PseudoFields, PseudoValue, RegularFields, StreamEnded, String — coincidencia exacta). Múltiples tipos de frame HTTP/2. `(*aRGN7PBOmo)` = http2.pipe (Close, Read). Usado para el transporte HTTP/2 de gRPC |
| `wD1vARBz` | **`golang.org/x/net/http2/hpack`** — compresión de headers HPACK (HTTP/2) | **194 funciones**. `(*A7ZSxr)` = `hpack.Decoder` (Close, DecodeFull, EmitEnabled, SetAllowedMaxDynamicTableSize, SetEmitEnabled, SetEmitFunc, SetMaxDynamicTableSize, SetMaxStringLength, Write). `(*GQqpJJVrwn)` = otro tipo hpack. Usado por `WU2ixCjkiE` para descomprimir headers HTTP/2 |
| `w3ZRgslh` | **`golang.org/x/net/http2`** internal o **`golang.org/x/net/http2/hpack`** | **194 funciones**. `(*C8Xlcll)` = `hpack.HeaderField` o tipo similar (IsPseudo, Size, String). `(*HR3xNc)` = encoder (MaxDynamicTableSize, SetMaxDynamicTableSize, SetMaxDynamicTableSizeLimit, WriteField). `(*gsHfjYtpG)` = framer interno con init propio. `(*VZ6CpVOtnb)` = otro tipo. Posiblemente `http2/internal` o un segundo nivel del stack HTTP/2 |
| `eSumh8L` | **`golang.org/x/net/dns/dnsmessage`** — parsing de mensajes DNS | **128 funciones**. `(*CV1d0H4O15i)` = `dnsmessage.Header` (ExtendedRCode, SetEDNS0, tS9cBwaLXa). `(*Ga4tQ8is)` = `dnsmessage.Parser` (AAAAResource, AdditionalHeader, AnswerHeader, AResource, CNAMEResource, Question, SkipAdditional, SkipAllAnswers). `(*doqguJg)` = `dnsmessage.MXResource` o similar con Error. El AC resuelve nombres DNS para verificar conectividad o detectar manipulación de DNS |
| `ki3UJTJ_` | **`golang.org/x/net/idna`** — nombres de dominio internacionalizados (IDNA) | **44 funciones**. `(*T1WGsZeLaSj)` = `idna.Profile` (ToASCII — función central IDNA para convertir nombre de dominio unicode a ASCII/punycode). `(*v86Cvw7Kb1)` = procesador interno. `(*bWgQTrKw)` = builder/lookup. `(*aed9scnyb)` = `idna.Error`. Usada para normalizar nombres de dominio antes de conexiones TLS (validación de certificados) |
| `hy8mSodjjSv` | **`github.com/google/gopacket/pcap`** — captura en vivo (confirma nombre en pclntab) | **171 funciones**. `(*QgQHFhn)` = `pcap.Handle` (Close, CompileBPFFilter, LinkType, ListDataLinks — métodos exclusivos de `pcap.Handle`). `(*OL2WhxTr)` = BPF filter (Matches, String). NOTA: `gG7Vmb7` (452 funciones, WritePacketData) es el nombre pclntab principal de gopacket/pcap; `hy8mSodjjSv` puede ser un sub-paquete o vista alternativa |

---

---

## Paquetes de Terceros Adicionales (identificados)

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `dvgs9C` | **`github.com/gin-gonic/gin`** — nombre en type table | Nombre garbled del paquete Gin en la tabla de tipos. Ver `hwvnZR_pbO` para el nombre en pclntab (mismo paquete). El AC ejecuta un servidor HTTP local con Gin — comunicación Wails UI↔backend o IPC multi-proceso. Rutas HTTP cifradas por garble -literals |
| `hwvnZR_pbO` | **`github.com/gin-gonic/gin`** — nombre en pclntab (**470 funciones**) | Mismo paquete que `dvgs9C`. Mapeo de tipos COMPLETO: `TebhSHwRpJd` = `gin.Engine` (GET/POST/PUT/DELETE/PATCH/HEAD/OPTIONS, Group, Handle, Run, RunTLS, RunUnix, RunFd, RunListener, NoMethod, NoRoute, LoadHTMLFiles, ServeHTTP, SecureJsonPrefix, SetHTMLTemplate, SetTrustedProxies). `PdQI6soR` = `gin.Context` (todos los métodos: Abort*, Bind*, Client/RemoteIP, Cookie, Data, File*, Get*, HTML, JSON, JSONP, Param, ProtoBuf, PureJSON, Query*, Redirect, Render, ShouldBind*, Set*, SSEvent, Stream, String, TOML, XML, YAML). `FIGaAyUT` = `gin.RouterGroup` (GET/POST/DELETE/PATCH, Static*, StaticFile, Use). `fVs5MuND` = `gin.responseWriter` (Flush, Header, Hijack, Pusher, Size, Status, Write, WriteHeader). `mv__wObu5MX` = internal FS (Close, Read, Readdir, Seek, Stat). `UUAEWKt` = `gin.LogFormatter` (IsOutputColor, MethodColor, StatusCodeColor). `_FC1UaV` = `gin.Errors` (ByType, JSON, Last, MarshalJSON). `TZIHLRTrrZZo` = `gin.Params` (ByName, Get) |
| `cXI2MODy` | **`github.com/gin-gonic/gin/binding`** — sub-paquete Gin para binding de request data — **154 funciones** | Tiene `decFunc` (garble -literals activo) y 115+ init closures. **CONFIRMADO** como `gin/binding` por sus dependencias: importa `ogLSXbC1MZp` (yaml.v3) para binding YAML y `FXIaelQr` (go-toml/v2) para binding TOML — exactamente las dependencias que `gin/binding` declara en su código fuente para soportar múltiples formatos (`application/x-yaml`, `application/toml`, `application/json`, etc.). Esto también confirma que el servidor Gin del AC puede recibir/enviar YAML y TOML, no solo JSON |
| `Ve0nLKBMzmC` | **`github.com/go-playground/validator/v10`** — validación de datos | `(*adPnywIlKf75)`: Field, GetTag, ExtractType, Param, Parent, Top, ReportError, ReportValidationErrors, Validator. Valida los datos del protocolo gRPC y posiblemente los campos protobuf recibidos del servidor |
| `vfFlma` | **`google.golang.org/protobuf/reflect/protoregistry`** — registro global de tipos protobuf | **171 funciones**. `decFunc` PRESENTE (string literals cifrados). Identificación DEFINITIVA por las firmas de interfaces en la type table del binario: `FindFileByPath(string) (vfFlma.TaNgMXIgI, error)` = `protoregistry.Files.FindFileByPath()`; `RegisterFile(vfFlma.TaNgMXIgI) error` = `protoregistry.GlobalFiles.RegisterFile()`; `FindExtensionByNumber(vfFlma.IgTYts3, t5W7d4Vh7E.Btf9ew8En) (vfFlma.GBbXxrK4, error)` = `protoregistry.GlobalExtensions.FindExtensionByNumber()`; `FindMessageByURL(string) (vfFlma.IiQTgkc, error)` = `protoregistry.GlobalTypes.FindMessageByURL()`; `RegisterMessage(vfFlma.IiQTgkc) error` y `RegisterExtension(vfFlma.GBbXxrK4) error` = registro de tipos proto. Tipos: `TaNgMXIgI`=FileDescriptor, `IiQTgkc`=MessageType, `GBbXxrK4`=ExtensionType, `IgTYts3`=MessageDescriptor, `ZyZwoqNYuq7`=Value (interface{}), `CCwyMQn`/`Gz2fJnWEdG6R`/`HahfZlHJZ0N` = tipos internos del registry. Cross-refs: `oJ5pXwZm7R.ICaHN1kQ` (otro paquete proto), `t5W7d4Vh7E.Btf9ew8En` (FieldNumber). Aparece inmediatamente ANTES de `er9_yp` en pclntab |
| `G1kVlqH3` | **`golang.org/x/crypto/cryptobyte`** — builder/parser de ASN.1 y TLS | **78 funciones**. `(*AqFLYEKa)` = `cryptobyte.String` (CopyBytes, ReadAnyASN1, ReadAnyASN1Element, ReadASN1, ReadASN1BitString, ReadASN1Boolean, ReadASN1Element, ReadASN1GeneralizedTime, ReadASN1Integer, ReadASN1ObjectIdentifier, ReadASN1UTCTime, ReadBytes — coincidencia exacta con métodos de `cryptobyte.String`). Usada en parsing de certificados X.509 en el handshake TLS |
| `t89_Iw9sGL1` | **`golang.org/x/crypto/edwards25519`** — aritmética de curva Ed25519 | **78 funciones**. `(*FsfHydC)` = `edwards25519.Point` (Add, Bytes, Negate, ScalarBaseMult, Set, SetBytes, VarTimeDoubleScalarBaseMult — `VarTimeDoubleScalarBaseMult` es método exclusivo de esta librería para verificación de firma Ed25519). Tipos de coordenadas proyectivas: `aHn80Kmd` (CondNeg, FromP3, Select, Zero), `cicNkFE` (FromP1xP1, FromP3, Zero), `k6povwZXcy`, `l5Lf3UA3Z1`, `oeoqOQpfce` (FromP3, SelectInto). Usada por `crypto/ed25519` para firmas digitales |
| `GTShob4MX` | **`golang.org/x/sys/windows/registry`** — acceso al registro de Windows | **26 funciones**. `GTShob4MX.D3AAR5j4` = `registry.Key` (Close, GetIntegerValue, GetStringValue, SetStringValue — API exacta de `registry.Key`). Helpers: `Dl4YcPukm`, `hOXBdzRS`, `MiaLOHfEN` = funciones OpenKey/CreateKey. 11 init closures (string literals de claves del registro). Usada para leer HWID desde registro: `HKLM\SOFTWARE\Microsoft\Cryptography\MachineGuid`, etc. |
| `TVgAOO` | **`golang.org/x/sys/windows/registry`** (vista alternativa) o **paquete auxiliar de registro** | **24 funciones**. `TVgAOO.ALQe7a` = tipo registry.Key extendido (Close, GetMUIStringValue, GetStringValue, ReadSubKeyNames — `GetMUIStringValue` = método para leer strings localizadas). `foAgmvA6QCF`, `H2_NoxMO`, `HOV0O2`, `iBWyxkUN`, `sJxe1Gx` = helpers. 10 init closures (claves de registro cifradas). Probable: sub-package de registry o wrapper para leer DisplayName del software instalado (HKLM\SOFTWARE) |
| `WpWmMn` | **`golang.org/x/text/language`** — BCP 47 language tags | **101 funciones**. `(*HlmSUag)` = `language.Region` (Canonicalize, Contains, IsCountry, IsGroup, ISO3, IsPrivateUse, M49, String, TLD — coincidencia exacta). `(*Ox20D92m3uog)` = otro tipo Region. `(*Ma114HivAjwF)` = `language.Script` (IsPrivateUse, String). `(*RvInEbJ3LlQE)` = `language.Tag` (Base, Extension, Extensions). `(*gslBMex)` = `language.Coverage` (BaseLanguages, Regions, Scripts, Tags). `(*GH2NMHwVA)` = `language.Matcher` (String). Usada probablemente para detección de idioma del sistema o validación de locale |
| `xbojjLBFq` | **`golang.org/x/text/internal/language`** — implementación interna de language tags | **223 funciones**. `(*KP1Go6d20r)` = Tag interno (Extension, Extensions, HasExtensions, HasString, HasVariants, IsPrivateUse, IsRoot, MarshalText, Maximize, Parent). `(*F5YfdexG)` = Builder (AddExt, AddVariant, Make, SetTag). `(*A6Pxm9dwpF)` = SubtagError (Error, Subtag). Usada por `WpWmMn` como dependencia interna |
| `j8F4gZc7iLj` | **`google.golang.org/protobuf/internal/filedesc`** — descriptores de archivos proto | **100 funciones**. `(*BLQIsGP)` = registro de descriptores (FindDescriptorByName, FindFileByPath, NumFiles, NumFilesByPackage). `(*apaoYff)` = stack interno (Pop). Función principal `kJGGuROwQ_O` (13 closures = carga de descriptores proto). Usada para el registry de tipos protobuf en runtime |
| `xYYlVHODE3qa` | **`google.golang.org/protobuf/reflect/protoreflect`** — reflexión de mensajes protobuf | **364 funciones**. `(*BAbJq9d)` y `(*CPP62_ijp)` = EnumValueDescriptor (GoString, IsValid, String). `(*HahfZlHJZ0N)` = protobuf Value (Bool, Int, Interface, IsValid, String, Uint, Value — exactamente `protoreflect.Value`). `(*IgTYts3)` = lista de descriptores (Append, IsValid, Name, Parent). `(*QteYnOvkj)` = tipo descriptor con muchos métodos garbled. Usada para introspección dinámica de mensajes proto |
| `yzULHj8` | **`google.golang.org/protobuf/internal/impl`** sub-codec — serialización de mensajes | **83 funciones**. `(*M5clpmDMoB)` = marshaler (Marshal, MarshalAppend, MarshalState, Size). `(*LFYo4r_y)` = unmarshaler (Unmarshal, UnmarshalState). `h2yZePC66H.*` = funciones internas de codec. Complemento del paquete `mPmrATx3LG` para la serialización/deserialización real de mensajes proto |
| `puIQ09B` | **`github.com/gin-gonic/gin/render`** — renderers de respuestas HTTP de Gin | **96 funciones**. TODOS los tipos implementan `Render()` + `WriteContentType()`: `AUmAalGr5waR`=JSON, `CM68bz_Z7s`=TOML, `EMrTyP`=YAML, `GDpUon0`=IndentedJSON, `GeqcFg`=SecureJSON, `Gk3cfe`=AsciiJSON, `HDuzk3`=XML, `IX6ZTEMX`=Redirect, `KXuet9EV6`=Data, `NqzKKn`=HTML, `TDCNJy9OA3qI`=HTMLDebug, `TeJcG4b`=String. `(*Fz_Hd1)` = HTMLRender.Instance. Permite al servidor Gin del AC responder en JSON/YAML/TOML/XML |
| `oZTp4NP` | **Sistema de menús de Wails** (`wailsapp/wails/v2/pkg/menu`) | **236 funciones**. `(*HgZA4imGFle)` = `menu.Menu` (AddCheckbox, AddRadio, AddSeparator, AddSubmenu, AddText, Append, Merge, Prepend). `(*PN_Va0)` = `menu.MenuItem` (Append, Disable, Enable, Hide, InsertAfter, InsertBefore, IsCheckbox, IsRadio, IsSeparator, OnClick, Parent, Prepend, Show, SetLabel, SetChecked). `(*xJ9lgawHZ)` = helper interno. El AC tiene un menú de sistema con checkboxes/radio buttons (probable system tray o menú de ventana) |
| `sRymfbGH` | **Logger de Wails** (`wailsapp/wails/v2/internal/logger`) | **11 funciones**. `(*IoxEaTXSCpi)` = `logger.Logger` (Debug, Error, Fatal, Info, Print, Trace, Warning — 7 niveles de log). `(*Lr79bm7yBk)` = `logger.LogLevel` (String). `WHMq6j` = función auxiliar. Wails tiene su propio logger interno que el AC usa para output de consola |
| `xOy29u` | **Generador de structs/código** (probable `gorm.io/gen` o similar) | **111 funciones**. `(*AQ2s03MYHxdb)` = struct principal (AddEnum, AddType, Convert, GetGeneratedStructs, WithBackupDir, WithInterface, WithPrefix, WithSuffix + métodos garbled). `(*nmVUNRtFCRYi)` = builder de campos (AddArrayOfStructsField, AddEnumField, AddMapField, AddSimpleArrayField). `dF7FS3dTIB` (20 closures) = función de conversión/generación principal. Probablemente para generar structs Go desde esquemas de BD o API para el sistema de configuración del AC |
| `WBJsMwj` | **`google.golang.org/protobuf/internal`** — decodificador de wire format protobuf | **32 funciones**. `(*MQvtzY)` = wire reader (DecodeVarint32, DecodeVarintSlow, Done, Remaining, Skip, SkipFixed32, SkipFixed64, SkipGroup, SkipVarint). `(*Yo4hGq4Cc)` = unmarshaler (AllowedPartial, AppendField, Buffer, FindFieldInProto, SetBuffer, SetIndex, SetUnmarshalFlags, SizeField, UnmarshalFlags). Decodificador de bajo nivel para el wire format de protobuf |
| `dXBWjyeJNPdG` | **`github.com/go-ole/go-ole`** — interfaces COM/OLE de Windows para Go | Identificado por `SCODE`/`WCode` (métodos de error COM Windows), `IDispatch`, `IEnumVARIANT`, `QueryInterface`/`AddRef`/`Release` (trío IUnknown). Usado por `asYMlWeBL6f6` (WMI) y `A39Z4i` (go-wmi) para ejecutar queries WMI sobre COM/DCOM. Sin este paquete no hay WMI en Go |
| `FnLlWO` | **`golang.org/x/crypto/hpke`** — Hybrid Public Key Encryption (RFC 9180) | Identificado por métodos únicos del RFC 9180: `LabeledExtract`, `LabeledExpand`, `Encap`, `ExtractAndExpand`. HPKE combina KEM + KDF + AEAD en un solo esquema. Usado en el protocolo de autenticación del AC para encriptar el HWID u otros datos sensibles antes de enviarlos al servidor |
| `defy4D` | **`filippo.io/bigmod`** — aritmética modular de gran precisión (constante en tiempo) | Implementación de aritmética modular segura en tiempo constante para criptografía. Alternativa a `math/big` para operaciones RSA/DH donde el tiempo variable podría causar side-channel attacks. Usada en la capa TLS junto con `LYUqBZOV7RHi` (x509) |
| `TTJYj7n6_` | **`google.golang.org/protobuf/internal/encoding/json`** — codificador JSON interno de protobuf | Identificado por la API exacta del codificador JSON interno de protobuf: `StartMessage`, `EndMessage`, `WriteBool`, `WriteName`, `WriteFloat`, `WriteLiteral`, `WriteUint`, `Position`, `Snapshot`. Usado para serializar mensajes protobuf en formato JSON (además del wire format binario). El AC puede enviar/recibir mensajes en ambos formatos |
| `KyXsgao` | **`github.com/ugorji/go/codec`** — encoder/decoder multi-formato (msgpack, cbor, binc, json) | **572 funciones** — uno de los paquetes más grandes. Identificación DEFINITIVA por métodos únicos del codec ugorji: `AddExt/SetExt/SetBytesExt/Intf2Impl/TimeBuiltin` (métodos de `codec.Handle`), `DecodeBuiltin/EncodeBuiltin`, `TryNil/CheckBreak/ContainerType` (decDriver interface), `WriteArrayElem/WriteMapElemKey/WriteMapElemValue` (encDriver interface), `ReadArrayElem/ReadArrayEnd/ReadMapElemKey/ReadMapElemValue` (decoder interface), `ResetBytes/ResetString/NumBytesRead` (codec.Decoder), `MustDecode/MustEncode` (convenience), `TransientAddr2K/TransientAddrK` (map key transition interna). Múltiples sort.Interface para ordenamiento de claves de maps. `lyV9a7gF8C3J` = decoder principal con DecodeBool/DecodeBytes/DecodeExt/DecodeFloat64/DecodeInt64/DecodeNaked/DecodeStringAsBytes/DecodeTime/DecodeUint64. Usado probablemente por Wails IPC o serialización de datos internos del AC |
| `XFsN_Dbyaa1` | **`github.com/leodido/go-urn`** — parser de Uniform Resource Names (URN) | **28 funciones**. `EUIcCwW` = `urn.URN` (Equal, FComponent, IsSCIM, MarshalJSON, Normalize, QComponent, RComponent, RFC, SCIM, String, UnmarshalJSON — API exacta del tipo URN). `ou3UqBeAaxhA` = parser (Error, Parse, WithParsingMode — builder pattern). `SIczfkC` = tipo adicional con MarshalJSON/String/UnmarshalJSON. Importado por `Ve0nLKBMzmC` (go-playground/validator) para validación de campos URN en los mensajes del protocolo |
| `vypSO2` | **`wailsapp/wails/v2/internal/assetserver`** — servidor de assets/WebView de Wails | **66 funciones**. `YeuPlJHHiE` = AssetServer (AddPluginScript, ServeHTTP, ServeWebViewRequest, UseRuntimeHandler — `ServeWebViewRequest` es método EXCLUSIVO del AssetServer de Wails). `oxvD2t` = ResponseRecorder interno (Body, Code, Header, Write, WriteHeader — como httptest.ResponseRecorder). `ela5NnFM0` = http.ResponseWriter wrapper. `fKnyCzhrWZ` = ServeHTTP handler. Sirve los assets del frontend Wails (HTML/CSS/JS) al WebView2 |
| `GaVTbNOdw` | **`github.com/gin-gonic/gin/binding`** — implementaciones de binders Gin | **97 funciones**. Tipos con métodos `Bind`, `BindBody`, `Name`, `BindUri`: `bHrkqWjaJz`, `d83VbIcV`, `gczYp8G5X`, `ca3MJVW6` (binders Form/JSON/XML/Multipart). `aQkyqhVfno` = BindUri binder. `cOnDdZCu.TrySet` — método de form decoder. `HaRt5VBo8i0v.Error` — error de binding. NOTA: `cXI2MODy` (154 funcs) es la vista pclntab del mismo paquete que incluye las dependencias yaml/toml; `GaVTbNOdw` son las implementaciones core de binders |
| `H61m4ib4MK` | **`golang.org/x/net/http/httpproxy`** — configuración de proxy HTTP | **23 funciones**. `EetmwuusE_W.ProxyFunc` — `ProxyFunc()` es el método EXCLUSIVO de `httpproxy.Config` en x/net (devuelve una función que determina el proxy para cada request). Múltiples tipos con método `.t3ZxKPIsvi` (nombre de interfaz garbled). Usado por el HTTP transport de gRPC para detectar y usar proxies del sistema en las conexiones al servidor |
| `bCDs2NME30rs` | **`golang.org/x/text/unicode/norm`** — normalización Unicode NFC/NFD/NFKC/NFKD | **81 funciones**. `K6rIWyZnahNa` = `norm.Properties` — métodos preservados: BoundaryAfter, BoundaryBefore, Decomposition + 8 garbled (aB_krQPS, b_GsFQkgfkB, m4SGnt9TF, uNqLGw9En, vFLVaWZAb, xpsc3LISJ_C, zbPSwkC, zWaqaECsm). `ZRLRKCWnhV3` = `norm.Form` — métodos preservados: Bytes, IsNormalString, QuickSpan, String. Tipos internos: `aiCAsEN4qhLQ` (18 métodos = normWriter o reader interno), `evMxA_spS`, `ewL_xq`, `hBIwYYT` (3 métodos), `hfWGcVOBmSG` (9 métodos), `kaGy_uY3UW` (3 métodos), `rSJEk6Q7o2` (3 métodos), `Z_K2uJFs_z4` (2 métodos). 1 init closure, sin `decFunc`. Usado por `fm9Lu8y5M2` (x/text/transform) para normalización en pipelines de texto; probablemente también por `ki3UJTJ_` (x/net/idna) para normalizar nombres de dominio Unicode a punycode |
| `fm9Lu8y5M2` | **`golang.org/x/text/transform`** — interfaz Transformer y pipeline de transformaciones de texto | **82 funciones**. `NjWooFOGaHB` = tipo con BoundaryAfter/BoundaryBefore/Decomposition embebidos (norm.Properties dentro de transform). `Z3QOzQs` = tipo con Bytes/IsNormalString/Properties/QuickSpan/String (norm.Form extendida con método Properties() = `transform.NormalizingTransformer`). Tipos Transformer: `k0OGSxr3P0A` y `t3m7x3OSoD` comparten set idéntico {f0R4x06NAF, sN_ByIOgY, zfaSMB} = {Transform, Reset, Span} de `transform.Transformer` — dos implementaciones distintas del mismo interfaz. `fbvyfXUwRs3` (14 métodos) = transformer complejo (Chain o NopCloser). `ezoeH8RzZQ` (10 métodos) = transformer con estado. `glT6E9` (1 método: sN_ByIOgY) = wrapper mínimo. `Bydvv0dhu0dC` (2 métodos), `kfNqppH` (2 métodos), `r0xu5k` (1 método). 1 init closure, sin `decFunc`. Depende de `bCDs2NME30rs` (unicode/norm) para normalización Unicode dentro de los transformers |
| `hRyHlb` | **`golang.org/x/text/internal/number`** — formateo y redondeo de números Unicode | **197 funciones**. `eovpeu7y` = `number.Decimal` — métodos preservados exactos de la API: Assign, Round, RoundDown, RoundedInteger, RoundUp, Shift. 52 init closures (tablas de formateo numérico localizado). `decFunc` PRESENTE (string literals cifrados). Posición en pclntab: adyacente a `PkUgQB64wN09` (filippo.io/nistec). Dependencia interna de `WpWmMn` (`golang.org/x/text/language`) para formateo de números con redondeo Unicode-aware |

---

## Paquetes de Firmas de Detección (Detection Signature Registries)

Estos paquetes consisten casi enteramente en `init.funcX` closures, cada una registrando
una firma de cheat en el motor de detección. El patrón es idéntico al de `ra_94HIlnc6`
(299 firmas) documentado en sesiones previas. Juntos suman **694 firmas de cheat**
registradas antes de que el AC conecte al servidor.

| Paquete Garbled | Firmas | Descripción |
|-----------------|--------|-------------|
| `ra_94HIlnc6` | **299** | Paquete de firmas 1 — el más grande. 299 `init.funcX` closures = 299 patrones de detección de cheats registrados en el motor |
| `EyjsrRr` | **162** | Paquete de firmas 2. 162 `init.funcX` (rango func1-func162) con sub-closures (`.func100.1`, `.func113.1`, etc.) = 162 firmas de cheat. Función adicional `FHUq_cLQ` (helper de registro) |
| `WGfDxX0zz2M` | **117** | Paquete de firmas 3. 117 `init.funcX` (rango func1-func117) con sub-closures = 117 firmas. Un receiver `(*lg7_RxsCD).dNWdv6axQb26` = struct de datos de firma. 396 entradas pclntab totales |
| `YCJ5PUz_M` | **116** | Paquete de firmas 4. 116 `init.funcX` (rango func1-func116) con sub-closures = 116 firmas. Un receiver `(*d7eXzVsSSu).dHFZRa17` = struct de datos de firma. 395 entradas pclntab totales |
| `kNpc1A53` | **84** | Paquete de firmas 5. 84 `init.funcX` secuenciales (func1-func84, SIN gaps = sin sub-closures). Adyacente a `Moh1QXpPW` (Wails Events) en pclntab. 104 entradas totales |
| **TOTAL CONFIRMADO** | **778** | `ra_94HIlnc6`(299) + `EyjsrRr`(162) + `WGfDxX0zz2M`(117) + `YCJ5PUz_M`(116) + `kNpc1A53`(84) = 778 firmas de cheat en los 5 paquetes principales |
| **TOTAL CON ADICIONALES** | **~825** | + `BdDIRo5Tv42`(~10) + `msvCchDB`(~19) + `er9_yp`(~18) = ~825 firmas registradas en init() antes de conectar al servidor |

Paquetes relacionados (pequeños, init closures con strings garble-literals):
- `er9_yp` (57 funcs): **18 init closures + `decFunc` + `init.0` (pre-init)** → paquete AC custom con strings cifrados. Función `h0INzvC179I` (4 closures), `nQt7jN_pa` (3 closures), `YadKWKe` (1 closure), `ii5oZ3o5kV` (deferwrap1), `qtF5zU_s` (deferwrap1). Vecinos en pclntab: ANTES `vfFlma.decFunc` + `qavYW93.CoXO8f`; DESPUÉS `G2Ab89kPnVq`, luego `CNEPNkZW_`. Las 18 init closures probablemente registran 18 patrones de detección cifrados adicionales (posible blacklist de window titles cifrada)
- `G2Ab89kPnVq` (8 funcs): `(*ZZuYqyV1).Read` + `(*ZZuYqyV1).Read.func1/2` + `fahZxk` + `fahZxk.func1` + `Gl5WmfAG` + `rV0OyJUa` + `sEXjMc`. Tipo principal `ZZuYqyV1` implementa `io.Reader`. Posición: después de `er9_yp`, antes de `CNEPNkZW_`. Probable: pequeño wrapper de reader para el pipeline de detección de er9_yp
- `vfFlma` (171 funcs): **`decFunc` PRESENTE** (strings cifrados). Tipos con pointer receivers: `BAbJq9d`, `CCwyMQn`, `IgTYts3`, `LimqqEq`, `NZRYUsa`, `IiQTgkc`. El `FindExtension` ya documentado + nuevos tipos sugieren paquete AC propio de encoding o sub-módulo de extensiones protobuf. Aparece inmediatamente ANTES de `er9_yp` en pclntab — probable paquete companion del módulo de detección
- `qavYW93` (desconocido): aparece entre `vfFlma` y `er9_yp` en pclntab. Al menos función `CoXO8f`. No analizado en profundidad
- `w0bqp0RaeD` (125 funcs): **paquete de detección AC — inmediatamente anterior a `main` en pclntab**. 11 init closures. Función crítica `TnC04uk4`: estructura compleja con **6 goroutines lanzadas desde `func6`** — `func6` tiene 15+ sub-closures propios (func6.1 a func6.15, con sub-sub-closures func6.X.1) más una función NOMBRADA interna `dhCJhjGa` (referenciada como `func6.dhCJhjGa.16`) y `gowrap1-6`. Total `TnC04uk4`: func1-8 + (func6.1-15 + sub-closures) + func6.dhCJhjGa.16 + gowrap1-6. Funciones adicionales complejas: `QKaVdXPL` (8 closures + deferwrap1), `aiV8LM9eoaot` (5), `bxCiu5XSx1` (5), `OY6DwAo` (5), `F9SUnlRNJ` (4), `ugEeRXqA` (4 + deferwrap1), `wrBVrQ` (4), `V65onxa3a` (3), `YxgPBvwS` (3), `GomyLb` (2). 2 tipos comparables: `RIpbVXrKn` y `O1pxFVY` (usados como map keys). Funciones con deferwraps: `PHkNuSrB_TI` (1), `QKaVdXPL` (1), `QwD77pJ6RN` (1), `ugEeRXqA` (1), `xqXiqnUJ` (2). Sin `decFunc` (strings cleartext = detecciones visibles). Cadena pclntab confirmada: `GTShob4MX` (registry) → `T1XTTlVrw4yb` → `w0bqp0RaeD` → `main.(*HpP4qwz).Startup.func36`
- `lI61JSyj` (30 funcs): ~10 init closures, adyacente a `HEirZZGS` (error types) → posible paquete helper con strings cifrados
- `DQTJuVp_jZ` (115 funcs confirmadas): posición en pclntab entre `E5D2QU` (`internal/poll`) y `u26yruH2Ly_u` (`net` stdlib). Sin `decFunc`. **56 init closures** incluyendo `init.AAaUijjhdsR.func61/62` (función nombrada inner en init, igual a `ZrMEw4hJW` de main) y `init.GFwvMRC3zna[go.shape.bool].func57-60` (función genérica `bool`). 9 tipos comparables como map keys: `E5Cq49a`, `NgA3XBFdmdb`, `OK6p4yyx`, `SgNrJN`, `TB3Q9SaVnrtE`, `TKS90l`, `VmoFu3c`, `XvC0liM9`, `ZHTCBMPJZQf`. **Sin pointer receivers en pclntab** (0 métodos visibles en pclntab). Vecinos en área de memoria: `reflect`, `internal/bytealg`, `internal/runtime/atomic`, `bVtNY51`, `f3SkGnYxZwK`, `GYOujgH`, `HQCYVEFsqR`. `atomic.(*Pointer[go.shape.func(string) func()]).Store` = registro de handlers tipo `func(string) func()` vía atomic (factory pattern). DESCARTADO como `net/netip` (ya identificado como `LbVoc9TjklSB`). Hipótesis actual: **`github.com/go-playground/validator/v10`** (registra ~56 validators en init: required, len, min, max, email, url, uuid, etc. usando reflect; la función nombrada `AAaUijjhdsR` podría ser el loop de registro). **NOTA:** `Ve0nLKBMzmC` = `validator/v10` por receiver `(*adPnywIlKf75)` = FieldLevel — podría haber dos paquetes del ecosistema go-playground
- `xzmWSJlaR` (25 funcs): posición EXACTA en pclntab entre `DdtGTasS` (Wails WebView2 HTTP handler) y `OeqkvWSN2RR`. No tiene `decFunc`. Funciones: `ahNgAICOg1`, `bmy74wwJdwZ` (1 closure), `crVPWkXE`, `gkQ9qyR4FqU`, `ID_zIBF`, `kZasw0eN3Xna` (2 closures), `lyAIGupS`, `p5HcBGu` (2 closures), `u05jRTQAa3A`, `VArO4zl_KeQ`, `ZB3K74E9S`, `ZypiPGKd76`. 5 init closures. Probable: **`wailsapp/wails/v2/internal/binding`** (capa de binding JS↔Go de Wails, entre el HTTP handler y el runtime)

---

## Paquetes de UI (Wails)

| Paquete Garbled | Descripción | Evidencia |
|-----------------|-------------|-----------|
| `VKiZI7` | Integración WebView2 | Solo UI, no detección |
| `eiW4NTKLkx` | **Parser/template engine interno** — múltiples tipos con `.String()` | 277 funciones. Tipos `p8BgiJAaEcfj`, `p9u3cis5i`, `xkCm2WkY`, `itVPQUeJM`, `pNWZdwv`, `jGrO8NkzL3k`, `bc9029PFC5` todos implementan `String()` + tipo `R7XPqVqUIAYH` con type equality. Patrón típico de nodos AST. Adyacente a `KyXsgao` (ugorji codec) en pclntab — puede ser un parser de templates o un renderer de UI |
| `H5_Lib8AHNQN` | Runtime de Wails (ExecJS, WebView) | Wails runtime |
| `Moh1QXpPW` | **Sistema de eventos + IPC de Wails** — CONFIRMADO | `LFkHvs5` = IPC handler (DesktopIPC, RuntimeDesktopJS, WebsocketIPC — métodos EXCLUSIVOS de Wails IPC interface). `Ozow48G3Ot_` = EventManager (AddFrontend, Emit, Notify, Off, OffAll, Once, OnMultiple). **RESUELVE** la incógnita del paquete WebSocket IPC — es `Moh1QXpPW` |
| `aarD4xK0csQ` | **Generador de TypeScript/Go bindings de Wails** | `(*PWPBE1oU_)`: AddEnumToGenerateTS, AddStructToGenerateTS, DB, GenerateGoBindings, GenerateModels, SetOutputType, SetTsPrefix, SetTsSuffix, ToJSON, WriteModels. `(*AtRJSI)`: AddMethod, GetMethod, GetMethodFromStore, **GetObfuscatedMethod**, **UpdateObfuscatedCallMap** — maneja el mapeo de métodos ofuscados en IPC. `(*CMj0VyxJP)`: Call, InputCount, OutputCount, ParseArgs. Genera los bindings JS/TS para llamar métodos Go desde el frontend |
| `DnVABQn` | **Wails Windows File Dialog** (wrapper COM de `IFileDialog`) | `(*h7cWwdrR4)` y `(*msO2eZ_rGHF)`: GetResult, GetResults, Release, SetDefaultExtension, SetDefaultFolder, SetFileFilters, SetFileName, SetFolder, SetParentWindowHandle, SetRole, SetSelectedFileFilterIndex, SetTitle, Show, ShowAndGetResult, ShowAndGetResults — API exacta de `IFileDialog` COM de Windows. 29 funciones totales |
| `pNqsJkaQYkb` | **Wails Windows runtime message processor** | `(*QalYtrXV)`: **ProcessMessage**, **NewErrorCallback** (métodos preservados por interfaz), `dwyZaWNISgQ` con 22 gowraps = 22 goroutines de manejo de mensajes IPC, `aG_smaqv9a` con 33 closures, `fTRlStm`/`fuvIaFHv`. 108 funciones totales. Procesa mensajes del frontend WebView2 |
| `oFJMvWezJ9O` | **Wails Windows System Tray + Window management** — **148 funciones** | `DNwmM3_M8`: AddSubMenu, Dispose, IsDisposed, Show = **System Tray** menu. `DS8RzXk3kAd`: AddItem, AddItemCheckable, AddItemRadio = Tray items. `XVcwgpKUXH`: Text, ToggleVisible, Visible, Width, wPssRrtIyvyj = **Window control**. `CEUKoe2fX.String` = tipo con Stringer. Types con hash/equality: `DX_TpxDZJUY7`, `Kh_hcGr`, `Y0Kb5CIAy`, `HgjwOGdyXmU`. El AC **TIENE UN ÍCONO DE BANDEJA DEL SISTEMA** con menú contextual (AddItem/AddSubMenu/AddItemCheckable) |
| `Kd1BZ5fvE` | **Windows Toast Notification COM** (Wails) | `(*sTbdcks9).CreateToastNotifierWithId`, `(*GbfcJn_pKOW).Show`, `(*aDtFqDMMkjY).Show/VTable` — API exacta de `IToastNotificationManagerStatics` COM de Windows. `BPL0UpzsT8N9` y `F3aUca` = funciones de construcción de notificaciones. El AC envía **notificaciones toast de Windows** (probablemente para alertar sobre cheats detectados o mensajes del servidor) |
| `T1XTTlVrw4yb` | **Wails Windows COM Notifications** — capa WinRT para notificaciones del sistema | **77 funciones**. `gIXt71Y` con **4 deferwraps** = 4 COM objects liberados via defer (patrón típico de IToastNotification COM). `rQASPDFSlM` (5 closures), `rut8pO1Cygha` (8 closures), `kIBAtNDV` (4 closures) = handlers de eventos de notificación. `eBIHeYi1` (deferwrap1), `wBXcEJapFTP` (1 closure). `init.0` + 19 init closures = string literals cifrados para nombres de interfaces COM WinRT. Posición EXACTA: después de `Kd1BZ5fvE` (Toast) y `cjwjoC`/`aGxe6x3` (html/template), antes de `H5_Lib8AHNQN.(*SzXwEOvmo).InitializeNotifications/CleanupNotifications/IsNotificationAvailable` = Wails notification runtime. Este paquete implementa la capa COM intermedia entre Toast (`Kd1BZ5fvE`) y el runtime de Wails (`H5_Lib8AHNQN`). Sin `decFunc` en pclntab |
| `Z1uy7iC` | **Wails Windows UI** — window rendering/events | 74 funciones. 27 init closures. Adyacente a `oFJMvWezJ9O` (system tray) en pclntab. Funciones CamelCase preservadas: `Ae9n`, `BfPgt3`, `BlFDD8B`, `GpY8cZuaNwwF`, `JsAV44jb` |
| `DdtGTasS` | **Wails HTTP/WebView utility** | 28 funciones. `mya7fH2M8Uac`: Header, Write, WriteHeader, Finish (ResponseWriter-like + Finish). `odivUgRII`: URL, Method, Header, Body, Response, Close = envuelve `*http.Request` + `*http.Response`. `qbQ6Uc`: Read, Close = io.ReadCloser. `JdWBgfVE` = función libre. Probable `wailsapp/wails/v2/internal/http` o recorder de requests WebView |
| `IUEIXURnge4` | **Generic utility para Moh1QXpPW** — slices genéricos | 8 funciones. `F2QMrOIMF[go.shape.*uint8, go.shape.[]*Moh1QXpPW._lf3CKBawiY]` = función genérica que opera sobre slices del tipo privado de EventEmitter de Wails. 6 init closures. Probable `golang.org/x/exp/slices` o helper genérico interno |
| `cjwjoC` | Micro-paquete adjacente a Toast | 2 funciones, init con 1 closure. Stub o wrapper mínimo |
| `_g8PFjjvV` | **WebView2 COM interfaces** (Wails — capa COM baja) | **126 funciones**. `(*A6WSzA)` y `(*Gyh5Axsxa)` = COM interfaces (AddRef, QueryInterface, Release — trío IUnknown COM). Genéricos `G81XPK[n40Xq0xFt.bUOuJUuRWj]` y `G81XPK[n40Xq0xFt.ff_xOZxAyPpW]` = wrappers COM tipados. `(*h39tSl)` = handler COM con 4 métodos garbled (each con deferwrap = COM Release). Implementación de las interfaces COM de WebView2 |
| `n40Xq0xFt` | **WebView2 options/handlers** (Wails — capa alta) | **148 funciones**. `(*f1WnsiLnXC)` = `ICoreWebView2CreateCoreWebView2EnvironmentCompletedHandler` (EnvironmentCompleted). `(*gGu76LwS)` = `ICoreWebView2EnvironmentOptions` (AdditionalBrowserArguments, AllowSingleSignOnUsingOSPrimaryAccount, ExclusiveUserDataFolderAccess, Language, TargetCompatibleBrowserVersion). Tipos genéricos `bUOuJUuRWj` y `ff_xOZxAyPpW` = IDs de WebView2 handlers. El AC usa WebView2 como UI nativa de Windows |
| `uqFev5WTtxU` | **`wailsapp/wails/v2/internal/frontend/desktop/windows`** — frontend Windows de Wails | **482 funciones** — mayor paquete UI. `(*ABcS660pTk)` = Window (IsPopup — método de ventana Wails). `(*DKpXdvt3mv6l)` = otro tipo UI (String). Decenas de funciones libres garbled = handlers de eventos Windows (WM_*, WndProc, dialogs, drag-drop, etc.). El AC tiene una ventana de desktop completa con Wails sobre WebView2 |
| `YcVKoYA` | **`golang.org/x/net/html`** — parser HTML5 completo | **1243 funciones** — el paquete más grande de la lista (mayor que mPmrATx3LG). `(*OWcrC8mQD7HB)` = `html.Tokenizer` (AllowCDATA, Err, Next, NextIsNotRawText — métodos exactos del tokenizador HTML5). `(*hycInUFKki)` = parser interno. Incluye tablas de entidades HTML, especificaciones de elementos y atributos del spec HTML5. Usado por Wails para procesar el HTML de la interfaz |

---

## Paquetes de Infraestructura Go

| Paquete Garbled | Paquete Real | Evidencia |
|-----------------|-------------|-----------|
| `sNAkh4` | **Paquete de monitoreo de red propio del AC** — usa gopacket (`q4ajG0RM`) como dependencia | Tipos adyacentes a gopacket: `GrRzvM2` implementa `SetNetworkLayer`/`DecodeFromBytes`; `eeWJva_vkG` (arrays de 8 elementos, procesamiento por lotes), `cd1Utgxh0`, `gvBDt2BpBt7`, `wqT1oO`, `vaf3j0Z`, `ySbqKa9f0p`, `encTsS0_`. **CORRECCIÓN:** previamente identificado como gopsutil — incorrecto. El paquete gopsutil/v3/process real es `WvPUk5UlmHG` |
| `WvPUk5UlmHG` | **`github.com/shirou/gopsutil/v3/process`** | Confirmado por `CmdlineWithContext`, `CmdlineSliceWithContext`, `NameWithContext`, `CreateTimeWithContext`, `CmdlineSlice`, `Name`. Struct principal: `(*USEy__R7Bor9)`. Métodos en tabla de tipos: `CPUPercent`, `CreateTime`, `Foreground`, `IOCounters`, `MemoryInfo`, `MemoryMaps`, `NumThreads`, `PageFaults`, `SendSignal`. Llamado directamente desde `dUgTofmw.rpfbMOh` para leer cmdline del proceso del juego |
| `q4ajG0RM` | **`github.com/google/gopacket`** (biblioteca principal de captura de paquetes) | Métodos: `SetNetworkLayer`, `DecodeFromBytes`, `SetTransportLayer`, `SetApplicationLayer` — todas interfaces estándar de gopacket. Tipos: `Gt6tsVi`, `TXYGsDR`, `QOxXU6`, `Aa8DB2bIK66`, `UXODB2hYq`, `JorhuVUPZ`, `KAwWPEYU`, `DQcWKXdd`. Adyacente a `sNAkh4` en la tabla de tipos |
| `aFzxJpzp2` | **`crypto/hmac`** — HMAC implementation | `(*cESPyjL5y).ConstantTimeSum` (método exclusivo de `crypto/hmac`), `BlockSize`, `Size`, `Sum`, `Write`, `Reset`, `MarshalBinary`, `UnmarshalBinary` |
| `PcTWfu` | **`crypto/aes`** — cifrado AES con AES-NI | `(*pAWn49)` = AES block cipher; `(*myqAnYT)` = AES-GCM; `(*pRzjXqs)`, `(*gU1WIgp)` = modos CBC/CFB; funciones ensamblador: `_expand_key_128/192a/192b/256a/256b` |
| `pa9q8q` | **`crypto/des`** — DES y Triple DES | **42 funciones**. Dos tipos de cifrado: `(*pXyC3Ai)` = `des.desCipher` (BlockSize=8, Encrypt, Decrypt + `gOCZyg` garbled). `(*lPQ6ELOmD)` = `des.tripleDESCipher` (BlockSize=8, Encrypt, Decrypt con 3 sub-closures). `(*LmD7Ea4lI7).Error` = `des.KeySizeError`. 6 init funcs = carga de S-boxes y tablas de permutación DES. Funciones helper: `ab6TaeoX1`, `blXoJ6r`, `Cj7lDA2QBSl`, `dqjwtW`, `j_vT4LahNZ`, `mb3obkWZykT`, `n_Td_raz`, `oh_6BvwMUlz`, `uxh_Nfo3tKi`, `YA2aCsE` |
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
| `D9eGQ5PG` | **Lector de bits / decodificador de protocolo** | **39 funciones**. `(*h5VwJ9tCk)` = bit reader (Err, ReadBit, ReadBits, ReadBits64) — lee bits individuales de un stream de datos. `(*jshpAxPwIozu)` = lector/decodificador (Read + 4 métodos garbled). `(*xqiLWkRkgiw).Decode` = decoder principal. `(*P61aMY8x9u_r).Error` = error type. `t6BK6y2pRfht.Decode` + `t6BK6y2pRfht.First` = decoder con First(). Tipos usados en generics: `dvbpBubCS`, `rkkYPmZ5`. Probablemente `compress/flate` (DEFLATE bit reader) o un decodificador de protocolo de red (gopacket/layers internals) |
| `Lvy2smJECjF` | **`math/rand`** | `(*J3sgRye).ExpFloat64`, `(*J3sgRye).Uint32` = generación de números pseudoaleatorios |
| `sa96sWj2YdO` | **`crypto/rand`** o `math/rand v2` | `(*j2se_tz1Yd).Uint64`, `(*V_IwguYI).Uint64` — generación de uint64 aleatorio |
| `f3SkGnYxZwK` | **`time`** stdlib | **522 funciones**. `GRgO8es4xSt` = `time.Time` (Add, AddDate, After, AppendFormat, Before, Clock, Compare, Date, Day, Equal, Format, GobDecode, GobEncode, GoString, Hour, In, IsDST, ISOWeek, IsZero, Local, Location, MarshalBinary, MarshalJSON, MarshalText, Minute, Month, Nanosecond, Round, Second, String, Sub, Truncate, Unix, UnixMicro, UnixMilli, UnixNano, UnmarshalBinary, UnmarshalJSON, UTC, Weekday, Year, YearDay, Zone, ZoneBounds). `BwwazZtU.Error` = `time.ParseError`. `NOZFARjOV` = probablemente `time.Now()` (llamado desde `sOAbtRgFLa6_.GXErxuFMY9Z`). Genéricos: `eQS8EDEGHQI[go.shape.[]uint8]`, `[go.shape.string]`. 107+ init closures = carga de timezone data |
| `gF8Cxkh_m` | **`bytes`** stdlib | **316 funciones**. `aFoLnPeIZXz` = `bytes.Buffer` (Available, AvailableBuffer, Bytes, Cap, Grow, Len, Next, Read, ReadByte, ReadBytes, ReadFrom, ReadRune, ReadString, Reset, String, Truncate, UnreadByte, UnreadRune, Write, WriteByte, WriteRune, WriteTo, WriteString). Segundo tipo `n4RzlLuBp4GM` = probablemente `bytes.Reader` |
| `uaIqvkWry1B` | **`github.com/google/gopacket`** core — nombre en pclntab | **203 funciones** (mismo paquete que `q4ajG0RM` en type table). Mapeo: `o6FQLB36GI` = `gopacket.packet` (AddLayer, ApplicationLayer, Data, DecodeOptions, DumpPacketData, ErrorLayer, Layer, LayerClass, Layers, LinkLayer, Metadata, NetworkLayer, NextDecoder, SetApplicationLayer, etc.). `ryBzAPTZNP0` = gopacket.eagerPacket. `b3WyVCk2HTL1` = packet builder. `ZihZIXB` = `gopacket.PacketSource` (NextPacket, Packets con gowrap1). `KAwWPEYU` = `gopacket.Flow` (Dst, Endpoints, EndpointType, FastHash, Reverse, Src). `Ds_7K2NX5VxQ` = `gopacket.Endpoint` (EndpointType, FastHash, LessThan, Raw). `E7i4ZG8H6wo`, `HE1vNwA_vu61` = protocol layer implementations (CanDecode, DecodeFromBytes, LayerContents, LayerPayload, LayerType, NextLayerType, Payload, SerializeTo). `FKAQBww0pVJi` = `gopacket.DecodeFailure`. `Aa8DB2bIK66`, `NVAiuLvwuA`, `OfO2vubjULFe` = decoder collections (Contains, LayerTypes). `I8Be4EUk3p` = `gopacket.CaptureInfo` |
| `FXIaelQr` | **`github.com/pelletier/go-toml/v2`** — parser TOML completo | **266 funciones**. Mapeo de tipos CONFIRMADO: `(*fhGQ5e)` = TOML parser visitor (EnterArrayTable, EnterKeyValue, EnterTable, ExitKeyValue, MissingField, MissingTable — conceptos directos del spec TOML: `[[array.table]]`, `[table]`, `key = value`). `(*JWTweH7OD)` = `toml.LocalDateTime` (AsTime→time.Time, MarshalText, String, UnmarshalText). `(*XgEHC64)` = `toml.LocalDate` / `toml.LocalTime` (AsTime, MarshalText, String, UnmarshalText). `(*QVIuu8)` = error de parseo TOML (Error, Key, Position, String — posición de línea/columna). `(*eK_ZclYo).FromParser` = factory del parser. `(*WdMbCqYk).Decode` = decoder TOML. `(*DUu7Y3e).Encode` = encoder TOML. **NOTA CRÍTICA:** el nombre garble `JWTweH7OD` NO tiene relación con JWT — garble usa nombres pseudoaleatorios. Adyacente a `ogLSXbC1MZp` (yaml.v3) en pclntab, ambos importados por `cXI2MODy` (gin/binding) |
| `ogLSXbC1MZp` | **`gopkg.in/yaml.v3`** — parser YAML completo | **534 funciones**. Mapeo de tipos CONFIRMADO: `(*H_BNpi)` = `yaml.Node` (IsZero, LongTag, ShortTag, SetString, Encode con 3 defers, Decode con 1 defer — LongTag/ShortTag son métodos exclusivos de yaml.Node para conversión de tags YAML). `(*kQ5xrOiB)` = `yaml.mapSorter` (Len, Less, Swap — sort.Interface para ordering de claves YAML). `(*LQq6xAgupl_a).Decode` = decoder YAML principal. `(*aFYf1NlEWMK).String` = `yaml.Style` (estilo de nodo: Flow, Block, etc.). `(*CuVr5oh5).Error` = `yaml.TypeError` (error de tipo). Tipos internos: `aalXkyr6`, `cGxVYlaCG`, `gOt1AEQU`, `hZOq0fQW` = scanner/parser/emitter internos de yaml. Adyacente a `FXIaelQr` (go-toml/v2) en pclntab, ambos importados por `cXI2MODy` (gin/binding) |
| `J4pbxzDN6T` | Paquete helper pequeño | Funciones: `JxJlCi2H` (func1), `psK3gqUeJD` (func1, func2) — muy pequeño, probablemente wrapper o utility |
| `i7OyDUpCDA3q` | Helper de comparación/matching | Función `D5UnAj`; aparece entre funciones de `ncRaYk_Ke` |
| `lAEmxoi9bxt` | **`strings`** stdlib | **138 funciones**. `(*DJwHDX)` = `strings.Builder` (Cap, Grow, Len, Reset, String, Write, WriteByte, WriteRune, WriteString). `(*Ag5Lnmg_WHv)` = `strings.Reader` (Len, Read, ReadAt, ReadByte, ReadRune, Reset, Seek, Size, UnreadByte, UnreadRune, WriteTo). `(*AiEta36hdu)` = `strings.Replacer` (Replace, WriteString, + internos a6lKwZ, s0wSg4dlCOY). `(*drzvhmB8I)`, `(*eRMy0sP7)`, `(*ibaEalqjD1l)`, `(*n4I5Or07cwA)` = tipos internos replacer. `(*aBUwSdEWXG)` = `strings.byteReplacer` (Write, WriteString). Funciones de package: EeRyKGk, QEqxeJW10a (7 closures = strings.Fields/Split?), plus ~100 funciones de búsqueda/manipulación de strings |
| `ux88b3Kaxj2` | **`io/fs`** — filesystem abstraction | `L_MN3FaO2` = `fs.FileMode` (String, IsDir, IsRegular, Perm, Type); `(*DayNID)` = `fs.PathError` (Error, Unwrap, Timeout); `HqQopxeAAae` = `fs.FileInfo`; `BMjI1OjH7G` = `fs.DirEntry` (slice type en ge3tkaLtE) |
| `ge3tkaLtE` | **Generics runtime helpers** — funciones genéricas del AC | **141 funciones**, todas especializaciones `[go.shape.X]`. Funciones polimórficas: `V7lZyo` (especializado para []string, []int, []int32), `HgQqVJffBv` (especializado para TODOS los slice types del AC: `[]dvbpBubCS`, `[]jK8057_`, `[]ehTpuLs`, `[]BMjI1OjH7G`, etc.), `WasTXvziIF_` (especializado para [][]uint8, []MyTTk7_I.*, []string, []uint16, []uint8), `aB2rbmkLxw3` (4 shapes: *uint8, []uint8, interface, struct). `(*anzMQ02OY).Next` = iterador genérico. Probablemente `golang.org/x/exp/slices` (Contains, Index, Sort, etc.) o funciones runtime de Go 1.21+ generics. Nota: los tipos de package `MyTTk7_I`, `D9eGQ5PG`, `gF8Cxkh_m`, `ux88b3Kaxj2`, `xOy29u`, `zD7bA8PweG`, `FPB_79`, `D_t1gg3dia` fueron descubiertos gracias a las anotaciones de tipo de este package |
| `BhCuafOD` | **`github.com/cespare/xxhash/v2`** — hash xxHash de 64 bits | `(*BDap2x).Write` + `(*BDap2x).Sum64` = hash.Hash64 estándar; **`(*BDap2x).ResetWithSeed`** = método EXCLUSIVO de `cespare/xxhash/v2` (v2.2.0+), no existe en hash/fnv ni en zeebo/xxhash. 18 funciones totales. `BDap2x` = `xxhash.Digest`. Usado en el pipeline HWID (`FXWqsvy_`) para hashear los 5 componentes de hardware y producir el identificador HWID de 64 bits enviado al servidor |
| `xzFpaM3Mq3` | **`google.golang.org/grpc`** — cliente gRPC | 219 entradas; función principal `iwhm6fWm4v` (23 closures = bidirectional stream handler), `qd_m0I_QYkH7` (42 closures = conexión gRPC), `rKGI40jJGm` (42 closures = TLS handshake gRPC), `nL6NzEwvxPE` (27 closures = frame processor); referencias `DialContext`, `ClientStream`, `Invoke` confirman gRPC |
| `keNa_4m` | Paquete de sesión/token del AC | `(*pvusDPtY5vCp)` con métodos `evojx0Jc`, `_GoIAD`, `h7OdHS4`; funciones: `AAs7V4vmMZA`, `TAZv8E`, `DPCCiaoS`, `IUJ55okf`, `uE66JFzHSqr`, `Lnr6NFAWAA` — posiblemente manejo de session tokens o key derivation |
| `lNdBoswFIJHP` | **Paquete de reader/error interno** (pequeño helper) | 12 funciones. `NkWkVR` = tipo error (Error, Unwrap). `CznIJq20IbZP` = Reader (Read, ReadAll, iXQC04tIzT, t0zVf3). Funciones libres: `hJMGame`, `M2RaILaH_`, `txWaHrZ8`, `zA2B6WI81`. 5 init closures. Adyacente a `WGfDxX0zz2M` (detection pkg 3) en pclntab |
| `XEpNubH_5A5` | **HTTP/2 utility** — adyacente a `w3ZRgslh` (HPACK) | 61 funciones. Función principal `x2EsJ09LntzA` con 5+ closures. 1 init closure. Funciones: `re3sM0lu`, `bvmyWeOPNa`. Parte del stack HTTP/2 de gRPC |
| `HEirZZGS` | **Error types** del AC | 16 funciones. Tipos `fVlYVq` y `gTfmQ8uFof` con métodos garbled; tipos error `fiAWx26dm` y `ofV3sGd` con `Error()`. Paquete de definición de errores custom del AC |

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

## Paquetes del Runtime de Go (NO garbled — runtime internals)

Estos paquetes aparecen con sus nombres ORIGINALES en pclntab porque forman parte
del runtime de Go y garble no los obfusca (hacerlo rompería la ejecución):

| Paquete | Funciones | Observación |
|---------|-----------|-------------|
| `runtime` | **1594** — el paquete MÁS GRANDE del binario | `_defer`, `_func`, `_panic`, `_typePair` = internals del scheduler Go. Gestiona goroutines, GC, stacks, defer, panic. No garbled por necesidad del runtime |
| `reflect` | **634** | `aAhNyM` = `reflect.rtype` (Align, ArrayType, ChanDir, Common, ExportedMethods). Reflexión de tipos Go. Parcialmente garbled en tipo-tabla pero el paquete aparece sin garble |
| `syscall` | **599** | `BI7j6lzjkgw.Sockaddr` = `syscall.Sockaddr`; `FnICOY.Continued/CoreDump/Exited` = `syscall.WaitStatus` (métodos ungarbled!). Wrapper de syscalls de Windows |
| `sync` | **51** | `Cond.Wait` (ungarbled!); `ev96n7K.Lock/Unlock` (mismos tipos garbled del sync de usuario). Versión runtime-interna de sync usada por el scheduler |
| `math` | **32** | `Abs` = `math.Abs` (ungarbled!). Funciones matemáticas básicas. La mayoría de funciones están garbled pero el package name no |

---

## Paquetes Pendientes / Parcialmente Identificados

Estos paquetes existen en el binario pero NO han sido completamente identificados.
Pueden ser paquetes custom del AC, sub-paquetes de librerías ya documentadas, o
librerías de terceros menos conocidas.

### Potencialmente Identificados (requieren confirmación)

| Paquete | Funciones | Identificación | Evidencia |
|---------|-----------|----------------|-----------|
| `PkUgQB64wN09` | **71** | **CONFIRMADO: `filippo.io/nistec`** — implementación de curvas NIST en tiempo constante | Dos tipos de curva: `AgFtY5` y `D_cBbyrdL7N` — métodos preservados idénticos a la API de nistec: Add, Double, ECDH, Equal, IsOnCurve, Params, Public, ScalarBaseMult, ScalarMult, Sign. `ECDH` y `Sign` son métodos exclusivos de `filippo.io/nistec` (no existen en `crypto/elliptic`). Posición: entre `G1kVlqH3` (cryptobyte) y `t89_Iw9sGL1` (edwards25519) en pclntab — cluster cryptográfico de filippo.io. Usado por `OibOJGGM67` (ecdh stdlib) como backend en tiempo constante |
| `anGbYY2EWPw` | **314** | **Probable `google.golang.org/protobuf/internal/filedesc`** u otro sub-paquete protobuf grande | **249+ init closures** = carga masiva de descriptores proto en init(). `HEGKR0lOPr3` con métodos preservados `Parent` y `String` = `protoreflect.Descriptor` interface (`Parent()` devuelve el descriptor padre). Posición: DESPUÉS de `YCJ5PUz_M` (firmas de detección 4), ANTES de `vypSO2` (Wails assetserver). La combinación de 249+ closures de init + Parent/String = carga de descriptores protobuf de todos los archivos `.proto` del AC |
| `g7rfSp_eNaON` | **112** | **CONFIRMADO: wrapper custom de `bytes.Buffer` + `bytes.Reader`** | `ITLFV1h` = `bytes.Buffer` — API completa preservada: Available, AvailableBuffer, Bytes, Cap, Grow, Len, Next, Read, ReadByte, ReadBytes, ReadFrom, ReadRune, ReadString, Reset, String, Truncate, UnreadByte, UnreadRune, Write, WriteByte, WriteRune, WriteTo, WriteString. `WfaC7enF` = `bytes.Reader` — métodos: Len, Read, ReadAt, ReadByte, ReadRune, Reset, Seek, Size, UnreadByte, UnreadRune, WriteTo. `decFunc` PRESENTE (strings cifrados). NOTA: `sLqxHdT` aparece ANTES en pclntab (también tiene `aFoLnPeIZXz` = bytes.Buffer) — probable dependencia o paquete companion |
| `saxLnjfOw` | **244** | **CONFIRMADO: `github.com/json-iterator/go`** — encoder JSON de alto rendimiento | `VfCOmhAX5v.Encode` + `VfCOmhAX5v.EncodeValue` = API exclusiva de json-iterator (no existe en encoding/json). `d3No1nSZ94wm` = `jsoniter.Stream` (Bytes, Len, Reset, Write, WriteString). `eiQLC3Onif` = `jsoniter.Iterator` (Bytes, Drop, Len, ReadByte). Múltiples encoders: a7Xb4Ta, gTAEP2cR, n_83R1, oG_m2mVyC, WNyliq1kjKj = ptrEncoder/structEncoder/mapEncoder/sliceEncoder/arrayEncoder. `init.0/1/2` = tres fases de init (encoder cache, type registry, fallback). Llamado por `Ve0nLKBMzmC` (validator) y `cXI2MODy` (gin/binding). Gin usa json-iterator por defecto |
| `I2I3kG0xgKU` | 34 | **Utility HTTP/2** (posiblemente `golang.org/x/net/http2` sub-paquete interno) | 18 init closures de 34 funciones (53% ratio = muy alto, sugiere tablas de constantes). Posición CONFIRMADA: entre `ki3UJTJ_` (x/net/idna) y `i1aqCskEISkX` (net/http + http2) en pclntab. Llamado 27 veces por `i1aqCskEISkX` y 24 veces por `ki3UJTJ_`. Las 18 init closures probablemente registran constantes de protocolo HTTP/2 o tablas de strings de headers |

### Nombres Alternativos en Type Table (mismos paquetes, nombres garbled distintos)

El compilador garble asigna diferentes nombres en pclntab vs la type table. Estos son
la misma librería vista desde dos ángulos del binario:

| Nombre Type Table | Paquete Real | Evidencia |
|-------------------|-------------|-----------|
| `fuV0LEHC4VnD` | **`google.golang.org/protobuf/internal/impl`** = `mPmrATx3LG` en pclntab | `fuV0LEHC4VnD._c4YMU8F` = `mPmrATx3LG.(*_c4YMU8F)` (protoreflect.Message). Mismo tipo `_c4YMU8F` con métodos Clear/Get/Set/Range. Position: ANTES `bmV09O0` (protobuf internal) |
| `praU9nxF77Z` | **`google.golang.org/protobuf/internal/impl`** codec = `yzULHj8` en pclntab | `praU9nxF77Z.M5clpmDMoB` = `yzULHj8.(*M5clpmDMoB)` (marshaler), `praU9nxF77Z.LFYo4r_y` = `yzULHj8.(*LFYo4r_y)` (unmarshaler), `praU9nxF77Z.h2yZePC66H` = h2yZePC66H codec. ANTES: `eUmM3IpzIrV` (connectivity layer) |
| `mdv8HmLNJp6` | **`reflect`** stdlib o wrapper reflect-adjacent | `mdv8HmLNJp6.aAhNyM` = `reflect.rtype` (mismo garbled que en runtime reflect). Método preservado `Method` = `reflect.Type.Method()`. Tipos `Uc2Bsq`, `JniTXRl4udG`, `W_14mAruM4W` usados en type sigs protobuf. ANTES: `ldZb0s` (io), `q7klgJ` (json), `sMYs0N1` (os). Tiene `decFunc` |
| `t5W7d4Vh7E` | **`D3c7OwSvpJ`** en pclntab (FieldNumber/ExtensionNumber protobuf) | `t5W7d4Vh7E.Btf9ew8En` = `D3c7OwSvpJ.Btf9ew8En` (tiene IsValid). Usado como parámetro en `FindExtensionByNumber(IgTYts3, t5W7d4Vh7E.Btf9ew8En)`. Struct con json:"Score" json:"Port" json:"Visibility" = struct de servidor del juego. Tiene `decFunc` |

### Cluster de Paquetes Tiny con decFunc (strings cifrados, entre protoregistry y er9_yp)

Aparecen en la type table entre `vfFlma` (protoregistry) y `er9_yp`. Todos tienen `decFunc` = strings cifrados.
Probablemente son paquetes AC custom muy pequeños o micro-wrappers del protocolo.

| Paquete | Funciones | Observación |
|---------|-----------|-------------|
| `_UEudt` | ~1 (solo decFunc) | Solo `decFunc`. Micro-paquete con strings cifrados pero sin funciones visibles |
| `l3aebF` | ~6-7 | `eMJ6MxX`, `eMPj3Z`, `C24TiAXHaA`, `oD2fTcTIAaP`, `TrkCKKfkDgK` + decFunc. Struct con json:"g" y json:"a" (campos cortos = posiblemente compresión de protocolo) |
| `ftob0d` | desconocido | Aparece en closures de `_UEudt`. Tiene `decFunc` |
| `kXZSs4` | desconocido | Aparece en closures de `_UEudt`. Tiene `decFunc` |
| `fn_8wo` | desconocido | Aparece en closures de `l3aebF`. Tiene `decFunc` |
| `afvGbcC8` | desconocido | Tipo `S5aVCtzsnT` usado en `func(uint32) (uint32, uint32, bool, bool, []afvGbcC8.S5aVCtzsnT)` = firma de función de detección. `Yo4hGq4Cc` = posiblemente `WBJsMwj` (protobuf unmarshaler) visto desde type table |

### Paquetes Muy Pequeños (difíciles de identificar sin análisis dinámico)

| Paquete | Funciones | Observación |
|---------|-----------|-------------|
| `GtN_WZiIQ` | 14 | Error types: `dpN7za.Error/Unwrap`, `EKV_aagFA.Error` |
| `msvCchDB` | 23 | **Probable 7mo paquete de firmas de detección** — **19 init closures** con sub-closures (func1-func19 + sub-closures como func10.1, etc.). Sin `decFunc`, sin tipos propios, sin funciones no-init. Posición EXACTA: DESPUÉS de `w0bqp0RaeD` (detección activa), ANTES de `GztdjdTxnX1w`. Patrón idéntico a `BdDIRo5Tv42` y `kNpc1A53` |
| `GztdjdTxnX1w` | desconocido | Aparece DESPUÉS de `msvCchDB` en pclntab. Posición entre módulos de detección. No analizado en profundidad |
| `h7lxVr` | 27 | Métodos garbled, custom package. `buwDgX7Zkn.GetOriginFromURL` aparece ANTES en pclntab — `buwDgX7Zkn` es un helper de URL parsing |
| `sLqxHdT` | desconocido | Aparece ANTES de `g7rfSp_eNaON` en pclntab. Tiene `aFoLnPeIZXz` = `bytes.Buffer`. Probable companion o dependencia del wrapper de bytes |
| `buwDgX7Zkn` | desconocido | Función preservada `GetOriginFromURL` visible en binario. Aparece antes de `h7lxVr`. Helper de parsing de URLs (extrae origen de una URL) |
| `Q2v0Ja734` | desconocido | Tiene función `Sum64` — posible paquete hash adicional (fnv64? murmur? otro xxhash?) |
| `BdDIRo5Tv42` | 12 | **Probable 6to paquete de firmas de detección** — init.func1-func10 + init.func10.1 (sub-closure) = ~10 firmas adicionales de cheat. Mismo patrón que `kNpc1A53` (84 firmas) pero más pequeño. Sin decFunc |
| `Ga7B40suNkY` | 8 | 4 init closures |
| `BSjj4d` | 5 | 4 init closures |
| `F5ImErztW` | 6 | deferwrap1 = COM release probable |
| `b_hDwUv` | 5 | 3 init closures |
| `OBYSp0KPBjBZ` | 4 | `VyE6_84BwI` con 3 métodos garbled |
| `DqYZI2` | 3 | Micro-paquete, 2 funciones libres |
| `AnGoYbf` | 2 | Micro-paquete stub |
| `x_znjuZe` | 3 | Micro-paquete, 2 funciones libres |
| `XsrsAJ7` | 6 | 4 init closures |
| `OeqkvWSN2RR` | 75 | **Paquete AC propio** — módulo entre Wails binding y firmas de detección. Tipo principal `(*wpuRTCttiJE)` con métodos garbled: `i0MmnFa3242T`, `jDXVcOt`, `mEj4Tr3nZ` (1 closure), `mmdV1Ykl` (2 closures: func1, func1.1), `xshtxE_s1vq` (1 closure). 23+ funciones libres con closures: `xnAwpTnm76UG` (func1/2/2.1/3/3.1 = 5 closures), `d5kTRY6P6` (func1/1.1), `tDwaBFGXx2xX`, `i1YkpWm5lmUW`, `mdg_prpe`, `hlm6xNZm`, `tDwaBFGXx2xX`, `js7a2ELe2U`, `bOcp0h7Ie`. Sin `decFunc`. Cadena pclntab confirmada: `xzmWSJlaR` (Wails binding) → `OeqkvWSN2RR` → `YCJ5PUz_M` (firmas de detección 4). Probablemente módulo de orquestación que conecta el runtime de Wails con el sistema de firmas de detección |
| `D3c7OwSvpJ` | 33 | `Btf9ew8En.IsValid` — type con IsValid |

### Estadísticas Finales del Censo

| Categoría | Paquetes |
|-----------|---------|
| Paquetes del AC (código propio) | ~15 |
| Firmas de detección estáticas | 7 (~825 firmas total con msvCchDB + BdDIRo5Tv42) |
| Stdlib garbled | ~50 |
| Stdlib sin garble (runtime) | 5 |
| Terceros identificados | ~47 (+ hRyHlb, PkUgQB64wN09, saxLnjfOw, g7rfSp_eNaON confirmados) |
| Wails/UI packages | ~15 |
| Pendientes/no identificados | ~24 |
| **TOTAL** | **183+** |

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
