# LTX-2.5 Google Colab Test Plan

## 1. Feasibility Assessment

Based on [idea.md](file:///home/master/Software/content_maker/ttv_apps/idea.md) and Lightricks' LTX-2.5 requirements:

**Yes, we can test LTX-2.5 on Google Colab**, provided we match the model variant to the Colab GPU tier:

| Colab Tier | Typical GPU / VRAM | Feasibility | Recommended Setup |
| :--- | :--- | :--- | :--- |
| **Free** | T4 (16GB VRAM), ~12.7GB RAM, ~78–100GB disk | **Not recommended** (high OOM/disk-full risk) | Only possible with heavily quantized community GGUF (Q3/Q4) + aggressive CPU/disk offloading |
| **Colab Pro** | L4 (24GB VRAM) | **Feasible** | LTX-2.5 **Distilled** + INT8/GGUF quantized weights + quantized text encoder + offloading |
| **Colab Pro / Pro+** | A100 (40GB / 80GB VRAM) | **Recommended for Colab testing** | LTX-2.5 **Distilled** (8-step schedule), INT8/FP8 on 40GB or BF16 on 80GB |

### Key Constraints to Plan Around
1. **Gated Weights:** `Lightricks/LTX-2.5` on Hugging Face requires accepting the *LTX-2.x Community License Agreement* and authenticating with an `HF_TOKEN`.
2. **Text Encoder & VRAM Footprint:** The unquantized Gemma 4 12B text encoder alone is ~26GB. On <= 40GB VRAM, use quantized text encoder / offloading and start with the **Distilled** transformer (8-step schedule, no heavy CFG).
3. **Disk & Session Persistence:** Colab local disk is limited and ephemeral. Download **only** the specific checkpoint files needed (do not `snapshot_download` the entire repo) and save outputs to Google Drive.
4. **Runtime Dependencies:** `Lightricks/LTX-2` targets Python >= 3.12, CUDA >= 12.7, and PyTorch ~2.7 (or upstream `diffusers` + `transformers==5.14.1` + `kernels`/`natten`).

---

## 2. Step-by-Step Test Plan

### Phase 1: Prerequisites (Before Opening Colab)
1. Create or log into a Hugging Face account and accept the license on `Lightricks/LTX-2.5`.
2. Generate a read-only Hugging Face access token (`HF_TOKEN`).
3. In Google Colab, add `HF_TOKEN` to **Colab Secrets** (key icon in the left sidebar) so it is never hardcoded in notebooks.

### Phase 2: Runtime & Storage Setup (Notebook Cell 1–2)
1. Select a GPU runtime: **A100** (preferred) or **L4** (fallback).
2. Verify hardware and environment:
   - Run `nvidia-smi`, `python --version`, `df -h`, and `free -h`.
3. Mount Google Drive (`/content/drive/MyDrive/ltx_test`) for:
   - Persisting generated `.mp4` outputs.
   - Optionally caching downloaded model weights to avoid re-downloading across sessions.

### Phase 3: Dependency Installation (Notebook Cell 3)
1. Clone `https://github.com/Lightricks/LTX-2.git` (or install upstream `diffusers` main branch + `transformers`, `accelerate`, `huggingface_hub`, `safetensors`, `av`, `imageio-ffmpeg`, `kernels`).
2. Install with `uv` (preferred by `Lightricks/LTX-2`) or `pip`, matching the Colab runtime's CUDA/PyTorch version.
3. Authenticate Hugging Face Hub using `userdata.get('HF_TOKEN')`.

### Phase 4: Selective Weight Download (Notebook Cell 4)
1. Download **only** the LTX-2.5 **Distilled** checkpoint (`ltx-2.5-22b-distilled-transformer-bf16.safetensors` for A100 80GB, or INT8/quantized variant for L4/A100 40GB) plus the required Gemma 4 12B text encoder, VAE, and audio vocoder.
2. Verify remaining disk space with `df -h` before running inference.

### Phase 5: Smoke Test — Text-to-Video + Audio (Notebook Cell 5)
1. Run a **minimal smoke test** first:
   - Pipeline: Text-to-Video with synchronized audio (T2V+A).
   - Schedule: 8-step distilled inference.
   - Resolution & duration: Low/medium resolution (e.g., 512x320 or 768x512) and short duration (2–4 seconds) to validate VRAM headroom.
2. Log peak VRAM (`torch.cuda.max_memory_allocated()`) and wall-clock generation time.
3. Save the `.mp4` to Google Drive and preview it inline in the notebook.

### Phase 6: Evaluation & Next Steps
- **If the smoke test succeeds:**
  - Test a higher-resolution / longer prompt, then optionally test Image-to-Video (I2V).
  - If moving to regular production, transition to **RunPod** (`L40S 48GB` + ComfyUI + 150–200GB network volume) as outlined in [idea.md](file:///home/master/Software/content_maker/ttv_apps/idea.md).
- **If VRAM/RAM OOM occurs (e.g., on L4 24GB or T4 16GB):**
  - Enable model/text-encoder CPU offloading and switch to INT8/GGUF quantized weights.
