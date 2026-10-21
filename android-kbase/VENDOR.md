# android-kbase (vendored)

Local copy of the Android mali_kbase patch/build layer used by
this repository's CI.

- Maintainer account: TheoLore
- Driver identity string: PanLore (set at build via PANVK_KBASE_RELEASE)
- Source git: https://github.com/zenithblue-oss/panvk-kbase-android.git
- Source ref: main
- Source commit: afbafab74c68dca982004281613e8539078eb55b
- Vendored date: 2026-10-21T12:58:44Z

Licenses of vendored headers and scripts remain as in upstream
files (see LICENSES/, NOTICE, and per-file headers). Do not strip
SPDX / copyright lines from UAPI or kmod sources.

Build with the workflow **Build kbase Android ICD**, which uses
only this directory (no network clone of the layer).
