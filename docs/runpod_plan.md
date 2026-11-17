# LTX-2.5 RunPod Production & Testing Plan

## 1. Goal & Why Move to RunPod
Move from ephemeral Google Colab sessions (where ~50GB+ of weights must be re-downloaded on every new runtime) to **RunPod** with a persistent **Network Volume**, enabling:
- **Zero re-download wait:** Weights (`Lightricks/LTX-2.5` distilled + optional dev/quantized checkpoints) stay on the Network Volume at `/workspace/models`.
- **Higher quality experimentation:** Test both `ltx_pipelines.distilled` (fast 8-step + 3-step) and higher-quality/higher-bitrate pipelines (e.g., full 2-stage dev pipeline or ComfyUI workflows) without Colab idle timeouts.
- **On-demand billing:** Spin up a GPU pod only while generating, then **terminate** the pod (keeping only the Network Volume).

---

## 2. Available RunPod GPU Options & Pricing

*(On-demand rates fluctuate slightly by region/supply; Secure Cloud is RunPod-operated data centers, Community Cloud is third-party hosts.)*

| GPU | VRAM | Community Cloud ($/hr) | Secure Cloud ($/hr) | Fit for LTX-2.5 |
| :--- | :--- | :--- | :--- | :--- |
| **L40S** | 48 GB | ~$0.79/hr | ~$1.09/hr | **Best value starting point** for LTX-2.5 Distilled + ComfyUI |
| **A100 (PCIe / SXM)** | 80 GB | ~$1.19 – $1.39/hr | ~$1.49 – $1.79/hr | **Best for quality/speed** (full `bf16`, zero offloading, fast batch runs) |
| **H100 (PCIe / SXM)** | 80 GB | ~$1.99 – $2.69/hr | ~$2.89 – $3.99/hr | Fastest throughput for heavy production batches |

### Storage Pricing & Do You Need a Network Volume?
- **For a one-off 1–2 hour test:** You do **not** strictly need a Network Volume. You can deploy a single pod with a `100 GB` Pod Volume (`~$0.10/GB/month` prorated = a few cents for 2 hours), run your test, download the `.zip`, and **Terminate** the pod. The downside is that the `~50 GB` of LTX-2.5 weights and `LTX-2` environment are erased on termination, so you pay ~15 minutes of GPU setup time (`~$0.35`) if you spin up again later.
- **For regular use (>2–3 runs/month):** Use a **Network Volume (< 1 TB): `$0.07 / GB / month`** (portable across pods; persists when pods are terminated).
  - `150 GB` Network Volume = **$10.50 / month** (`~$0.35/day`)
  - `200 GB` Network Volume = **$14.00 / month**
- **Pod Volume Disk (avoid for idle storage):** `$0.10 / GB / month` while running, **$0.20 / GB / month** when stopped. Always **terminate** (don't just stop) pods when finished.
- **Container Disk (ephemeral):** `$0.10 / GB / month` while running (erased on terminate). Keep small (e.g., `20–30 GB`).

### Measured A100 80GB Benchmarks (from Colab `run_v3_20261115_010646`)
- **Throughput:** `15` clips (`193`–`241` frames) in `1987.77s` (`~132.5s` / clip average; `$0.8227` at `$1.49/hr`).
- **Peak VRAM:** `47.8 GB` at `896x1536` (`9:16`, `193f`); `52.9 GB` at `1536x1024` (`193f`); **`64.3 GB`** at `1536x1024` (`241f` / 10s).
  - Because `241`-frame full-res clips peak at `64.3 GB`, **A100 80GB** is required for full resolution at 10s; L40S 48GB must step down to `1152x768` / `768x1344` for 241-frame clips.
- **Required Runtime Settings:** Patch `media_io/encode.py` (`_SDR_CRF = 14`, `_SDR_PRESET = "medium"`) for `3–5.5 Mbps` output, and set `MPLBACKEND=Agg` in the environment if invoking `ltx_pipelines.ic_lora`.

---

## 3. Cost Estimate (Example Scenarios)

Assuming a **150 GB Network Volume** (`$10.50/mo`) + **10 hours/month** of active generation (and ~2 min per 5–7s distilled clip on A100 80GB, ~2.5–3 min on L40S):

| Setup | Storage / mo | 10 hrs GPU / mo | 25 hrs GPU / mo | Est. Cost / Video (GPU only) | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **L40S 48GB (Community)** | $10.50 | ~$7.90 (**~$18.40 total**) | ~$19.75 (**~$30.25 total**) | **~$0.03 – $0.04** | Great budget option |
| **L40S 48GB (Secure)** | $10.50 | ~$10.90 (**~$21.40 total**) | ~$27.25 (**~$37.75 total**) | **~$0.05 – $0.06** | Reliable availability |
| **A100 80GB (Community)** | $10.50 | ~$11.90–$13.90 (**~$22–$24 total**) | ~$29.75–$34.75 (**~$40–$45 total**) | **~$0.04 – $0.05** | Matches Colab A100 80GB headroom |
| **A100 80GB (Secure)** | $10.50 | ~$14.90–$17.90 (**~$25–$28 total**) | ~$37.25–$44.75 (**~$48–$55 total**) | **~$0.05 – $0.06** | Best reliability + 80GB VRAM |

*Note: Including amortized Network Volume storage (`$10.50/mo`), effective cost per video depends on monthly volume (e.g., at 300 videos/month on A100 Community, ~$10.50/300 = +$0.035/video, or **~$0.08/video all-in**).*

---

## 4. Step-by-Step Deployment Plan

### Phase 1: Create Network Volume (One-Time)
1. In RunPod Console → **Storage → Network Volumes → + New Network Volume**.
2. Choose **150 GB** (or **200 GB** if you plan to keep both Distilled + Full Dev checkpoints + outputs).
3. Pick a region with good **L40S** and **A100 80GB** availability (e.g., `US-TX`, `US-CA`, or `EU-RO` depending on live stock).

### Phase 2: Launch Pod & Bootstrap Environment (One-Time on Volume)
1. Deploy a pod attached to your Network Volume (mount path `/workspace`):
   - Template: **RunPod PyTorch** (CUDA 12.7+ / Python 3.12+) or a **ComfyUI** template.
   - Container Disk: `25 GB` (ephemeral).
2. SSH or open JupyterLab on `/workspace` and set up `LTX-2` + `natten` directly on the Network Volume:
   ```bash
   cd /workspace
   git clone https://github.com/Lightricks/LTX-2.git
   cd /workspace/LTX-2
   pip install -q uv
   uv python install 3.12
   uv sync --python 3.12
   uv sync --python 3.12 --inexact --package ltx-core --extra natten
   ```
3. Export `HF_TOKEN` (never commit it) and download only the needed weights into `/workspace/models/ltx-2.5` (same selective list as `notebooks/ltx25_coach_colab.ipynb`).

### Phase 3: Quality Optimization & Production Runs
1. **CLI / Pipeline tuning (using logs from Colab `ltx25_coach_colab.ipynb`):**
   - Increase video encode bitrate / lower CRF if `ltx_pipelines` supports it (or re-encode / export higher-bitrate intermediate frames).
   - Compare **Distilled** vs. **Full 2-Stage Dev** checkpoint on A100 80GB for talking-head skin texture and micro-expressions.
2. **ComfyUI (Optional Visual Workflow):**
   - Point ComfyUI's `extra_model_paths.yaml` to `/workspace/models/ltx-2.5` so CLI scripts and ComfyUI share the exact same weights without duplicating storage.
3. **Teardown:**
   - Download finished `.mp4` files from `/workspace/outputs/` (or sync to S3/Drive).
   - **Terminate** the Pod (not just Stop) so you only pay `$0.07/GB/month` for the Network Volume.
