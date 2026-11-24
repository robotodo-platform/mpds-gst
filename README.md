# GStreamer Cerbero bridge

`mpds-gst` is a **single Conda package** for the upstream GStreamer SDK
built by [Cerbero](https://gitlab.freedesktop.org/gstreamer/cerbero). Despite its
package name, it does not depend on MPDS or contain MPDS-specific code.

Pixi's Rattler-build backend checks out the complete upstream Cerbero Git tree
at a pinned commit, builds its `gstreamer-1.0` SDK, and installs Cerbero's
runtime and development archives into **one** Conda package. Cerbero controls
the plugins and native dependencies; this repository has no plugin allowlist,
downstream SDK staging code, Python wheel, or separate development package.

## Build

From this directory, with Pixi installed:

```sh
pixi publish --path . --target-dir dist
```

For an x86-64 `manylinux_2_28` build, the ABI baseline is supplied
by the container. Cerbero installs the distribution's system prerequisites
through its own bootstrap; the Dockerfile does not duplicate that list:

```sh
docker build -f Dockerfile.manylinux -t gstreamer-manylinux .
docker run --rm \
    --mount "type=bind,src=$PWD,dst=/workspace" \
    gstreamer-manylinux \
    pixi publish --path . --target-dir dist
```

This is a native-source build, not a repack of arbitrary conda-forge GStreamer
libraries. Cerbero's upstream bootstrap requires network access to install its
pinned build tools. The Conda recipe declares OpenSSL 3 as a transitive runtime
dependency because the Cerbero SDK archives refer to its shared libraries.
The `run_constraints` prevent installation alongside the conda-forge GStreamer
packages. Consumers depend on `mpds-gst` without selecting individual
GStreamer plugin packages.

## Validation still required

The project is currently a packaging bridge, **not a certified manylinux SDK**.
The release process must verify the lowest possible glibc baseline and the
complete library/codec plugin dependency closure on x86-64 and AArch64. Cerbero
also needs all relevant system prerequisites for its target platform. Package
relocation, plug-in discovery, development metadata and installed-prefix behavior
must be tested before publishing. These validations must not be bypassed by
excluding libraries from auditwheel or dropping upstream plugins.

GStreamer's libraries and plugins retain their upstream licenses; the Cerbero
package includes its upstream license notices.
