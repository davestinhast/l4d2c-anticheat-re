# Protocolo de Red

## Stack de Transporte

```
Capa        Implementación
───────────────────────────────────────────────
TLS 1.2/3   crypto/tls de Go (compilado en el binario)
HTTP/2      net/http2 de Go (compilado en el binario)
Protobuf    Protocol Buffers v3 (google.golang.org/protobuf)
Servidor    https://l4d2center.com/
```

El stack TLS está completamente embebido en el binario de Go — no usa WinSSL/SChannel ni OpenSSL. Esto implica que el MITM requiere interceptar a nivel de proxy del sistema, no hookeando librerías externas.

---

## Certificate Pinning

**No se detectó certificate pinning.**

El string `BuildNameToCertificate` encontrado corresponde a una API de TLS deprecada en Go — indica que NO hay implementación de pinning personalizado. El AC confía en el store de certificados de Windows.

Consecuencia: Burp Suite / Fiddler con su CA importada al sistema funciona sin modificaciones adicionales.

---

## Endpoint Conocido

```
https://l4d2center.com/0
```

El `/0` al final del path es inusual. Puede indicar:
- Versión del API: `/v0/` o `/0` como ruta base de gRPC
- En gRPC, las rutas siguen el formato `/package.Service/Method` — el `/0` podría ser una ruta obfuscada

---

## Sistema de Tokens y Sesiones

Los mensajes protobuf revelan un sistema de doble sesión:

```
Session    (varint, int64)   — sesión principal
Session2   (varint, int64)   — sesión secundaria
Auth       (bytes)           — token de autenticación
AppKey     (bytes)           — clave de la aplicación
```

El paquete `BEwVDQOh5` maneja la generación de tokens:

```
BEwVDQOh5.(*HUBoNKO).Token          — token activo
BEwVDQOh5.(*HUBoNKO).RawToken       — token en formato crudo
BEwVDQOh5.(*YDWWSd4K).EncodeToken   — encoder de tokens
  .func1 / .func2 / .func2.1 / .func3 / .func4 / .func5 / .func6
```

El `EncodeToken` tiene 6 sub-goroutines, lo que sugiere un proceso de construcción complejo (posiblemente HMAC o firma digital).

El paquete HTTP (`i1aqCskEISkX`) implementa autenticación básica:

```
i1aqCskEISkX.(*JC1j0AALEr5).BasicAuth
i1aqCskEISkX.(*JC1j0AALEr5).SetBasicAuth
i1aqCskEISkX.(*eJ7sJnVS).Authenticate
```

---

## Funciones de Red Identificadas

```
ConnectL4D2Server    — conexión principal al servidor
GetClientConn        — obtener conexión gRPC del pool
NewClientConn        — crear nueva conexión
GetServer            — obtener servidor asignado
GetServerIP          — IP del servidor de juego
IsStreamingClient    — check de streaming gRPC
IsStreamingServer    — check de streaming gRPC
EncodeToken          — codificar auth token
DecryptTicket        — desencriptar ticket del servidor
EncryptTicket        — encriptar ticket para envío
ReportError          — reportar error al servidor
ReportZombies        — reporte de estado de zombies (game state validation)
ReportZerolen        — reporte de longitud cero (posible detección de manipulación)
ReportValidationErrors — reporte de errores de validación
```

---

## ReportZombies y Validación de Estado

`ReportZombies` y `ReportZerolen` son funciones de reporte que van más allá del escaneo de memoria. Sugieren que el AC valida el estado interno del juego:

- `ReportZombies` — valida que el estado de los zombies en la simulación es coherente
- `ReportZerolen` — detecta paquetes o datos de longitud cero (posible anti-speedhack o validación de netcode)
- `ReportValidationErrors` — errores genéricos de validación del estado del juego

---

## TLS — Detalles

```
TLSConfig
TLSClientConfig
TLSHandshakeStart
TLSNextProto         ← HTTP/2 ALPN negotiation ("h2")
DialTLSContext
```

La presencia de `TLSNextProto` con valor probable "h2" confirma HTTP/2 con ALPN. El handshake es estándar TLS 1.2/1.3 sin extensiones propietarias.

---

## Método 1 — Interceptar con Burp Suite

```
1. Abrir Burp Suite
2. Proxy → Options → Export CA cert como DER
3. Importar en Windows: certlm.msc → Autoridades de certificación raíz de confianza → Importar
4. Configurar proxy del sistema: Configuración → Red → Proxy → Manual: 127.0.0.1:8080
5. Lanzar l4d2c_anticheat.exe
6. Burp → HTTP History → buscar requests a l4d2center.com
7. El body de cada request son bytes protobuf
```

Decodificar protobuf capturado:
```bash
cat capturado.bin | protoc --decode_raw
# o con el esquema:
cat capturado.bin | protoc --decode=P4mAKk.BEt_icchsrxy proto/esquema.proto
```

---

## Método 2 — Hook del TLS Write (In-Process)

Dado que Go compila el stack TLS internamente, se puede hookear `crypto/tls.(*Conn).Write` para interceptar el texto plano antes de que sea encriptado.

```cpp
// Encontrar la dirección de tls.(*Conn).Write en Ghidra (con GoReSym labels)
// Luego hookear:

int64_t HOOK_tls_write(void* conn, void* slice_ptr, int64_t slice_len) {
    // Go usa representación de slice: { data ptr, len, cap }
    struct go_slice { void* data; int64_t len; int64_t cap; };
    go_slice* slice = (go_slice*)slice_ptr;
    
    // Volcar el plaintext antes de encriptar
    FILE* f = fopen("tls_dump.bin", "ab");
    fwrite(slice->data, 1, slice->len, f);
    fclose(f);
    
    return original_tls_write(conn, slice_ptr, slice_len);
}
```

---

## Método 3 — DNS Redirect + Servidor Local

```
1. Resolver l4d2center.com → 127.0.0.1 (modificar hosts file)
2. Montar servidor HTTPS local con nginx + certificado propio
3. Importar CA propia como confiable en Windows
4. El AC conecta al servidor local → logear todo → reenviar al servidor real
```

Este método permite análisis completo del protocolo sin hooking.

---

## Replay Attack para Investigar Condiciones de Ban

Una vez capturado el tráfico:

```python
import struct

def encode_varint(value):
    bits = []
    while value > 127:
        bits.append((value & 0x7f) | 0x80)
        value >>= 7
    bits.append(value)
    return bytes(bits)

def pb_field(field_num, wire_type, data):
    tag = (field_num << 3) | wire_type
    if wire_type == 2:  # len-delimited
        encoded = data if isinstance(data, bytes) else data.encode()
        return encode_varint(tag) + encode_varint(len(encoded)) + encoded
    return encode_varint(tag) + encode_varint(data)

def build_test_report(steam_id: str, hwid_disk_serial: str) -> bytes:
    msg = b""
    msg += pb_field(3, 2, steam_id)            # SteamID
    msg += pb_field(14, 0, 1)                  # AddonsProvided = true
    # Agregar HWData modificado para probar qué HWID trigerea ban
    return msg
```
