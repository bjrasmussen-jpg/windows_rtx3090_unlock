# Windows RTX 3090 unlock - experimental exact-card adaptation

**A Windows port already existed.** This project adapts
[impacifist/Windows-GPU-UNLOCK](https://github.com/impacifist/Windows-GPU-UNLOCK),
commit `9c541591317b00e580aec818988df8a795dd0069`, to one EVGA RTX 3090 with a
separate RTX 5070 Ti display. The underlying unlock technique and GPU core are
upstream work. This is not the first Windows implementation.

**Adaptation, test tooling and documentation were developed with GPT-6.1 Sol
in Codex.** The human operator performed the physical USB boots and manual
unlock. Original authorship, copyright and license notices are retained.

| Component | Validation status |
| --- | --- |
| Read-only identification probe | Run on this machine before any unlock attempt |
| Manual UEFI unlock | Successful on the exact hardware below; measured in Windows |
| Automatic countdown and Windows handoff | Built and host-mock tested only; not installed or validated in a real automatic boot |

## Identification first: how the IDs and board profile were obtained

The first USB application was a separate **read-only diagnostic probe**, not
the unlock application. The procedure was:

| Step | What was checked | GPU effect |
| --- | --- | --- |
| 1. Windows inventory | Enumerate both NVIDIA GPUs and PCI locations; read device/subsystem IDs, driver and board information | Read only |
| 2. Cold-boot UEFI probe | Read PCI configuration, target BDF, BAR descriptor, GPU identity/cold registers and PROM/ROM; enumerate GOP display handles and paths | Read only; no firmware payload, DMA, GPU reset or driver release |
| 3. Compare captures | Match the target board and separate display parent; derive exact guards and a PROM SHA-256 from the real capture | Local file analysis only |
| 4. Host verification | Exercise the actual app/core with the captured configuration and local firmware assets, simulated EFI/PCI/MMIO/DMA, and injected failures | No hardware access |
| 5. Manual hardware trial | Boot the guarded UEFI unlock, press U once, verify successful cleanup, then enter Windows without rebooting | One explicit unlock transaction |
| 6. Windows measurements | Run the same frozen synthetic benchmark before/after; separately inspect the other session's real Strata request counters | Measurements; no new firmware modification |

The UEFI probe wrote its report and optional PROM/ROM captures to its own USB
log directory. Those captures remain private and are **not in this repository**.
Model name and GPU index alone were not used as identity proof.

| Role | GPU | PCI vendor:device | Subsystem vendor:device | Captured PCI BDF |
| --- | --- | --- | --- | --- |
| Unlock target | EVGA RTX 3090 24 GB | `10de:2204` | `3842:3987` | `0000:05:00.0` |
| Display, kept separate | RTX 5070 Ti | `10de:2c05` | `19da:2765` | `0000:01:00.0` |

The tested board is an ASUS PRIME Z690-P WIFI with an i7-13700K. The profile
also checks BAR0 (`0x88000000`, 16 MiB), the captured PROM digest, cold state
and the actual display GOP/PCI route. Matching only the IDs, or changing those
numbers on another PC, is insufficient. This is **not a general RTX 3090 installer**.

## Speed: synthetic CUDA/cuBLAS GEMM

4096 x 4096 GEMM, three fresh processes per phase, two-second warmup and
30 samples per format/process. Every element of the final output matrix was
checked for correctness. Driver 617.42, CUDA 13.1, RTX 3090 power limit 420 W.
FP16/BF16 use FP32 accumulation; FP32 is the pedantic reference.

| Calculation | Before, TFLOP/s | After manual unlock, TFLOP/s | Change |
| --- | ---: | ---: | ---: |
| FP32 pedantic | 27.76 | 27.51 | -0.9% |
| FP16 Tensor, FP32 accumulation | 69.31 | 130.44 | +88.2% |
| BF16 Tensor, FP32 accumulation | 69.24 | 131.33 | +89.7% |

The executable and cuBLAS DLL hashes were verified identical. Baseline had an
idle model resident; after-unlock had no resident model. That condition differs.
These results are synthetic throughput, not a comparable percentage gain in
all AI workloads or a long-term stability test. See [BENCHMARK.json](BENCHMARK.json).

## Speed: approximately 10% higher Strata decode throughput observed

This is a **separate observational comparison from another local session**,
using real requests on 20 and 21 October 2026. Filter: the same long-context
workload, 70k-100k prompt tokens and at least 100 generated tokens; warm subset
requires >90% cache reuse. Decode throughput is total generated tokens divided
by total decode time, not the average of individual request rates.

| Strata selection | Before, tokens/s | After, tokens/s | Change | Requests before / after |
| --- | ---: | ---: | ---: | ---: |
| All matching requests | 92.2 | 103.3 | +12.1% | 205 / 3 |
| Warm cache (>90% reused) | 92.2 | 100.4 | +8.9% | 200 / 2 |

The after snapshot has only **three** matching requests (two warm). Content,
output length, speculative-decoding acceptance, cache reuse and the fresh engine
start differ. This does **not isolate the unlock as the cause** and does not
prove a 10% reduction in total response time. Both GPUs were used, with the
RTX 3090 connected at PCIe 4.0 x4. These observations did not use the new
autostart variant. [STRATA-OBSERVATION.json](STRATA-OBSERVATION.json) contains
aggregate counters only; raw request logs, prompts and API keys are excluded.
The source archive includes `tools/summarize_strata_observation.py` for the
aggregation method, with explicit private input paths supplied by the caller.

## Prepared automatic boot behavior

The automatic variant validates the exact card, PROM, cold state, separate
display and pinned firmware. It first preloads Windows Boot Manager from one
explicitly configured internal GPT partition. A five-second countdown replaces
U confirmation; Esc skips unlocking and boots Windows with the existing state.
Exactly one transaction is permitted. Windows starts after safe cleanup;
unsafe post-release or GPU states retain the hard stop. There is no retry,
BootOrder/BootNext rewrite, internal ESP change or GPU VBIOS flash.

This automatic variant has passed host mocks only. No automatic hardware boot
was performed or installed. Strata/Qwen configuration was left unchanged.

## Source, firmware and building

`SOURCE.zip` contains the curated source tree, bundled EDK2 headers, licenses,
host tests, benchmark source and `prepare.ps1`. Unzip it into a dedicated folder.
`host-tests.log` and [VALIDATION.json](VALIDATION.json) record source-export tests
with a dummy Windows ESP profile; UEFI, GPU and Windows execution were simulated.

**No NVIDIA firmware, generated V67 payload, GPU ROM/PROM dump, driver/CUDA DLL
or ready-to-run EFI binary is included, even inside SOURCE.zip.** Personal
filesystem paths and the real Windows partition GUID are omitted. Expected
filenames/sizes/hashes in `unlock/assets.json` are verification metadata only.
Use [FIRMWARE.md](FIRMWARE.md) for separate, pinned upstream preparation.

Building requires PowerShell, Clang/lld-link (validated with LLVM 20.1.8),
Python 3 and Visual Studio C/C++ Build Tools for host CRT headers/libraries.
Supply local pinned assets and private `PROM.BIN` / `cold-normalized.txt` fixtures.
`prepare.ps1` requires `-WindowsEspGuid`, `-WindowsEspStartLba`,
`-WindowsEspSizeLba`, `-Assets`, `-Fixtures` and `-VcVars`. Configure `-Clang`,
`-LldLink` and `-Python` if absent from PATH. The ESP profile is generated locally
in ignored `unlock/machine_profile.h`. LBA units are logical device sectors.
A dummy-profile test build must not be installed for booting. This script
builds and runs host tests locally; it does not install, reboot, mount USB/ESP
partitions or perform an unlock.

## Original projects and licensing

The direct base is [impacifist/Windows-GPU-UNLOCK](https://github.com/impacifist/Windows-GPU-UNLOCK),
building on [rough2515-hub/rtx3080-unlock-public](https://github.com/rough2515-hub/rtx3080-unlock-public)
and [Zagitta/gpu-unlock](https://github.com/Zagitta/gpu-unlock). The GPU transaction
core and SS1-preserve policy were retained. Our contribution is the exact-card
adaptation, display checks/captured tests, measurement tooling and prepared
automatic handoff. See [CHANGES.md](CHANGES.md) and [RELATED-PROJECTS.md](RELATED-PROJECTS.md).

GPU core and UEFI port: GPL-2.0-only. EDK2 headers: BSD-2-Clause-Patent. Full
licenses and original notices are retained. See LICENSE, THIRD_PARTY_NOTICES.md
and FIRMWARE-NOTICE.txt. NVIDIA firmware is obtained separately.

The source license does not settle NVIDIA software restrictions. Section 2.6
of the [NVIDIA Driver License](https://www.nvidia.com/en-eu/drivers/nvidia-license/)
restricts circumventing technical limitations in NVIDIA software. Applicability
and enforceability for this technique have not been legally assessed; this
project claims no NVIDIA approval or legal clearance.
