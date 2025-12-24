# ComfyUI-on-RX-9060-XT-RDNA-4---Docker-Setup-Guide
This guide documents how to run ComfyUI on the AMD Radeon RX 9060 XT (RDNA 4 / gfx1200) using Docker on Linux (Fedora/Ubuntu).


**The Problem:** RDNA 4 cards require ROCm 7.0+ libraries, which are not yet available as of now 2025 in standard Linux repositories or the default Docker images. The Solution: We use the standard ROCm Docker container but manually upgrade the internal PyTorch version to the "Nightly" build specifically compiled for gfx120x (RDNA 4).
