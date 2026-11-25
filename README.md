# 🚀 CachyOS Kernel Porter for Debian

### The CachyOS Kernel. Re-engineered for Debian.

<p align="center">
  <strong>Automated kernel porting · Intelligent ISA detection · Native Debian packaging</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Debian%20%7C%20Debian--based-A81D33?style=for-the-badge&logo=debian&logoColor=white" alt="Debian">
  <img src="https://img.shields.io/badge/Language-Bash-121011?style=for-the-badge&logo=gnubash&logoColor=white" alt="Bash">
  <img src="https://img.shields.io/badge/Architecture-x86__64-0071C5?style=for-the-badge" alt="x86-64">
  <img src="https://img.shields.io/badge/Packaging-Debian%20DEB-orange?style=for-the-badge" alt="Debian packaging">
  <img src="https://img.shields.io/badge/License-See%20repository-lightgrey?style=for-the-badge" alt="License">
</p>

---

## 🌟 Overview

**CachyOS Kernel Porter for Debian** is an automated kernel porting and packaging system designed to bring the performance-oriented CachyOS Linux kernel to Debian GNU/Linux and compatible distributions.

The project aims to reproduce the relevant CachyOS kernel configuration, patches and build decisions while adapting the compilation and packaging infrastructure to Debian.

Instead of simply downloading a precompiled kernel, the Porter manages the build process:

- Automatically discovers the kernel version selected by the official CachyOS build recipe.
- Retrieves the corresponding kernel source and patches.
- Detects the host processor's effective x86-64 ISA capabilities.
- Selects the highest compatible ISA level.
- Adapts CachyOS kernel configuration to the available toolchain.
- Compiles the kernel.
- Generates native Debian `.deb` packages.
- Validates generated packages.
- Supports optional installation and boot integration.
- Maintains build fingerprints and version history.

The objective is straightforward:

**Bring the CachyOS kernel experience to Debian without depending on Arch Linux packaging infrastructure.**

> **Project status:** Experimental implementation. Automated tests and local CLI checks have been performed. A complete production kernel build and physical-machine boot validation are still pending.

## ✨ Features

| Feature | Description |
| --- | --- |
| 🔄 Automatic source discovery | Reads the official CachyOS BORE build recipe |
| 🧠 Intelligent ISA detection | CPUID, XGETBV and operating-system state validation |
| ⚡ CPU optimization | Selects x86-64, v2, v3 or v4 |
| 🧩 CachyOS integration | Preserves supported BORE configuration and patches |
| 🛠️ GCC / LLVM | Compiler selection based on recipe and toolchain support |
| 📦 Native Debian packaging | Uses the kernel's `bindeb-pkg` target |
| 🔐 Source verification | PGP signature verification of the kernel source |
| ♻️ Incremental builds | Tracks source, patch and configuration fingerprints |
| 📊 Resource management | CPU, RAM and cgroup-aware parallelism |
| 🧪 Automated testing | ISA simulation and package validation tests |
| 🛡️ Installation control | Installation and bootloader operations require explicit requests |
| 📝 Build history | Records validated builds and their metadata |

## 🏗️ Project Architecture

The Porter is organized into independent Bash modules.

```text
cachyos-debian-porter/
│
├── cachyos-debian-porter.sh
├── README.md
├── .gitignore
│
├── config/
│   └── porter.conf
│
├── lib/
│   ├── common.sh
│   ├── cpu-detection.sh
│   ├── isa-detection.sh
│   ├── source-manager.sh
│   ├── patch-manager.sh
│   ├── kernel-config.sh
│   ├── debian-packaging.sh
│   ├── build-manager.sh
│   ├── validation.sh
│   └── install-manager.sh
│
├── tests/
│   ├── run-tests.sh
│   ├── test-isa-detection.sh
│   └── test-packaging-validation.sh
│
├── sources/
│
├── build/
│
├── packages/
│
├── cache/
│
├── logs/
│
└── history/
```

### Core modules

| Module | Responsibility |
| --- | --- |
| `common.sh` | Logging, shared utilities and error handling |
| `cpu-detection.sh` | Processor information |
| `isa-detection.sh` | CPUID/XGETBV and ISA compatibility |
| `source-manager.sh` | Official source discovery and downloads |
| `patch-manager.sh` | CachyOS patch application |
| `kernel-config.sh` | Kernel configuration adaptation |
| `build-manager.sh` | Toolchain, resources and compilation |
| `debian-packaging.sh` | Debian package generation |
| `validation.sh` | Package and build validation |
| `install-manager.sh` | Optional installation and boot integration |

The main executable integrates these modules and provides both interactive and command-line operation.

## 🔗 Official Sources

The project prioritizes the original CachyOS repositories.

| Repository | Purpose |
| --- | --- |
| [CachyOS/linux-cachyos](https://github.com/CachyOS/linux-cachyos) | Kernel build recipes and configurations |
| [CachyOS/linux](https://github.com/CachyOS/linux) | Integrated CachyOS kernel source |
| [CachyOS/kernel-patches](https://github.com/CachyOS/kernel-patches) | Additional kernel patches |
| [CachyOS Website](https://cachyos.org/) | Official project information |

### Source discovery

The Porter reads the official `linux-cachyos-bore/PKGBUILD` instead of assuming that the newest GitHub release is necessarily the kernel selected by the CachyOS build system.

It extracts relevant metadata, including:

- Kernel major and minor versions.
- Release revision.
- Package release.
- Patch source.
- Compiler and LTO settings.
- Scheduler configuration.
- Official configuration options.
- Authorized PGP fingerprints.

The version is discovered dynamically. Kernel version numbers are not permanently hardcoded into the script.

The source archive must pass PGP signature verification before preparation continues.

## 🧠 Intelligent CPU and ISA Detection

One of the most important components is the automatic x86-64 ISA detection system.

The Porter identifies the instruction-set capabilities effectively available to the operating system.

It does not rely on the commercial CPU name or blindly use `-march=native`.

### Supported ISA levels

| ISA | Description |
| --- | --- |
| x86-64 | Original AMD64 baseline |
| x86-64-v2 | SSE3, SSSE3, SSE4.1, SSE4.2, POPCNT and required features |
| x86-64-v3 | AVX, AVX2, BMI1, BMI2, FMA and additional features |
| x86-64-v4 | AVX-512 feature set required by the ABI level |

### Detection methodology

The ISA detector uses a temporary C probe to:

1. Read the CPU affinity mask.
2. Enumerate CPUs available to the process.
3. Pin the probe to each available CPU.
4. Execute CPUID instructions.
5. Check XSAVE and OSXSAVE.
6. Execute XGETBV when permitted.
7. Validate the XCR0 state required by AVX and AVX-512.
8. Calculate the lowest common ISA level.

This conservative approach is particularly important for:

- Virtual machines.
- Hypervisors masking CPU features.
- Hybrid processors.
- Systems with different CPU capabilities.
- Restricted CPU affinity environments.

### ISA selection

```text
                Detect CPU capabilities
                         |
                         v
                  Validate CPUID
                         |
                         v
                 Validate XCR0
                         |
                         v
               Evaluate ISA features
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
           x86-64       v2          v3/v4
             |           |           |
             +-----------+-----------+
                         |
                         v
              Select highest compatible
                    common ISA
                         |
                         v
                Validate Kconfig
```

The selected ISA is applied through kernel configuration mechanisms, not indiscriminately through userspace compiler flags.

### Manual ISA restriction

The user can request a lower ISA level:

```bash
./cachyos-debian-porter.sh --build --isa 2
```

A requested level above the detected host capability is rejected.

## ⚡ CachyOS Kernel Integration

The initial implementation targets the **CachyOS BORE variant**.

It reproduces supported settings from the official build recipe, including:

- CachyOS kernel configuration.
- BORE scheduler integration.
- Relevant scheduler patches.
- Preemption configuration.
- Tick-rate configuration.
- Tickless configuration.
- Transparent Huge Pages configuration.
- Supported compiler optimizations.
- Supported LTO settings.

The Porter does not blindly copy every Arch-specific build operation.

Unsupported recipe features that could affect the resulting kernel are treated as compatibility errors rather than silently ignored.

### Compiler selection

The project supports:

- GCC.
- LLVM/Clang.
- LLD when required by the selected configuration.

The default compiler selection follows the relevant CachyOS recipe settings and actual kernel requirements.

LTO is not forced when the official recipe does not enable it.

PGO, AutoFDO and other experimental optimization systems are not presented as implemented features.

## 📦 Native Debian Packaging

The Porter uses the Linux kernel's own Debian packaging infrastructure.

The principal build target is:

```bash
make -j"$(nproc)" bindeb-pkg
```

The actual build process uses the resource-aware job calculation implemented by the Porter.

### Generated packages

The primary expected packages are:

| Package | Contents |
| --- | --- |
| `linux-image-*` | Kernel image and kernel modules |
| `linux-headers-*` | Kernel headers for external modules |

The kernel's Debian packaging infrastructure stages `modules_install` inside the image package.

The Porter therefore does not install modules directly into the host system during package creation.

It also avoids replacing Debian's independently managed `linux-libc-dev` package.

### Package isolation

Each build receives a distinct kernel release identifier incorporating relevant build information, including:

- Kernel version.
- CachyOS variant.
- ISA level.
- Build fingerprint.

This allows different builds to coexist without intentionally overwriting the files of existing kernels.

## 🛠️ Requirements

### Supported platform

- Debian GNU/Linux.
- Debian-based distributions with compatible `dpkg`, APT and kernel packaging infrastructure.
- x86-64 / AMD64.

Compatibility depends on the installed toolchain and the requirements of the selected kernel revision.

The project does not assume that every Debian Stable release has sufficiently recent GCC, Clang, Rust or bindgen versions.

### Main dependencies

The dependency installer uses Debian packages, including:

```text
build-essential
gcc-x86-64-linux-gnu
bc
bison
flex
libssl-dev
libelf-dev
libdw-dev
libncurses-dev
dwarves
cpio
rsync
kmod
fakeroot
dpkg-dev
debhelper
git
curl
ca-certificates
gnupg
patch
python3
python3-yaml
perl
xz-utils
zstd
clang
llvm
lld
rustc
rust-src
bindgen
libclang-dev
```

Additional requirements are checked against the actual kernel source.

### Toolchain validation

The Porter checks the minimum tool versions declared by the selected kernel source, including applicable requirements for:

- GCC.
- Binutils.
- LLVM.
- Rust.
- bindgen.
- pahole.

It does not silently replace the distribution's repositories or install a third-party toolchain.

## 🚀 Installation

Clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/cachyos-debian-porter.git

cd cachyos-debian-porter
```

Make the scripts executable:

```bash
chmod +x cachyos-debian-porter.sh
chmod +x tests/*.sh
```

Install dependencies explicitly:

```bash
./cachyos-debian-porter.sh --install-deps
```

The dependency operation uses APT but does not automatically run `apt-get update` or modify repository configuration.

## 🎮 Usage

The Porter supports command-line operation and an interactive terminal menu.

### Display help

```bash
./cachyos-debian-porter.sh --help
```

### Display system information

```bash
./cachyos-debian-porter.sh --info
```

### Detect CPU and ISA

```bash
./cachyos-debian-porter.sh --detect-isa
```

### Check official updates

```bash
./cachyos-debian-porter.sh --check-updates
```

### Prepare kernel sources

```bash
./cachyos-debian-porter.sh --prepare
```

This operation performs source discovery, download verification, patch application and kernel configuration preparation.

### Compile and package

```bash
./cachyos-debian-porter.sh --build
```

### Validate generated packages

```bash
./cachyos-debian-porter.sh --validate
```

### Force recompilation

```bash
./cachyos-debian-porter.sh --build --force
```

### Select ISA

```bash
./cachyos-debian-porter.sh --build --isa 2
```

### Select compiler

```bash
./cachyos-debian-porter.sh --build --compiler gcc
```

### Limit compilation parallelism

```bash
./cachyos-debian-porter.sh --build --jobs 4
```

### Interactive interface

```bash
./cachyos-debian-porter.sh --menu
```

## ⚙️ Command-Line Reference

| Argument | Description |
| --- | --- |
| `--help` | Display help |
| `--menu` | Interactive interface |
| `--info` | Display system information |
| `--detect-isa` | Detect CPU and ISA |
| `--check-updates` | Check official CachyOS source changes |
| `--prepare` | Prepare sources, patches and configuration |
| `--build` | Compile and package |
| `--validate` | Validate the latest build |
| `--install` | Install the latest validated build |
| `--clean` | Clean temporary build files |
| `--install-deps` | Install Debian build dependencies |
| `--variant bore` | Select BORE variant |
| `--isa auto\|1\|2\|3\|4` | Select ISA |
| `--compiler auto\|gcc\|clang` | Select compiler |
| `--jobs N` | Set maximum compilation jobs |
| `--force` | Force recompilation |
| `--initramfs` | Explicitly request initramfs generation |
| `--update-grub` | Explicitly request GRUB update |

Run only one primary action per invocation.

### Configuration

The project provides:

```text
config/porter.conf
```

The configuration supports resource and build defaults.

Important parameters include:

```bash
PORTER_VARIANT=bore
PORTER_ISA=auto
PORTER_COMPILER=auto
PORTER_JOBS=auto
PORTER_MIN_FREE_GB=30
```

Parallelism considers available CPUs, memory and detected cgroup limits.

The minimum free-space requirement is configurable.

## 🔐 Installation and Boot Management

Kernel installation is optional and separate from compilation.

The Porter does not automatically:

- Remove existing kernels.
- Replace the currently running kernel.
- Change the default boot entry.
- Modify APT repositories.
- Create background services.
- Create cron jobs or systemd timers.

### Explicit installation

```bash
./cachyos-debian-porter.sh --install
```

### Installation with initramfs and GRUB

```bash
./cachyos-debian-porter.sh --install --initramfs --update-grub
```

These operations require administrative privileges.

### Secure Boot

Source PGP verification and Secure Boot kernel signing are different operations.

The Porter verifies the authenticity of the source archive, but does not automatically sign the generated kernel or modules with a key trusted by UEFI.

Systems with Secure Boot enabled need an appropriate signing and MOK enrollment procedure, or Secure Boot must be disabled through UEFI configuration.

Do not assume that a successfully generated `.deb` is automatically bootable with Secure Boot enabled.

## ♻️ Build Cache and Update Management

The Porter records information about previous builds.

The build fingerprint considers relevant inputs, including:

- Source version and revision.
- Official build recipe.
- Patch repository revision.
- Source archive checksum.
- Official kernel configuration.
- Final kernel configuration.
- ISA.
- Compiler.
- LTO configuration.

If the fingerprint matches a previously validated build, unnecessary recompilation can be avoided.

The `--force` argument requests a new build regardless of the existing cache.

### Build history

Build metadata is stored under:

```text
packages/
history/
cache/
logs/
```

The history allows previously generated kernels and their configurations to be identified.

## 🧪 Automated Testing

Run the test suite:

```bash
./tests/run-tests.sh
```

The test suite covers:

| Test | Purpose |
| --- | --- |
| Bash syntax | Check script syntax |
| ISA v1 | Baseline compatibility |
| ISA v2 | SSE4 compatibility |
| ISA v3 | AVX/AVX2 compatibility |
| ISA v4 | AVX-512 compatibility |
| Virtualized CPU | Masked CPU features |
| XCR0 validation | Operating-system AVX state |
| Resource management | CPU and memory limits |
| Recipe parsing | Version and build metadata |
| Package validation | Debian package structure |
| Manifest reload | Build metadata recovery |

Synthetic `.deb` packages are used to test package validation without requiring a complete kernel compilation.

### Validation status

The initial implementation has undergone:

- Bash syntax checks.
- Automated ISA tests.
- Synthetic package validation.
- Resource calculation tests.
- Local command-line tests.

A complete kernel compilation, installation and physical boot test have not yet been established as passing.

The test suite must not be interpreted as proof of hardware compatibility.

## 📁 Output Structure

After a successful build, the project maintains its output in:

```text
packages/
├── <KERNELRELEASE>/
│   ├── linux-image-*.deb
│   ├── linux-headers-*.deb
│   ├── build.env
│   └── SHA256SUMS
```

The exact generated package filenames depend on the selected kernel revision and build identity.

Logs are stored in:

```text
logs/
```

Temporary compilation files are stored in:

```text
build/work/
```

The `--clean` operation removes temporary build files without deleting the package archive, source cache or build history.

## ⚠️ Current Limitations

This project is an experimental porting system, not an officially supported CachyOS distribution.

Current implementation boundaries:

1. **Architecture:** x86-64 / AMD64 only.
2. **Variant:** BORE is the implemented variant.
3. **ARM64 and RISC-V:** not implemented.
4. **Other schedulers:** BMQ, RT and other variants are not implemented.
5. **External modules:** separate NVIDIA, ZFS and r8125 packaging is not implemented.
6. **Secure Boot:** automatic kernel and module signing is not implemented.
7. **Full integration testing:** physical boot validation remains necessary.
8. **Toolchain compatibility:** older Debian releases may not provide sufficiently recent build tools.
9. **Upstream changes:** unsupported changes in the CachyOS build recipe can cause preparation to abort.

The project deliberately prefers an explicit compatibility error over silently producing a kernel that differs from the intended CachyOS configuration.

## 🗺️ Roadmap

The following are development objectives, not claims of completed functionality.

- [x] Modular Bash architecture.
- [x] Official CachyOS BORE recipe discovery.
- [x] CPUID/XGETBV ISA detection.
- [x] x86-64 through x86-64-v4 selection.
- [x] Debian native package generation integration.
- [x] Synthetic package validation.
- [x] Build fingerprinting and metadata.
- [ ] Complete real-world kernel build validation.
- [ ] Physical Debian boot testing.
- [ ] Secure Boot signing integration.
- [ ] Additional CachyOS kernel variants.
- [ ] External module packaging.
- [ ] Expanded failure-injection testing.
- [ ] Continuous integration on multiple Debian releases.

## 🛠️ Troubleshooting

### Build dependencies are missing

Run:

```bash
./cachyos-debian-porter.sh --install-deps
```

Then inspect the toolchain:

```bash
./cachyos-debian-porter.sh --info
```

### ISA detection fails

Check:

```bash
lscpu
grep -m1 '^flags' /proc/cpuinfo
```

The detector also requires a working C compiler and access to the necessary CPU instructions.

### Source download fails

Check connectivity:

```bash
curl -I https://github.com/CachyOS/linux
```

Check the latest build log under:

```text
logs/
```

Do not bypass PGP verification to work around a failed download.

### Kernel configuration fails

Inspect:

```text
build/work/
logs/
```

Pay particular attention to Kconfig dependencies and minimum compiler versions.

### Compilation fails

Inspect the build log and the kernel's own error output.

Useful system diagnostics:

```bash
free -h
df -h
nproc
```

### Boot problems

Boot the previous working kernel from the bootloader.

Inspect kernel logs:

```bash
journalctl -k -b
journalctl -k -b -1
```

Do not remove the previous working kernel while troubleshooting.

## 🤝 Contributing

Contributions are welcome.

Areas that would benefit from additional development:

- Debian compatibility testing.
- Kernel configuration validation.
- CachyOS recipe tracking.
- ISA detection test coverage.
- Build failure diagnostics.
- Secure Boot integration.
- Additional kernel variants.

Before submitting changes:

1. Preserve the official CachyOS source attribution.
2. Do not silently discard upstream patches.
3. Add tests for new functionality.
4. Verify Bash syntax.
5. Document unsupported features.
6. Do not claim successful compilation without an actual build.

## 📚 References

- [CachyOS Official Website](https://cachyos.org/)
- [CachyOS Kernel Packaging](https://github.com/CachyOS/linux-cachyos)
- [CachyOS Kernel Source](https://github.com/CachyOS/linux)
- [CachyOS Kernel Patches](https://github.com/CachyOS/kernel-patches)
- [Linux Kernel Documentation](https://docs.kernel.org/)
- [Debian Kernel Handbook](https://kernel-team.pages.debian.net/kernel-handbook/)
- [Debian Packaging Documentation](https://www.debian.org/doc/manuals/maint-guide/)

## 📜 License and Attribution

The Linux kernel and CachyOS components retain their respective original licenses and copyright notices.

The Porter is an integration and packaging tool. Its redistribution must respect the licenses of all included external components.

A project-specific license should be added to this repository before claiming a single license for the entire project.

---

<p align="center">
  <strong>Built for Debian. Powered by CachyOS.</strong>
  <br>
  <sub>Bringing performance-oriented Linux kernel engineering to the Debian ecosystem.</sub>
</p>
