# wolfBoot Target Selection and Cross-Compilation

## Selecting a Target

wolfBoot uses a `.config` file in the repository root to specify the target hardware. The
`config/examples/` directory contains ready-to-use configurations for 60+ supported targets.

Copy the example that matches your hardware:

```bash
cp config/examples/stm32h7.config .config
make keytools
make
```

To see all available example configs:

```bash
ls config/examples/*.config
```

To interactively generate a new `.config` from scratch:

```bash
make config
```

## Target Families

### STM32 (ARM Cortex-M)

The most extensively supported family. HAL implementations in `hal/stm32*.c`.

Representative configs:
- `stm32f1.config`, `stm32f407-discovery.config`, `stm32f7.config`
- `stm32h7.config`, `stm32h5.config`, `stm32h5-tz.config` (TrustZone)
- `stm32l4-cube.config`, `stm32l5.config`, `stm32u5.config`
- `stm32wb.config`, `stm32g0.config`, `stm32c0.config`
- `stm32n6.config` (Cortex-M55)

Many STM32 configs have `-dualbank`, `-tz`, `-lms`, and `-tpm` variants.

Cross-compiler: `arm-none-eabi-gcc`

### NXP (i.MX RT, LPC, Kinetis, S32K, QorIQ)

HAL implementations cover both Cortex-M and PowerPC NXP families.

Representative configs:
- `imx-rt1060.config`, `imx-rt1050.config` (Cortex-M7)
- `lpc55s69.config`, `lpc55s69-tz.config`
- `kinetis-k64f.config`, `kinetis-k82f.config`
- `nxp-s32k144.config`, `nxp-s32k148.config`
- `nxp-t1024.config`, `nxp-t2080.config`, `nxp-ls1028a.config` (PowerPC/ARM64 SoCs)

Cross-compilers: `arm-none-eabi-gcc` (Cortex-M), `powerpc-linux-gnu-gcc` (QorIQ)

### Nordic Semiconductor (nRF)

HAL in `hal/nrf52.c`, `hal/nrf5340.c`, `hal/nrf54l.c`.

Representative configs:
- `nrf52840.config`
- `nrf5340.config`, `nrf5340-tz.config`
- `nrf54l15.config`, `nrf54l15-wolfcrypt-tz.config`

Cross-compiler: `arm-none-eabi-gcc`

### Microchip (SAM, PIC32)

HAL in `hal/same51.c` (via STM32 HAL pattern), IDE projects in `IDE/MPLAB/`.

Representative configs:
- `same51.config`, `same51-dualbank.config`
- `pic32ck.config`, `pic32cz.config`

### Renesas (RX, RZ)

IDE project files in `IDE/Renesas/e2studio/`. HAL in `hal/renesas*.c`.

Representative configs:
- `renesas-rx65n.config`, `renesas-rx72n.config`

### Raspberry Pi / Broadcom

HAL in `hal/raspi.c`.

Representative configs:
- `raspi3.config`, `raspi3-encrypted.config`

### RISC-V

HAL in `hal/hifive1.c`, `hal/mpfs250.c`.

Representative configs:
- `hifive1.config` (SiFive HiFive1)
- `polarfire_mpfs250.config` (Microchip PolarFire SoC)

Cross-compiler: `riscv64-unknown-elf-gcc` or `riscv32-unknown-elf-gcc`

### PowerPC (NXP QorIQ, NXP P-series)

HAL in `hal/nxp_ppc.c`, `hal/nxp_t1024.c`, etc.

Representative configs:
- `nxp-p1021.config`, `nxp-t1024.config`, `nxp-t1040.config`, `nxp-t2080.config`

### x86-64 (EFI, FSP/QEMU)

wolfBoot can run as a UEFI application or as a pre-OS FSP-based bootloader.

Representative configs:
- `x86_64_efi.config` — UEFI application
- `x86_fsp_qemu.config` — FSP on QEMU (runnable without hardware)

### QEMU Simulator (x86 FSP)

The x86 FSP QEMU target can be tested without real hardware using QEMU:

```bash
cp config/examples/x86_fsp_qemu.config .config
make
# Then run with QEMU (see docs/Loader.md for the QEMU command line)
```

### Simulator (Linux/macOS native)

`TARGET=sim` compiles wolfBoot as a native process for development and CI. See
[build-native.md](build-native.md) for details.

## Cross-Compilation

wolfBoot auto-selects the toolchain based on `ARCH` and `TARGET`. You can override:

```bash
make CROSS_COMPILE=arm-none-eabi-
```

The `CROSS_COMPILE` prefix is prepended to `gcc`, `objcopy`, `size`, etc.

Typical cross-compilers by architecture:

| Architecture | Toolchain prefix |
|---|---|
| ARM Cortex-M (32-bit) | `arm-none-eabi-` |
| ARM Cortex-A (64-bit) | `aarch64-none-elf-` or `aarch64-linux-gnu-` |
| RISC-V 32-bit | `riscv32-unknown-elf-` |
| RISC-V 64-bit | `riscv64-unknown-elf-` |
| PowerPC | `powerpc-linux-gnu-` |
| x86-64 EFI | `x86_64-w64-mingw32-` or native gcc |

## Verifying a Target Config

```bash
cp config/examples/<target>.config .config
make check_config
```

`check_config` validates that required parameters (partition addresses, sizes, sign algorithm)
are all set consistently before attempting a build.

## IDE Targets

For IDE-based workflows (IAR, MPLAB X, Code Composer Studio, Renesas e2Studio, AURIX):

```
IDE/IAR/           IAR Embedded Workbench project for STM32
IDE/MPLAB/         MPLAB X project for SAM E51
IDE/CCS/           Code Composer Studio project for TMS570
IDE/Renesas/       e2Studio projects for RX and RZ
IDE/AURIX/         AURIX Development Studio projects for TC3xx
IDE/pico-sdk/      Raspberry Pi Pico SDK projects for RP2350
```

See the `README.md` in each IDE subdirectory for setup instructions.
