# LTX-2.5 LoRAs, Filters & Audio Post-Processing Options

> **Compatibility Warning:** **LTX-2.5 uses a 22B transformer**, whereas older **LTX-2** LoRAs were trained for the **19B** architecture. Always use **`LTX-2.5-22b-*`** LoRAs with LTX-2.5 checkpoints; 19B LoRAs (such as the older `LTX-2-19b-LoRA-Camera-Control-*` series) are not directly compatible with 22B weights.

---

## 1. Official LTX-2.5 Video LoRAs (Hugging Face `Lightricks` Creative Lab)

| Repo ID | Type | What It Does | Best Use Case for Our Workflow |
| :--- | :--- | :--- | :--- |
| `Lightricks/LTX-2.5-22b-IC-LoRA-Refine-Details` | V2V In-Context LoRA | Rebuilds fine textures (skin, beard, fabric), sharpens edges, and adds natural film-like micro-contrast to soft/compressed clips. | **Top pick for quality:** Run as a second-pass V2V detailer on finished talking-head clips to eliminate compression softness. |
| `Lightricks/LTX-2.5-22b-IC-LoRA-Pixel-Spatial-Upscaler` | V2V In-Context LoRA (2x) | Generatively upscales lower-res clips while inventing plausible high-frequency detail. | Useful if generating faster at lower res (e.g., `768x512`) and upscaling to `1536x1024`+. |
| `Lightricks/LTX-2.5-22b-LoRA-Cinemagraph` | Standard LoRA (I2V) | Locks the camera on a static tripod and freezes background elements while isolating motion. | Worth testing if you want zero background drift in the wine bar (though watch for stiff posture). |
| `Lightricks/LTX-2.5-22b-IC-LoRA-Layout-To-Render` | In-Context LoRA (BF16) | Converts 3D viewport playblasts/depth layouts into photorealistic video while locking camera path. | Not needed for talking-head shots; useful for complex multi-shot camera choreography. |

### How to Pass a Standard LoRA in `ltx_pipelines.distilled`
From `logs/cli_help.txt`, `distilled.py` supports `--lora PATH [STRENGTH ...]`:
```bash
--lora /content/models/loras/my_lora.safetensors 0.8
```
*(Note: **IC-LoRAs** like `IC-LoRA-Refine-Details` are Video-to-Video / In-Context adapters that take an input video/conditioning pass rather than simple T2V `--lora` stacking—best run via `LTX-2`'s IC-LoRA / V2V pipeline or ComfyUI on RunPod.)*

---

## 2. Audio Quality Options

There is currently **no separate audio-only LoRA ecosystem** for LTX-2.5 because video and audio are generated jointly in the same diffusion transformer. Best levers for cleaner, richer speech:
1. **Prompting (in-model):** Specify mic distance and room acoustics (`"off-camera boom mic, no visible microphone, dry studio-like speech, minimal room tone, no music"`).
2. **Post-processing (FFmpeg / Python, zero extra GPU cost):**
   - **Loudness normalization & vocal clarity (`ffmpeg`):**
     ```bash
     ffmpeg -i in.mp4 -c:v copy -af "highpass=f=80,acompressor=threshold=-18dB:ratio=3,loudnorm=I=-16:TP=-1.5:LRA=11" -c:a aac -b:a 192k out.mp4
     ```
   - **Neural speech cleanup (optional):** `DeepFilterNet` or `noisereduce` on the extracted audio track if a clip has faint hiss or room reverb.

---

## 3. Film Look & Color Filters

Rather than baking heavy color filters into the diffusion prompt (which can destabilize skin tones), apply a lightweight **post-process film look** in `ffmpeg` (or DaVinci Resolve / CapCut):
- **Subtle contrast + warm lounge grade + fine grain (`ffmpeg`):**
  - Apply a `.cube` 3D LUT with `lut3d=file=film_look.cube`
  - Or add subtle organic film grain (`noise=alls=4:allf=t+u`) to mask any remaining diffusion smoothness.
