# ComfyUI Colab Notebook Optimization Plan

## Summary
Clean up `ComfyUIonColab.ipynb` so it installs ComfyUI correctly, installs ComfyUI-Manager explicitly, gives users simple fill-in-the-blank model and custom-node lists, and launches through a single robust Cloudflare-only path.

## Key Changes
- Rebuild the notebook flow into: config, ComfyUI install/update, ComfyUI-Manager install/repair, model downloads, custom nodes, verification, Cloudflare launch.
- Pin the canonical path everywhere:
  - `WORKSPACE = "/root/comfy/ComfyUI"`
  - `CUSTOM_NODES_PATH = Path(WORKSPACE) / "custom_nodes"`
- Replace the previous `comfy-cli` install attempt with direct, non-interactive setup:
  - use `/root/comfy/ComfyUI` as the ComfyUI install directory
  - clone/update `https://github.com/comfyanonymous/ComfyUI.git` when the directory is missing or is a git checkout
  - accept an existing non-git ComfyUI install at `/root/comfy/ComfyUI` when `main.py` is present
  - install `requirements.txt`
  - fail early if `main.py` is missing
- Install ComfyUI-Manager as required infrastructure:
  - clone/update `https://github.com/ltdrdata/ComfyUI-Manager.git`
  - install its `requirements.txt` if present
  - verify `custom_nodes/ComfyUI-Manager` exists
  - tell users to restart ComfyUI after Manager installation or repair
- Add a repair cell for current runtimes where ComfyUI is already running without Manager.
- Keep editable lists for customization:
  - `MODEL_DOWNLOADS = [...]`
  - `CUSTOM_NODE_URLS = [...]`
  - ComfyUI-Manager is excluded from `CUSTOM_NODE_URLS` because it is always installed separately.

## Robust Cloudflare Launch
- Use one Cloudflare-only launch cell.
- Start ComfyUI from `/root/comfy/ComfyUI`.
- Add bounded waits for:
  - local ComfyUI port readiness
  - local HTTP health
  - Cloudflare tunnel URL detection
- Print recent ComfyUI/cloudflared logs when startup or URL discovery fails.
- Make clear that the launch cell remaining busy after printing the URL is normal.

## Test Plan
- Validate notebook JSON with `python3 -m json.tool ComfyUIonColab.ipynb`.
- Static-check runnable notebook source for `/root/comfy/ComfyUI`.
- Confirm Manager is installed before launch by checking `custom_nodes/ComfyUI-Manager`.
- In a clean Colab runtime, run install through launch and confirm Manager appears in the ComfyUI web UI.
- In a dirty/running runtime without Manager, run the repair cell, restart ComfyUI, and confirm Manager appears.

## Assumptions
- “ComfyUI Manager” means `https://github.com/ltdrdata/ComfyUI-Manager`.
- Manager should always be installed, independent of the custom-node URL list.
- Cloudflare remains the only tunnel method.
- Existing model and custom-node examples should stay as editable starter entries.
