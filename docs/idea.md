Both Google Colab and RunPod can run LTX-2.5. However, I recommend RunPod for serious experimentation.

Your RTX 4060 Ti 16GB is below the recommended hardware for reliable full local generation, but you can experiment with optimized configurations.
1. Hardware requirements
Lightricks' published guidance:
Resource	Minimum guidance	Recommended
GPU VRAM	32GB+	80GB
System RAM	32GB	64GB+
Storage	100GB	200GB+
CUDA	12.7+	12.7+


LTX Documentation+1
2. Google Colab vs RunPod

Google Colab

Good for testing

- Free-tier GPU availability is not guaranteed.
- Paid tiers may provide better GPU access.
- GPU memory depends on the assigned machine.
- Session timeouts and storage persistence can be inconvenient.
- Possible to use A100-class hardware when available.
Verdict: Good for notebooks and initial experiments.

Google Colab

RunPod

My recommendation

- Choose your GPU and VRAM.
- Persistent network volumes.
- ComfyUI templates available.
- Better suited to longer generation experiments.
- Easier to build repeatable production workflows.
Verdict: Best for regular video production.

RunPod


3. Which RunPod GPU should you rent?
These are suggested configurations, not guaranteed minimums.
GPU	VRAM	Recommendation
RTX A5000	24GB	Quantized experiments
L40S	48GB	Good starting point
A100	80GB	Excellent
H100	80GB	Excellent
RTX PRO 6000	96GB	Excellent headroom

RunPod's October 2026 LTX-2.5 deployment guide recommends starting with a 48GB L40S, with 80–96GB cards preferred for more demanding workflows. 

Runpod

4. How I'd set it up
Recommended configuration
GPU

L40S 48GB


Platform

RunPod


Interface

ComfyUI


Model

LTX-2.5 Distilled


Storage

150–200GB network volume


Workflow: Text-to-video with synchronized audio, then image-to-video and audio-to-video.

Useful resources:
- Official LTX-2 repository
- Official ComfyUI instructions
- LTX-2.5 model weights
- RunPod deployment guide
One thing I'd emphasize: Start with the distilled model rather than the full development checkpoint. It uses an 8-step generation schedule and is designed for faster inference. 

Lightricks/LTX-2.5 at main

I can also put together a RunPod deployment guide with the exact ComfyUI template, GPU selection, model downloads and workflow JSON for your first synchronized video-and-audio generation.