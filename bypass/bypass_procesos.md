# Bypass — Ocultación de Procesos y Módulos DLL

El AC monitorea dos cosas: la lista de procesos del sistema (para detectar herramientas RE) y los módulos DLL cargados en el juego (para detectar inyecciones). Esta página documenta ambos vectores y sus bypasses.

---

## Vector A — Lista de Procesos del Sistema

### Cómo detecta el AC los procesos

El AC enumera procesos del sistema usando WMI (`Win32_Process`) o `CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0)`. Compara cada nombre de proceso contra su blacklist.

El módulo `_6di6zc0se2v` (WatchList) gestiona esta vigilancia de forma continua.

### Bypass Simple — Renombrar el Ejecutable

El chequeo es por nombre del ejecutable. Renombrar el binario evita la detección:

```
fiddler.exe    → datos_editor.exe
cmd.exe        → (bloqueado — usar PowerShell que no está en la lista)
x32dbg.exe     → debugger.exe
discord.exe    → (usar versión web en vez del cliente)
```

PowerShell (`powershell.exe`) no está en la blacklist — funciona como reemplazo de `cmd.exe`.

### Bypass Avanzado — DKOM (Direct Kernel Object Manipulation)

Eliminar el proceso de la lista enlazada de procesos del kernel. El proceso existe pero no aparece en ninguna enumeración de userland:

```c
// Driver kernel — ocultar proceso de la EPROCESS list
VOID OcultarProceso(ULONG pid) {
    PEPROCESS proceso = NULL;
    PsLookupProcessByProcessId((HANDLE)(ULONG_PTR)pid, &proceso);
    
    if (!proceso) return;
    
    // Offset de ActiveProcessLinks en EPROCESS (varía por versión de Windows)
    // Win10 20H2+: offset 0x448
    PLIST_ENTRY lista = (PLIST_ENTRY)((ULONG_PTR)proceso + 0x448);
    
    // Desconectar el proceso de la lista doblemente enlazada
    lista->Blink->Flink = lista->Flink;
    lista->Flink->Blink = lista->Blink;
    
    // Apuntarse a sí mismo para no crashear si alguien lo referencia
    lista->Flink = lista;
    lista->Blink = lista;
    
    ObDereferenceObject(proceso);
}
```

Este método hace el proceso completamente invisible para `CreateToolhelp32Snapshot`, WMI, `EnumProcesses`, Task Manager, y el AC.

---

## Vector B — Módulos DLL en el Proceso del Juego

### Cómo detecta el AC los DLLs inyectados

```
main.(*HpP4qwz).StartL4D2.func3  — goroutine de vigilancia de inyección
```

Usa `CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid_del_juego)` con `Module32First`/`Module32Next` para enumerar todos los módulos DLL cargados en el proceso del juego. Compara contra una lista de módulos esperados.

### Bypass — Manual Mapping

El manual map no registra el módulo en el PEB (`LDR_DATA_TABLE_ENTRY`). No aparece en:
- `CreateToolhelp32Snapshot`
- `EnumProcessModules`
- `Module32First` / `Module32Next`
- `LdrEnumerateLoadedModules`

```cpp
// Implementación completa en bypass_firmas.md — función ManualMap()
// Después de ManualMap:
// 1. El DLL está en memoria pero no en ninguna lista de módulos
// 2. El escáner del AC no puede verlo por enumeración
// 3. Solo es visible por escaneo directo de memoria (VirtualQuery + ReadProcessMemory)
//    → combinarlo con cifrado polimórfico del método 1
```

### Bypass — Ocultar Módulo del PEB

Si ya se usó LoadLibrary y el módulo está registrado, se puede remover manualmente de las listas del PEB:

```cpp
#include <windows.h>
#include <winternl.h>

void OcultarModuloDePEB(HMODULE hMod) {
    // Obtener PEB del proceso actual
    PPEB peb = (PPEB)__readgsqword(0x60);
    PPEB_LDR_DATA ldr = peb->Ldr;
    
    // Iterar InLoadOrderModuleList
    PLIST_ENTRY entrada = ldr->InLoadOrderModuleList.Flink;
    while (entrada != &ldr->InLoadOrderModuleList) {
        PLDR_DATA_TABLE_ENTRY modulo = CONTAINING_RECORD(
            entrada, LDR_DATA_TABLE_ENTRY, InLoadOrderLinks);
        
        if (modulo->DllBase == (PVOID)hMod) {
            // Remover de las tres listas del LDR
            RemoveEntryList(&modulo->InLoadOrderLinks);
            RemoveEntryList(&modulo->InMemoryOrderLinks);
            RemoveEntryList(&modulo->InInitializationOrderLinks);
            
            // Limpiar el HashLinks si existe
            RemoveEntryList(&modulo->HashLinks);
            
            return;
        }
        
        entrada = entrada->Flink;
    }
}
```

---

## Vector C — Títulos de Ventana (EnumWindows)

### Cómo funciona la detección

El goroutine 3 de detección llama `EnumWindows` + `GetWindowTextW` para cada ventana del sistema. El título se chequea contra los tokens de la blacklist de ventanas.

### Bypass

Para herramientas de debugging y RE:

**x32dbg / x64dbg:** En el menú Options → Preferences → GUI, se puede cambiar el nombre de la ventana. Si no hay opción, hookear `SetWindowTextW` para cambiar el título.

**Ghidra:** Cambiar el nombre de la ventana en el código fuente no es trivial, pero Ghidra puede renombrarse en el lanzador o mediante agentes Java.

**Fiddler:** Renombrar el ejecutable es suficiente — el título de Fiddler incluye el nombre del proceso.

Para cualquier herramienta:

```cpp
// Hook de GetWindowTextW para falsificar el título de ventanas específicas
typedef int (WINAPI *GetWindowTextW_t)(HWND, LPWSTR, int);
GetWindowTextW_t GetWindowTextW_original;

int WINAPI GetWindowTextW_hook(HWND hWnd, LPWSTR lpString, int nMaxCount) {
    int resultado = GetWindowTextW_original(hWnd, lpString, nMaxCount);
    
    // Si el título contiene tokens de la blacklist, reemplazarlo
    std::wstring titulo(lpString);
    if (titulo.find(L"x32dbg") != std::wstring::npos ||
        titulo.find(L"ghidra") != std::wstring::npos ||
        titulo.find(L"windbg") != std::wstring::npos) {
        wcsncpy_s(lpString, nMaxCount, L"Ventana", _TRUNCATE);
        return 7;
    }
    
    return resultado;
}
```

Este hook se instala en el proceso del AC, no en el proceso de la herramienta.

---

## Ocultar Herramientas de la Blacklist de Procesos

Para herramientas que deben correr con su nombre original (por compatibilidad con plugins, etc.):

```cpp
// Hook de CreateToolhelp32Snapshot + Process32Next para filtrar procesos
typedef BOOL (WINAPI *Process32NextW_t)(HANDLE, LPPROCESSENTRY32W);
Process32NextW_t Process32NextW_original;

// Lista de procesos a ocultar del AC
const wchar_t* PROCESOS_OCULTOS[] = {
    L"fiddler.exe",
    L"x32dbg.exe",
    L"ghidra.exe",
    nullptr
};

BOOL WINAPI Process32NextW_hook(HANDLE hSnapshot, LPPROCESSENTRY32W lppe) {
    while (Process32NextW_original(hSnapshot, lppe)) {
        bool ocultar = false;
        for (int i = 0; PROCESOS_OCULTOS[i]; i++) {
            if (_wcsicmp(lppe->szExeFile, PROCESOS_OCULTOS[i]) == 0) {
                ocultar = true;
                break;
            }
        }
        if (!ocultar) return TRUE;
        // Si hay que ocultarlo, saltar al siguiente proceso
    }
    return FALSE;
}
```

---

## Estrategia Combinada Recomendada

Para análisis del AC con herramientas RE mientras el AC está corriendo:

1. Renombrar los ejecutables de las herramientas RE (más simple)
2. Para x32dbg: cambiar el título de ventana en las opciones
3. Para Fiddler: renombrar como `traffic_editor.exe`
4. Usar una VM separada para el análisis dinámico — el AC no puede ver el host
5. Si se necesita análisis en la misma máquina: DLL inyectada en el AC que hookee `EnumWindows` y `CreateToolhelp32Snapshot`
