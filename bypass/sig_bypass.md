# Cheat Signature Bypass — Complete Guide

The AC downloads byte pattern signatures from the server and scans game process memory.
This explains what it scans and how to make your code invisible to it.

---

## How the Scanner Works

```
1. Client connects → requests CheatSigs from server
2. Server sends array of byte patterns (repeated bytes CheatSigs = 5)
3. AC iterates over game process memory using VirtualQuery
4. For each committed readable region:
   memmem(region, region_size, pattern, pattern_size)
5. On match → fills Pattern{Name, Data, Size} → sends to server → BAN
```

The patterns are byte sequences — specific opcodes, strings, or data structures
characteristic of known cheats.

---

## Bypass Methods

### Method 1 — Polymorphic Runtime Decryption

XOR-encrypt your payload with a session-unique key. The encrypted bytes on disk/in memory 
don't match any signature. Decrypt JIT into a freshly allocated executable region:

```cpp
#include <windows.h>
#include <cstdint>
#include <cstring>

// Your cheat payload (encrypted with XOR key at build time)
// Key changes every build → different bytes every time
extern const uint8_t ENCRYPTED_PAYLOAD[];
extern const size_t  PAYLOAD_SIZE;
extern const uint8_t XOR_KEY;

void* load_and_decrypt() {
    // Allocate a random-address executable region
    void* exec_mem = VirtualAlloc(
        nullptr,                           // Let OS choose base → ASLR
        PAYLOAD_SIZE,
        MEM_COMMIT | MEM_RESERVE,
        PAGE_EXECUTE_READWRITE             // Temp: need write for decryption
    );
    
    if (!exec_mem) return nullptr;
    
    // Decrypt in-place with rolling XOR
    uint8_t* out = (uint8_t*)exec_mem;
    for (size_t i = 0; i < PAYLOAD_SIZE; i++) {
        out[i] = ENCRYPTED_PAYLOAD[i] ^ (uint8_t)(XOR_KEY + i * 0x13 + (i >> 3));
    }
    
    // Lock to read-execute only (prevents trivial memory dump detection)
    DWORD old_prot;
    VirtualProtect(exec_mem, PAYLOAD_SIZE, PAGE_EXECUTE_READ, &old_prot);
    
    return exec_mem;
}

// Build-time encryption tool (compile separately):
// Takes plaintext cheat DLL → outputs encrypted array with random key
void encrypt_payload(const uint8_t* input, size_t len, uint8_t key, uint8_t* output) {
    for (size_t i = 0; i < len; i++)
        output[i] = input[i] ^ (uint8_t)(key + i * 0x13 + (i >> 3));
}
```

**Why it works**: The scanner sees encrypted random-looking bytes. No static signature survives re-keying. Every build produces different bytes.

---

### Method 2 — Module-less Injection (Manual Map)

Normal DLL injection registers the module in PEB's module list. AC can enumerate modules.
Manual mapping bypasses this:

```cpp
// Manual PE loader — map DLL without LoadLibrary
// No entry in CreateToolhelp32Snapshot, EnumProcessModules, etc.

void* ManualMap(const uint8_t* dllData, size_t dllSize) {
    PIMAGE_DOS_HEADER dosHdr = (PIMAGE_DOS_HEADER)dllData;
    PIMAGE_NT_HEADERS ntHdr  = (PIMAGE_NT_HEADERS)(dllData + dosHdr->e_lfanew);
    
    // Allocate at preferred base (or let ASLR choose)
    uint8_t* base = (uint8_t*)VirtualAlloc(
        (LPVOID)ntHdr->OptionalHeader.ImageBase,
        ntHdr->OptionalHeader.SizeOfImage,
        MEM_COMMIT | MEM_RESERVE,
        PAGE_EXECUTE_READWRITE
    );
    
    // Copy headers
    memcpy(base, dllData, ntHdr->OptionalHeader.SizeOfHeaders);
    
    // Copy sections
    PIMAGE_SECTION_HEADER section = IMAGE_FIRST_SECTION(ntHdr);
    for (int i = 0; i < ntHdr->FileHeader.NumberOfSections; i++, section++) {
        if (section->SizeOfRawData) {
            memcpy(
                base + section->VirtualAddress,
                dllData + section->PointerToRawData,
                section->SizeOfRawData
            );
        }
    }
    
    // Process imports, relocations...
    // (standard PE loading steps)
    
    return base;
}
```

**Combined with Method 1**: Manual-map your encrypted DLL, decrypt in-place → double bypass.

---

### Method 3 — Code Cave in Game Binary

The legitimate game binary (`left4dead2.exe`) is already in memory and likely NOT scanned 
(scanning your own trusted binary would be expensive and self-defeating).

Find unused code space (gaps between functions) and inject there:

```python
# Find code caves in left4dead2.exe
# Look for sequences of 0xCC (INT3 padding) or 0x00 blocks of sufficient size

def find_caves(pe_data: bytes, min_size: int = 256) -> list:
    caves = []
    i = 0
    while i < len(pe_data) - min_size:
        if pe_data[i] == 0x00:
            j = i
            while j < len(pe_data) and pe_data[j] == 0x00:
                j += 1
            if j - i >= min_size:
                caves.append((i, j - i))
            i = j
        else:
            i += 1
    return caves

# Then: write shellcode into cave → hook game function to call it
```

---

### Method 4 — Kernel Allocation

Allocate memory from kernel mode using a driver. VirtualQuery from userland 
returns info on user-mode allocations only. Kernel allocations (pool memory mapped 
into user space via MDL) are not visible to a userland scanner.

```c
// Kernel driver: allocate and map memory invisible to VirtualQuery
PVOID KernelAllocateUserVisible(PEPROCESS process, SIZE_T size) {
    // Allocate non-paged pool
    PVOID kernel_mem = ExAllocatePoolWithTag(NonPagedPoolNx, size, 'FAKE');
    
    // Build MDL to map into user process VA space
    PMDL mdl = IoAllocateMdl(kernel_mem, size, FALSE, FALSE, NULL);
    MmBuildMdlForNonPagedPool(mdl);
    
    // Map with user-accessible permissions
    PVOID user_va = MmMapLockedPagesSpecifyCache(
        mdl, UserMode, MmCached, NULL, FALSE, NormalPagePriority
    );
    
    return user_va;  // VirtualQuery on this returns nothing useful
}
```

---

## Understand What Gets Scanned

The AC scanner likely scans these memory types (based on VirtualQuery usage + typical AC behavior):

```
MEM_COMMIT regions with:
  PAGE_EXECUTE_READ         → executable code (main target)
  PAGE_EXECUTE_READWRITE    → JIT compiled or injected code (suspicious)
  PAGE_EXECUTE_WRITECOPY    → copy-on-write executable
  
Skipped (likely):
  PAGE_NOACCESS             → access violation on read
  MEM_FREE                  → not allocated
  Known system DLL ranges   → skip ntdll.dll, kernel32.dll etc.
  Game binary ranges        → likely skip left4dead2.exe regions
```

The scanner reads each region with `ReadProcessMemory` (since VirtualQuery doesn't give content)
or directly via pointer if in-process. Given the AC is a separate launcher process
(not injected into the game), it likely uses `ReadProcessMemory`.

**Counter**: If you allocate with `PAGE_GUARD`, the first access raises a guard exception.
The AC would need to handle this — most don't. (Not verified for this AC.)

---

## Pattern Categories (What Gets Signatured)

Based on `CheatSigs` protobuf field and cheat scene knowledge, patterns likely include:

```
1. Cheat DLL PE headers (MZ + specific sections)
2. Known function prologues of popular cheats
3. Specific string literals from cheat configs
4. Magic bytes / struct headers from cheat frameworks
5. Characteristic import sequences
6. Config file signatures if loaded into memory
```

**Practical**: If you're writing a cheat from scratch (not using known public ones),
you have NO signature → Method 1/2 are overkill. Only public/leaked cheats have signatures.
