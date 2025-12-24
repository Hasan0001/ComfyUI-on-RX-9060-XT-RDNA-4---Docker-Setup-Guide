# ComfyUI-on-RX-9060-XT-RDNA-4---Docker-Setup-Guide
This guide documents how to run ComfyUI on the AMD Radeon RX 9060 XT (RDNA 4 / gfx1200) using Docker on Linux (Fedora/Ubuntu).


**The Problem:** RDNA 4 cards require ROCm 7.0+ libraries, which are not yet available as of now 2025 in standard Linux repositories or the default Docker images. The Solution: We use the standard ROCm Docker container but manually upgrade the internal PyTorch version to the "Nightly" build specifically compiled for gfx120x (RDNA 4).


**Prerequisites**
  Docker installed and running.
  GPU drivers installed on the host.

**Step 1: Host System Permissions**
Before starting the container, we must ensure Docker has permission to read/write to the external drive where ComfyUI is stored.
(Run these on your host machine)

  Bash

  # 1. Set the target directory variable
  export COMFY_DIR="/run/media/hasan/ComfyUI/native_comfy"

  # 2. Fix SELinux context (Crucial for Fedora)
  sudo chcon -Rt svirt_sandbox_file_t "$COMFY_DIR"

  # 3. Ensure read/write permissions
  chmod -R 777 "$COMFY_DIR"

**Step 2: Create the Persistent Container**
We create a container named comfy-rdna4 that runs in the background. We map the ComfyUI folder to /workspace.

Bash

  docker run -d --name comfy-rdna4 \
    --restart unless-stopped \
    --device=/dev/kfd --device=/dev/dri --group-add=video \
    -p 8188:8188 \
    -v "$COMFY_DIR":/workspace \
    -e HSA_OVERRIDE_GFX_VERSION=12.0.0 \
    rocm/pytorch:latest \
    tail -f /dev/null
  
  Note: HSA_OVERRIDE_GFX_VERSION=12.0.0 is the recommended override for RDNA 4 if native detection fails.

**Step 3: Install RDNA 4 Drivers (One-Time Setup)**
  We need to replace the standard PyTorch version inside the container with the RDNA 4 compatible version.

  1. Enter the container shell:

  Bash
  docker exec -it comfy-rdna4 /bin/bash

2. Run these commands INSIDE the container:

  Bash

  # A. Uninstall the incompatible stable version
    pip uninstall -y torch torchvision torchaudio

  # B. Install the RDNA 4 Nightly version (Specific Index URL)
    pip install --pre torch torchvision torchaudio --index-url https://rocm.nightlies.amd.com/v2/gfx120X-all/

  # C. Install ComfyUI dependencies
    pip install -r requirements.txt

  # D. Install missing dependencies for Custom Nodes (Fixes Impact Pack crashes)
    pip install scikit-image

**Step 4: Usage**
Once setup is complete, you can launch ComfyUI anytime using this single command:

  Bash
  docker exec -it comfy-rdna4 python main.py --listen
