# Building the Android kbase ICD (PanLore)

## Mesa line

- Tree in this repo tracks `staging/26.3` for reference.
- **Android ICD builds** use the kbase layer’s pinned Mesa from `sources.lock`
  (`26.3.0-devel`, commit `5a07217f…`) so patches apply cleanly.

## CI

Workflow: **Build kbase Android ICD**

1. Clone [zenithblue-oss/panvk-kbase-android](https://github.com/zenithblue-oss/panvk-kbase-android)
2. `fetch-mesa.sh` → pinned commit
3. `apply-patches.sh --profile g615-v11-csf`
   - Includes **kbase-common** (UAPI + `kbase_kmod`), android, wsi, app-loader
   - CSF series + **jm-v9** + **jm-v7**
4. `build-android.sh` with `-Dpanfrost-kmds=kbase`
5. `PANVK_KBASE_RELEASE=PanLore` → `driverInfo`
6. Artifact: `.so`, ICD JSON, `meta.json`, checksums

See also [RUNTIME.md](RUNTIME.md).

## Local

```sh
git clone https://github.com/zenithblue-oss/panvk-kbase-android.git
cd panvk-kbase-android
./scripts/fetch-mesa.sh
./scripts/apply-patches.sh --profile g615-v11-csf
export PANVK_KBASE_RELEASE=PanLore
./scripts/build-android.sh --profile g615-v11-csf --api 29
```

## Patch families (what “UAPI support” actually is)

| Family | Contents |
|--------|----------|
| kbase-common | Meson `panfrost-kmds=kbase`, kmod backend, vendored UAPI headers, enumerate/device |
| android | Bionic, gralloc/AHB |
| app-loader | ICD entry points |
| wsi | Swapchain / present |
| jm-v7 / jm-v9 | Job Manager backends |
| csf / csf-v* | CSF backends |

Maintainer: TheoLore
