# Firmware is obtained separately

This source snapshot does not contain NVIDIA firmware binaries, GPU ROM/PROM
dumps, Windows driver/CUDA DLLs, a generated V67 payload or a compiled EFI image.
Names, expected byte counts and SHA-256 digests in `unlock/assets.json` are
metadata for local verification; they are not the firmware contents.

The code reads the five assets from local files. There are no embedded NVIDIA
firmware arrays. Host fixtures generate small synthetic parser test data or
read the user's own private PROM capture; a real GPU ROM is not supplied.

For obtaining the assets locally, use the pinned upstream instructions:

- [Firmware preparation and required Python version](https://github.com/Zagitta/gpu-unlock/blob/bb1088b6b5b8cbdafa1ea69601bd118f7f317158/firmware/README.md)
- [Pinned input URLs and checksums](https://github.com/Zagitta/gpu-unlock/blob/bb1088b6b5b8cbdafa1ea69601bd118f7f317158/firmware/artifacts-610.43.03.json)
- [Firmware provenance and licensing](https://github.com/Zagitta/gpu-unlock/blob/bb1088b6b5b8cbdafa1ea69601bd118f7f317158/firmware/NOTICE.md)

Upstream `make inputs` fetches the pinned archives with SHA-256 verification;
`make firmware` generates the required files locally. Follow the linked
upstream prerequisites; its Python-based zstd extraction requires Python 3.14+.
These are preparation/build commands, not this project's install targets.
Do not run an upstream `install-*` target to prepare Windows assets.

The five resulting files must match this project's `unlock/assets.json`:

- `booter_load_ga102.bin`
- `v67-ga102-610.43.03.bin`
- `gsp_ga10x.bin`
- `gsp_boot_ga102.bin`
- `gsp_boot_desc_ga102.bin`

Keep these files in a local directory outside the repository and pass that
directory as `-Assets` to `prepare.ps1`. Keep the extracted `NVIDIA-LICENSE`
with your local assets. Real PROM/cold-state fixtures are separate private
inputs supplied through `-Fixtures`; never commit those captures.

The code and header licenses do not grant permission to redistribute NVIDIA
firmware or settle the NVIDIA software terms for the unlock technique.
For this export, all such binary assets remain outside GitHub.
