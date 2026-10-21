# android-kbase (in-repo layer)

Maintainer: **TheoLore**  
Driver string: **PanLore**

## Goal

All kbase / JM / CSF / Android patch support lives under **`android-kbase/`**
inside this repository. CI builds do **not** clone a third-party layer repo.

## Import / refresh

Workflow: **Vendor kbase layer into repo**

1. Clones a public layer source (default: the public panvk-kbase-android tree).
2. Copies `patches/`, `scripts/`, `profiles/`, `sources.lock`, etc. into `android-kbase/`.
3. Writes `android-kbase/VENDOR.md` (provenance + licenses note).
4. Commits to `main`.

Re-run that workflow when you want to refresh the layer. Patch *bodies* stay
as upstream so `apply-patches.sh` remains valid; branding for packages is
PanLore / TheoLore at build and packaging time.

## Build

Workflow: **Build kbase Android ICD**

Uses only `android-kbase/` + NDK + host tools. Still downloads **pinned Mesa**
from `sources.lock` (that is Mesa source, not another driver product).

## Where to look after vendor

| Path | Content |
|------|---------|
| `android-kbase/patches/kbase-common/` | kbase UAPI + kmod |
| `android-kbase/patches/jm-v7/` | Bifrost JM |
| `android-kbase/patches/jm-v9/` | Valhall JM |
| `android-kbase/patches/csf/` | CSF |
| `android-kbase/patches/android/` | Android |
| `android-kbase/scripts/` | fetch / apply / build |
| `android-kbase/sources.lock` | Pinned Mesa commit |

Until you run **Vendor** once, `android-kbase/` may be missing and build will fail on purpose.
