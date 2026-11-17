# LTX-2.5 Video & Audio Generation (`ttv_apps`)

Workflows, Google Colab notebooks, and deployment plans for generating synchronized Text-to-Video + Audio (T2V+A) and Image-to-Video + Audio (I2V+A) using **Lightricks LTX-2.5 (22B Distilled)**.

---

## Directory Layout

- **`README.md`** — Project overview, directory structure, and Colab workflow.
- **`GEMINI.md`** — Workspace instructions and safety/resource rules for AI assistants.
- **`docs/`**
  - [`docs/idea.md`](docs/idea.md) — Hardware requirements and Google Colab vs. RunPod comparison.
  - [`docs/colab_test_plan.md`](docs/colab_test_plan.md) — Phased Colab smoke-test plan.
  - [`docs/runpod_plan.md`](docs/runpod_plan.md) — RunPod deployment plan, GPU pricing, and per-video cost estimates.
  - [`docs/lora_and_filters_options.md`](docs/lora_and_filters_options.md) — LTX-2.5 LoRA options (detailers, camera/layout) and audio/film post-processing.
- **`notebooks/`**
  - [`notebooks/ltx25_colab_smoke_test.ipynb`](notebooks/ltx25_colab_smoke_test.ipynb) — Initial A100 smoke test + 1-hour batch loop.
  - [`notebooks/ltx25_consistent_coach_colab.ipynb`](notebooks/ltx25_consistent_coach_colab.ipynb) — First consistent-character (`--image coach_ref.jpg`) 6-clip run.
  - [`notebooks/ltx25_coach_hq_colab.ipynb`](notebooks/ltx25_coach_hq_colab.ipynb) — Latest high-quality run (CRF/bitrate boost, `0.85` + `guide` A/B conditioning, character range test, 3rd-person narrator test, zip + auto-shutdown).
- **`assets/`**
  - `coach_ref.jpg` — Reference frame (`1536x1024`) extracted from `coach_01_game_seed201.mp4` for consistent-character conditioning.
- **`runs/`**
  - Downloaded Google Drive run archives (`.zip`) and extracted `videos/` + `logs/` (`cli_help.txt`, `gpu_info.txt`, `run_summary.json`, per-clip `.log`).

---

## Google Colab Workflow

1. **Prerequisites (one-time):**
   - Accept the **LTX-2.x Community License Agreement** on [`Lightricks/LTX-2.5`](https://huggingface.co/Lightricks/LTX-2.5).
   - Add your Hugging Face read token as `HF_TOKEN` in **Colab Secrets** (key icon in the left sidebar).
2. **Open & Configure Notebook:**
   - Upload the target `.ipynb` from `notebooks/` to [Google Colab](https://colab.research.google.com) (**File → Upload notebook**).
   - Select **Runtime → Change runtime type → A100 GPU (High-RAM)**.
3. **Run Unattended:**
   - Click **Runtime → Run all**.
   - The notebook mounts Google Drive, clones `Lightricks/LTX-2`, installs `uv` + `natten`, selectively downloads only the required distilled weights, runs inference, and saves `videos/` + `logs/` into `/MyDrive/ltx25_coach_runs/<RUN_ID>.zip`.
   - The final cell flushes/unmounts Google Drive and calls `runtime.unassign()` to disconnect and delete the Colab runtime automatically.
4. **Analyze Locally:**
   - Download `<RUN_ID>.zip` from Google Drive into `runs/`, unzip it, and inspect `logs/run_summary.json` and the `.mp4` clips to iterate on prompts, conditioning, and encoding.
