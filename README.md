# GStreamer Cerbero bridge

`mpds-gst` provides GStreamer runtime and development Conda packages for MPDS,
built with upstream [Cerbero](https://gitlab.freedesktop.org/gstreamer/cerbero).
It does not depend on MPDS or contain MPDS-specific code.

Pixi's Rattler-build backend checks out the upstream Cerbero Git tree at a
pinned commit and builds its full `gstreamer-1.0` SDK once, using Cerbero's
upstream package selection. Cerbero itself
separates runtime and development files into two tarballs. Rattler-build
installs each tarball directly into its corresponding Conda output:
`mpds-gst` for runtime and `mpds-gst-devel` for development files. The
development output depends on its exact matching runtime build. Conda does
not maintain a second file classification or retain the intermediate tarballs.

Cerbero controls the plugins and native dependencies; this repository has
no plugin allowlist or separate Python wheel.

## Build

From this directory, with Pixi installed:

```sh
pixi exec -c conda-forge --spec rattler-build \
  rattler-build build --experimental --recipe recipe.yaml \
  --channel conda-forge --test native --output-dir dist
```

Rattler-build runs the tests defined in each output's `tests:` section in
isolated environments as part of the build, and writes the packages under
`dist/<platform>/`.

Cerbero installs its platform prerequisites through upstream bootstrap.
GitHub Actions builds both Linux architectures in the official manylinux 2.28
containers (glibc 2.28).

This is a native-source build, not a repack of arbitrary conda-forge GStreamer
libraries. Cerbero's upstream bootstrap requires network access to install its
pinned build tools. Release validation must check the native shared-library
dependencies of the complete SDK and confirm that they resolve after installation.
The `run_constraints` prevent installation alongside the conda-forge GStreamer
packages. Consumers depend on `mpds-gst` without selecting individual
GStreamer plugin packages.

## CI and publishing

CI builds and tests `linux-64` and `linux-aarch64`. Windows and macOS are
deferred until after MPDS v1a0. Publishing a GitHub release triggers the build
and publication of tested packages to Anaconda.org after every build passes.
Prereleases publish to the `dev` label; stable releases publish to `main`.
Configure the GitHub `conda` environment with variable `ANACONDA_OWNER`
and secret `ANACONDA_API_KEY`.

The package version follows the upstream GStreamer version in `recipe.yaml`,
which also selects the Cerbero revision. The Conda build number starts at `0`.
Increment `build.number` for packaging changes at the same upstream version
and reset it on upstream upgrades.
Tag releases as `v<version>-<build number>` (for example, `v1.28.9-0`).
Until MPDS v1a0, publish GitHub prereleases and use
`robotodo-platform/label/dev::mpds-gst`. Promote validated packages to the
`main` label for stable distribution.

GStreamer's libraries and plugins retain their upstream licenses; the Cerbero
package includes its upstream license notices.

## Known issues

Cerbero 1.28.9 has the following Linux runtime packaging issues on both
x86-64 and AArch64:

- OpenSSL (`libcrypto.so.3`, `libssl.so.3`) is intentionally omitted from native Linux runtime archives by `packages/base-crypto.package`. Consider an ABI-compatible Conda runtime dependency later.
- `liba52.so` is classified as development-only, but `libgsta52dec.so` requires it at runtime. Report upstream.
- `libgstva.so` requires `libgstva-1.0.so.0`, which is missing from the runtime archives. The upstream bad-plugins recipe omits the library declaration. Report upstream.

Rattler-build preserves these upstream omissions. Fixes and manual library
copying are deferred. Conda-forge builds individual GStreamer modules directly
with Meson and uses Conda dependencies instead of Cerbero SDK archive classification.

Evidence: [CI run 45565953029](https://github.com/robotodo-platform/mpds-gst/actions/runs/45565953029),
[CI run 45657754522](https://github.com/robotodo-platform/mpds-gst/actions/runs/45657754522),
and native Orin archive inspection. Separate ARM64 system discovery issue:
[Cerbero #590](https://gitlab.freedesktop.org/gstreamer/cerbero/-/work_items/590).
