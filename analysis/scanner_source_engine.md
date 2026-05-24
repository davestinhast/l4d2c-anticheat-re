# Scanner de Estado Source Engine — Paquete sOAbtRgFLa6_

Documentación completa del paquete `sOAbtRgFLa6_` — el scanner de estado interno
del juego Left 4 Dead 2, que usa el motor Source Engine como vector de detección.

---

## Identificación

| Atributo | Valor |
|----------|-------|
| Nombre garbled | `sOAbtRgFLa6_` |
| Función | Scanner de estado del juego / ConVar Source Engine |
| Total funciones | 20 |
| Llamado por | `dUgTofmw.KT300841` — evaluador central |
| Usa | `f3SkGnYxZwK` (time stdlib), `ncRaYk_Ke` (regexp) |
| Vector | **Vector 6** del anticheat |

---

## Todas las Funciones

```
sOAbtRgFLa6_
│
├── init                         — inicialización del paquete
│
├── GXErxuFMY9Z                  — función de setup/inicialización del scanner
│   ├── func1                    — sub-etapa
│   │   └── func1.1              — sub-closure
│   ├── func2                    — llama f3SkGnYxZwK.NOZFARjOV (time.Now)
│   ├── func3                    — sub-etapa
│   └── func4                    — finalización setup
│
├── UxvaZNFzHI                   — función principal de detección (feeds dUgTofmw)
│   ├── func1                    — etapa de scan 1
│   ├── func2                    — etapa de scan 2
│   │   └── func2.1              — sub-closure de scan 2
│   ├── func3                    — etapa de scan 3
│   │   └── func3.1              — sub-closure de scan 3
│   ├── func4                    — etapa de scan 4
│   │   └── func4.1              — sub-closure de scan 4
│   ├── func5                    — etapa de scan 5
│   ├── func6                    — etapa de scan 6
│   └── func7                    — reporte de detección
│       └── func7.1              — sub-closure del reporte
│
├── BY2TsEzeKap                  — punto de entrada llamado desde dUgTofmw.KT300841
│
└── H1gN3xqm                     — función helper interna
```

---

## Relación con el Evaluador Central

```
dUgTofmw.KT300841
   │
   ├── [llama directamente]
   ▼
sOAbtRgFLa6_.BY2TsEzeKap     ← punto de entrada externo
   │
   ▼
sOAbtRgFLa6_.GXErxuFMY9Z    ← setup: inicializa scanner, llama time.Now()
   │
   ▼
sOAbtRgFLa6_.UxvaZNFzHI     ← scanner principal: 7 etapas de verificación
   │
   ▼
dUgTofmw.(*fcje4l4dl_uV).Feed(evento)   ← reporta detección al evaluador
```

---

## Mecanismo de Detección

### Anchor de Localización

El AC usa el string `i4localSurvivorGunFire` (encontrado en binario offset 29623296)
como **anchor para localizar la tabla ConVar del Source Engine en memoria**.

El proceso aproximado:
```
1. ReadProcessMemory(left4dead2.exe) — escanea espacio de memoria del juego
2. Busca string "i4localSurvivorGunFire" en memoria (ConVar conocida del juego)
3. Desde ese anchor, navega la estructura interna de ConVarSystems del Source Engine
4. Lee valores de ConVars específicas via offsets calculados
5. Compara valores contra umbrales esperados
6. Si valor ≠ esperado → genera evento de detección
```

### Qué Detecta

| Indicador | Valor Normal | Valor Sospechoso | Tipo de Cheat |
|-----------|-------------|------------------|---------------|
| `sv_cheats` | `0` | `1` | Consola de cheats habilitada |
| Salud de survivor | 0-100 | >100 o <0 | Godmode / modificación directa |
| Velocidad de movimiento | normal Source | anormal | Speed hack |
| Rate de disparo | limitado por arma | sin límite | No-recoil / aimbot |
| Flags de noclip | `0` | `1` | Noclip activado |
| Flags de godmode | `0` | `1` | Invulnerabilidad |

### Uso de Regexp

La presencia de `ncRaYk_Ke` (regexp) en este paquete sugiere que los patrones
de comparación se compilan como expresiones regulares, posiblemente para:
- Validar formato de strings de ConVar
- Matching de nombres de ConVar contra whitelist/blacklist
- Detección de ConVars modificadas vs lista esperada

---

## Paquetes Adyacentes en Pclntab

```
... (packages anteriores) ...
f3SkGnYxZwK.*      — stdlib time (llamado desde GXErxuFMY9Z)
hwvnZR_pbO.*       — gin-gonic/gin
Hx5_5u.*           — paquete pequeño (CHF_KROE, Hx6HQk9q6jn)
─────────────────────────────────────────
sOAbtRgFLa6_.*     ← este paquete
─────────────────────────────────────────
uaIqvkWry1B.*      — github.com/google/gopacket (core)
```

La adyacencia con `uaIqvkWry1B` (gopacket) sugiere que el scanner de Source Engine
puede complementarse con captura de paquetes para correlacionar estado del juego
con tráfico de red (ej: detectar discrepancia entre lo que el cliente reporta y
lo que se ve en el tráfico).

---

## Bypass

| Vector | Bypass |
|--------|--------|
| Lectura de ConVars via ReadProcessMemory | Hook `ReadProcessMemory` para devolver valores limpios |
| Localización via anchor string | Patch el string `i4localSurvivorGunFire` en el proceso o en el scan del AC |
| Comparación de valores | Asegurarse que todas las ConVars cheatable estén en valores base antes de conectar |

**Dificultad:** Media. El anchor string puede moverse o actualizarse. Los offsets
desde el anchor dependen de la versión específica del engine.
El AC calcula offsets en runtime — sin análisis dinámico no se conocen exactamente.

---

## Referencias Cruzadas

```
sOAbtRgFLa6_ ← dUgTofmw.KT300841  (el evaluador central lo invoca)
sOAbtRgFLa6_ → f3SkGnYxZwK         (time — para timing del loop)
sOAbtRgFLa6_ → ncRaYk_Ke           (regexp — para matching)
sOAbtRgFLa6_ → dUgTofmw.Feed()     (reportar detecciones al pipeline)
```
