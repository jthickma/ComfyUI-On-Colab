# ComfyUI on Google Colab

Run ComfyUI in Google Colab with optional Google Drive persistence, one-click model downloads, and public access via Cloudflare Tunnel or Localtunnel.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nazdridoy/ComfyUI-On-Colab/blob/main/ComfyUIonColab.ipynb)

---

## Features
- Mount or unmount Google Drive to persist your ComfyUI workspace
- Clone and optionally update the upstream ComfyUI repository
- Download the configured Hugging Face models with authenticated `hf download`
- Clone/update the configured custom nodes and install their requirements
- Colab-oriented cache, transfer, and low-VRAM launch defaults
- Start ComfyUI and expose it publicly via:
  - Cloudflare Tunnel (trycloudflare)
  - Localtunnel (fallback)

## Quick start
1) Click the Colab badge above to open `ComfyUIonColab.ipynb`.
2) In the first cell, set `MODE` to `MOUNT` and run to mount Google Drive (recommended for persistence).
3) (Optional) Set `DRIVE_PATH` in the Setup cell, e.g. `/content/drive/MyDrive`. If set, the workspace will be created at `<DRIVE_PATH>/ComfyUI` and will persist across sessions. If left empty, the workspace is `/content/ComfyUI` (ephemeral).
4) Run the Setup cell to clone/update ComfyUI and install dependencies.
5) Add a Colab user secret named `HF_TOKEN` with access to the requested Hugging Face repositories.
6) Use the Models section to download the diffusion model, GGUF text encoder, and VAE into the correct ComfyUI model folders.
7) Run the Custom Nodes section to clone/update the listed nodes and install any `requirements.txt` files.
8) Start ComfyUI:
   - Cloudflare Tunnel cell prints a public URL ending with `trycloudflare.com` once port 8188 is ready
   - Or use the Localtunnel cell as an alternative

## Notebook structure (what each section does)
- Mount/Unmount Google Drive
  - `MODE = "MOUNT" | "UNMOUNT"` mounts to `/content/drive` and cleans up when unmounting
- Setup and Update ComfyUI
  - `DRIVE_PATH` (string, optional). When provided, workspace is set to `<DRIVE_PATH>/ComfyUI`
  - `COMFYUI_LAUNCH_ARGS` defaults to `--dont-print-server --lowvram --disable-auto-launch` for Colab runtimes
  - Clones `https://github.com/comfyanonymous/ComfyUI` if not present
  - If `UPDATE_COMFY_UI = True`, runs `git pull`
  - Installs dependencies with `pip install xformers!=0.0.18 -r requirements.txt` and CUDA wheels extra-indexes
- Models download helpers
  - Installs `huggingface_hub[cli]` and `hf_transfer`
  - Uses authenticated `hf download` with `HF_TOKEN` from Colab user secrets:
    ```python
    from google.colab import userdata
    HF_TOKEN = userdata.get("HF_TOKEN")
    ```
  - Configured model targets:
    - `Dervlex/Private` / `VeniceV2_bf16.safetensors` -> `./models/diffusion_models`
    - `ponpoke/flux2-klein-9b-uncensored-text-encoder` / `flux2-klein-9b-uncensored-q8_0.gguf` -> `./models/text_encoders`
    - `black-forest-labs/FLUX.2-klein-9B` / `vae/diffusion_pytorch_model.safetensors` -> `./models/vae/FLUX.2-klein-9B-vae.safetensors`
- Custom nodes
  - Clones into `/content/ComfyUI/custom_nodes`, updates existing clones with `git pull --ff-only`, and installs each node's `requirements.txt` when present
  - Included nodes: `rgthree-comfy`, `ComfyUI-KJNodes`, `ComfyUI-GGUF`, `ComfyUI-Easy-Use`, `ComfyUI-Crystools`, and `ComfyUI-Lora-Manager`
- Start and expose ComfyUI
  - Cloudflare: downloads `cloudflared` `.deb`, starts tunnel, and prints a `trycloudflare.com` URL when port 8188 is ready
  - Localtunnel: global `lt` install and public URL; also prints your endpoint IP for password prompt if required

## Persistence tips
- Recommended `DRIVE_PATH`: `/content/drive/MyDrive`
- With Drive mounted and `DRIVE_PATH` set, your ComfyUI install, models, and custom nodes live in Drive and survive runtime restarts
- Without Drive, everything under `/content` is temporary and will be lost when the Colab session ends
- For fastest installs, leave `DRIVE_PATH` empty and redownload into `/content`; Drive persistence trades speed for convenience
- Hugging Face transfer cache is kept under `/content/hf-cache` to avoid slow Drive metadata operations
- `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True,max_split_size_mb:256` is set to reduce CUDA memory fragmentation

## Troubleshooting
- Cloudflared hangs or prints no URL:
  - Re-run the cell after ComfyUI fully starts, or use the Localtunnel cell as fallback
- Hugging Face auth errors:
  - Ensure `HF_TOKEN` is set in Colab user secrets, has access to the private/gated repos, and any required model terms are accepted
- GGUF text encoder does not load:
  - Run the Custom Nodes section so `ComfyUI-GGUF` is installed, then restart ComfyUI
- Out-of-memory or slow starts:
  - Keep `--lowvram` in `COMFYUI_LAUNCH_ARGS`, avoid heavy custom nodes, or try a high-VRAM Colab runtime

## Folder layout (within the workspace)
- `./models/diffusion_models` – diffusion model weights
- `./models/text_encoders` – GGUF/text encoder weights
- `./models/vae` – VAE files
- `./custom_nodes` – optional custom node repos

## License
This repository is licensed under the AGPL-3.0. See [`LICENSE`](./LICENSE).

The underlying ComfyUI project is developed by the ComfyUI authors; please refer to their repository and license for details.

## Acknowledgements
- ComfyUI by comfyanonymous
- Community model authors on Hugging Face
