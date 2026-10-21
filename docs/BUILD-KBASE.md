# Building the Android kbase ICD

This repository tracks Mesa **26.3** (`staging/26.3`). Upstream PanVK uses DRM.
Stock Android uses **mali_kbase** (`/dev/mali0`).

The production Android ICD is built with the public patch/build layer:

https://github.com/zenithblue-oss/panvk-kbase-android

## What the CI job does

Workflow: **Build kbase Android ICD**

1. Clones the kbase patch layer (pinned Mesa in `sources.lock`).
2. `fetch-mesa.sh` — exact Mesa commit.
3. `apply-patches.sh --profile g615-v11-csf` — universal series:
   - common, android, kbase-common, app-loader, wsi
   - csf + csf-v11
   - **jm-v9** (Valhall JM)
   - **jm-v7** (Bifrost JM, experimental)
4. `build-android.sh` with `-Dpanfrost-kmds=kbase` → **`libvulkan_panfrost.so`**
5. Sets `PANVK_KBASE_RELEASE` (default **PanLore**) for `driverInfo`.

Output artifact:

- `libvulkan_panfrost.so`
- `panfrost_icd.aarch64.json`
- `BUILD.txt`

## Local equivalent

```sh
git clone https://github.com/zenithblue-oss/panvk-kbase-android.git
cd panvk-kbase-android
./scripts/fetch-mesa.sh
./scripts/apply-patches.sh --profile g615-v11-csf
export PANVK_KBASE_RELEASE=PanLore
./scripts/build-android.sh --profile g615-v11-csf --api 29
# dist/android-g615-v11-csf/libvulkan_panfrost.so
```

## Runtime notes

- Place the `.so` where the launcher/ICD loader can `dlopen` it (e.g. `jniLibs/arm64-v8a` or loader path).
- Point `VK_ICD_FILENAMES` at `panfrost_icd.aarch64.json` when required.
- Mali-G52 (v7 JM) is **experimental** in this series; feature level is lower than v9+.

Maintainer: TheoLore
