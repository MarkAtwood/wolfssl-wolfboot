# wolfBoot Native Build Guide

## Build System Overview

wolfBoot uses **GNU Make** as its primary build system. A `.config` file in the repository root
controls all build parameters (target MCU, flash layout, signature algorithm, etc.). Command-line
`make` variables override `.config` values.

**CMake** is also supported as an alternative, mainly for host-native builds and CI integration.
See [docs/CMake.md](../docs/CMake.md) for CMake-specific details.

## Dependencies

wolfBoot has minimal dependencies:

- **GNU Make** and a suitable C compiler (GCC or Clang)
- **wolfssl** — included as a git submodule at `lib/wolfssl`. Always initialize submodules first:
  ```bash
  git submodule update --init
  ```
- Optional submodules in `lib/`: wolfTPM, wolfHSM, wolfPKCS11, wolfPSA, wolfHAL. These are only
  needed when the corresponding feature is enabled in `.config`.
- **Python 3** with `wolfcrypt-py` — required only for the Python signing tool
  (`tools/keytools/sign.py`). The C keytools (`tools/keytools/keygen` and `tools/keytools/sign`)
  work without Python.

## Simulator Target (Recommended for Development)

The simulator target (`TARGET=sim`) runs wolfBoot as a native Linux/macOS process. It is the
fastest path to verify a configuration change or develop new features without hardware.

```bash
git submodule update --init
cp config/examples/sim.config .config
make keytools          # builds keygen and sign tools
make                   # builds wolfboot.bin and a signed test-app image
```

The `factory.bin` output concatenates wolfboot and the signed firmware image, matching what
would be flashed to a real device.

## Key Make Targets

| Target | Description |
|---|---|
| `make` (or `make all`) | Build wolfBoot and the signed test application |
| `make keytools` | Build the C keygen and sign tools under `tools/keytools/` |
| `make wolfboot.bin` | Build only the bootloader binary |
| `make factory.bin` | Build the combined flash image (bootloader + signed firmware) |
| `make keys` | Generate a new signing keypair (type set by `SIGN=` in `.config`) |
| `make clean` | Remove build artifacts (keeps keys) |
| `make keysclean` | Remove generated key files |
| `make distclean` | Remove all generated files including keys |
| `make config` | Interactive wizard to generate a new `.config` |
| `make check_config` | Validate the current `.config` |
| `make cppcheck` | Run cppcheck static analysis |

## Unit Tests

Unit tests run on the host (no hardware or cross-compiler needed):

```bash
make -C tools/unit-tests run
```

Individual tests can be built and run directly from `tools/unit-tests/`. The tests cover
firmware image parsing, NVM update logic, delta updates, encryption, flash sector flags,
and more. See `tools/unit-tests/README.md` for details.

## CMake Alternative

```bash
mkdir build && cd build
cmake -DWOLFBOOT_TARGET=sim \
      -DWOLFBOOT_PARTITION_BOOT_ADDRESS=0x80000 \
      -DWOLFBOOT_PARTITION_SIZE=0x40000 \
      -DWOLFBOOT_PARTITION_UPDATE_ADDRESS=0x100000 \
      -DWOLFBOOT_PARTITION_SWAP_ADDRESS=0x180000 \
      -DBUILD_TEST_APPS=yes ..
cmake --build .
```

CMake Presets are defined in `CMakePresets.json` for common configurations. See
`docs/CMake.md` for the full list of CMake variables.

## Build Variables

Common variables that can be passed on the make command line or set in `.config`:

| Variable | Description |
|---|---|
| `TARGET` | Hardware target (e.g., `sim`, `stm32h7`, `nrf52840`) |
| `ARCH` | Architecture (`ARM`, `AARCH64`, `RISCV`, `PPC`, `x86_64`, `sim`) |
| `SIGN` | Signature algorithm (`ED25519`, `ECC256`, `RSA2048`, `ML_DSA`, etc.) |
| `HASH` | Hash algorithm (`SHA256`, `SHA384`, `SHA3`) |
| `DEBUG` | Set to `1` to enable debug output |
| `NO_ASM` | Set to `1` to disable assembly optimizations (smaller but slower) |
| `WOLFBOOT_SMALL_STACK` | Set to `1` for targets with very limited stack |

## Output Files

After a successful build:

- `wolfboot.bin` — the bootloader binary to flash at address 0
- `wolfboot.elf` — ELF with debug symbols
- `test-app/image_v1_signed.bin` — signed firmware image for the primary partition
- `factory.bin` — combined image ready to flash (bootloader + firmware)
- `wolfboot_signing_private_key.der` — generated private key (keep secure, never commit)
