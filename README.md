<p align="center">

<a href="https://github.com/d-asnaghi/bazel-riscv-none-elf/blob/master/LICENSE">
    <img alt="GitHub license" src="https://img.shields.io/github/license/d-asnaghi/bazel-riscv-none-elf?color=success">
</a>

<a href="https://github.com/d-asnaghi/bazel-riscv-none-elf/stargazers">
    <img alt="GitHub stars" src="https://img.shields.io/github/stars/d-asnaghi/bazel-riscv-none-elf?color=success">
</a>

<a href="https://github.com/d-asnaghi/bazel-riscv-none-elf/issues">
    <img alt="GitHub issues" src="https://img.shields.io/github/issues/d-asnaghi/bazel-riscv-none-elf">
</a>

<a href="https://github.com/d-asnaghi/bazel-riscv-none-elf/actions">
    <img alt="Linux" src="https://github.com/d-asnaghi/bazel-riscv-none-elf/workflows/Linux/badge.svg">
</a>

<a href="https://github.com/d-asnaghi/bazel-riscv-none-elf/actions">
    <img alt="macOS" src="https://github.com/d-asnaghi/bazel-riscv-none-elf/workflows/macOS/badge.svg">
</a>

<a href="https://github.com/d-asnaghi/bazel-riscv-none-elf/actions">
    <img alt="Widnows" src="https://github.com/d-asnaghi/bazel-riscv-none-elf/workflows/Windows/badge.svg">
</a>

</p>

<p align="center">

<img src="https://asnaghi.me/images/bazel-arm.png" width="400px"/>

</p>


The goal of the project is to illustrate how to use a custom riscv-none-elf embedded toolchain with Bazel.

If this project was useful to you, give it a ⭐️ and I'll keep improving it!

You can follow the post [Bazel for ARM embedded toolchains](https://asnaghi.me/post/embedded-bazel/) to get more details about this code.

## Use the toolchain from this repo

To get started with the arm none eabi embedded toolchain, copy the appropriate `WORKSPACE` setup
from the [releases](https://github.com/d-asnaghi/bazel-riscv-none-elf/releases) page.

Using a stable commit from the repo is also an option,for example:

```python
# WORKSPACE

#---------------------------------------------------------------------
# ARM none eabi GCC
#---------------------------------------------------------------------
git_repository(
    name = "riscv_none_elf",
    commit = "<commit>",
    remote = "https://github.com/hexdae/bazel-riscv-none-elf",
    shallow_since = "<value>",
)

load("@riscv_none_elf//:deps.bzl", "riscv_none_elf_deps")

riscv_none_elf_deps()
#---------------------------------------------------------------------
```

And this to your `.bazelrc `
```bash
# .bazelrc

# Build using platforms by default
build --incompatible_enable_cc_toolchain_resolution
build --platforms=@riscv_none_elf//platforms:riscv_none_generic
```

If for you are using Bazel rules that do not support platforms, you can also use this instead
```bash
# .bazelrc

# Use the legacy crosstool-top config
build:legacy --crosstool_top=@riscv_none_elf//toolchain
build:legacy --host_crosstool_top=@bazel_tools//tools/cpp:toolchain
```

Now Bazel will automatically use `riscv-none-elf-gcc` as a compiler

## Platforms

By default, this repo defines the `riscv-none-elf-generic` platform as:
```python
platform(
    name = "riscv_none_generic",
    constraint_values = [
        "@platforms//os:none",
        "@platforms//cpu:arm",
    ],
)
```

If you want to further differentiate your project's platforms (for example to support a certain board/cpu) you can extend it using this template:

```python
platform(
    name = "your_custom_platform",
    constraint_values = [
        "<your additional constraints>",
    ],
    parents = [
        "@riscv_none_elf//platforms:riscv_none_generic"
    ],
)
```

## Configurable build attributes

If you want to select some build attributes based on their compatibility with the riscv-none-elf-gcc toolchain, you can use the `config_settings` available at `@riscv_none_elf//config:riscv_none_compatible`, defined as:

```python
config_setting(
    name = "riscv_none_compatible",
    constraint_values = [
        "@platforms//cpu:arm",
        "@platforms//os:none",
    ],
)
```

You can always add onto this `config_setting` by creating your own `config_setting_group` that inherits from this one:

```python
load("@bazel_skylib//lib:selects.bzl", "selects")

config_setting()
    name = "your_config_setting",
    ...
)

selects.config_setting_group(
    name = "your_config_setting_group",
    match_all = [
        "@riscv_none_elf//config:riscv_none_compatible",
        ":your_config_setting"
    ],
)
```

and then use these definitions to `select` in rules

```python
filegroup(
    name = "riscv_none_srcs",
    srcs = [...],
)

cc_binary(
    name = "your_binary",
    srcs = select({
        "@riscv_none_elf//config:riscv_none_compatible": [":riscv_none_srcs"],
        ...
    })
)
```

## Direct access to gcc tools

If you need direct access to `gcc` tools, they are available as `@riscv_none_elf//:<tool>`. For example, the following `genrules` could be used to produce `.bin` and `.hex` artifacts from a generic `.out` target.

```python

cc_binary(
    name = "target.out"
    srcs = [...],
    deps = [...],
    copts = [...],
    ...
)

genrule(
    name = "bin",
    srcs = [":target.out"],
    outs = ["target.bin"],
    cmd = "$(execpath @riscv_none_elf//:objcopy) -O binary $< $@",
    tools = ["@riscv_none_elf//:objcopy"],
)

genrule(
    name = "hex",
    srcs = [":target.out"],
    outs = ["target.hex"],
    cmd = "$(execpath @riscv_none_elf//:objcopy) -O ihex $< $@",
    tools = ["@riscv_none_elf//:objcopy"],
)
```

## Integrate the toolchain into your project

Follow these steps if you want to test this repo before using it to integrate
the toolchain into your local project.

### Bazel

[Install Bazel](https://docs.bazel.build/versions/master/install.html) for your platform. Installing with a package manager is recommended, especially on windows where additional runtime components are needed.

- [Ubuntu Linux](https://docs.bazel.build/versions/master/install-ubuntu.html): `sudo apt install bazel`
- [macOS](https://docs.bazel.build/versions/master/install-os-x.html): `brew install bazel`
- [Windows](https://docs.bazel.build/versions/master/install-windows.html): `choco install bazel`

### Bazelisk

`bazelisk` is a user-friendly launcher for `bazel`. Follow the install instructions in the [Bazelisk repo](https://github.com/bazelbuild/bazelisk)

Use `bazelisk` as you would use `bazel`, this takes care of using the correct Bazel version for each project by using the [.bazelversion](./.bazelversion) file contained in each project.

### Clone the repo

```bash
git clone https://github.com/d-asnaghi/bazel-riscv-none-elf.git
```

### Build

Use this command to build any target (a mock target `examples` is provided).

```bash
# build the examples
bazelisk build examples
```

This will take care of downloading the appropriate toolchain for your OS and compile all the source files specified by the target.

## Folder structure

```bash
├── WORKSPACE
│
├── examples
│   ├── BUILD.bazel
│   └── /* SOURCE CODE */
│
└── toolchain
    ├── BUILD.bazel
    ├── compiler.BUILD
    ├── config.bzl
    └── riscv-none-elf
        ├── darwin_x86_64
        │   └── /* DARWIN TOOLCHAIN   */
        ├── linux_x86_64
        │   └── /* LINUX TOOLCHAIN    */
        ├── windows_x86_32
        │   └── /* WINDOWS TOOLCHAIN  */
        └── ...
            └── /* OTHER TOOLCHAINS   */

```