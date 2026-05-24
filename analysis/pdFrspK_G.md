# Motor Principal de Detección — Paquete `pdFrspK_G`

Documentación del paquete `pdFrspK_G`, el núcleo procedural del anticheat
que contiene la función de detección masiva `hlavBkMcO` con 644+ sub-closures.

---

## Identificación

| Atributo | Valor |
|----------|-------|
| Nombre garbled | `pdFrspK_G` |
| Función | Motor principal de detección — análisis de estado del juego |
| Funciones totales | **789** |
| Funciones top-level | 5 (sin contar closures) |
| Init closures | 0 (no registra firmas, ejecuta directamente) |
| Struct types | 0 (puramente procedimental — ningún receiver) |
| Llamado por | Orquestador AC (vía pclntab, adyacente a `aGxe6x3`/html/template) |

---

## Funciones Top-Level

```
pdFrspK_G
│
├── hlavBkMcO         ← MOTOR DE DETECCIÓN PRINCIPAL (644+ closures)
│   ├── func1         ← Verificación #1
│   ├── func2         ← Verificación #2
│   ├── ...
│   └── func644       ← Verificación #644
│
├── DaZXKAoyfrI       ← Setup / inicialización del motor
├── i8xZKDcoOE        ← Helper interno
├── PviJufUQhhN       ← Helper interno
└── decFunc           ← Descifrador de string literals (garble -literals)
```

---

## La Función `hlavBkMcO` — 644 Verificaciones

`hlavBkMcO` es la función de detección más grande del binario. Sus 644
sub-closures representan 644 verificaciones de estado del juego ejecutadas
en secuencia (o en paralelo según goroutines del orquestador).

### Patrones de Agrupación de Closures

Basado en el análisis de los paquetes de firmas (`ra_94HIlnc6` etc.),
los 644 closures de `hlavBkMcO` probablemente se agrupan por categoría:

```
hlavBkMcO.func1    - func~150   → Verificaciones de memoria (ReadProcessMemory)
hlavBkMcO.func~151 - func~300   → Verificaciones de ConVars (sOAbtRgFLa6_)
hlavBkMcO.func~301 - func~450   → Verificaciones de procesos/ventanas (bszAWJqu)
hlavBkMcO.func~451 - func~550   → Verificaciones de red/paquetes (sNAkh4)
hlavBkMcO.func~551 - func~644   → Verificaciones de archivos/DLLs
```

**Nota:** La agrupación anterior es inferida. Sin análisis dinámico
no es posible determinar el orden exacto de verificaciones.

### Diferencia con los Paquetes de Firmas (ra_94HIlnc6 etc.)

Los paquetes de firmas registran sus patrones en `init()`:
```
ra_94HIlnc6.init.func1  → registra firma "NombreCheat1"
ra_94HIlnc6.init.func2  → registra firma "NombreCheat2"
...
```

`hlavBkMcO` NO usa `init()` — sus closures son EJECUTADAS directamente
durante el ciclo de detección, no registradas en tablas de firmas.
Esto significa que `hlavBkMcO` es un **scanner activo** mientras que
`ra_94HIlnc6` et al. son **bases de datos de patrones**.

---

## Función `DaZXKAoyfrI` — Setup del Motor

`DaZXKAoyfrI` tiene solo 1 aparición en el binario (desde el orquestador).
Su rol probable es:
- Inicializar el handle de `ReadProcessMemory` (obtener handle del juego)
- Obtener el PID de `left4dead2.exe`
- Inicializar buffers de memoria para el scanner
- Configurar los parámetros del ciclo de detección

---

## `decFunc` — Descifrador de String Literals

La presencia de `decFunc` en `pdFrspK_G` confirma que este paquete
tiene **string literals cifrados por garble `-literals`**.

Esto significa que los nombres de variables de ConVar, los nombres de
funciones de cheat conocidas, las rutas de archivos sospechosos, y
otros strings usados en las 644 verificaciones están cifrados en el
binario y solo se descifran en runtime.

El patrón de cifrado de garble es XOR con claves rotativas:
```
string_cifrado XOR clave_aleatoria_por_paquete → string_original
```

Para obtener los strings reales, se requiere análisis dinámico.

---

## Relación con el Sistema Global

```
main.QSUMsCa (goroutines del main)
   │
   ├── goroutine 1: ciclo de detección principal
   │     │
   │     └── pdFrspK_G.DaZXKAoyfrI  ← inicializa scanner
   │           │
   │           └── pdFrspK_G.hlavBkMcO  ← ejecuta 644 checks
   │                 │
   │                 ├── sOAbtRgFLa6_.UxvaZNFzHI  (Source Engine scanner)
   │                 ├── bszAWJqu.FkjkBraZo        (ventanas blacklist)
   │                 ├── bszAWJqu.JnwG33U           (procesos blacklist)
   │                 ├── bszAWJqu.PlRmoX6cuU        (DLLs blacklist)
   │                 ├── ra_94HIlnc6 patterns       (firmas estáticas)
   │                 └── dUgTofmw.(*fcje4l4dl_uV).Feed() ← reporte
   │
   ├── goroutine 2: pipeline HWID
   │     └── FXWqsvy_.Z6ey1EJD → BhCuafOD → dUgTofmw
   │
   ├── goroutine 3: gRPC bidirectional stream
   │     └── xzFpaM3Mq3.iwhm6fWm4v → l4d2center.com:443
   │
   └── goroutine 4: captura de paquetes de red
         └── gG7Vmb7/uaIqvkWry1B → sNAkh4 → dUgTofmw
```

---

## Contexto pclntab

`pdFrspK_G` está ubicado en pclntab entre:
- **Anterior:** `aGxe6x3` (html/template) / Wails UI packages
- **Posterior:** `GVADC0m` (text/template/parse) / más Wails packages

Esta ubicación en el espacio de funciones del binario sugiere que el
compilador ubicó el código del motor AC después del código de la UI,
probablemente en orden de compilación (los paquetes AC propios se
compilan después de las dependencias de terceros).

---

## Bypass de `hlavBkMcO`

Para bypassear el motor de detección principal se requiere:

### Opción 1: Análisis Dinámico Completo (Recomendado)

```python
# Con frida: trazar todas las llamadas a Feed() desde hlavBkMcO
import frida

def on_message(message, data):
    print(message['payload'])

session = frida.attach("l4d2c_anticheat.exe")
script = session.create_script("""
    // Encontrar dirección de dUgTofmw.(*fcje4l4dl_uV).Feed
    var feedAddr = /* dirección desde análisis */ null;
    
    if (feedAddr) {
        Interceptor.attach(feedAddr, {
            onEnter: function(args) {
                send("Feed llamado: " + args[0].readByteArray(64).toString());
            }
        });
    }
""")
script.on('message', on_message)
script.load()
```

### Opción 2: Patch del Loop de Detección

Si se conoce la dirección de `DaZXKAoyfrI`, patchear la función para
que retorne inmediatamente sin ejecutar `hlavBkMcO`.

### Opción 3: Hook de ReadProcessMemory

`hlavBkMcO` usa `ReadProcessMemory` para leer el estado del juego.
Hookear esta función para devolver bytes limpios (sin evidencia de cheats)
es el bypass más genérico pero requiere conocer qué busca exactamente.

---

## Estadísticas

| Métrica | Valor |
|---------|-------|
| Total funciones en paquete | 789 |
| Sub-closures de hlavBkMcO | 644+ |
| Otras funciones top-level | 4 |
| Init closures | 0 |
| String literals cifrados | Sí (`decFunc` presente) |
| Tipos de struct | 0 |
| Paquete más cercano (función clave) | `hlavBkMcO` comparable a `pdFrspK_G.main detection loop` |
