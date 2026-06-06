# ComfyUI on Google Colab

Run ComfyUI in Google Colab with a direct ComfyUI install, required ComfyUI-Manager install, editable model/custom-node lists, and a single Cloudflare tunnel launch path.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nazdridoy/ComfyUI-On-Colab/blob/main/ComfyUIonColab.ipynb)

## Features
- Canonical ComfyUI workspace: `/root/comfy/ComfyUI`
- Direct non-interactive ComfyUI install/update at `/root/comfy/ComfyUI`
- Accepts an existing ComfyUI install at `/root/comfy/ComfyUI` when `main.py` is present
- Required `ltdrdata/ComfyUI-Manager` install before launch
- Repair cell for runtimes where ComfyUI is already running without Manager
- Fill-in-the-blank model downloads via Hugging Face specs or direct URLs
- Fill-in-the-blank custom-node Git URLs
- Cloudflare-only launch with bounded readiness checks and log output

## Quick Start
1. Open `ComfyUIonColab.ipynb` in Colab.
2. Optionally mount Google Drive. The ComfyUI install path stays `/root/comfy/ComfyUI`.
3. Edit the config cell:
   - `MODEL_DOWNLOADS` for Hugging Face or direct URL model downloads.
   - `CUSTOM_NODE_URLS` for extra custom node Git repositories.
   - Do not add ComfyUI-Manager to `CUSTOM_NODE_URLS`; it is installed separately.
4. Run `Install or Update ComfyUI`. It clones ComfyUI if missing, updates it if it is a git checkout, or validates an existing `/root/comfy/ComfyUI` install.
5. Run `Install ComfyUI-Manager`.
6. Run model downloads and custom node installs.
7. Run `Verify Install`.
8. Run `Launch ComfyUI with Cloudflare`.

The Cloudflare launch cell remains running after it prints the public URL. That is normal because the cell owns both the ComfyUI process and the tunnel.

## Model Download Entries
Use Hugging Face fields:

```python
{
    "repo_id": "black-forest-labs/FLUX.2-klein-9b-fp8",
    "filename": "flux-2-klein-9b-fp8.safetensors",
    "dest": "diffusion_models",
    "target_name": "",
}
```

Or a direct URL:

```python
{
    "url": "https://example.com/model.safetensors",
    "dest": "checkpoints",
    "target_name": "model.safetensors",
}
```

Relative `dest` values are created under `/root/comfy/ComfyUI/models`. Absolute `dest` values are used as-is.

## Custom Nodes
Add custom nodes as Git URLs:

```python
CUSTOM_NODE_URLS = [
    "https://github.com/rgthree/rgthree-comfy.git",
    "https://github.com/kijai/ComfyUI-KJNodes.git",
]
```

The notebook derives the folder name from the Git URL, updates existing repos with `git pull --ff-only`, clones missing repos, and installs `requirements.txt` when present.

## ComfyUI-Manager
ComfyUI-Manager is installed from:

```text
https://github.com/ltdrdata/ComfyUI-Manager.git
```

If you open the ComfyUI web UI and Manager is missing, run the `Repair Manager In Current Runtime` cell, then stop and restart the Cloudflare launch cell. Manager loads only during ComfyUI startup.

## Troubleshooting
- `main.py` missing: rerun `Install or Update ComfyUI`; the notebook verifies `/root/comfy/ComfyUI/main.py` before launch.
- Manager missing: run `Install ComfyUI-Manager` or the repair cell, then restart ComfyUI.
- Cloudflare URL timeout: review the printed `/tmp/comfyui.log` and `/tmp/cloudflared.log` excerpts in the launch cell output.
- Cell appears stuck after URL prints: leave it running; interrupt it only when you want to stop the server.
- Hugging Face auth errors: set `HF_TOKEN`, `HUGGINGFACE_TOKEN`, or fill `HF_TOKEN` in the config cell.

## License
This repository is licensed under the AGPL-3.0. See [`LICENSE`](./LICENSE).
