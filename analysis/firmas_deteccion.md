# Firmas de Detección — Mapa Completo

Este documento centraliza el análisis de todos los paquetes de firmas de detección
del AC de L4D2Center. Incluye los 5 registros estáticos de firmas de cheat y el
paquete de detección adicional `er9_yp`.

---

## Resumen Ejecutivo

| Componente | Tipo | Firmas/Checks | Descripción |
|------------|------|--------------|-------------|
| `ra_94HIlnc6` | Registro estático firmas | **299** | Paquete de firmas más grande |
| `EyjsrRr` | Registro estático firmas | **162** | Paquete de firmas 2 |
| `WGfDxX0zz2M` | Registro estático firmas | **117** | Paquete de firmas 3 |
| `YCJ5PUz_M` | Registro estático firmas | **116** | Paquete de firmas 4 |
| `kNpc1A53` | Registro estático firmas | **84** | Paquete de firmas 5 |
| `er9_yp` | Detección AC custom | **18** init closures | Strings cifrados, detección especializada |
| `pdFrspK_G.hlavBkMcO` | Motor de detección directo | **644** closures | Checks hardcodeados en la función principal |
| **TOTAL** | | **~1440** | Detecciones individuales combinadas |

---

## Arquitectura del Sistema de Firmas

```
init() de cada paquete de firmas
    └── init.funcX() (1 por firma)
            └── Registra firma en dUgTofmw (motor central)
                    └── dUgTofmw.ga4oovjHCfg() (scanner principal, 33 goroutines)
                            └── Evalúa cada firma contra la memoria del proceso
```

Las firmas se registran **antes de conectar al servidor**. Son estáticas y
hardcodeadas en el binario. El servidor (`l4d2center.com:443`) envía
firmas adicionales en el mensaje `H1oahxz1l3iY.CheatSigs` (campo 5)
con el struct `W99qYP` que contiene `Name` + `Pattern` de bytes.

---

## Paquete ra_94HIlnc6 — 299 Firmas

**Estructura:**
- 299 closures `init.func1` ... `init.func299`
- Cada closure registra una firma de cheat en el motor `dUgTofmw`
- Algunas firmas tienen sub-closures (`.func100.1`, `.func113.1`, etc.)
  indicando lógica de matching más compleja (wildcards, ranges de bytes)

**Características:**
- Paquete MÁS GRANDE del registro de firmas
- Sin `decFunc` (no confirmado en este paquete)
- Tipo de firma: patrones de bytes en memoria del proceso del juego

**Análisis estructural:**
```
ra_94HIlnc6.init
├── func1   → firma de cheat #1
├── func2   → firma de cheat #2
│   └── ...
├── func99  → firma de cheat #99
├── func100 → firma con lógica extra
│   └── func100.1 → sub-closure (wildcard/range)
└── func299 → firma de cheat #299
```

---

## Paquete EyjsrRr — 162 Firmas

**Estructura:**
- 162 closures `init.func1` ... `init.func162`
- Función auxiliar **`FHUq_cLQ`** = helper de registro (facilita el proceso
  de registrar firmas con formato particular)
- Sub-closures en: `.func100.1`, `.func113.1` y otros — lógica multi-paso

**Características:**
- Segundo paquete más grande
- Tiene función helper propia `FHUq_cLQ` (única entre los 5 paquetes)
- Esto sugiere que usa un formato de firma distinto o un mecanismo
  de registro más sofisticado

---

## Paquete WGfDxX0zz2M — 117 Firmas

**Estructura:**
- 117 closures `init.func1` ... `init.func117`
- Struct adicional: `(*lg7_RxsCD).dNWdv6axQb26` = struct de datos de firma
  con método `dNWdv6axQb26` — el único receptor en este paquete
- 396 entradas totales en pclntab

**Características:**
- Tipo struct `lg7_RxsCD` sugiere que las firmas de este paquete
  tienen metadatos adicionales (nombre del cheat, versión, etc.)
- El método garbled `dNWdv6axQb26` podría ser un getter/serializer
  del struct de firma

---

## Paquete YCJ5PUz_M — 116 Firmas

**Estructura:**
- 116 closures `init.func1` ... `init.func116`
- Struct adicional: `(*d7eXzVsSSu).dHFZRa17` = struct de datos de firma
- Funciones adicionales: `FBYoadgUZP8` (8 closures), `SwIdLuTt7VG_`,
  `HOcnuopELKLl` = helpers del paquete
- 395 entradas totales en pclntab

**Características:**
- Similar a `WGfDxX0zz2M` (mismo tamaño, misma estructura con struct)
- Posición pclntab: **después de `OeqkvWSN2RR`** (módulo AC)
- `FBYoadgUZP8` con 8 closures es función de utilidad interna

---

## Paquete kNpc1A53 — 84 Firmas

**Estructura:**
- 84 closures `init.func1` ... `init.func84`
- **SIN sub-closures** (todos secuenciales, sin `.funcX.1`)
- 104 entradas totales en pclntab

**Características:**
- Paquete más simple — firmas directas sin lógica compleja
- Adyacente a `Moh1QXpPW` (Wails Events) en pclntab
- Las firmas sin sub-closures sugieren que son patrones de bytes simples
  (sin wildcards ni lógica de multi-match)

---

## Paquete er9_yp — Detección Especializada

**Estructura:**
```
er9_yp.init
├── init.0     → pre-init (inicialización de variables de paquete)
├── func1-func18 → 18 closures de registro
└── decFunc    → descifrador de string literals (garble -literals)

er9_yp.h0INzvC179I → 4 closures (func1-4)
er9_yp.nQt7jN_pa   → 3 closures (func1-3)
er9_yp.YadKWKe     → 1 closure
er9_yp.ii5oZ3o5kV  → deferwrap1 (cleanup de recurso)
er9_yp.qtF5zU_s    → deferwrap1 (cleanup de recurso)
er9_yp.pjnG4U      → 1 closure
er9_yp.ky04dT      → 1 closure
[+40 funciones más]
Total: 57 funciones
```

**Diferencias clave vs los otros 5 paquetes:**
1. Tiene `decFunc` → **todos los strings están cifrados** (garble -literals)
2. Tiene `init.0` → inicialización de variables de paquete antes del init()
3. Tiene funciones no-init con lógica propia (h0INzvC179I, nQt7jN_pa)
4. Tipo wrapper `(*G2Ab89kPnVq.ZZuYqyV1)` adyacente = io.Reader wrapper
   para leer datos de detección

**Hipótesis del propósito:**
- Las 18 init closures probablemente registran **firmas adicionales cifradas**
  (evitando que sean visibles en strings analysis del binario)
- Las funciones con closures (h0INzvC179I × 4, nQt7jN_pa × 3) sugieren
  **detección activa con múltiples etapas** (setup → scan → cleanup)
- Los 2 deferwraps indican **manejo de recursos** (handles del SO, memoria)
- Posición en pclntab: entre `vfFlma` (protoregistry) y `G2Ab89kPnVq` (io.Reader)
  → accede al protocolo protobuf para reportar hallazgos

**Vecinos en pclntab:**
```
vfFlma (protoregistry) → qavYW93 (1 func) → [er9_yp] → G2Ab89kPnVq (io.Reader) → CNEPNkZW_ (mime/multipart)
```

---

## Motor de Detección pdFrspK_G.hlavBkMcO — 644 Closures

Este NO es un paquete de "firmas registradas" sino el motor que las ejecuta:

**Estructura:**
```
pdFrspK_G.hlavBkMcO (función principal)
├── func1   → check de detección #1
├── func2   → check de detección #2
├── ...
└── func644 → check de detección #644
```

**Diferencias con los paquetes de firmas:**
- Las firmas de `ra_94HIlnc6` et al. se **registran** en el motor durante `init()`
- `hlavBkMcO.func1...644` son **las verificaciones ejecutadas** cuando el motor corre
- Los 644 closures probablemente corresponden a los 778 patrones registrados
  más verificaciones adicionales de integridad

**El paquete pdFrspK_G:**
- 789 funciones totales
- Solo 5 funciones top-level (no closure): `hlavBkMcO`, `DaZXKAoyfrI`, `i8xZKDcoOE`,
  `PviJufUQhhN`, `decFunc`
- Sin tipos con receivers (0 structs propios)
- Puro procedural: todas las detecciones son funciones directas

---

## Flujo de Detección Completo

```
[Inicio del AC]
    │
    ├── init() de ra_94HIlnc6 → registra 299 firmas en dUgTofmw
    ├── init() de EyjsrRr     → registra 162 firmas en dUgTofmw
    ├── init() de WGfDxX0zz2M → registra 117 firmas en dUgTofmw
    ├── init() de YCJ5PUz_M   → registra 116 firmas en dUgTofmw
    ├── init() de kNpc1A53    → registra 84 firmas en dUgTofmw
    └── init() de er9_yp      → registra 18 firmas cifradas en dUgTofmw
                                              Total: 796 firmas locales
    │
    ├── Conexión a l4d2center.com:443 (gRPC/TLS 1.3)
    │       └── Recibe H1oahxz1l3iY.CheatSigs → firmas adicionales del servidor
    │               (W99qYP: Name + Pattern de bytes)
    │
    ├── main.(*HpP4qwz).Startup() lanza:
    │       ├── dUgTofmw.ga4oovjHCfg() — scanner principal (33 goroutines)
    │       │       └── Aplica las 796 firmas registradas contra la memoria
    │       ├── pdFrspK_G.hlavBkMcO() — 644 checks hardcodeados
    │       ├── bszAWJqu.{FkjkBraZo,JnwG33U,PlRmoX6cuU,X10KDNt6j,X80N6Rs52TTi}
    │       │       └── 5 scanners paralelos (ventanas/procesos/DLLs/archivos/red)
    │       │               Aplican las 4 blacklists (BL1-BL4) vía ncRaYk_Ke (regexp)
    │       └── sOAbtRgFLa6_.UxvaZNFzHI() — scanner de memoria Source Engine
    │               (ConVars, velocidad de disparo, noclip, sv_cheats)
    │
    └── Loop de monitoreo continuo (main.ZrMEw4hJW + ConnectL4D2Server)
            └── Heartbeat + re-evaluación periódica
```

---

## Vectores de Detección por Categoría

### Vector 1: Firmas de Bytes en Memoria (778 firmas + servidor)
- `ra_94HIlnc6` + `EyjsrRr` + `WGfDxX0zz2M` + `YCJ5PUz_M` + `kNpc1A53`
- Escanean memoria del proceso buscando patrones de bytes conocidos de cheats
- Matching vía `dUgTofmw.ga4oovjHCfg` (33 goroutines concurrentes)

### Vector 2: Blacklists de Ventanas/Procesos/DLLs/Archivos/Red
- Paquete `bszAWJqu` (5 scanners)
- ~62 entradas en 4 blacklists (BL1-BL4), ver `listas_negras.md`
- RE Tools: x32dbg, windbg, ghidra, ollydbg, fiddler, x64_dbg, Cheat Engine...
- Cheats específicos: aimware, PhantOm, WPE PRO, harmony, pizza...

### Vector 3: Motor de Detección Hardcodeado
- `pdFrspK_G.hlavBkMcO` — 644 checks ejecutados directamente
- Sin overhead de registro — los checks se ejecutan en secuencia fija

### Vector 4: Scanner de Estado Source Engine
- `sOAbtRgFLa6_.UxvaZNFzHI` — lee memoria del proceso de L4D2
- Detecta: flags ConVar (`i4localSurvivorGunFire` como anchor)
- Detecta: sv_cheats, noclip, godmode, velocidad de disparo anómala

### Vector 5: Fingerprint HWID (Hash xxhash64)
- `asYMlWeBL6f6` → WMI queries → `FXWqsvy_.Z6ey1EJD` (11 colectores)
- Componentes: CPU, Disk, GPU, Motherboard, OS
- Hash final: `BhCuafOD.BDap2x.Sum64()` = xxhash64 de los 5 componentes
- Enviado al servidor en `ENOV9d` (proto struct, campo HWData)

### Vector 6: Detección en er9_yp (Cifrado)
- 18 patrones de detección con strings cifrados (garble -literals)
- Naturaleza desconocida sin análisis dinámico
- Probablemente: cheats específicos o métodos de bypass documentados internamente

### Vector 7: Addons/VPK Scanner
- `main.QSUMsCa` — 4 goroutines de escaneo paralelo
- Busca VPKs con pattern `%s\%sk1e1y*.vpk` (cheats de addon conocidos)
- Aplica 2 regexes especiales (`MDfyjAe.func7`, `RaR0kkqIl.func8`)

---

## Técnicas de Bypass por Vector

### Bypass Vector 1 (Firmas de Bytes)
- Las firmas escanean patrones de bytes específicos — **evitar firma** cambiando
  bytes al compilar el cheat (recompilación con ofuscación de código)
- Las firmas se actualizan desde el servidor → bypass temporal hasta nueva firma

### Bypass Vector 2 (Blacklists)
- **Renombrar procesos**: `fiddler.exe` → `fiddlerx.exe` (fuera del pattern)
- **Renombrar ventanas**: cambiar el título de la ventana de herramientas RE
- Las blacklists usan **substring matching parcial** (no nombre exacto),
  lo que significa que algunas variaciones de nombre pueden evadir

### Bypass Vector 3 (hlavBkMcO)
- 644 checks hardcodeados → requiere análisis individual de cada check
- Sin análisis dinámico no es posible documentar qué verifica cada closure

### Bypass Vector 4 (Source Engine)
- El AC usa `i4localSurvivorGunFire` como anchor para ubicar la memoria de ConVars
- Cambiar el anchor invalidaría la detección (si es una string cifrada)
- Manipular el proceso de escritura de ConVars para evitar que sean leídas

### Bypass Vector 5 (HWID)
- El HWID es xxhash64 de 5 componentes WMI
- **Spoofer de HWID**: falsificar los valores WMI retornados para CPU/Disk/GPU/MB/OS
- Componentes que se leen vía WMI: ver `pipeline_hwid.md` para detalles
- **NOTA**: No implementar spoofer hasta confirmación explícita de LO

### Bypass Vector 6 (er9_yp cifrado)
- Requiere análisis dinámico o debug para descifrar las 18 firmas
- `decFunc` descifra los strings en runtime — debugger puede capturarlos

### Bypass Vector 7 (VPK Scanner)
- El pattern `k1e1y` en VPKs es específico — cheats que no usan este naming
  no son detectados por este scanner

---

## Estadísticas Totales de Detección

| Componente | Checks | Observaciones |
|------------|--------|---------------|
| Firmas estáticas (5 paquetes) | 778 | Registradas en init(), ejecutadas en runtime |
| er9_yp (cifradas) | ~18 | Strings cifrados, naturaleza desconocida |
| hlavBkMcO closures | 644 | Hardcodeados, sin registro previo |
| Blacklist entries | ~62 | 4 listas (BL1-BL4) |
| Source Engine scanner | 1 función | 7 closures internos |
| VPK scanner | 1 patrón glob | + 2 regexes especiales |
| Firmas del servidor | Variable | W99qYP.Pattern, actualizables |
| **TOTAL ESTÁTICO** | **~1502** | Checks sin contar firmas del servidor |
