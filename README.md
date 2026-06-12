<div align="center">
  <img src="analysis/ui_logo.png" width="80" />
  <h1>l4d2c-anticheat-re</h1>
  <p>Ingeniería inversa completa del binario <code>l4d2c_anticheat.exe</code></p>
  <img src="https://img.shields.io/badge/analysis-static_only-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/binary-Go_%2B_garble-00ADD8?style=flat-square&logo=go&logoColor=white" />
  <img src="https://img.shields.io/badge/framework-Wails_v2-9E4784?style=flat-square" />
  <img src="https://img.shields.io/badge/protocol-gRPC_%2B_protobuf-4285F4?style=flat-square" />
</div>

---

## Contenido

- [Identificación del Binario](#identificación-del-binario)
- [Lo que hace el AC](#lo-que-hace-el-ac--resumen-técnico)
- [Metodología](#metodología)
- [Estructura del Repositorio](#estructura-del-repositorio)
- [Hallazgos Clave](#hallazgos-clave)
- [Mapa de Vectores de Bypass](#mapa-de-vectores-de-bypass)

---

## Identificación del Binario

```
Archivo:        l4d2c_anticheat.exe
Tamaño:         44,877,952 bytes (42.80 MB)
Arquitectura:   x86-64 (PE32+)
Lenguaje:       Go (confirmado por múltiples indicadores del runtime)
Ofuscador:      garble (obfuscación de nombres, tipos, strings y timestamp PE)
Framework UI:   Wails v2 (Go + WebView2/Chromium embebido)
Servidor:       https://l4d2center.com/0
Protocolo:      HTTP/2 + TLS 1.2/1.3 + Protocol Buffers v3 (gRPC)
Firma digital:  Authenticode -- "Editor verificado: L4D2Center" en UAC
Certificados:   Auto-firmados (Root CA 10 años, hoja ~2.3 años)
Certificate Pinning: NO (usa el store de certificados de Windows)
```

---

## Lo que hace el AC -- Resumen Técnico

### Al iniciar (Startup -- 87 goroutines)
- Carga la interfaz WebView2 (Chromium embebido) con UI de consola oscura
- Inicializa blacklists de ventanas, procesos y herramientas RE
- Carga el sistema WMI (COM/DCOM) para el HWID
- Configura el cliente HTTP/2 hacia `l4d2center.com`
- Configura el sistema de tokens de autenticación
- Expone tres métodos al frontend JS: `StartL4D2` (ID=2), `ConnectL4D2Server` (ID=0), `FrontendReady` (ID=1)

### Al conectar al servidor (ConnectL4D2Server -- 4 goroutines)

**Goroutine 1 -- Autenticación**
- Lee el SteamID del usuario desde el cliente de Steam local
- Ejecuta consultas WMI para recolectar HWID completo:
  - `Win32_Processor.ProcessorId` -- CPU ID
  - `Win32_DiskDrive.SerialNumber` / `Win32_DiskDrive.Model` -- Disco
  - `Win32_BaseBoard.SerialNumber` / `Win32_BaseBoard.Model` -- Placa madre
  - `Win32_VideoController.PNPDeviceID` / `.Name` -- GPU
  - `Win32_OperatingSystem.BuildNumber` / `.Version` -- OS
- Lee fecha de instalación de Steam (anti-smurf) via `AAUUUh.InstallDate`
- Solicita token de auth al servidor l4d2center.com
- Verifica si la cuenta es smurf (`GetSmurf`)
- Verifica si la cuenta está baneada (`GetBanned` via `Qi1Z8I.Banned`)

**Goroutine 2 -- Configuración de detección**
- Descarga `CheatSigs` (W99qYP[]) desde el servidor, patrones de bytes en memoria
- Enumera todos los archivos `.vpk` del directorio de addons del juego
- Inicializa el WatchList (`_6di6zc0se2v`)

**Goroutine 3 -- Loop de detección (ticker cada ~5-60s)**
- Scan de títulos de ventanas: `EnumWindows` + `GetWindowTextW` contra blacklist
- Scan de procesos: `CreateToolhelp32Snapshot` + `Process32First/Next` contra blacklist
- Scan de memoria del juego: `ReadProcessMemory` contra CheatSigs descargados
- Monitoreo de módulos DLL: `Module32First/Next` buscando inyecciones
- Heartbeat al servidor cada ~30 segundos
- Capturas de pantalla con `BitBlt` cada ~120 segundos (formato PNG NRGBA)

**Goroutine 4 -- Handler de resultados**
- Al detectar algo: captura screenshot, reporta al servidor, desconecta/banea
- Mensajes de consola en rojo (`#FF0000`) = ban/error

### Motor de Detección Principal (dUgTofmw.ga4oovjHCfg -- 37 closures)
- 33 goroutines de detección (func1-func33) + 4 sub-goroutines anidadas
- Cada goroutine maneja un vector de detección independiente
- Patrón productor-consumidor: `(*fcje4l4dl_uV).Feed` alimenta datos entre goroutines

---

## Metodología

Todo el análisis fue realizado de forma **estática**. El binario no fue ejecutado en ningún momento.

Técnicas utilizadas:

- **pclntab extraction**: Go conserva una tabla de funciones incluso con garble, permite recuperar el conteo exacto de closures y el árbol de llamadas aproximado
- **Type reflection metadata**: Go embebe metadatos de tipos en `.rdata`. Con garble los nombres están ofuscados pero la estructura queda expuesta
- **Protobuf annotations**: las anotaciones `protobuf:"..."` y `json:"..."` en `.rdata` no son ofuscadas por garble, lo que permitió reconstruir el esquema completo de mensajes gRPC
- **String extraction**: búsqueda de strings con encoding Windows-1252 para extraer blacklists, endpoints y nombres de WMI classes
- **Wails binding analysis**: Wails expone métodos al frontend JS con nombres en texto plano, filtrando parcialmente la ofuscación de garble

183 paquetes Go identificados de un total estimado de ~190 en el binario.

---

## Estructura del Repositorio

```
analysis/
  binario.md                  Análisis PE: secciones, imports, metadatos
  goroutines.md               Arquitectura de goroutines y timing
  protobuf_esquema.md         Esquema protobuf completo reconstruido
  hwid.md                     HWID: queries WMI, structs, bypass detallado
  pipeline_hwid.md            Pipeline HWID completo
  listas_negras.md            Blacklists completas de procesos y ventanas
  logica_deteccion.md         Lógica interna de detección (20 vectores)
  protocolo_red.md            Protocolo de red, autenticación, MITM
  motor_deteccion.md          Motor de detección principal
  pdFrspK_G.md                Motor hlavBkMcO: 644 verificaciones activas
  paquetes_identificados.md   Mapa completo de paquetes garble (183 paquetes)
  frontend_ui.md              Análisis de la UI: HTML, JavaScript, WebSocket IPC
  bypass_guide.md             Guía de bypass (20 vectores)

bypass/
  bypass_firmas.md            Bypass de firmas estáticas y CheatSigs
  bypass_hwid.md              Bypass del pipeline HWID con hook de vtable WMI
  bypass_procesos.md          Bypass de blacklist de procesos y ventanas
  bypass_red.md               MITM del protocolo gRPC sin certificate pinning

proto/
  esquema.proto               Definición protobuf reconstruida (16 mensajes)
```

---

## Hallazgos Clave

### Seguridad del Protocolo
- **Sin certificate pinning** -- MITM con Burp/Fiddler funciona sin modificaciones
- El stack TLS está embebido en Go, no usa WinSSL/SChannel ni OpenSSL
- Tres métodos de MITM documentados: Burp proxy, hook TLS in-process, DNS redirect

### HWID
- **5 componentes:** CPU, Disco, GPU, Motherboard, OS via WMI COM (`IWbemServices::ExecQuery` vtable índice 20)
- Hash final: xxHash64 de los 5 componentes concatenados (confirmado por `BhCuafOD`)
- Bypass documentado con hook de vtable WMI

### Detección
- **CheatSigs dinámicos** -- patrones de bytes descargados del servidor en cada sesión
- **778 firmas hardcoded** en 5 paquetes: `ra_94HIlnc6`(299) + `EyjsrRr`(162) + `WGfDxX0zz2M`(117) + `YCJ5PUz_M`(116) + `kNpc1A53`(84)
- **`hlavBkMcO`** con **644 sub-closures** ejecutándose en paralelo en `pdFrspK_G`
- **Blacklist de ~62 entradas en 4 listas:** ventanas (x32dbg, windbg, ghidra...), procesos (aimware, fiddler, ollydbg...), herramientas RE (dump, peek, kgdb), herramientas .NET (dnspy, ilspy, ida -)
- **Screenshot automático** cada ~120s como evidencia (PNG NRGBA)

### Anti-Análisis
- **garble** -- todos los nombres de paquetes, tipos y funciones son strings aleatorios
- **Timestamp PE = 0** -- impide correlacionar con fecha de build
- **APIs por ordinal** -- ReadProcessMemory, OpenProcess no aparecen como strings

### Correcciones de Análisis Previo
- `reportZombies` = runtime de Go (GC), no función del AC
- `ReportZerolen`/`IsZerolen` = librería gopacket/802.11, no función del AC
- `mTkHARFkg8` = paquete `sync` de Go (WaitGroup, Mutex)

---

## Mapa de Vectores de Bypass

| Vector | Dificultad | Método |
|--------|-----------|--------|
| Blacklist ventanas | Baja | Renombrar ventana del debugger |
| Blacklist procesos | Baja | Renombrar ejecutable |
| HWID pipeline | Media | Hook Z6ey1EJD.func1-11 con frida |
| CheatSigs memoria | Alta | No inyectar en proceso / hook VirtualQuery |
| 778 firmas estáticas | Alta | Polimorfismo / código sin firmas conocidas |
| `hlavBkMcO` 644 checks | Muy Alta | Análisis dinámico completo + hook Feed() |
| DLL modules | Alta | Manual mapping (sin LoadLibrary) |
| os/exec procesos | Media | Hook CreateProcess |
| SteamID | Media | Requiere cuenta legítima |
| InstallDate | Media | Hook de lectura de registro |
| Source Engine ConVars | Media | Hook ReadProcessMemory |
| Captura de paquetes (pcap) | Muy Alta | VM con NIC virtual |
| Protocolo red | Baja | Burp Suite (sin certificate pinning) |
