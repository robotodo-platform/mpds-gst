# Known issues

## Linux runtime library completeness (deferred)

Cerbero 1.28.9 SDK archive inspection on x86-64 and AArch64 confirmed:

- OpenSSL (`libcrypto.so.3`, `libssl.so.3`) is intentionally omitted from native Linux runtime archives by `packages/base-crypto.package`. Consider an ABI-compatible Conda runtime dependency later.
- `liba52.so` is classified as development-only, but `libgsta52dec.so` requires it at runtime on both architectures. Report upstream.
- On AArch64 with VA enabled, `libgstva.so` requires `libgstva-1.0.so.0`, absent from both SDK archives. The upstream bad-plugins recipe omits the library declaration. x86-64 VA has not been independently verified.

Rattler-build preserves these omissions; no fixes or manual library copying for now. Conda-forge builds individual GStreamer modules directly with Meson and uses Conda dependencies instead of Cerbero SDK archive classification.

Evidence: [CI run 45565953029](https://github.com/robotodo-platform/mpds-gst/actions/runs/45565953029) and native Orin archive inspection. Separate ARM64 system discovery issue: [Cerbero #590](https://gitlab.freedesktop.org/gstreamer/cerbero/-/work_items/590).
