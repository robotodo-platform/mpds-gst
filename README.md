# GStreamer Cerbero bridge

`mpds-gst` provides Conda runtime and development packages for the upstream GStreamer SDK
built by [Cerbero](https://gitlab.freedesktop.org/gstreamer/cerbero). Despite its
package name, it does not depend on MPDS or contain MPDS-specific code.

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
containers (glibc 2.28), while Windows and macOS use native GitHub runners.

This is a native-source build, not a repack of arbitrary conda-forge GStreamer
libraries. Cerbero's upstream bootstrap requires network access to install its
pinned build tools. Release validation must check the native shared-library
dependencies of the complete SDK and confirm that they resolve after installation.
The `run_constraints` prevent installation alongside the conda-forge GStreamer
packages. Consumers depend on `mpds-gst` without selecting individual
GStreamer plugin packages.

## CI and publishing

CI builds and tests `linux-64`, `linux-aarch64`, `win-64`, `osx-64`,
and `osx-arm64`. Linux builds use manylinux 2.28 containers; Windows and macOS
use native GitHub runners. Version tags trigger publication of the tested
packages to Anaconda.org after every build passes. Configure
`ANACONDA_OWNER` and `ANACONDA_API_KEY` in the GitHub `conda` environment.

GStreamer's libraries and plugins retain their upstream licenses; the Cerbero
package includes its upstream license notices.
