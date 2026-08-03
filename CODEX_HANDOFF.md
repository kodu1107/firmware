# Codex handoff: OpenMANET Raspberry Pi 5 + Wio-WM6108

## Goal

Produce an SD-card image for this exact hardware combination, boot it, and then
port/fix any runtime or boot errors together with the user:

- Raspberry Pi 5 (`BCM2712`)
- Seeed WM1302 Raspberry Pi HAT used as the SPI carrier
- Seeed Wio-WM6108 Wi-Fi HaLow mini-PCIe module (`MM6108`)

The required OpenMANET target is `bcm2712_mm6108-spi`. Do not switch this build
to the MM6108 SDIO or MM8108 USB profiles.

## Repository state

- Upstream: <https://github.com/OpenMANET/firmware>
- Public build fork: <https://github.com/kodu1107/firmware>
- Working branch: `agent/rpi5-wm6108-spi`
- Pi 5 source baseline: upstream closed PR
  [#54](https://github.com/OpenMANET/firmware/pull/54), commit
  `488577b7bb37d5f56a505f740fb45855ac669a30`
- Initial build configuration commit: `398fbac8c40f055ee9e3f4da7a1bd55d19dfa955`

Do not open a pull request, add comments, or otherwise write to the upstream
OpenMANET repository unless the user explicitly requests that later.

## Changes made

`boards/ekh-bcm2712/target_diffconfig` was narrowed to the single required
device profile:

```text
CONFIG_TARGET_DEVICE_bcm27xx_bcm2712_DEVICE_bcm2712_mm6108-spi=y
```

The generic Pi 5, MM6108 SDIO, and MM8108 USB profiles/packages were removed
from this build selection. Required BCM2712, MM6108 firmware, Morse driver and
OpenMANET packages remain enabled.

`.github/workflows/build-rpi5-manual.yml` provides a manual, public GitHub
Actions build using the repository's reusable `build-firmware.yml` workflow.
The expected artifact name is `firmware-ekh-bcm2712`.

## Current GitHub Actions run

- Run: <https://github.com/kodu1107/firmware/actions/runs/30798122760>
- Workflow: `Build Raspberry Pi 5 SD images`
- Event: manual `workflow_dispatch`
- Run ID: `30798122760`
- Last checked: 2026-08-03, while `Download dependencies` was in progress

The red-X `Build Kernel` run triggered by the branch push is a separate
upstream CI workflow and is not the SD-card image build. Track the manual run
linked above.

## Continue on another PC

Sign in to GitHub as `kodu1107`, install Git and GitHub CLI if needed, then:

```powershell
gh auth login
git clone https://github.com/kodu1107/firmware.git
cd firmware
git switch agent/rpi5-wm6108-spi
gh run view 30798122760 --repo kodu1107/firmware
```

If the run succeeds, download its artifact:

```powershell
New-Item -ItemType Directory -Force ..\openmanet-pi5-artifact
gh run download 30798122760 --repo kodu1107/firmware `
  --name firmware-ekh-bcm2712 `
  --dir ..\openmanet-pi5-artifact
```

Inspect the downloaded files and SHA-256 sums. The SD-card file should be the
only image for `RPI5-MM6108-SPI` and normally ends in `sysupgrade.img.gz`.
Raspberry Pi Imager can write the `.img.gz` directly using **Use custom**; do
not copy it as a normal file onto an already formatted SD card.

If the run fails, download `build-log-ekh-bcm2712`, inspect the first relevant
compiler/build failure rather than the final cascading error, apply a focused
fix on this same branch, push it, and rerun `build-rpi5-manual.yml`. Preserve
the MM6108-SPI-only target unless evidence shows the profile itself is the
cause.

## First boot and porting plan

1. Power off the Raspberry Pi 5.
2. Install the WM1302 Pi HAT and Wio-WM6108 securely.
3. Attach the correct 868/915 MHz antenna before powering or transmitting.
4. Write the generated `.img.gz` with Raspberry Pi Imager and insert the card.
5. Connect Ethernet for initial access and capture HDMI/serial boot output if
   it does not become reachable.
6. Record the exact Pi 5 revision, SD-card model/capacity, power supply, radio
   region, LEDs, console output, and networking behavior.
7. Diagnose and fix one reproducible boot/runtime error at a time.

Do not add passwords, device keys, private BCF/calibration data, or other
secrets to this public repository or to public Actions logs.
