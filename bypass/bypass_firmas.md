# Bypass del Escáner de Firmas en Memoria

El AC descarga patrones de bytes desde el servidor y escanea la memoria del proceso del juego. Esta página explica cómo funciona y cómo hacer código invisible para él.

---

## Cómo Funciona el Escáner

```
1. Cliente conecta → solicita CheatSigs al servidor
2. Servidor envía array de CheatSignature { Name, Pattern } (campo CheatSigs, field 5)
3. AC obtiene handle al proceso left4dead2.exe con OpenProcess
4. AC itera regiones de memoria con VirtualQuery
5. Para cada región MEM_COMMIT y legible:
   ReadProcessMemory → buscar patrón con memmem()
6. Si hay match → rellena W99qYP { Name, Pattern } → envía al servidor → BAN
```

El AC es un proceso externo — usa `ReadProcessMemory` para leer la memoria del juego. No está inyectado en el proceso.

---

## Método 1 — Descifrado Polimórfico en Runtime

Cifrar el payload con XOR + clave única por build. Los bytes en memoria son datos cifrados aleatorios — ningún patrón estático puede matchear.

```cpp
#include <windows.h>
#include <cstdint>
#include <cstring>

// Payload del cheat cifrado con XOR en tiempo de compilación
// La clave cambia en cada build → bytes diferentes cada vez
extern const uint8_t PAYLOAD_CIFRADO[];
extern const size_t  PAYLOAD_TAMANO;
extern const uint8_t CLAVE_XOR;

void* cargar_y_descifrar() {
    // Asignar región ejecutable en dirección aleatoria (ASLR)
    void* mem_exec = VirtualAlloc(
        nullptr,
        PAYLOAD_TAMANO,
        MEM_COMMIT | MEM_RESERVE,
        PAGE_EXECUTE_READWRITE     // temporal: necesita escritura para descifrar
    );
    
    if (!mem_exec) return nullptr;
    
    // Descifrar en-place con XOR rodante
    uint8_t* salida = (uint8_t*)mem_exec;
    for (size_t i = 0; i < PAYLOAD_TAMANO; i++) {
        salida[i] = PAYLOAD_CIFRADO[i] ^ (uint8_t)(CLAVE_XOR + i * 0x13 + (i >> 3));
    }
    
    // Cambiar a solo ejecutable (previene dump trivial)
    DWORD prot_anterior;
    VirtualProtect(mem_exec, PAYLOAD_TAMANO, PAGE_EXECUTE_READ, &prot_anterior);
    
    return mem_exec;
}

// Herramienta de cifrado en tiempo de compilación (compilar por separado):
// Toma el DLL del cheat en texto plano → produce array cifrado con clave aleatoria
void cifrar_payload(const uint8_t* entrada, size_t len, uint8_t clave, uint8_t* salida) {
    for (size_t i = 0; i < len; i++)
        salida[i] = entrada[i] ^ (uint8_t)(clave + i * 0x13 + (i >> 3));
}
```

Por qué funciona: el escáner ve bytes cifrados aleatorios. Ninguna firma estática sobrevive una nueva clave.

---

## Método 2 — Manual Map (Inyección sin Módulo)

La inyección DLL normal registra el módulo en la lista de módulos del PEB. El AC puede enumerarlos. El manual map bypasea esto:

```cpp
// Cargador PE manual — mapea DLL sin LoadLibrary
// No aparece en CreateToolhelp32Snapshot, EnumProcessModules, Module32First

void* ManualMap(const uint8_t* datosDll, size_t tamDll) {
    PIMAGE_DOS_HEADER dosHdr = (PIMAGE_DOS_HEADER)datosDll;
    PIMAGE_NT_HEADERS ntHdr  = (PIMAGE_NT_HEADERS)(datosDll + dosHdr->e_lfanew);
    
    // Asignar en la base preferida (o dejar que ASLR elija)
    uint8_t* base = (uint8_t*)VirtualAlloc(
        (LPVOID)ntHdr->OptionalHeader.ImageBase,
        ntHdr->OptionalHeader.SizeOfImage,
        MEM_COMMIT | MEM_RESERVE,
        PAGE_EXECUTE_READWRITE
    );
    
    if (!base) {
        // Fallback: dejar que el SO elija la dirección
        base = (uint8_t*)VirtualAlloc(nullptr,
            ntHdr->OptionalHeader.SizeOfImage,
            MEM_COMMIT | MEM_RESERVE,
            PAGE_EXECUTE_READWRITE);
    }
    
    // Copiar cabeceras
    memcpy(base, datosDll, ntHdr->OptionalHeader.SizeOfHeaders);
    
    // Copiar secciones
    PIMAGE_SECTION_HEADER seccion = IMAGE_FIRST_SECTION(ntHdr);
    for (int i = 0; i < ntHdr->FileHeader.NumberOfSections; i++, seccion++) {
        if (seccion->SizeOfRawData) {
            memcpy(
                base + seccion->VirtualAddress,
                datosDll + seccion->PointerToRawData,
                seccion->SizeOfRawData
            );
        }
    }
    
    // Procesar imports, relocaciones...
    // (pasos estándar de carga PE)
    
    return base;
}
```

Combinado con Método 1: manual map del DLL cifrado, descifrar in-place → doble bypass.

---

## Método 3 — Code Cave en el Binario del Juego

El binario legítimo del juego (`left4dead2.exe`) ya está en memoria y probablemente NO es escaneado (escanear el propio binario confiable sería costoso y contraproducente).

Buscar espacio no usado (gaps entre funciones):

```python
# Buscar code caves en left4dead2.exe
# Buscar secuencias de 0xCC (padding INT3) o bloques 0x00 de tamaño suficiente

def buscar_caves(datos_pe: bytes, tam_minimo: int = 256) -> list:
    caves = []
    i = 0
    while i < len(datos_pe) - tam_minimo:
        if datos_pe[i] == 0x00:
            j = i
            while j < len(datos_pe) and datos_pe[j] == 0x00:
                j += 1
            if j - i >= tam_minimo:
                caves.append((i, j - i))
            i = j
        else:
            i += 1
    return caves

# Luego: escribir shellcode en el cave → hookear función del juego para llamarlo
```

---

## Método 4 — Asignación desde el Kernel

Asignar memoria desde modo kernel usando un driver. `VirtualQuery` desde userland solo ve asignaciones en modo usuario. Las asignaciones kernel (pool memory mapeada al espacio de usuario via MDL) no son visibles para un escáner de userland.

```c
// Driver kernel: asignar y mapear memoria invisible para VirtualQuery
PVOID KernelAsignarVisible(PEPROCESS proceso, SIZE_T tamano) {
    // Asignar en non-paged pool
    PVOID mem_kernel = ExAllocatePoolWithTag(NonPagedPoolNx, tamano, 'CUST');
    
    // Construir MDL para mapear en el espacio VA del proceso usuario
    PMDL mdl = IoAllocateMdl(mem_kernel, tamano, FALSE, FALSE, NULL);
    MmBuildMdlForNonPagedPool(mdl);
    
    // Mapear con permisos de usuario
    PVOID va_usuario = MmMapLockedPagesSpecifyCache(
        mdl, UserMode, MmCached, NULL, FALSE, NormalPagePriority
    );
    
    return va_usuario;  // VirtualQuery en esto no devuelve info útil
}
```

---

## Qué Regiones Escanea el AC

```
Escaneadas (MEM_COMMIT con):
  PAGE_EXECUTE_READ          código ejecutable principal (objetivo)
  PAGE_EXECUTE_READWRITE     código inyectado / JIT (sospechoso)
  PAGE_EXECUTE_WRITECOPY     copy-on-write ejecutable

Probablemente omitidas:
  PAGE_NOACCESS              excepción de acceso en lectura
  MEM_FREE                   no asignado
  Rangos de DLLs del sistema (ntdll, kernel32, etc.)
  Rango propio del binario del juego
```

Técnica adicional: asignar con `PAGE_GUARD`. El primer acceso lanza una excepción guard. La mayoría de ACs no manejan esto. (No verificado para este AC.)

---

## Categorías de Firmas (Qué se Signa Típicamente)

```
1. Cabeceras PE de cheats conocidos (MZ + secciones específicas)
2. Prólogos de funciones conocidas de cheats
3. Strings del cheat (nombres de menú, opciones de config)
4. Magic bytes de frameworks de cheats (aimware, skeet, etc.)
5. Secuencias características de imports
6. Archivos de config cargados en memoria
```

Si el cheat fue escrito desde cero (no usando bases públicas), no tiene firma conocida. Los métodos 1 y 2 son overkill — simplemente no habrá firma que detectar.

Solo los cheats públicos/filtrados tienen firmas en la base de datos del servidor.
