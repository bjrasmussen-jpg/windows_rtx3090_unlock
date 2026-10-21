# Local modifications — 2026-10-21

Based on impacifist/Windows-GPU-UNLOCK commit
9c541591317b00e580aec818988df8a795dd0069. Original authorship and license notices
are retained. Modified code files additionally carry a dated local-change notice.

- Exact EVGA RTX 3090 subsystem, PCI bus/BAR and captured PROM checks.
- Separate RTX 5070 Ti display parent/path and two-GOP validation fixtures.
- Captured EVGA ROM-parser geometry in private-input host tests.
- SS1-preserve GPU transaction core retained from the manual variant.
- Automatic arming after a cancellable five-second countdown.
- Preload the configured internal Windows loader before GPU effects; start it
  after proven cleanup; preserve hard stops on unsafe states.
- Windows ESP profile and tool/fixture locations become explicit build inputs
  in this export; machine identifiers and personal filesystem paths omitted.
- Synthetic CUDA benchmark source, full source host mocks and build/test script.
- Firmware is obtained separately using the linked pinned upstream instructions.
