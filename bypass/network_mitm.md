# Network MITM — Intercepting Protobuf Traffic

The AC communicates with `https://l4d2center.com/` using HTTP/2 + TLS + Protocol Buffers.
No certificate pinning was detected — standard MITM works.

---

## What We Know About the Protocol

```
Transport:      HTTPS (TLS 1.2/1.3) + HTTP/2
Format:         Protocol Buffers (binary)
Server:         https://l4d2center.com/
Auth:           Token-based (AppKey + Auth + Session tokens)
Operations:     Connect, Heartbeat, Report, Ban check
```

---

## Method 1 — Burp Suite / Fiddler (Easiest)

**No cert pinning** → standard proxy MITM works.

```
1. Open Burp Suite
2. Proxy → Options → Import CA cert as DER
3. Import Burp CA cert into Windows:
   certlm.msc → Trusted Root Certification Authorities → Import → burp_ca.der

4. Set Windows system proxy:
   Settings → Network → Proxy → Manual: 127.0.0.1:8080

5. Launch l4d2c_anticheat.exe
6. Burp → HTTP History → look for l4d2center.com requests
7. Raw body = protobuf bytes
```

**Decode protobuf**:
```bash
# Install protoc
# Use our schema
cat captured_body.bin | protoc --decode=l4d2c.ClientReport proto/schema.proto

# Or use protobuf-inspector for unknown schemas:
pip install protobuf-inspector
python -m protobuf_inspector captured_body.bin
```

---

## Method 2 — Go TLS Hook (In-Process)

The Go binary compiles the TLS stack into itself (`crypto/tls` package).
There's no separate openssl or schannel — the Go runtime handles TLS internally.

**Find the TLS write function**:
```
In Ghidra, after loading with GoReSym labels:
Search → "crypto/tls.(*Conn).Write"
This function is called every time the AC sends data over TLS
```

**Hook approach**:
```cpp
// After finding the address of tls.(*Conn).Write in the binary:
// Patch the first bytes to redirect to our logger

void* tls_write_addr = (void*)(base + 0xXXXXXX);  // from Ghidra

// Our hook: log the plaintext before it gets encrypted
int64_t HOOK_tls_write(void* conn, void* slice_ptr, int64_t slice_len) {
    // slice_ptr/slice_len is Go's []byte representation
    go_slice* slice = (go_slice*)slice_ptr;
    LogBytes(slice->data, slice->len);  // dump plaintext
    return original_tls_write(conn, slice_ptr, slice_len);
}
```

---

## Method 3 — DNS Redirect + Custom Server

For full protocol analysis without executing:

```
1. Set up local DNS: resolve l4d2center.com → 127.0.0.1
2. Run a local HTTPS server (nginx + self-signed cert with our CA)
3. Import our CA into Windows trusted store
4. Launch AC → it connects to our server
5. Our server receives the full protobuf request
6. Log everything, forward to real server, relay response
```

**nginx config**:
```nginx
server {
    listen 443 ssl;
    server_name l4d2center.com;
    
    ssl_certificate     /path/to/fake_cert.pem;
    ssl_certificate_key /path/to/fake_key.pem;
    
    location / {
        # Log full request body
        access_log /tmp/ac_requests.log;
        
        # Forward to real server
        proxy_pass https://actual.l4d2center.com;
        proxy_ssl_server_name on;
        
        # Log responses too
        # (body logging requires nginx-mod-http-lua or similar)
    }
}
```

---

## Reading Captured Protobuf

Using our reconstructed schema:

```bash
# Install protoc
# https://github.com/protocolbuffers/protobuf/releases

# Decode a captured request:
cat request.bin | protoc --decode=l4d2c.ClientReport ./proto/schema.proto

# If field numbers are wrong (schema not exact), use raw decode:
cat request.bin | protoc --decode_raw

# Example raw decode output:
# 3: "76561198xxxxxxxxx"        ← SteamID (field 3)
# 5: "<binary>"                  ← CheatSigs (field 5, bytes)
# 9: "..."                       ← Smurf (field 9)
# 13: "addon_name.vpk"           ← Addons (field 13, repeated)
# 14: 1                          ← AddonsProvided (field 14, bool)
# 15: 0                          ← CheatSigsRequested (field 15)
```

---

## Replay Attack (Ban Research)

Once you can intercept traffic, you can:

1. Capture a valid session from an unbanned account
2. Replay the same `ClientReport` with modified fields
3. Test which fields trigger bans by:
   - Sending a known-bad HWID (from a banned account's HWData)
   - Sending a fake Pattern match
   - Sending a bad SteamID

This lets you reverse-engineer the exact ban conditions without risking a real account.

```python
import socket
import ssl
import struct

# Build a minimal ClientReport
def build_protobuf_field(field_num, wire_type, data):
    tag = (field_num << 3) | wire_type
    # wire_type: 0=varint, 2=len-delim, 1=64bit, 5=32bit
    if wire_type == 2:  # string/bytes
        return encode_varint(tag) + encode_varint(len(data)) + data
    return encode_varint(tag) + encode_varint(data)

def build_minimal_report(steam_id: str, hwid_disk: str) -> bytes:
    msg = b""
    msg += build_protobuf_field(3, 2, steam_id.encode())   # SteamID
    msg += build_protobuf_field(14, 0, 1)                  # AddonsProvided = true
    # Add HWData fields for HWID testing...
    return msg
```
