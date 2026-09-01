# GL.iNet OpenWrt Build Guide

This document explains how to build supported GL.iNet firmware images from the
OpenWrt 25.12 source tree. Common source and feed preparation steps are shared
by every model, while each model uses its own complete OpenWrt `.config`, DTS,
and image profile.

## Supported models

| Model | Complete configuration | Target |
| --- | --- | --- |
| GL-BE10000 | `gl_configs/gl-be10000.config` | MediaTek Filogic |

Only models listed in this table have a complete configuration ready for a
direct build. Additional models must provide a separate configuration named
`gl_configs/<model>.config` before they are added here.

## 1. Clone the source code

Install the standard OpenWrt build dependencies for your Linux distribution,
then clone the `openwrt-25.12` branch:

```sh
git clone --branch openwrt-25.12 --single-branch \
  https://github.com/gl-inet/gl-openwrt.git
cd gl-openwrt
```

If the source tree has already been cloned, update it before building:

```sh
git checkout openwrt-25.12
git pull --ff-only
```

The repository only needs to be cloned once. The same source tree can build any
model listed in the supported-model table.

## 2. Update and install the feeds

```sh
./scripts/feeds update -a
./scripts/feeds install -a
```

Run both commands again whenever `feeds.conf.default` or a feed revision is
updated. Feed preparation is shared by all supported models.

## 3. Select a model and copy its complete configuration

Set `MODEL` to a supported configuration name. For GL-BE10000:

```sh
MODEL=gl-be10000
test -f "gl_configs/${MODEL}.config"
cp "gl_configs/${MODEL}.config" .config
make defconfig
```

Each file under `gl_configs/` is a full `.config`, not a configuration fragment.
Do not enable multiple device profiles in one configuration.

The GL-BE10000 configuration selects:

```text
Target System: MediaTek Ralink ARM
Subtarget: Filogic 8x0
Target Profile: GL.iNet GL-BE10000
```

## 4. Build the firmware

Use all available CPU cores for a normal build:

```sh
make -j"$(nproc)"
```

If the build fails, rebuild serially with verbose output to obtain the complete
error message:

```sh
make -j1 V=s
```

The generated images are written under:

```text
bin/targets/<target>/<subtarget>/
```

For GL-BE10000, the output directory is:

```text
bin/targets/mediatek/filogic/
```

## Switching models

To build another supported model in the same source tree, copy that model's
complete configuration and normalize it before building:

```sh
# Replace this value with a model listed in the supported-model table.
MODEL=gl-be10000
test -f "gl_configs/${MODEL}.config"
cp "gl_configs/${MODEL}.config" .config
make defconfig
make -j"$(nproc)"
```

When switching to a different target or subtarget, clean target-specific build
artifacts first:

```sh
make dirclean
```

Then copy the new model configuration and start the build. Models on the same
target and subtarget normally do not require `make dirclean`.

## Complete GL-BE10000 command sequence

For a clean GL-BE10000 build environment, run:

```sh
git clone --branch openwrt-25.12 --single-branch \
  https://github.com/gl-inet/gl-openwrt.git
cd gl-openwrt
./scripts/feeds update -a
./scripts/feeds install -a
MODEL=gl-be10000
test -f "gl_configs/${MODEL}.config"
cp "gl_configs/${MODEL}.config" .config
make defconfig
make -j"$(nproc)"
```

## Adding another model

Keep model support isolated and reproducible:

1. Add a separate DTS and image profile for the model.
2. Build and verify the model configuration.
3. Save the complete configuration as `gl_configs/<model>.config`.
4. Add the verified model to the supported-model table in this document.

Share a common `.dtsi` only when models genuinely use the same hardware nodes.
Do not combine unrelated board definitions or multiple model profiles in one
configuration.
