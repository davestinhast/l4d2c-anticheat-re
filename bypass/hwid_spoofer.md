# HWID Bypass — Complete Guide

The AC collects HWID via WMI queries. These are the exact WMI classes and properties it reads.

---

## What Gets Collected

```
Win32_DiskDrive.SerialNumber          ← PRIMARY — most weight in HWID hash
Win32_DiskDrive.PNPDeviceID
Win32_BaseBoard.SerialNumber          ← SECONDARY  
Win32_BaseBoard.Manufacturer
Win32_Processor.ProcessorId           ← TERTIARY
Win32_Processor.Microcode
Win32_Processor.MaxClockSpeed
Win32_Processor.NumberOfCores
Win32_Processor.Stepping
```

HWID ban = server stores `Hash(DiskSerial + MoboSerial + CPUId)` → if you reconnect with same hash → instant ban.

---

## Method 1 — WMI Provider Subscription (Recommended, Userland)

Register a permanent WMI subscription that intercepts and replaces properties for specific classes.

```csharp
// C# WMI provider override — run ONCE before launching AC
using System.Management;
using System;

class HWIDSpoofer {
    static void SpoofSerial() {
        // Override Win32_DiskDrive SerialNumber via WMI permanent subscription
        // NOTE: This approach uses WQL event subscription — fires before AC reads
        
        string fakeSerial  = GenerateRandomSerial(20);
        string fakeMoboSN  = GenerateRandomSerial(16);
        
        // The actual override requires a MOF-registered provider or
        // Registry-level spoofing (see Method 2)
        Console.WriteLine($"[*] Target serial: {fakeSerial}");
    }
    
    static string GenerateRandomSerial(int len) {
        var chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        var rng = new Random();
        var sb = new System.Text.StringBuilder(len);
        for (int i = 0; i < len; i++)
            sb.Append(chars[rng.Next(chars.Length)]);
        return sb.ToString();
    }
}
```

---

## Method 2 — Registry Volume Serial Override

Windows caches disk volume serial numbers in the registry:

```
HKLM\SYSTEM\CurrentControlSet\Services\disk\Enum
HKLM\SYSTEM\MountedDevices
```

The volume serial (not physical serial, but often used) can be changed:
```cmd
:: Change volume serial (requires admin)
:: Tool: VolumeID by Sysinternals
VolumeID.exe C: XXXX-YYYY
```

> This changes the logical volume serial. The physical WMI serial (Win32_DiskDrive) requires a driver-level approach.

---

## Method 3 — Kernel Driver Filter (Most Effective)

A kernel filter driver sitting between the storage stack and upper layers intercepts `IOCTL_STORAGE_QUERY_PROPERTY` and returns fake serials.

```c
// Kernel driver — StorageDeviceProperty spoofing
NTSTATUS FilterDispatchIoctl(
    PDEVICE_OBJECT DeviceObject,
    PIRP Irp
) {
    PIO_STACK_LOCATION stack = IoGetCurrentIrpStackLocation(Irp);
    
    if (stack->Parameters.DeviceIoControl.IoControlCode == 
        IOCTL_STORAGE_QUERY_PROPERTY) {
        
        PSTORAGE_PROPERTY_QUERY query = Irp->AssociatedIrp.SystemBuffer;
        
        if (query->PropertyId == StorageDeviceProperty) {
            // Let the real driver respond first
            NTSTATUS status = ForwardAndWait(DeviceObject, Irp);
            
            // Then overwrite the serial number in the response
            PSTORAGE_DEVICE_DESCRIPTOR desc = Irp->AssociatedIrp.SystemBuffer;
            if (desc->SerialNumberOffset > 0) {
                char* serial = (char*)desc + desc->SerialNumberOffset;
                strncpy(serial, "FAKESN12345678901234", 20);
            }
            return status;
        }
    }
    
    return ForwardToDriver(DeviceObject, Irp);
}
```

**WMI chain for disk serial**:
```
WMI query → winmgmt service → WMI provider (discdump.dll) →
  DeviceIoControl(IOCTL_STORAGE_QUERY_PROPERTY) →
    Storage driver stack ← *** HOOK HERE ***
```

---

## Method 4 — Hook Win32 WMI in AC Process

Since the AC is a Go binary running in userland, its WMI calls go through `ole32.dll` and `wbemprox.dll`. You can hook these:

```
Target function: IWbemServices::ExecQuery
Hook:            After getting results, patch the IWbemClassObject for Win32_DiskDrive
Technique:       COM interface vtable hook
```

```cpp
// COM vtable hook for IWbemServices::ExecQuery
// The vtable is at a known offset — hook slot 20 (ExecQuery)
typedef HRESULT (STDMETHODCALLTYPE *ExecQuery_t)(
    IWbemServices* pThis,
    const BSTR strQueryLanguage,
    const BSTR strQuery,
    long lFlags,
    IWbemContext* pCtx,
    IEnumWbemClassObject** ppEnum
);

ExecQuery_t orig_ExecQuery = nullptr;

HRESULT STDMETHODCALLTYPE Hooked_ExecQuery(
    IWbemServices* pThis,
    const BSTR strQueryLanguage,
    const BSTR strQuery,
    long lFlags,
    IWbemContext* pCtx,
    IEnumWbemClassObject** ppEnum
) {
    HRESULT hr = orig_ExecQuery(pThis, strQueryLanguage, strQuery, 
                                lFlags, pCtx, ppEnum);
    
    if (wcsstr(strQuery, L"Win32_DiskDrive") && SUCCEEDED(hr)) {
        // Wrap the enumerator to replace SerialNumber property
        *ppEnum = new FakeSerialEnumerator(*ppEnum);
    }
    return hr;
}
```

---

## Verification — Check What AC Sees

After applying spoofer, verify with PowerShell:
```powershell
# Check what WMI returns (same as what AC reads)
Get-WmiObject Win32_DiskDrive | Select-Object SerialNumber, PNPDeviceID
Get-WmiObject Win32_BaseBoard | Select-Object SerialNumber, Manufacturer
Get-WmiObject Win32_Processor | Select-Object ProcessorId, MaxClockSpeed

# Compare before and after spoofer
```

If the values changed — the AC will compute a different HWID hash → bypass ban.

---

## HWID Hash Reconstruction (Estimated)

Based on crypto strings found (`SHA-256`, `MD5`) and field names:

```python
import hashlib

def compute_hwid(disk_serial, mobo_serial, cpu_processor_id):
    combined = f"{disk_serial}:{mobo_serial}:{cpu_processor_id}"
    # Likely SHA-256 or MD5 — try both
    sha256 = hashlib.sha256(combined.encode()).hexdigest()
    md5    = hashlib.md5(combined.encode()).hexdigest()
    return sha256, md5

# Example with fake values:
h256, hmd5 = compute_hwid("FAKESN12345", "MBSERIAL001", "BFEBFBFF000906ED")
print(f"SHA256: {h256}")
print(f"MD5:    {hmd5}")
```

If you can capture two ban entries from the server (via MITM), you can verify the hash algorithm by checking if your computed hash matches the stored one.
