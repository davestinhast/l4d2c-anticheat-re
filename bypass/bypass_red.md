# Bypass — Interceptar Tráfico de Red (MITM)

El AC se comunica con `https://l4d2center.com/` usando HTTP/2 + TLS + Protocol Buffers. No hay certificate pinning — el MITM estándar funciona sin modificaciones adicionales.

---

## Lo que Sabemos del Protocolo

```
Transporte:     HTTPS (TLS 1.2/1.3) + HTTP/2
Formato:        Protocol Buffers v3 (binario)
Servidor:       https://l4d2center.com/
Auth:           Tokens + sesiones (AppKey + Auth + Session + Session2)
Operaciones:    Conectar, Heartbeat, Reportar, Verificar ban
```

---

## Método 1 — Burp Suite / Fiddler (El más Simple)

Sin certificate pinning → proxy MITM estándar funciona.

```
1. Abrir Burp Suite
2. Proxy → Options → Exportar CA cert como DER
3. Importar CA de Burp en Windows:
   certlm.msc → Entidades de certificación de confianza → Importar → burp_ca.der

4. Configurar proxy del sistema Windows:
   Configuración → Red e Internet → Proxy → Configuración manual:
   127.0.0.1:8080

5. Lanzar l4d2c_anticheat.exe
6. Burp → HTTP History → buscar requests a l4d2center.com
7. El body de cada request son bytes protobuf

8. Decodificar protobuf:
   protoc --decode_raw < capturado.bin
   # o con el esquema:
   protoc --decode=l4d2c.ConnectRequest proto/esquema.proto < capturado.bin
```

Decodificar con Python si no se tiene protoc:

```python
import struct

def decode_varint(data, pos):
    result = 0
    shift = 0
    while True:
        byte = data[pos]
        pos += 1
        result |= (byte & 0x7f) << shift
        if not (byte & 0x80):
            break
        shift += 7
    return result, pos

def decode_protobuf_raw(data):
    pos = 0
    fields = []
    while pos < len(data):
        tag, pos = decode_varint(data, pos)
        field_number = tag >> 3
        wire_type = tag & 0x7
        
        if wire_type == 0:  # varint
            value, pos = decode_varint(data, pos)
            fields.append((field_number, 'varint', value))
        elif wire_type == 2:  # len-delimited
            length, pos = decode_varint(data, pos)
            value = data[pos:pos+length]
            pos += length
            try:
                fields.append((field_number, 'string', value.decode('utf-8')))
            except:
                fields.append((field_number, 'bytes', value.hex()))
        elif wire_type == 1:  # 64-bit
            value = struct.unpack('<Q', data[pos:pos+8])[0]
            pos += 8
            fields.append((field_number, 'int64', value))
        elif wire_type == 5:  # 32-bit
            value = struct.unpack('<I', data[pos:pos+4])[0]
            pos += 4
            fields.append((field_number, 'int32', value))
    
    return fields

# Uso:
with open('capturado.bin', 'rb') as f:
    datos = f.read()
    
for field_num, tipo, valor in decode_protobuf_raw(datos):
    print(f"Campo {field_num} [{tipo}]: {valor}")
```

---

## Método 2 — Hook TLS en el Proceso (In-Process)

El binario Go compila el stack TLS internamente (`crypto/tls`). No hay openssl ni schannel externo — el Go runtime maneja TLS directamente.

Encontrar la función `crypto/tls.(*Conn).Write`:

```
En Ghidra (después de cargar con etiquetas GoReSym):
  Buscar → "crypto/tls.(*Conn).Write"
  Esta función es llamada cada vez que el AC envía datos por TLS
```

Hook:

```cpp
// Después de encontrar la dirección de tls.(*Conn).Write en el binario:
// Patchear los primeros bytes para redirigir a nuestro logger

// Representación de slice de Go: { ptr_datos, longitud, capacidad }
struct GoSlice {
    void*   datos;
    int64_t longitud;
    int64_t capacidad;
};

typedef int64_t (*TLS_Write_t)(void* conn, GoSlice* slice, int64_t slice_len);
TLS_Write_t TLS_Write_original = nullptr;

int64_t TLS_Write_hook(void* conn, GoSlice* slice, int64_t slice_len) {
    // slice contiene el plaintext antes de encriptar
    if (slice && slice->datos && slice->longitud > 0) {
        // Guardar plaintext en archivo para análisis
        FILE* f = fopen("C:\\tls_dump.bin", "ab");
        if (f) {
            fwrite(slice->datos, 1, slice->longitud, f);
            fclose(f);
        }
    }
    
    // Llamar a la función original
    return TLS_Write_original(conn, slice, slice_len);
}

void instalar_tls_hook(uintptr_t base_address) {
    // Offset obtenido de Ghidra para crypto/tls.(*Conn).Write
    uintptr_t tls_write_addr = base_address + 0xXXXXXX;  // offset de Ghidra
    
    // Instalar hook con detour de 14 bytes (jmp [rip+0] en x64)
    uint8_t hook[] = {
        0xFF, 0x25, 0x00, 0x00, 0x00, 0x00,  // JMP [RIP+0]
        0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00  // dirección de TLS_Write_hook
    };
    *(uint64_t*)(hook + 6) = (uint64_t)TLS_Write_hook;
    
    DWORD proteccion;
    VirtualProtect((void*)tls_write_addr, 14, PAGE_EXECUTE_READWRITE, &proteccion);
    
    // Guardar bytes originales
    TLS_Write_original = (TLS_Write_t)VirtualAlloc(nullptr, 32, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    memcpy(TLS_Write_original, (void*)tls_write_addr, 14);
    
    // Instalar hook
    memcpy((void*)tls_write_addr, hook, 14);
    
    VirtualProtect((void*)tls_write_addr, 14, proteccion, &proteccion);
}
```

---

## Método 3 — Redireccionamiento DNS + Servidor Propio

Para análisis completo sin ejecutar el AC en la máquina de producción:

```
1. Resolver l4d2center.com → 127.0.0.1
   Editar: C:\Windows\System32\drivers\etc\hosts
   Agregar: 127.0.0.1  l4d2center.com

2. Montar servidor HTTPS local (nginx + cert propio):
   openssl req -x509 -newkey rsa:4096 -keyout fake_key.pem -out fake_cert.pem -days 365 -nodes
   
3. Importar fake_cert.pem como CA confiable en Windows:
   certlm.msc → Entidades de certificación raíz de confianza → Importar

4. El AC conecta al servidor local → logear todo → reenviar al real
```

Configuración nginx:

```nginx
server {
    listen 443 ssl;
    server_name l4d2center.com;
    
    ssl_certificate     /path/to/fake_cert.pem;
    ssl_certificate_key /path/to/fake_key.pem;
    
    # Loguear body de requests (requiere módulo lua o echo)
    access_log /tmp/ac_requests.log;
    
    location / {
        # Reenviar al servidor real
        proxy_pass https://actual.l4d2center.com;
        proxy_ssl_server_name on;
        
        # Loguear bodies de request y respuesta con nginx-lua:
        # body_filter_by_lua_block { ... }
    }
}
```

---

## Análisis del Tráfico Capturado

Con `protoc --decode_raw`:

```bash
# Capturar request con Burp o nginx → guardar body como .bin
cat request.bin | protoc --decode_raw

# Salida de ejemplo (campos del mensaje principal ConnectRequest):
# 1: "token_hex_aqui"          <- Auth (field 1)
# 2: 1234567890123             <- Session (field 2)
# 3: 9876543210987             <- Session2 (field 3)
# 4: 4                         <- Version (field 4)
# 5: 550                       <- Game (field 5, L4D2 AppID = 550)
# 6: "192.168.1.100:27015"     <- Server (field 6)
# 10: <binary>                 <- HWData (field 10, bytes del ENOV9d)
# 11: "appkey_aqui"            <- AppKey (field 11)
# 13: "addon_name.vpk"         <- Addons (field 13, repeated)
# 14: 1                        <- AddonsProvided (field 14)
# 15: 1                        <- CheatSigsRequested (field 15)
```

---

## Replay Attack — Investigar Condiciones de Ban

Una vez interceptado el tráfico, construir mensajes personalizados para probar qué trigerea el ban:

```python
def encode_varint(value):
    bits = []
    while value > 127:
        bits.append((value & 0x7f) | 0x80)
        value >>= 7
    bits.append(value)
    return bytes(bits)

def pb_len_delimited(field_num, data):
    if isinstance(data, str):
        data = data.encode()
    tag = encode_varint((field_num << 3) | 2)
    return tag + encode_varint(len(data)) + data

def pb_varint_field(field_num, value):
    tag = encode_varint((field_num << 3) | 0)
    return tag + encode_varint(value)

def construir_reporte_prueba(steam_id: str, serial_disco: str) -> bytes:
    msg = b""
    msg += pb_len_delimited(3, steam_id)        # SteamID
    msg += pb_varint_field(14, 1)               # AddonsProvided = true
    msg += pb_varint_field(15, 1)               # CheatSigsRequested = true
    
    # HWData con serial modificado para probar ban por HWID
    hw_data = b""
    # ... construir ENOV9d con serial falso
    msg += pb_len_delimited(10, hw_data)        # HWData
    
    return msg

# Enviar el reporte:
import ssl, http.client
ctx = ssl.create_default_context()
conn = http.client.HTTPSConnection("l4d2center.com", context=ctx)
conn.request("POST", "/0", body=construir_reporte_prueba("76561198...", "FAKESN123"))
resp = conn.getresponse()
print(resp.status, resp.read())
```

Con este enfoque se puede probar sistemáticamente qué campo del HWID causa el ban, sin arriesgar una cuenta real.
