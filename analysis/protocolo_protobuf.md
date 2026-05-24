# Protocolo Protobuf — Esquema Completo de Mensajes

Análisis completo del protocolo gRPC/protobuf que usa el anticheat
para comunicarse con `l4d2center.com`. Todos los tipos extraídos del paquete
**`P4mAKk`** (pclntab) mediante sus métodos getter `ProtoMessage`/`ProtoReflect`.

---

## Paquete P4mAKk — Todos los Tipos de Mensaje

### 1. `BEt_icchsrxy` — Mensaje Principal de Autenticación (Cliente → Servidor)

El mensaje más importante. Se envía al servidor con TODOS los datos del cliente.

```protobuf
message BEt_icchsrxy {
    bytes    Data               = ?;  // payload cifrado (AES-GCM)
    bytes    Auth               = ?;  // token de autenticación
    bytes    AppKey             = ?;  // clave de aplicación/licencia
    string   Game               = ?;  // identificador del juego ("left4dead2")
    string   Version            = ?;  // versión del AC cliente
    string   Server             = ?;  // servidor L4D2 al que se conecta
    string   Session            = ?;  // ID de sesión primaria
    string   Session2           = ?;  // ID de sesión secundaria
    ENOV9d   HWData             = ?;  // datos de hardware (HWID completo)
    AAUUUh   Smurf              = ?;  // datos anti-smurf (fecha instalación OS)
    string   Code               = ?;  // código de estado/error
    repeated ZwlL2R6Gvy Addons  = ?;  // addons instalados
    repeated ZwlL2R6Gvy AddonsProvided = ?;  // addons del servidor
    repeated string CheatSigsRequested = ?;  // firmas de cheats a descargar
    bytes    ExtraInfo           = ?;  // información extra (capturas, etc.)
}
```

**Campos clave:**
- `HWData` = fingerprint de hardware completo (CPU + Disk + GPU + Motherboard + OS)
- `Session` + `Session2` = doble token de sesión para evitar replay attacks
- `CheatSigsRequested` = el AC puede solicitar firmas de cheats específicas al servidor
- `ExtraInfo` = campo de propósito general para datos adicionales (posiblemente screenshots)

---

### 2. `ENOV9d` — Datos de Hardware (HWID Agregado)

```protobuf
message ENOV9d {
    EetHP7O  CPU       = ?;  // datos del procesador
    EetHP7O  Disk      = ?;  // datos del disco duro
    BlhNU1RKfjg GPU    = ?;  // datos de la GPU
    EetHP7O  MotherB   = ?;  // datos de la motherboard
    AAUUUh   OS        = ?;  // datos del sistema operativo
    bool     Success   = ?;  // si la recolección fue exitosa
}
```

---

### 3. `EetHP7O` — Hardware con Serial (CPU / Disk / Motherboard)

```protobuf
message EetHP7O {
    string Model    = ?;  // nombre del modelo
    string Serial   = ?;  // número de serie
    bool   Success  = ?;  // éxito de la consulta WMI
}
```

**Mapeo WMI:**
- CPU: `Win32_Processor` → Model, Serial (MaxClockSpeed)
- Disk: `Win32_DiskDrive` → Model, Serial
- Board: `Win32_BaseBoard` → Model, Serial

---

### 4. `BlhNU1RKfjg` — Datos de GPU (Win32_VideoController)

```protobuf
message BlhNU1RKfjg {
    string Model    = ?;  // nombre del modelo de GPU
    string PnpID    = ?;  // Plug-and-Play ID del dispositivo
    bool   Success  = ?;  // éxito de la consulta WMI
}
```

---

### 5. `AAUUUh` — Anti-Smurf / Datos de OS

```protobuf
message AAUUUh {
    string InstallDate = ?;  // fecha de instalación del SO (Win32_OperatingSystem)
    bool   Success     = ?;  // éxito de la consulta
    int32  Type        = ?;  // tipo de detección anti-smurf
}
```

---

### 6. `ZwlL2R6Gvy` — Addon o Servidor

```protobuf
message ZwlL2R6Gvy {
    string ID      = ?;  // identificador único del addon/servidor
    string Name    = ?;  // nombre del addon/servidor
    bool   Success = ?;  // indicador de éxito
}
```

---

### 7. `H1oahxz1l3iY` — Respuesta de Conexión (Servidor → Cliente)

```protobuf
message H1oahxz1l3iY {
    repeated W99qYP CheatSigs = ?;  // firmas de cheats actualizadas
    string   ServerIP         = ?;  // IP del servidor L4D2
    string   Error            = ?;  // mensaje de error
    bool     Success          = ?;  // éxito de la operación
}
```

---

### 8. `W99qYP` — Firma de Cheat (Cheat Signature)

```protobuf
message W99qYP {
    string Name    = ?;  // nombre del cheat (e.g. "aimbot_v2", "wallhack_xyz")
    bytes  Pattern = ?;  // patrón de bytes para detectar en memoria
}
```

**Implicación:** El servidor envía patrones de bytes actualizados en cada sesión.
El AC escanea la memoria de `left4dead2.exe` buscando estos patrones (`ReadProcessMemory`).

---

### 9. `Qi1Z8I` — Respuesta de Ban (Servidor → Cliente)

```protobuf
message Qi1Z8I {
    bool   Banned   = ?;  // si el jugador está baneado
    string SteamID  = ?;  // SteamID64 del jugador
    string Nickname = ?;  // nickname del jugador
    string Error    = ?;  // mensaje de error
    bool   Success  = ?;  // éxito de la consulta
}
```

---

### 10. `P8ePybs` — Renovación de Sesión

```protobuf
message P8ePybs {
    string Session  = ?;  // nuevo token de sesión
    string Error    = ?;  // error si la renovación falla
    bool   Success  = ?;  // éxito de la renovación
}
```

---

### 11. `QBb3vhiLW7n` — Bundle de Autenticación

```protobuf
message QBb3vhiLW7n {
    bytes  Auth    = ?;  // token de autenticación
    string Session = ?;  // ID de sesión
}
```

---

### 12. `Ok4xyD92c8` — Verificación de Versión

```protobuf
message Ok4xyD92c8 {
    string Version = ?;  // versión requerida del AC
    bool   Success = ?;  // si la versión es aceptable
}
```

---

### 13. `BiK8wj` — Resultado de Escaneo de Archivo

```protobuf
message BiK8wj {
    string Path = ?;  // ruta del archivo escaneado
    int64  Size = ?;  // tamaño del archivo
}
```

**Uso:** Reporta archivos del juego con tamaños incorrectos (modificaciones).

---

### 14. `KLfm8u7` — Hardware con Serial (alternativo)

```protobuf
message KLfm8u7 {
    string Model   = ?;  // nombre del modelo
    string Serial  = ?;  // número de serie
    bool   Success = ?;  // éxito de la consulta
}
```

---

### 15. `N0Iwv9FQ` — Respuesta Genérica

```protobuf
message N0Iwv9FQ {
    string Error   = ?;  // mensaje de error
    bool   Success = ?;  // éxito
}
```

---

### 16. `MdQhYNc` — Mensaje Vacío / Ping

```protobuf
message MdQhYNc {
    // Sin campos (o campo único sin getter separado)
    // Posiblemente usado como KeepAlive o Ping
}
```

---

## Flujo Completo del Protocolo

```
CLIENTE (l4d2c_anticheat.exe)                    SERVIDOR (l4d2center.com)
─────────────────────────────                    ─────────────────────────

1. Recolectar hardware (asYMlWeBL6f6 WMI):
   ├── CPU  → EetHP7O{Model, Serial}
   ├── Disk → EetHP7O{Model, Serial}
   ├── GPU  → BlhNU1RKfjg{Model, PnpID}
   ├── Board→ EetHP7O{Model, Serial}
   └── OS   → AAUUUh{InstallDate}
              └──> ENOV9d{CPU, Disk, GPU, MotherB, OS}

2. Cifrar payload (PcTWfu AES-GCM):
   └── HMAC-SHA256 de ENOV9d (aFzxJpzp2)
   └── AES-GCM(ENOV9d + session + game_info)
              └──> BEt_icchsrxy.Data (bytes cifrado)

3. Enviar por gRPC/HTTP2+TLS1.3:
   BEt_icchsrxy ──────────────────────────────> /auth endpoint
   {
     Data: <AES-GCM(HWID)>,
     Auth: <token>,
     AppKey: <license>,
     Game: "left4dead2",
     Version: "X.Y.Z",
     Server: "<server_ip>",
     Session: "<session_id>",
     HWData: <ENOV9d>,
     Smurf: <AAUUUh>,
     Addons: [<ZwlL2R6Gvy>...],
     CheatSigsRequested: ["aimbot", "wh", ...],
   }

4. Servidor responde:
   <──────────────────────── H1oahxz1l3iY
   {
     CheatSigs: [W99qYP{Name, Pattern}...],
     ServerIP: "X.X.X.X",
     Success: true
   }

5. AC aplica firmas (scan de memoria):
   ReadProcessMemory(left4dead2.exe) → busca W99qYP.Pattern

6. Servidor envía veredicto de ban:
   <──────────────────────── Qi1Z8I
   {
     Banned: true/false,
     SteamID: "76561198XXXXXXXXX",
     Nickname: "PlayerName",
     Success: true
   }

7. Renovación periódica de sesión:
   <──────────────────────── P8ePybs
   {
     Session: "<nuevo_token>",
     Success: true
   }
```

---

## Protecciones del Protocolo

| Capa | Tecnología | Función |
|------|-----------|---------|
| **Transporte** | TLS 1.3 | Cifrado de canal completo |
| **Certificate pinning** | X.509 cert en binario | Anti-MITM |
| **CRL check** | pkix.CertificateList.HasExpired | Anti-cert-revocado |
| **Payload** | AES-GCM | Cifrado autenticado del HWID |
| **Integridad** | HMAC-SHA256 (ConstantTimeSum) | Anti-tampering, anti-timing |
| **Sesión** | Token doble (Session + Session2) | Anti-replay |
| **Compresión** | gzip (aENMwam6VEd) | Compresión del payload |

---

## Bypass del Protocolo

### Vector 1: Interceptar en TLS (MITM)
Bloqueado por certificate pinning. El AC verifica que el cert sea exactamente
el de `l4d2center.com` (DigiCert Trusted G4 RSA4096 SHA256 2024 CA1, offset 0x2A3E4B3).

**Bypass posible:** Patchear el binario para saltar la verificación del cert (offset 0x2A3E4B3),
o reemplazar el cert hardcodeado con uno propio y redirigir el DNS.

### Vector 2: Falsificar el HWID en BEt_icchsrxy
El HWID está cifrado con AES-GCM — sin la clave (embedded, garbled), no se puede
modificar el payload sin que falle el tag de autenticación del GCM.

**Bypass:** Extraer la clave AES del binario en tiempo de ejecución (antes del cifrado)
e interceptar la llamada a `PcTWfu.(*myqAnYT).Seal()`.

### Vector 3: Modificar respuesta del servidor (Qi1Z8I)
Si se puede interceptar el stream gRPC (rompiendo TLS), modificar
`GetBanned() = false` en la respuesta del servidor.

**Prerequisito:** Romper certificate pinning primero.

### Vector 4: Hookear ReadProcessMemory en el AC
Hacer que el AC no encuentre los patrones de cheats porque
`ReadProcessMemory` retorna memoria falsa/limpia.

**Técnica:** Inyectar código en el proceso del AC (difícil porque está protegido)
y hookear `MustFindProcByOrdinal`-loaded ReadProcessMemory.
