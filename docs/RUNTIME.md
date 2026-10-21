# Runtime requirements (PanLore kbase ICD)

What the built `libvulkan_panfrost.so` needs on a stock Android device.
This is not theoretical: it matches the public kbase layer used by the CI build.

## Already inside the binary

The ICD is not plain upstream Mesa. The build applies the kbase patch series, which includes:

| Piece | Role |
|--------|------|
| `kbase_uapi.h` / `mali_base_*.h` / ioctl headers | Userspace ABI against proprietary `mali_kbase` |
| `kbase_kmod.c` | Open `/dev/mali0`, version handshake, MEM_ALLOC, submit, sync |
| JM path (jm-v7 / jm-v9) | Bifrost / Valhall Job Manager atoms |
| CSF path (csf-v11 + …) | Newer Mali command stream frontend |
| Android WSI + AHB patches | Surfaces / hardware buffers where the platform allows |
| App-loader exports | `vk_icdGetInstanceProcAddr` + negotiate (works with Android WSI) |

Target kbase range stated by that backend: **JM uAPI 11.x** and **CSF uAPI 1.x** (roughly DDK r32–r44 era). Example: Galaxy A06 G52 tested with **kbase uAPI 11.38**.

You do **not** install separate UAPI headers on the phone. The driver talks to the kernel module already on the device.

## Kernel / device node

| Item | Expectation |
|------|-------------|
| Node | `/dev/mali0` (sometimes only this; not a DRM render node) |
| Access | App or container must be able to open RDWR (policy depends on ROM/SELinux) |
| Frontend | Auto-selected from GPU id: JM on v7/v9, CSF on v10+ |

If open fails: no physical device, crash, or "no Mali" in logs.

## Loader / ICD

Minimum files:

```
libvulkan_panfrost.so
panfrost_icd.aarch64.json   # or panvk_icd*.json from a launcher zip
```

Example ICD JSON:

```json
{
  "file_format_version": "1.0.0",
  "ICD": {
    "library_path": "libvulkan_panfrost.so",
    "api_version": "1.3"
  }
}
```

Common load methods:

1. **Adrenotools-style package** — install zip with `meta.json` + `.so`; launcher sets the driver path.
2. **Direct ICD** — set in the container/environment:

   `VK_ICD_FILENAMES=/absolute/path/to/panfrost_icd.aarch64.json`

3. **GameHub-style** — same binary renamed to `libvulkan_freedreno.so` (loader hard-codes that name).

Place the `.so` where the process can `dlopen` it (app private dir, jniLibs extraction path, or adrenotools contents dir). Assets-only without a real filesystem path often fails.

## Useful environment (optional)

| Variable | When |
|----------|------|
| `VK_ICD_FILENAMES` | Force this ICD |
| `MESA_VK_WSI_DEBUG=sw` | If X11 present path fails in some Winlator-like setups |
| `PANVK_DEBUG=…` | Mesa panvk debug categories (verbose; for logcat) |
| `PAN_I_WANT_A_BROKEN_VULKAN_DRIVER` | **Not required** for current jm-v7 experimental path in the series used by CI |

Do **not** set `MESA_LOADER_DRIVER_OVERRIDE=zink` for this ICD. This is native PanVK/kbase, not Zink.

## Arch status (honest)

| Arch | Frontend | Notes |
|------|----------|--------|
| v7 (G52, …) | JM | Experimental; basic create/draw/compute; lower feature level |
| v9 (G57, …) | JM | Much stronger in recent series |
| v10–v13 | CSF | Primary validation path |
| v14 | CSF | Built; less field data |

## Verify on device

1. Confirm logcat / wine log shows `driverInfo` containing **PanLore** (when built with `PANVK_KBASE_RELEASE=PanLore`).
2. Confirm `/dev/mali0` open does not fail for the app UID.
3. Prefer a probe app (e.g. PanProbe-class) before full games.

## What is not “missing UAPI”

Rewriting ioctl headers by hand is the wrong fix. The binary already embeds the kbase userspace path. Failures are usually:

- wrong load path / missing ICD
- SELinux or permissions on `/dev/mali0`
- expecting full DXVK feature level on v7
- mixing Zink overrides with this ICD

Maintainer account: TheoLore
