# Tabla de Funciones Completa — pclntab

El binario Go embebe en la sección `.rdata` una tabla llamada `pclntab` (program counter to line number table) que contiene los nombres de TODAS las funciones del binario en orden de dirección de memoria. Este análisis extrae todos los nombres de funciones AC relevantes de esa tabla.

---

## Paquete main — Funciones Completas

```
main.(*HpP4qwz).Startup
main.(*HpP4qwz).Startup.func1  ... .func27 (con sub-closures .1)
main.(*HpP4qwz).FrontendReady
main.(*HpP4qwz).ConnectL4D2Server
main.(*HpP4qwz).ConnectL4D2Server.func1
main.(*HpP4qwz).ConnectL4D2Server.func2
main.(*HpP4qwz).ConnectL4D2Server.func3
main.(*HpP4qwz).ConnectL4D2Server.func4
main.(*HpP4qwz).StartL4D2
main.(*HpP4qwz).StartL4D2.func1
main.(*HpP4qwz).StartL4D2.func2
main.(*HpP4qwz).StartL4D2.func3
main.(*HpP4qwz).StartL4D2.func4
main.(*HpP4qwz).StartL4D2.func4.1
main.(*HpP4qwz).BeforeClose
main.(*HpP4qwz).BeforeClose.func1
main.(*HpP4qwz).BeforeClose.func2
main.(*HpP4qwz).BeforeClose.func3
main.(*HpP4qwz).BeforeClose.func4
main.(*HpP4qwz).BeforeClose.deferwrap1
main.(*HpP4qwz).BeforeClose.deferwrap2

main.FLqUmcop           — función auxiliar con 1 closure (func1)
main.FLqUmcop.func1

main.(*ip4dz1).s3WIwGIayRx   — tipo ip4dz1, método garblado
main.(*fyBBzjy).exsr9Q9      — tipo fyBBzjy, método garblado
main.(*fyBBzjy).Replace       — tipo fyBBzjy, método Replace (string replacement?)

main.jBzxLnQ2u    — función utilitaria
main.cmzj5vg7g    — función utilitaria

main.QSUMsCa               — FUNCIÓN PRINCIPAL desconocida (muchos goroutines)
main.QSUMsCa.gowrap1
main.QSUMsCa.gowrap2
main.QSUMsCa.gowrap3
main.QSUMsCa.gowrap4
main.QSUMsCa.func1
main.QSUMsCa.func1.1
main.QSUMsCa.func2
main.QSUMsCa.func3
main.QSUMsCa.func4
main.QSUMsCa.func4.1
main.QSUMsCa.func5
main.QSUMsCa.func6

main.c5l5shUHY6XB
main.c5l5shUHY6XB.gowrap1
main.c5l5shUHY6XB.func1
main.c5l5shUHY6XB.func1.deferwrap1

main.main.func1
main.main.func2
main.main.func3
main.main.func3.1
main.main.func4
main.main.func4.1
main.main.func5
main.main.func7
```

### Observaciones

**`main.QSUMsCa`** es la función más notable del listado. Análisis confirmado por búsqueda en pclntab:

Funciones inlined EN su código (aparecen entre `main.QSUMsCa` y `main.QSUMsCa.gowrap4` en la memoria):
```
ncRaYk_Ke.FS0IhhJL                        — regexp function (inlined)
d8a0oX50i.FJYZBfxOBhA[go.shape.int32]    — generic comparison/sort para int32 (inlined)
ncRaYk_Ke.MDfyjAe                         — regexp function (inlined)
ncRaYk_Ke.RaR0kkqIl                       — regexp function (inlined)
i7OyDUpCDA3q.D5UnAj                       — helper function (inlined)
```

Sub-closures de QSUMsCa (adicionales a func1-func6):
```
main.QSUMsCa.MDfyjAe.func7    — closure dentro de QSUMsCa, sub-func 7 de MDfyjAe
main.QSUMsCa.RaR0kkqIl.func8  — closure dentro de QSUMsCa, sub-func 8 de RaR0kkqIl
```

La presencia de múltiples funciones regexp inlined sugiere que `QSUMsCa` realiza **compilación y matching de expresiones regulares** — probablemente construye el regex del blacklist combinado a partir de los strings individuales y lo aplica contra procesos/ventanas. El generic `FJYZBfxOBhA[go.shape.int32]` indica comparación/sorting de int32 (posiblemente PIDs de procesos).

**`main.(*fyBBzjy).Replace`** — el método `Replace` en un tipo garblado sugiere:
- Reemplazo de strings en paths de archivos (para normalizar rutas de VPKs)
- Sanitización de datos antes de enviar al servidor

**`main.FLqUmcop`** — función helper directamente llamada desde el flow de ConnectL4D2Server, probablemente para inicializar la sesión o configurar el estado.

**`bszAWJqu`** — paquete nuevo encontrado: tiene `FkjkBraZo` (15 closures + deferwrap) y `IvOAAX` (inlined en FrontendReady.func1). También usa regexp (`ncRaYk_Ke`). Candidato al escáner de procesos/ventanas.

---

## Paquete dUgTofmw — Lista Completa de Goroutines

La función `ga4oovjHCfg` lanza los goroutines numerados de la siguiente forma:

```
dUgTofmw.ga4oovjHCfg          — dispatcher principal
dUgTofmw.ga4oovjHCfg.func1    — check 1
dUgTofmw.ga4oovjHCfg.func2    — check 2
dUgTofmw.ga4oovjHCfg.func3    — check 3
dUgTofmw.ga4oovjHCfg.func4    — check 4
dUgTofmw.ga4oovjHCfg.func5    — check 5
dUgTofmw.ga4oovjHCfg.func6    — check 6
dUgTofmw.ga4oovjHCfg.func7    — check 7
dUgTofmw.ga4oovjHCfg.func8    — check 8
dUgTofmw.ga4oovjHCfg.func9    — check 9
dUgTofmw.ga4oovjHCfg.func9.1  — sub-goroutine de check 9
dUgTofmw.ga4oovjHCfg.func10   — check 10
dUgTofmw.ga4oovjHCfg.func11   — check 11
dUgTofmw.ga4oovjHCfg.func12   — check 12
dUgTofmw.ga4oovjHCfg.func12.1 — sub-goroutine de check 12
dUgTofmw.ga4oovjHCfg.func13   — check 13
dUgTofmw.ga4oovjHCfg.func14   — check 14
dUgTofmw.ga4oovjHCfg.func15   — check 15
dUgTofmw.ga4oovjHCfg.func16   — check 16
dUgTofmw.ga4oovjHCfg.func17   — check 17
dUgTofmw.ga4oovjHCfg.func18   — check 18
dUgTofmw.ga4oovjHCfg.func19   — check 19
dUgTofmw.ga4oovjHCfg.func20   — check 20
dUgTofmw.ga4oovjHCfg.func21   — check 21
dUgTofmw.ga4oovjHCfg.func21.1 — sub-goroutine de check 21
dUgTofmw.ga4oovjHCfg.func22   — check 22
dUgTofmw.ga4oovjHCfg.func23   — check 23
dUgTofmw.ga4oovjHCfg.func24   — check 24
dUgTofmw.ga4oovjHCfg.func25   — check 25
dUgTofmw.ga4oovjHCfg.func26   — check 26
dUgTofmw.ga4oovjHCfg.func27   — check 27
dUgTofmw.ga4oovjHCfg.func28   — check 28
dUgTofmw.ga4oovjHCfg.func28.1 — sub-goroutine de check 28
dUgTofmw.ga4oovjHCfg.func29   — check 29
dUgTofmw.ga4oovjHCfg.func30   — check 30
dUgTofmw.ga4oovjHCfg.func31   — check 31
dUgTofmw.ga4oovjHCfg.func32   — check 32
dUgTofmw.ga4oovjHCfg.func33   — check 33
```

**Total goroutines ga4oovjHCfg:** 33 goroutines principales + 4 sub-goroutines = 37 closures totales

Los checks 9, 12, 21, 28 tienen sub-goroutines anidadas (`.1`) — esto indica que estos checks son bidireccionales: el check principal detecta algo y la sub-goroutine maneja la respuesta/reporte.

### Otras Funciones Importantes de dUgTofmw

```
dUgTofmw.CGRKkahhRkP              — handler de reportes/resultados (12 sub-funciones)
dUgTofmw.CGRKkahhRkP.deferwrap1
dUgTofmw.CGRKkahhRkP.func1 ... .func12
dUgTofmw.CGRKkahhRkP.func11.1
dUgTofmw.CGRKkahhRkP.func11.2
dUgTofmw.CGRKkahhRkP.func11.2.1

dUgTofmw.MOKJ2g5p                 — análisis profundo (cuádruple anidamiento)
dUgTofmw.MOKJ2g5p.func1
dUgTofmw.MOKJ2g5p.func2
dUgTofmw.MOKJ2g5p.func3
dUgTofmw.MOKJ2g5p.func3.1
dUgTofmw.MOKJ2g5p.func3.1.1
dUgTofmw.MOKJ2g5p.func3.1.1.deferwrap1
dUgTofmw.MOKJ2g5p.func3.1.2
dUgTofmw.MOKJ2g5p.func3.1.2.1
dUgTofmw.MOKJ2g5p.func3.2
dUgTofmw.MOKJ2g5p.func3.3
dUgTofmw.MOKJ2g5p.func4
dUgTofmw.MOKJ2g5p.deferwrap1

dUgTofmw.(*fcje4l4dl_uV).Feed    — tipo detector con método Feed (alimenta datos al engine)
dUgTofmw.(*fcje4l4dl_uV).zUePZMa — método auxiliar del tipo detector

dUgTofmw.VA0jJhHuwb0l             — sub-sistema de detección (3 goroutines)
dUgTofmw.VA0jJhHuwb0l.func1
dUgTofmw.VA0jJhHuwb0l.func2
dUgTofmw.VA0jJhHuwb0l.func3
dUgTofmw.VA0jJhHuwb0l.func3.1
```

**`(*fcje4l4dl_uV).Feed`** es especialmente interesante: un tipo con un método `Feed` sugiere un patrón productor-consumidor donde datos de monitoreo se "alimentan" al motor de detección.

---

## Paquete HWARRxN — Estructura Completa

El paquete `HWARRxN` es el paquete principal de comunicación del AC. Contiene tanto el código de construcción del HWID como los descriptores protobuf del servicio gRPC.

```
HWARRxN.init
HWARRxN.init.func1

# Constructor de HWID
HWARRxN.KbBU6bdOa.Build        — construir HWID completo (llama 5 queries WMI)
HWARRxN.(*KbBU6bdOa)._h76SfBr  — helper privado del builder

# FileDescriptor — descriptor del archivo .proto
HWARRxN.(*SLltHXdFZqty).ParentFile
HWARRxN.(*SLltHXdFZqty).Parent
HWARRxN.(*SLltHXdFZqty).Index
HWARRxN.(*SLltHXdFZqty).Syntax
HWARRxN.(*SLltHXdFZqty).Edition
HWARRxN.(*SLltHXdFZqty).Name
HWARRxN.(*SLltHXdFZqty).FullName
HWARRxN.(*SLltHXdFZqty).IsPlaceholder
HWARRxN.(*SLltHXdFZqty).Options
HWARRxN.(*SLltHXdFZqty).Path
HWARRxN.(*SLltHXdFZqty).Package
HWARRxN.(*SLltHXdFZqty).Imports
HWARRxN.(*SLltHXdFZqty).Enums
HWARRxN.(*SLltHXdFZqty).Messages
HWARRxN.(*SLltHXdFZqty).Extensions
HWARRxN.(*SLltHXdFZqty).Services    ← confirma que HWARRxN define el servicio gRPC
HWARRxN.(*SLltHXdFZqty).SourceLocations
HWARRxN.(*SLltHXdFZqty).Format
HWARRxN.(*SLltHXdFZqty).ProtoType
HWARRxN.(*SLltHXdFZqty).ProtoInternal
HWARRxN.(*SLltHXdFZqty).GoPackagePath

# EnumDescriptor
HWARRxN.(*UzoZvHzYa8).Options
HWARRxN.(*UzoZvHzYa8).Values
HWARRxN.(*UzoZvHzYa8).ReservedNames
HWARRxN.(*UzoZvHzYa8).ReservedRanges
HWARRxN.(*UzoZvHzYa8).IsClosed

# MessageDescriptor
HWARRxN.(*Cv4Csm).Fields
HWARRxN.(*Cv4Csm).Oneofs
HWARRxN.(*Cv4Csm).ReservedNames
HWARRxN.(*Cv4Csm).ReservedRanges
HWARRxN.(*Cv4Csm).RequiredNumbers
HWARRxN.(*Cv4Csm).ExtensionRanges
HWARRxN.(*Cv4Csm).Enums
HWARRxN.(*Cv4Csm).Messages
HWARRxN.(*Cv4Csm).Extensions

# FieldDescriptor
HWARRxN.(*TvQBJqp99aCj).Number
HWARRxN.(*TvQBJqp99aCj).Cardinality
HWARRxN.(*TvQBJqp99aCj).Kind
HWARRxN.(*TvQBJqp99aCj).HasJSONName
HWARRxN.(*TvQBJqp99aCj).JSONName
HWARRxN.(*TvQBJqp99aCj).TextName
HWARRxN.(*TvQBJqp99aCj).HasPresence
HWARRxN.(*TvQBJqp99aCj).IsPacked
HWARRxN.(*TvQBJqp99aCj).IsExtension
HWARRxN.(*TvQBJqp99aCj).IsWeak
HWARRxN.(*TvQBJqp99aCj).IsLazy
HWARRxN.(*TvQBJqp99aCj).IsList
HWARRxN.(*TvQBJqp99aCj).IsMap
HWARRxN.(*TvQBJqp99aCj).Message
HWARRxN.(*TvQBJqp99aCj).MapKey
HWARRxN.(*TvQBJqp99aCj).MapValue
HWARRxN.(*TvQBJqp99aCj).HasDefault
HWARRxN.(*TvQBJqp99aCj).Default
HWARRxN.(*TvQBJqp99aCj).DefaultEnumValue
HWARRxN.(*TvQBJqp99aCj).ContainingOneof
HWARRxN.(*TvQBJqp99aCj).ContainingMessage
HWARRxN.(*TvQBJqp99aCj).Enum

# MethodDescriptor (gRPC)
# WPaeyWXKC3et — confirmado como protoreflect.MethodDescriptor

# OneofDescriptor
HWARRxN.(*IJUEa6ak).IsSynthetic
HWARRxN.(*IJUEa6ak).Fields
```

La presencia de `.Services` en el FileDescriptor `SLltHXdFZqty` confirma que el `.proto` compilado en este paquete define al menos un servicio gRPC.

---

## Paquete _6di6zc0se2v — WatchList Tipos

```
_6di6zc0se2v.(*_o3YyW).WatchList   — función principal de vigilancia
_6di6zc0se2v.(*_o3YyW).Add
_6di6zc0se2v.(*_o3YyW).AddWith
_6di6zc0se2v.(*_o3YyW).Remove
_6di6zc0se2v.(*_o3YyW).Close
_6di6zc0se2v.(*_o3YyW).Has
_6di6zc0se2v.(*GrRzvM2).Add      ← segundo tipo del WatchList
_6di6zc0se2v.(*GrRzvM2).Close
_6di6zc0se2v.(*Rs0QF1nUdhgw).Has
_6di6zc0se2v.(*UWzc59).Has
```

El paquete `_6di6zc0se2v` tiene al menos 4 tipos:
- `_o3YyW` — interfaz o tipo principal de WatchList
- `GrRzvM2` — implementación interna del watcher
- `Rs0QF1nUdhgw` — tipo con método Has (verificación)
- `UWzc59` — segundo tipo con método Has

---

## Paquete ncRaYk_Ke — Desconocido

```
ncRaYk_Ke.FS0IhhJL    — función 1
ncRaYk_Ke.MDfyjAe     — función 2
ncRaYk_Ke.RaR0kkqIl   — función 3
```

Aparece en el contexto de `ConnectL4D2Server`. Podría ser:
- Biblioteca de hashing/cifrado para el token de auth
- Utilidades de verificación de servidor
- Wrapper de syscalls de Windows

---

## Tabla de Funciones por Categoría

| Categoría | Función Principal | Goroutines |
|-----------|-----------------|-----------|
| Inicialización | `Startup` | 27+ |
| Conexión + Auth | `ConnectL4D2Server` | 4 |
| Detección principal | `dUgTofmw.ga4oovjHCfg` | 37 |
| Reporte/resultados | `dUgTofmw.CGRKkahhRkP` | 12+ |
| Análisis profundo | `dUgTofmw.MOKJ2g5p` | 4 niveles |
| Launcher juego | `StartL4D2` | 5 |
| Sub-detección | `dUgTofmw.VA0jJhHuwb0l` | 3 |
| Función desconocida | `main.QSUMsCa` | 4 gowraps + 6 |
| Cleanup | `BeforeClose` | 4 |
| UI ready | `FrontendReady` | N/A |
