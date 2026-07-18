# VYRO VR Firmware

Automated firmware builds and releases for every board sold by **VYRO VR**.

This repository does not contain firmware source code. It is a CI harness that pulls the
latest firmware from the VYRO VR source repositories, builds every product variant, and
publishes the resulting `.uf2` / `.hex` files as GitHub Releases — automatically, every
time the firmware is updated.

> 📥 **Looking for firmware?** Grab the newest files from the
> [**latest release**](../../releases/latest).

## Firmware files

| File | Product | Type | Format |
| --- | --- | --- | --- |
| `VYRO_VR_Tracker_Mochi.uf2` | Mochi tracker | Tracker | UF2 |
| `VYRO_VR_Tracker_ProMicro_Chrysalis.uf2` | Stacked Chrysalis (ProMicro) tracker | Tracker | UF2 |
| `VYRO_VR_Tracker_ProMicro_Default_I2C.uf2` | ProMicro tracker (default I2C wiring) | Tracker | UF2 |
| `VYRO_VR_Tracker_ProMicro_Default_SPI.uf2` | ProMicro tracker (default SPI wiring) | Tracker | UF2 |
| `VYRO_VR_Tracker_ProMicro_Stacked_I2C.uf2` | Stacked ProMicro tracker (I2C) | Tracker | UF2 |
| `VYRO_VR_Tracker_ProMicro_Stacked_SPI.uf2` | Stacked ProMicro tracker (SPI) | Tracker | UF2 |
| `VYRO_VR_Tracker_Styria_Mini_I2C.uf2` | Styria Mini tracker (I2C IMU) | Tracker | UF2 |
| `VYRO_VR_Tracker_Styria_Mini_SPI.uf2` | Styria Mini tracker (SPI IMU) | Tracker | UF2 |
| `VYRO_VR_Receiver_Fox_Dongle.uf2` | Fox dongle (nRF52840) | Receiver | UF2 |
| `VYRO_VR_Receiver_Fox_Dongle33.uf2` | Fox dongle (nRF52833) | Receiver | UF2 |
| `VYRO_VR_Receiver_Holyiot_21017.hex` | Holyiot 21017 dongle | Receiver | HEX |
| `VYRO_VR_Receiver_Holyiot_22046.hex` | Holyiot 22046 dongle | Receiver | HEX |
| `VYRO_VR_Receiver_Styria_R1.uf2` | Styria R1 receiver | Receiver | UF2 |
| `VYRO_VR_Receiver_ProMicro.uf2` | ProMicro receiver | Receiver | UF2 |

**UF2 files:** double-tap reset (or short RST to GND twice) to enter the bootloader, then
drag and drop the `.uf2` file onto the USB drive that appears.

**HEX files:** for boards without a UF2 bootloader (Holyiot dongles) — flash with a SWD
programmer or `nrfutil`.

## Firmware sources

| Repository | Branch | Used for |
| --- | --- | --- |
| [VYRO-VR/jitingcn-smol-slime-firmware](https://github.com/VYRO-VR/jitingcn-smol-slime-firmware) | `dev` | Tracker firmware |
| [VYRO-VR/SlimeVR-Tracker-nRF-Receiver](https://github.com/VYRO-VR/SlimeVR-Tracker-nRF-Receiver) | `dev` | Receiver firmware |

Both are built against [jitingcn/sdk-nrf](https://github.com/jitingcn/sdk-nrf) `v3.2-branch`
(nRF Connect SDK) with Zephyr SDK 0.17.4, matching each repository's own `west.yml`.

## How it works

The [build workflow](.github/workflows/build.yml) is modeled after
[Shine-Bright-Meow/SlimeNRF-Firmware-CI](https://github.com/Shine-Bright-Meow/SlimeNRF-Firmware-CI):

1. **Every 6 hours** (and on manual dispatch), the workflow resolves the current commit of
   each firmware source and SDK listed in [`boards.matrix.json`](boards.matrix.json).
2. It compares those commits against the `build-info.json` asset of the previous release.
   If nothing changed, the run stops — no duplicate releases.
3. If there is an update, every variant in the matrix is built in parallel with `west`
   (Zephyr SDK, nRF Connect SDK, and workspaces are cached between runs).
4. All firmware files are collected and published as a new GitHub Release, tagged
   `vYYYY.MM.DD-HHMM`, with release notes listing the exact source commits and a
   `build-info.json` recording the full build provenance.

Pushes and pull requests to this repository run build-only validation (no release).
A release can be forced at any time from the **Actions** tab via **Run workflow**.

## Adding or changing a build

Edit [`boards.matrix.json`](boards.matrix.json):

- **`profiles`** — an nRF Connect SDK repository + revision + Zephyr SDK version.
- **`sources`** — a firmware repository + revision, linked to a profile.
- **`variants`** — one output file: the Zephyr board target (`boardname`), the `source`
  to build from, the output `filename`, and the `fileformat` (`uf2` or `hex`).

Commit to the default branch and either wait for the next scheduled run or trigger the
workflow manually.

### Optional: `GH_TOKEN` secret

Releases are created with the workflow's built-in `GITHUB_TOKEN` by default. To publish
releases with a different identity (e.g., so they can trigger other workflows), add a
repository secret named `GH_TOKEN` containing a Personal Access Token with `contents: write`
access — the workflow will use it automatically.

## License

This CI repository is licensed under the [MIT License](LICENSE).

The firmware built here comes from its own repositories and keeps its own licensing
(the SlimeNRF / SmolSlime firmware is dual-licensed MIT / Apache-2.0). See each source
repository for details.
