# Ingeniería Inversa — L4D2Center Anticheat

Análisis estático completo del binario `l4d2c_anticheat.exe`, el sistema anti-cheat del servidor comunitario L4D2Center. Este repositorio documenta todo lo que hace el programa, cómo lo hace, y los vectores de bypass identificados.

**Restricción:** Todo el análisis fue realizado de forma estática. El binario no fue ejecutado.

---

## Identificación del Binario

```
Archivo:        l4d2c_anticheat.exe
Tamaño:         44,877,952 bytes (42.80 MB)
Arquitectura:   x86-64 (PE32+)
Lenguaje:       Go (confirmado por múltiples indicadores del runtime)
Ofuscador:      garble (obfuscación de nombres de paquetes, tipos y símbolos)
Framework UI:   Wails v2 (Go + WebView2/Chromium embebido)
Servidor:       https://l4d2center.com/
Protocolo:      HTTP/2 + TLS + Protocol Buffers (probablemente gRPC)
Firma digital:  Presente (Authenticode) — muestra "Editor verificado" en UAC
```

---

## Lo que hace el AC — Resumen Técnico

### Al iniciar (FrontendReady)
- Carga la interfaz WebView2 (Chromium embebido)
- Registra tres métodos accesibles desde el frontend JS: `StartL4D2`, `ConnectL4D2Server`, `FrontendReady`

### Al conectar al servidor (ConnectL4D2Server)
Lanza cuatro goroutines en paralelo:

**Goroutine 1 — Autenticación**
- Lee el SteamID del usuario desde el cliente de Steam local
- Ejecuta consultas WMI para recolectar HWID (hardware fingerprint)
- Solicita token de auth al servidor l4d2center.com
- Valida la cuenta contra la base de datos de smurfs (GetSmurf)
- Verifica si la cuenta está baneada (GetBanned)

**Goroutine 2 — Configuración de detección**
- Descarga las firmas de cheats (CheatSigs) desde el servidor
- Enumera addons VPK instalados en el juego
- Inicializa el loop de escaneo

**Goroutine 3 — Loop de detección (ticker)**
- Cada N segundos ejecuta:
  - Scan de títulos de ventanas contra blacklist
  - Scan de memoria del proceso del juego contra CheatSigs
  - Heartbeat al servidor

**Goroutine 4 — Handler de resultados**
- Al detectar algo: captura screenshot con BitBlt → reporta al servidor → desconecta/banea

### Al lanzar el juego (StartL4D2)
- `CreateProcess("left4dead2.exe")`
- Monitorea salud del proceso (crash/exit)
- Vigila la lista de módulos DLL cargados en el juego
- Capturas periódicas de pantalla como evidencia

---

## Estructura del Repositorio

```
analysis/
  binario.md              Análisis PE: secciones, imports, metadatos
  goroutines.md           Arquitectura de goroutines y timing
  protobuf_esquema.md     Esquema protobuf completo reconstruido
  fingerprint_hwid.md     Fingerprinting de hardware (WMI queries)
  listas_negras.md        Blacklists de procesos y títulos de ventana
  logica_deteccion.md     Lógica interna de detección
  protocolo_red.md        Protocolo de red y flujo de autenticación
  cadenas.md              Strings clave extraídas del binario

bypass/
  bypass_firmas.md        Bypass del escáner de firmas de memoria
  bypass_hwid.md          Spoofer de HWID
  bypass_procesos.md      Ocultación de procesos y módulos
  bypass_red.md           MITM de tráfico de red (inspección de protobuf)

proto/
  esquema.proto           Definición protobuf reconstruida
```

---

## Herramientas Utilizadas

- `strings.exe` (MinGW) — extracción masiva de strings (182,482 líneas)
- `objdump --all-headers` (MinGW) — análisis de cabeceras PE
- Análisis manual de type reflection metadata (Go embebe nombres de tipos en .rdata)

---

## Hallazgos Clave

- Sin certificate pinning — MITM con Burp/Fiddler funciona directamente
- HWID usa cinco componentes: CPU (ProcessorId WMI), disco (serial WMI), GPU (PnpDeviceID), motherboard (serial), OS
- La blacklist de herramientas RE incluye Ghidra, x32dbg, WinDbg, de4dot, Sysinternals y otros
- Los CheatSigs son patrones de bytes descargados dinámicamente desde el servidor — no están hardcodeados
- El AC lee memoria del juego para validar estado de la partida (i4localSurvivorGunFire)
- Timestamp PE deliberadamente en cero (técnica anti-análisis de garble)
- La fecha de instalación de Steam es uno de los factores del sistema anti-smurf
- VKiZI7 es el paquete de integración WebView2 — no es detección de procesos
- Token system con RawToken/EncodeToken — posiblemente JWT o formato propio cifrado
