# Esquema Protobuf — Reconstruido desde el Binario

El AC usa Protocol Buffers sobre HTTP/2 + TLS para toda la comunicación con el servidor.
El esquema fue reconstruido a partir de las anotaciones `protobuf:"..."` que Go embebe en el
binario para el sistema de reflexión. Los field numbers y tipos son exactos.

---

## Paquete de Mensajes — P4mAKk

El paquete `P4mAKk` (nombre ofuscado por garble) contiene todos los tipos protobuf.
El paquete contenedor de tipos de detección es `x1JPPahi`.

### Método de verificación

Las anotaciones `protobuf:"..."` se encontraron directamente en el binario:

```
protobuf:"bytes,1,opt,name=Auth,proto3"
protobuf:"varint,2,opt,name=Session,proto3"
protobuf:"bytes,10,opt,name=HWData,proto3"
protobuf:"bytes,13,rep,name=Addons,proto3"
protobuf:"varint,14,opt,name=AddonsProvided,proto3"
protobuf:"varint,15,opt,name=CheatSigsRequested,proto3"
... (lista completa abajo)
```

---

## Mensajes Identificados — 17 Tipos

### 1. BEt_icchsrxy — Mensaje Principal de Conexión (cliente → servidor)

El mensaje más completo. Enviado al conectarse a un servidor. Contiene todo.

```protobuf
message BEt_icchsrxy {
  bytes          Auth               = 1;   // token de autenticación
  int64          Session            = 2;   // ID de sesión primaria
  int64          Session2           = 3;   // ID de sesión secundaria
  int64          Version            = 4;   // versión del AC cliente
  int64          Game               = 5;   // AppID del juego (L4D2 = 550)
  bytes          Server             = 6;   // "IP:Puerto" del servidor de destino
  bytes          Code               = 7;   // código de operación
  bytes          Data               = 8;   // datos adicionales (capturas de pantalla, etc.)
  bytes          Smurf              = 9;   // resultado de detección anti-smurf
  bytes          HWData             = 10;  // ENOV9d serializado — hardware fingerprint completo
  bytes          AppKey             = 11;  // clave de aplicación
  bytes          ExtraInfo          = 12;  // info extra del sistema (arch, platform, buildType)
  repeated bytes Addons             = 13;  // VPK files instalados — lista de rutas
  bool           AddonsProvided     = 14;  // indica si se enviaron los addons en este mensaje
  bool           CheatSigsRequested = 15;  // pedir firmas de cheats activos al servidor
}
```

**Métodos Go:** GetAddons, GetAddonsProvided, GetAppKey, GetAuth, GetCheatSigsRequested, GetCode, GetData, GetExtraInfo, GetGame, GetHWData, GetServer, GetSession, GetSession2, GetSmurf, GetVersion

---

### 2. ENOV9d — Hardware Fingerprint (HWID completo)

Embebido en BEt_icchsrxy.HWData como bytes serializado. Todos los datos provienen de WMI.

```protobuf
message ENOV9d {
  bool           Success = 1;   // éxito de recolección
  repeated bytes OS      = 2;   // Win32_OperatingSystem (BuildNumber, Version)
  repeated bytes CPU     = 3;   // Win32_Processor::ProcessorId
  repeated bytes Disk    = 4;   // EetHP7O serializado (por cada disco)
  repeated bytes MotherB = 5;   // KLfm8u7 serializado (motherboard)
  repeated bytes GPU     = 6;   // BlhNU1RKfjg serializado (por cada GPU)
}
```

El HWID es construido por `HWARRxN.KbBU6bdOa.Build`.

---

### 3. EetHP7O — Disco Individual (Win32_DiskDrive)

Sub-mensaje embebido en ENOV9d.Disk.

```protobuf
message EetHP7O {
  bool  Success = 1;
  bytes Serial  = 2;  // SerialNumber (también via IOCTL_STORAGE_QUERY_PROPERTY)
  bytes Model   = 3;  // Model del disco
}
```

---

### 4. KLfm8u7 — Motherboard Individual (Win32_BaseBoard)

Sub-mensaje embebido en ENOV9d.MotherB.

```protobuf
message KLfm8u7 {
  bool  Success = 1;
  bytes Serial  = 2;  // SerialNumber (SMBIOS)
  bytes Model   = 3;  // Model de la motherboard
}
```

---

### 5. BlhNU1RKfjg — GPU Individual (Win32_VideoController)

Sub-mensaje embebido en ENOV9d.GPU.

```protobuf
message BlhNU1RKfjg {
  bool  Success = 1;
  bytes Model   = 2;  // nombre del adaptador de video
  bytes PnpID   = 3;  // PNPDeviceID — incluye VEN_XXXX&DEV_XXXX&SUBSYS_XXXXXXXX
}
```

---

### 6. AAUUUh — Anti-Smurf (fecha de instalación Steam)

Enviado en lista junto con BiK8wj y W99qYP en los mensajes de detección.
La fecha viene del registro de Windows (HKLM\SOFTWARE\WOW6432Node\Valve\Steam\InstallDate).

```protobuf
message AAUUUh {
  bool  Success     = 1;
  bytes Type        = 2;  // clasificación de la instalación
  int64 InstallDate = 3;  // timestamp UNIX de instalación de Steam
}
```

**Métodos Go:** GetSuccess, GetType, GetInstallDate

---

### 7. BiK8wj — Información de Archivo (verificación VPK)

Reporta archivos del directorio del juego al servidor. Enviado como lista repeated.
El glob pattern `*.vpk` en el binario confirma la búsqueda de addons VPK.

```protobuf
message BiK8wj {
  bytes Path = 1;  // ruta del archivo (probablemente relativa al dir del juego)
  int64 Size = 2;  // tamaño en bytes
}
```

**Métodos Go:** GetPath, GetSize

---

### 8. ZwlL2R6Gvy — Addon/Workshop Info

Reporta addons de Steam Workshop instalados.

```protobuf
message ZwlL2R6Gvy {
  bool  Success = 1;
  bytes ID      = 2;  // workshop ID o hash interno del addon
  bytes Name    = 3;  // nombre del addon
}
```

**Métodos Go:** GetSuccess, GetID, GetName

---

### 9. W99qYP — Firma de Cheat Individual

Descargada del servidor. El cliente busca este patrón en la RAM del juego.

```protobuf
message W99qYP {
  bytes Name    = 1;  // nombre del cheat (para el reporte de ban)
  bytes Pattern = 2;  // bytes del patrón a buscar en memoria
}
```

**Métodos Go:** GetName, GetPattern

---

### 10. Qi1Z8I — Estado de Cuenta Steam

Respuesta del servidor al validar el SteamID del jugador.

```protobuf
message Qi1Z8I {
  bool  Success  = 1;
  bytes Error    = 2;
  bytes SteamID  = 3;  // Steam ID del jugador (64-bit como string)
  bytes Nickname = 4;  // nombre en Steam
  bool  Banned   = 5;  // true si la cuenta está baneada en L4D2Center
}
```

**Métodos Go:** GetSuccess, GetError, GetSteamID, GetNickname, GetBanned

---

### 11. H1oahxz1l3iY — Respuesta de Conexión (servidor → cliente)

Primer mensaje recibido al conectar. Contiene las firmas de cheats activos.

```protobuf
message H1oahxz1l3iY {
  bool           Success   = 1;
  bytes          Error     = 2;
  bytes          ServerIP  = 3;   // IP real del servidor de juego
  // field 4: no identificado
  repeated bytes CheatSigs = 5;   // W99qYP serializado — firmas activas
}
```

**Métodos Go:** GetSuccess, GetError, GetServerIP, GetCheatSigs

---

### 12. Ok4xyD92c8 — Versión del AC

Verificación de versión en el handshake inicial.

```protobuf
message Ok4xyD92c8 {
  bool  Success = 1;
  int64 Version = 2;  // versión del AC que el servidor exige
}
```

**Métodos Go:** GetSuccess, GetVersion

---

### 13. P8ePybs — Respuesta de Auth (session token)

Enviado por el servidor al autenticar. Contiene el session token para el resto de la sesión.

```protobuf
message P8ePybs {
  bool  Success = 1;
  bytes Error   = 2;
  int64 Session = 3;  // session token asignado
}
```

**Métodos Go:** GetSuccess, GetError, GetSession

---

### 14. QBb3vhiLW7n — Auth Request / Heartbeat

Enviado por el cliente para autenticarse y en los heartbeats periódicos.

```protobuf
message QBb3vhiLW7n {
  bytes Auth    = 1;  // credencial de auth
  int64 Session = 2;  // session ID activo
}
```

**Métodos Go:** GetAuth, GetSession

---

### 15. N0Iwv9FQ — Respuesta Genérica

Usado como respuesta genérica success/error para operaciones simples.

```protobuf
message N0Iwv9FQ {
  bool  Success = 1;
  bytes Error   = 2;
}
```

**Métodos Go:** GetSuccess, GetError

---

### 16. MdQhYNc — Mensaje Vacío (Heartbeat Ping)

Sin campos identificados. Probablemente el ping de heartbeat más básico.

```protobuf
message MdQhYNc {}
```

---

### 17. jUaIcGWu — Tipo Interno (uso no determinado)

Presente en el paquete x1JPPahi pero sin métodos Get visibles. Posiblemente un tipo interno
de la infraestructura de serialización, no un mensaje enviado directamente.

---

## Campos JSON Adicionales Identificados

Estos campos existen en structs Go que envuelven los mensajes protobuf o en la respuesta
del servidor A2S (Source Engine server query). Se encontraron via anotaciones `json:"..."`:

```
# Info de sistema enviada al servidor:
arch        platform    buildType   cpu

# Info del servidor de juego (A2S_INFO query):
AppID       GameID      Name        Map
Players     MaxPlayers  Bots        VAC
Visibility  Port        Protocol    Folder
Mode        ServerOS    ServerType  EDF
ExtendedServerInfo      SourceTV    TheShip

# Stats de proceso del juego (gopsutil):
rss         vms         swap        hwm
stack       pid         Score       Deaths
Duration    Index       Count       Money
Witnesses   Blocked     Caught

# CPU stats (gopsutil):
user        system      idle        nice
iowait      irq         softirq     steal
voluntary   involuntary             (context switches)

# Pantalla:
width       height      physicalSize
```

**Witnesses** — jugadores presentes en el servidor cuando se detectó el cheat (testigos del ban).

---

## Sistema de Tokens (BEwVDQOh5)

El paquete `BEwVDQOh5` maneja la generación y ciclo de vida de los tokens de auth:

```
BEwVDQOh5.(*HUBoNKO).Token           — obtener token actual
BEwVDQOh5.(*HUBoNKO).RawToken        — token en formato raw
BEwVDQOh5.(*YDWWSd4K).EncodeToken    — codificar token (8 goroutines: func1-func8)
```

Tipos identificados con método Copy: C0Yj6LwhyuOz, EDzTvG, EmP0VH5taiU, PM1h8dEdvhDV, TZ2qP_qUjnK
→ Sugiere que los tokens son structs copiables/inmutables (posiblemente sync/atomic o token rotation).

El `EncodeToken` con 8 goroutines indica que el token se construye en paralelo, probablemente
combinando múltiples fuentes: HWID + SteamID + timestamp + clave de sesión.

---

## Flujo de Autenticación Completo

```
Cliente                                    Servidor
  |                                           |
  |-- Ok4xyD92c8 request (version check) --->|
  |<-- Ok4xyD92c8 (versión requerida) --------|
  |                                           |
  |-- QBb3vhiLW7n (auth request) ----------->|
  |<-- P8ePybs (session token) --------------|
  |                                           |
  |-- BEt_icchsrxy (connect, full data) ---->|
  |     Auth, HWData, Addons, ExtraInfo       |
  |     CheatSigsRequested=true               |
  |<-- H1oahxz1l3iY (CheatSigs) -------------|
  |<-- Qi1Z8I (ban check result) ------------|
  |                                           |
  | [cada ~30 segundos:]                      |
  |-- QBb3vhiLW7n (heartbeat) -------------->|
  |<-- N0Iwv9FQ (ok) / ban signal ------------|
  |                                           |
  | [al detectar cheat:]                      |
  |-- W99qYP (matched pattern) + screenshot ->|
  |<-- ban                                  --|
```
