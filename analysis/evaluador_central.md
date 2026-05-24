# Evaluador Central — Paquete dUgTofmw

Documentación del paquete `dUgTofmw` — el corazón del sistema de detección.
Con 308 funciones, es el paquete más grande del anticheat. Recibe eventos de todas
las goroutines de detección y los evalúa contra reglas para generar veredictos de ban.

---

## Identificación

| Atributo | Valor |
|----------|-------|
| Nombre garbled | `dUgTofmw` |
| Función principal | Motor de evaluación central (Central Evaluator) |
| Total funciones pclntab | 308 |
| Init functions | 13 (init.func1 – init.func12.1) |
| Tipos principales | `(*ZhEEW_uzlNn)`, `(*fcje4l4dl_uV)`, `(*gZDPQlzEDoPS)` |

---

## Tipos Principales

### `(*fcje4l4dl_uV)` — Ingesta de Eventos (Feed)

```
(*fcje4l4dl_uV).Feed    — recibir evento de detección de cualquier goroutine
(*fcje4l4dl_uV).zUePZMa — procesado interno / encolado del evento
```

Este tipo implementa el **productor-consumidor** del sistema:
- Los goroutines de WatchList (`_6di6zc0se2v`), bszAWJqu, y QSUMsCa llaman `Feed()`
- `Feed` encola el evento para evaluación asíncrona
- `zUePZMa` procesa la cola

```
Flujo de entrada:
  WatchList.qFouZjKyDKnV() ──→ fcje4l4dl_uV.Feed(deteccion)
  bszAWJqu.FkjkBraZo       ──→ fcje4l4dl_uV.Feed(deteccion)
  QSUMsCa.MDfyjAe           ──→ fcje4l4dl_uV.Feed(deteccion)
  sOAbtRgFLa6_.UxvaZNFzHI  ──→ fcje4l4dl_uV.Feed(deteccion)
```

---

### `(*ZhEEW_uzlNn)` — Motor de Evaluación

```
(*ZhEEW_uzlNn).gilRBz9eXb   — getter / estado del evaluador
(*ZhEEW_uzlNn).h7MNIMO       — LA función central de evaluación
  ├── func1                   — evaluación fase 1
  ├── func1.1                 — sub-evaluación (closure anidado)
  ├── func2                   — evaluación fase 2
  ├── func2.1                 — sub-evaluación
  ├── func3                   — evaluación fase 3
  ├── func4                   — evaluación fase 4
  ├── func5                   — evaluación fase 5
  ├── func6                   — evaluación fase 6
  ├── func7                   — evaluación fase 7
  └── func8                   — evaluación fase 8
      └── func8.1             — sub-evaluación fase 8
```

`h7MNIMO` con 8+ closures es el pipeline de evaluación multi-etapa.
Cada closure probablemente aplica un conjunto de reglas diferente:
1. Verificar si el evento es un falso positivo
2. Calcular score de confianza
3. Verificar contexto (en juego, en menú, etc.)
4. Consultar historial del jugador
5. Tomar decisión de ban / report / ignore
6. Emitir respuesta al servidor vía gRPC

---

### `(*gZDPQlzEDoPS)` — Error del Evaluador

```
(*gZDPQlzEDoPS).Error
gZDPQlzEDoPS.Error
```

Tipo de error específico del evaluador central.

---

## Funciones Principales

### `ga4oovjHCfg` — Función Crítica (33 closures)

La función más compleja del paquete — 33 sub-closures.

```
ga4oovjHCfg           — función principal
  ├── func1 – func11  — pipeline de análisis
  ├── func12          — decisión principal
  │   └── func12.1    — sub-decisión
  ├── func13 – func20 — procesado adicional
  ├── func21          — reporte
  │   └── func21.1    — sub-reporte
  └── func22 – func33 — post-procesado y cleanup
```

Probablemente es el **analizador de firmas en memoria** (CheatSigs) — el más costoso de todos los checks.

---

### `ZXugk2GxaotX` — Procesador de Eventos (14 closures)

```
ZXugk2GxaotX          — función principal
  ├── func1 – func8    — primera pasada
  │   └── func8.1      — sub-análisis
  └── func9 – func14   — segunda pasada
```

---

### `KT300841` — Handler con gowrap (17 closures)

```
KT300841              — función principal
  ├── func1 – func17  — procesado
  └── gowrap1         — goroutine wrapper (lanza goroutine)
```

Lanza goroutines para detección asíncrona.

---

### `CGRKkahhRkP` — Handler de Protocolo (12 closures + deferwrap)

```
CGRKkahhRkP           — función principal (protocolo gRPC)
  ├── deferwrap1       — cleanup en panic/defer
  ├── func1 – func9    — procesado de mensajes
  └── func10 – func12  — mensajes finales
      ├── func11.1     — sub-procesado
      └── func11.2     — alternativa
          └── func11.2.1 — sub-alternativa
```

Handler del protocolo gRPC para enviar reportes al servidor.

---

### `JcCuAB7zH` — Handler con gowrap (9 closures)

```
JcCuAB7zH             — función principal
  ├── func1 – func9   — stages de detección
  │   └── func8.1     — sub-stage
  └── gowrap1         — goroutine wrapper
```

---

### `MOKJ2g5p` — Función Anidada Compleja (closures de 3+ niveles)

```
MOKJ2g5p
  ├── deferwrap1
  ├── func1 – func2
  └── func3
      ├── func3.1
      │   ├── func3.1.1
      │   │   └── deferwrap1 (dentro de func3.1.1)
      │   └── func3.1.2
      │       └── func3.1.2.1
      ├── func3.2
      ├── func3.3
      └── func4
```

Lógica de evaluación con múltiples ramas anidadas — probablemente maneja casos complejos de detección donde múltiples vectores se combinan.

---

### `RB_nRb` — Analizador Multi-rama (6 closures con sub-niveles)

```
RB_nRb
  ├── func1
  │   ├── func1.1
  │   └── func1.2
  │       └── func1.2.1
  ├── func2 + func2.1
  ├── func3
  │   ├── func3.1
  │   ├── func3.2
  │   │   └── func3.2.1
  │   ├── func3.3
  │   └── func3.4
  ├── func4
  ├── func5
  └── func6
```

---

### `dfzoI5X7aj` — Función con Cleanup (5 closures)

```
dfzoI5X7aj
  ├── deferwrap1  — cleanup / report final
  └── func1 – func5
```

---

### Funciones de Reporte/Emisión

```
C7rmMxBQ           — emitir reporte al servidor (func1)
cafEKmBsI7W        — función de callback
CaVnmKHua          — función auxiliar de caché
Dll9_E_he          — función de decisión (dll = detect/log/log?)
DMrazc6            — función de estado
dWiyVdNx2Fv        — función de evaluación
EEnO9BVP           — función con deferwrap1 (cleanup en error)
egmY80_            — función auxiliar
EM4NEpmhW          — función con 4 closures
eOZqci             — función con func1
ETdFm1LN           — función de estado
etEBMu             — función auxiliar
Fa5wQ7bfv          — función con 4 closures anidados
  └── func1.1, func1.2, func1.3, func1.3.1
FmOyTPW            — función con func1
```

---

### Subsistemas Internos

```
bTOSrRwRD9, bXwPuh7bgI0          — colas de eventos internos
GaGfiYt, GnAAjxzf                — analizadores de patrones
  GnAAjxzf.bXwPuh7bgI0.func3     — sub-closure compartido
  Ku3NIva.bXwPuh7bgI0.func5      — otro sub-closure
g8Qnj_3_S                        — función de estado global
HWH9TFuU (5 closures + deferwrap1) — handler con cleanup
ix23fWF1BKm                       — función interna
JBCw5jG (+ deferwrap1)            — función con cleanup
JclT3ZyNnu (+ deferwrap1 + 4 closures) — función de transición
Jm9lCZuTayH, jslJ644O, JsO_uXo  — helpers internos
JYekK0C                           — función de estado
Ku3NIva (5+ closures)            — función de análisis
  Ku3NIva.func1, func1.1, func2, func3, func3.1, func4, func4.1
lgKnH05Wp                        — helper
Lxx8Q7N (+ deferwrap1)           — función con cleanup
LzxyCO (2 closures)              — función de transición
MOKJ2g5p                         — función anidada compleja (ver arriba)
MUbxHiHbA                        — función auxiliar
NVUoLGCRivS (4 closures)         — función de notificación
P6t_tYOk5O8 (4 closures + deferwrap1) — handler con cleanup
pV5JbQn9WF (3 closures)          — función de pipeline
pZV38F2F5p6H                     — función de configuración
Q100bRnHXe                        — función de query
RB_nRb                           — analizador multi-rama (ver arriba)
rpfbMOh (4 closures)             — función de reporte
t5rfyIM (3 closures)             — función de timer/ticker
t6BMqE0                           — función de timeout
T7nZWPA_QXN                       — función de transición
TmDL6w (2 closures)              — función de cleanup
tN6ha6DAaubp                     — función interna
UbS8cVmNACG2                     — función de caché
VA0jJhHuwb0l (4 closures)        — función de validación
wLSYYZ6n                         — función auxiliar
wmP3Hc                           — función interna
WSRrqZ_Tt[go.shape.int64]        — función genérica (Go generics)
wUPsw0yucxFy                     — función de write/update
XKhwdFnJ                         — función de estado
Xl9KPzkO1h                       — función de lookup
xT_i_Uv                          — función interna
yjl0SSj (3 closures)             — función de análisis
YjouPlOpm (2 closures)           — función de reporte
Z2RUaMq                           — función de estado
ZXugk2GxaotX (14 closures)      — procesador de eventos
```

---

## Init Functions (13)

```
init
init.func1   – registrar handler de evento tipo 1 (firma de cheat)
init.func2   – registrar handler de evento tipo 2 (proceso blacklist)
init.func3   – registrar handler de evento tipo 3 (ventana blacklist)
init.func4   – registrar handler de evento tipo 4 (módulo DLL)
init.func5   – registrar handler de evento tipo 5 (VPK hash)
init.func6   – registrar handler de evento tipo 6 (debugger)
init.func7   – registrar handler de evento tipo 7 (memoria anómala)
init.func9   – registrar handler adicional
init.func10  – registrar handler adicional
init.func11  – registrar handler adicional
init.func12  – registrar handler final
  └── init.func12.1 – sub-handler
```

Cada `init.funcN` registra un handler para un tipo de evento diferente.
Los eventos fluyen de las goroutines de detección al evaluador central.

---

## Flujo Completo del Pipeline de Detección

```
Goroutines de Detección (fuentes)
│
├── _6di6zc0se2v.(*_o3YyW).qFouZjKyDKnV  (WatchList)
├── bszAWJqu.FkjkBraZo        (scanner de ventanas)
├── bszAWJqu.JnwG33U          (scanner de procesos)
├── bszAWJqu.PlRmoX6cuU       (scanner de módulos DLL)
├── bszAWJqu.X10KDNt6j        (scanner de archivos)
├── bszAWJqu.X80N6Rs52TTi     (scanner de red)
├── sOAbtRgFLa6_.UxvaZNFzHI  (scanner de memoria / Source Engine)
└── QSUMsCa.MDfyjAe.func7     (scanner VPK)
       │
       ▼
dUgTofmw.(*fcje4l4dl_uV).Feed(evento)
       │
       ▼
dUgTofmw.(*fcje4l4dl_uV).zUePZMa  (encolado)
       │
       ▼
dUgTofmw.(*ZhEEW_uzlNn).h7MNIMO  (evaluación multi-etapa)
       │
   ┌───┴──────────────────────┐
   │  Score bajo → ignorar    │
   │  Score medio → log       │
   │  Score alto → report ban  │
   └──────────────────────────┘
       │
       ▼
dUgTofmw.CGRKkahhRkP (envío gRPC al servidor)
       │
       ▼
Servidor l4d2center.com → respuesta Qi1Z8I.Banned
```

---

## Generics en el Evaluador

```
WSRrqZ_Tt[go.shape.int64]
```

El evaluador usa Go generics para operaciones sobre tipos numéricos (scores, contadores).

---

## Referencias Cruzadas

```
dUgTofmw → xzFpaM3Mq3     (gRPC — para enviar reportes)
dUgTofmw → P4mAKk         (protobuf — para serializar reportes)
dUgTofmw ← _6di6zc0se2v   (WatchList — fuente de eventos)
dUgTofmw ← bszAWJqu       (blacklist scanner — fuente de eventos)
dUgTofmw ← sOAbtRgFLa6_  (memory scanner — fuente de eventos)
dUgTofmw ← QSUMsCa        (VPK scanner — fuente de eventos)
```
