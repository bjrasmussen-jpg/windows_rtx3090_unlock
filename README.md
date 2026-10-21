# Windows RTX 3090 unlock — experimental local port

Source snapshot of an exact-card Windows UEFI experiment. The preceding manual
unlock was successful on one EVGA RTX 3090 with a separate RTX 5070 Ti display.
The automatic variant is **prepared only**: built and host-mock tested, but
**not installed or tested in a real automatic boot**.

## Behavior

The automatic variant checks the target card, captured PROM digest, cold state,
display route and pinned firmware assets. It preloads Windows Boot Manager
from one explicitly configured internal GPT partition. A five-second countdown
replaces manual U confirmation; Esc skips unlocking and starts Windows with its
existing state. One transaction is allowed. Windows starts only after safe
cleanup; unsafe post-release or GPU states retain the original hard stop.
It does not rewrite BootOrder, BootNext, the internal ESP or GPU VBIOS.

This is **not a general RTX 3090 installer**. PCI paths, subsystem IDs, BAR and
PROM digest are tied to the tested board and motherboard topology. The GPU
transaction core and SS1-preserve policy are unchanged from the working manual
variant. Other cards, firmware versions and topologies have not been validated.

## Repository contents

`SOURCE.zip` contains the complete curated source tree, bundled EDK2 headers,
licenses, host tests, synthetic benchmark source and a build/test entry point.
Unzip it into a dedicated folder. Firmware binaries, GPU ROM/PROM dumps,
personal filesystem paths and the real Windows partition GUID are omitted.
No ready-to-run EFI binary or automatic USB installer is distributed here.

`host-tests.log` records the tests of the exported source with a dummy Windows
partition profile. UEFI, GPU MMIO/DMA and Windows execution are simulated.
`VALIDATION.json` records that distinction. `BENCHMARK.json` contains the earlier
manual before/after measurements; those are not an automatic-boot validation.

## Build and host tests

Requires PowerShell, Clang/lld-link (validated with LLVM 20.1.8), Python 3 and
Visual Studio C/C++ Build Tools for host CRT headers/libraries. Supply your
locally obtained pinned assets (`unlock/assets.json`) and private fixture folder
containing `PROM.BIN` and `cold-normalized.txt`. Fixtures are intentionally not
uploaded. The captured PROM must match the existing exact-card digest.

`prepare.ps1` requires explicit `-WindowsEspGuid`, `-WindowsEspStartLba`,
`-WindowsEspSizeLba`, `-Assets`, `-Fixtures` and `-VcVars` parameters. Configure
`-Clang`, `-LldLink` and `-Python` if they are not on PATH. The ESP profile is
written only to a git-ignored `unlock/machine_profile.h`. LBA values are logical
device sectors. Use a dummy GUID for host tests; a dummy-profile build must not
be installed for booting. The script only writes local build outputs and runs
host mocks. It does not install, mount USB/ESP partitions, reboot or touch GPUs.

## Earlier synthetic result

4096 x 4096 cuBLAS GEMM on RTX 3090; FP16/BF16 inputs with FP32 accumulation,
three fresh processes per phase, full output matrix checked for correctness.
Median FP16: 69.31 -> 130.44 TFLOP/s (+88.2%).
Median BF16: 69.24 -> 131.33 TFLOP/s (+89.7%).
Median FP32: 27.76 -> 27.51 TFLOP/s (-0.9%).
Driver 617.42, CUDA 13.1, 420 W power limit. Baseline had an idle model resident;
after-unlock had no resident model. These conditions differ. No Strata inference
speedup or long-term stability conclusion follows from this synthetic test.

## Attribution and license

Based on [impacifist/Windows-GPU-UNLOCK](https://github.com/impacifist/Windows-GPU-UNLOCK),
commit `9c541591317b00e580aec818988df8a795dd0069`, building on
[rough2515-hub/rtx3080-unlock-public](https://github.com/rough2515-hub/rtx3080-unlock-public)
and [Zagitta/gpu-unlock](https://github.com/Zagitta/gpu-unlock).
GPU core and UEFI port: GPL-2.0-only. EDK2 headers: BSD-2-Clause-Patent.
Original notices and full licenses are retained inside SOURCE.zip and at the
repository root. NVIDIA firmware is not bundled. See FIRMWARE-NOTICE.txt.

## Separate firmware preparation

See [FIRMWARE.md](FIRMWARE.md) for pinned upstream instructions and local-only
asset preparation. Firmware files are intentionally excluded, including from
SOURCE.zip. Expected file hashes are retained for verification.

The source license does not resolve NVIDIA software-license restrictions.
Section 2.6 of the [NVIDIA Driver License](https://www.nvidia.com/en-eu/drivers/nvidia-license/)
restricts circumventing technical limitations in NVIDIA software. Applicability
and enforceability for this technique have not been legally assessed. This
snapshot does not claim NVIDIA approval or legal clearance.
