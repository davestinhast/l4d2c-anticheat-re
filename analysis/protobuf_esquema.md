# Esquema Protobuf — Reconstruido desde el Binario

El AC usa Protocol Buffers sobre HTTP/2 + TLS para toda la comunicación con el servidor.
El esquema fue reconstruido a partir de los tags de reflection que Go embebe en el binario (campo `protobuf:"..."` de cada struct).

---

## Paquete de Mensajes — P4mAKk

El paquete `P4mAKk` (nombre ofuscado por garble) contiene todos los tipos protobuf. Se identificaron los siguientes mensajes:

### Mensaje: BEt_icchsrxy — Mensaje Principal de Conexión

El mensaje más completo. Usado para enviar datos del cliente al conectarse.

```
struct BEt_icchsrxy (mensaje principal):
  Auth             field 1   bytes   token de autenticación
  Session          field 2   int64   ID de sesión
  Session2         field 3   int64   ID de sesión secundario
  Version          field 4   int64   versión del AC cliente
  Game             field 5   int64   ID del juego (enum: L4D2=?)
  Server           field 6   bytes   IP:Puerto del servidor al que conecta
  Code             field 7   bytes   código de respuesta/error
  Data             field 8   bytes   datos adicionales (uso no determinado)
  Smurf            field 9   bytes   resultado de detección de smurf
  HWData           field 10  bytes   HWID completo embebido (ver ENOV9d)
  AppKey           field 11  bytes   clave de aplicación
  ExtraInfo        field 12  bytes   información extra
  Addons           field 13  []bytes addons VPK instalados (repeated)
  AddonsProvided   field 14  bool    indica si se enviaron addons
  CheatSigsRequested field 15 bool   indica si se solicitaron firmas
```

Métodos Go asociados: `GetAddons`, `GetAddonsProvided`, `GetAppKey`, `GetAuth`, `GetCheatSigsRequested`, `GetCode`, `GetData`, `GetExtraInfo`, `GetGame`, `GetHWData`, `GetServer`, `GetSession`, `GetSession2`, `GetSmurf`, `GetVersion`

---

### Mensaje: Qi1Z8I — Respuesta de SteamID / Estado de Cuenta

```
struct Qi1Z8I (respuesta de validación de cuenta):
  Success    field 1   bool    operación exitosa
  Error      field 2   bytes   mensaje de error
  SteamID    field 3   bytes   Steam ID del jugador
  Nickname   field 4   bytes   nombre del jugador en Steam
  Banned     field 5   bool    si la cuenta está baneada
```

Métodos: `GetSuccess`, `GetError`, `GetSteamID`, `GetNickname`, `GetBanned`

---

### Mensaje: H1oahxz1l3iY — Respuesta del Servidor al Conectar

```
struct H1oahxz1l3iY (respuesta inicial del servidor):
  Success    field 1   bool      conexión aceptada
  CheatSigs  field 5   []bytes   firmas de cheats (repeated bytes, patterns)
  Error      field 2   bytes     mensaje de error
  ServerIP   field 3   bytes     IP real del servidor de juego
```

Métodos: `GetSuccess`, `GetCheatSigs`, `GetError`, `GetServerIP`

---

### Mensaje: ENOV9d — HWID Completo (Hardware Fingerprint)

```
struct ENOV9d (hardware fingerprint):
  Success    field 1   bool      éxito de recolección
  OS         field 2   []bytes   información del sistema operativo (repeated)
  CPU        field 3   []bytes   datos de CPU (repeated — múltiples núcleos)
  Disk       field 4   []bytes   datos de disco (repeated — múltiples discos)
  MotherB    field 5   []bytes   datos de motherboard (repeated)
  GPU        field 6   []bytes   datos de GPU (repeated — múltiples GPUs)
```

Métodos: `GetSuccess`, `GetOS`, `GetCPU`, `GetDisk`, `GetMotherB`, `GetGPU`

---

### Mensaje: EetHP7O — Datos de Disco Individual (Win32_DiskDrive)

```
struct EetHP7O (un disco):
  Model      field 2   bytes   modelo del disco
  Serial     field 2   bytes   número de serie
  Success    field 1   bool
```

Nota: `KLfm8u7` tiene la misma estructura — posiblemente CPU individual.

---

### Mensaje: BlhNU1RKfjg — Datos de GPU Individual (Win32_VideoController)

```
struct BlhNU1RKfjg (una GPU):
  Success    field 1   bool
  Model      field 3   bytes   nombre del adaptador de video
  PnpID      field 3   bytes   PnpDeviceID de la GPU
```

---

### Mensaje: BiK8wj — Información de Archivo

```
struct BiK8wj (archivo/addon):
  Path       field 1   bytes   ruta del archivo
  Size       field 2   int64   tamaño en bytes
```

---

### Mensaje: W99qYP — Firma de Cheat Individual (CheatSig)

```
struct W99qYP (un patrón de firma):
  Name       field 1   bytes   nombre del cheat
  Pattern    field 2   bytes   secuencia de bytes del patrón
```

Métodos: `GetName`, `GetPattern`

---

### Mensaje: P8ePybs — Respuesta de Autenticación

```
struct P8ePybs (auth response):
  Success    field 1   bool
  Session    field 2   int64   token de sesión asignado
  Error      field 3   bytes   error si falló
```

---

### Mensaje: QBb3vhiLW7n — Request de Autenticación

```
struct QBb3vhiLW7n (auth request):
  Auth       field 1   bytes   credencial de auth
  Session    field 2   int64   session ID
```

---

### Mensaje: N0Iwv9FQ — Respuesta Genérica

```
struct N0Iwv9FQ:
  Success    field 1   bool
  Error      field 2   bytes
```

---

### Mensaje: Ok4xyD92c8 — Versión del AC

```
struct Ok4xyD92c8 (version check):
  Success    field 1   bool
  Version    field 2   int64   versión del AC
```

---

### Mensaje: AAUUUh — Fecha de Instalación

```
struct AAUUUh (install date check):
  Success        field 1   bool
  Type           field 2   bytes   tipo de respuesta
  InstallDate    field 3   int64   timestamp UNIX de instalación de Steam
```

---

### Mensaje: ZwlL2R6Gvy — Info de Servidor/Juego

```
struct ZwlL2R6Gvy:
  Success    field 1   bool
  ID         field 2   bytes   ID del servidor
  Name       field 3   bytes   nombre del servidor
```

---

## Campos Adicionales Identificados por Nombre JSON

Estos campos existen en el protocolo pero su field number exacto no fue determinado:

```
AppID       GameID      Duration    Index
Mode        Protocol    Port        Map
MaxPlayers  Players     VAC         Visibility
SourceTV    TheShip     EDF         Score
Deaths      Bots        Witnesses   Money
Folder      InstallDate PnpID       Serial
Model       Manufacturer
```

**Witnesses** — posiblemente jugadores que "atestiguan" el cheat (colocado en el reporte de ban para evidencia social).

---

## Sistema de Tokens (BEwVDQOh5)

El paquete `BEwVDQOh5` maneja la generación y decodificación de tokens:

```
BEwVDQOh5.(*HUBoNKO).Token          — obtener token actual
BEwVDQOh5.(*HUBoNKO).RawToken       — token en formato raw
BEwVDQOh5.(*YDWWSd4K).EncodeToken   — codificar token (6 goroutines func1-func6)
```

La presencia de `EncodeToken` con múltiples goroutines sugiere que el token se construye de forma asíncrona, posiblemente combinando múltiples fuentes de datos (HWID + SteamID + timestamp) y cifrando el resultado.

---

## Flujo de Autenticación

```
Cliente                          Servidor
  |                                 |
  |-- GetVersion request ---------->|
  |<- Ok4xyD92c8 (version OK) ------|
  |                                 |
  |-- QBb3vhiLW7n (auth request) -->|
  |<- P8ePybs (session token) ------|
  |                                 |
  |-- Qi1Z8I (SteamID + HWID) ----->|
  |<- Qi1Z8I (banned/nickname) -----|
  |                                 |
  |-- BEt_icchsrxy (full connect) ->|
  |<- H1oahxz1l3iY (CheatSigs) -----|
  |                                 |
  |-- heartbeat (cada ~30s) ------->|
  |<- OK / ban                  ----|
```
