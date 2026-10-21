# Related public projects

Checked on 2026-10-21. These are related implementations and upstream sources,
not evidence that this exact automatic boot variant is hardware validated.

| Project | Scope | Relation to this snapshot |
| --- | --- | --- |
| [impacifist/Windows-GPU-UNLOCK](https://github.com/impacifist/Windows-GPU-UNLOCK) | RTX 3090 Windows UEFI port, manual U and firmware-menu Windows handoff; published synthetic evidence | Direct base of this adaptation |
| [rough2515-hub/rtx3080-unlock-public](https://github.com/rough2515-hub/rtx3080-unlock-public) | Experimental exact-card MSI RTX 3080 Windows UEFI unlock | Predecessor of the 3090 port |
| [Zagitta/gpu-unlock](https://github.com/Zagitta/gpu-unlock) | Pre-driver Linux kernel module, including an RTX 3090 result | Source of the reused GPU transaction core |
| [WildFlash1st/cmp90hx-unlock-for-windows](https://github.com/WildFlash1st/cmp90hx-unlock-for-windows) | CMP 90HX Windows EFI bootloader with Windows-loader preloading | Related Windows preboot work, for a different GPU product |

The first three are explicitly credited in this repository's provenance. The
CMP project is related research, not a drop-in RTX 3090 build. Public availability
of another project does not establish legal clearance or compatibility.
