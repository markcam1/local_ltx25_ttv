# GEMINI.md

Project instructions and context for AI assistants working in `/home/master/Software/content_maker/ttv_apps`.

## Project Overview
This workspace is for building and testing text-to-video (T2V), image-to-video (I2V), and audio-to-video (A2V) workflows, currently centered on **Lightricks LTX-2.5** (22B audiovisual model with synchronized video and audio generation).

## Key Reference Files
- `docs/idea.md`: Hardware requirements, Google Colab vs. RunPod comparison, and recommended RunPod + ComfyUI configuration.
- `docs/colab_test_plan.md`: Phased plan for smoke-testing LTX-2.5 on Google Colab.
- `docs/runpod_plan.md`: RunPod deployment plan, GPU options, and cost estimate.
- `notebooks/`: Google Colab notebooks for smoke testing and batch/logged runs.

## Hardware & Environment Context
- **Local Machine:** RTX 4060 Ti (16GB VRAM) — below the recommended 32GB+ VRAM for full local LTX-2.5 generation; only suitable for heavily quantized/offloaded experiments.
- **Google Colab (Testing):**
  - Target **A100 (40GB/80GB)** or **L4 (24GB)** on Colab Pro. Free-tier T4 (16GB) is not recommended for baseline testing due to VRAM, system RAM, and disk constraints.
  - Always start with **LTX-2.5 Distilled** (8-step schedule) rather than the full dev checkpoint.
- **RunPod (Recommended for Production):**
  - **L40S 48GB** (or A100/H100 80GB), **ComfyUI**, **LTX-2.5 Distilled**, and a **150–200GB network volume**.

## Guidelines for Agents
- **Secrets & Auth:** `Lightricks/LTX-2.5` is a gated Hugging Face repository. Never hardcode or commit `HF_TOKEN` or other credentials; read them from environment variables or Colab Secrets (`google.colab.userdata`).
- **Resource Discipline:** When writing Colab notebooks or scripts, avoid downloading entire Hugging Face repositories (`snapshot_download` without `allow_patterns`). Download only the exact distilled/quantized weights needed and persist outputs to Google Drive or a RunPod network volume.
- **Incremental Validation:** Start inference scripts and notebooks with a short, low-resolution smoke test (e.g., 8-step distilled T2V+A) and log peak VRAM and runtime before scaling resolution or duration.
- **Colab Auto-Shutdown:** Always include a final cell in Colab notebooks to disconnect and delete the runtime (`from google.colab import runtime; runtime.unassign()`) after all outputs and logs are flushed to Google Drive, so long/unattended runs do not burn idle compute units.
