# ComfyUI_Rebels_LingBot — LoRA Fork (30B-3B & 1.3B)

> Fork of [RealRebelAI/ComfyUI_Rebels_LingBot](https://github.com/RealRebelAI/ComfyUI_Rebels_LingBot)
> adding a **LoRA Loader** node. Everything else (loaders, structured-prompt node, sampler,
> install steps) is unchanged — see the original README for the full reference. This repo:
> [carlos13332/ComfyUI_Rebels_LingBot](https://github.com/carlos13332/ComfyUI_Rebels_LingBot).

ComfyUI custom nodes for **LingBot-Video** (Robbyant) — text-to-video and text+image-to-video
on consumer GPUs. Built and tested on an **RTX 3070 8GB / 16GB RAM**.

Two model families are supported:
- **LingBot-Video-Dense-1.3B**
- **LingBot-Video-MoE-30B-A3B** (run from GGUF)

## What's new in this fork: LoRA support

A new **LingBot LoRA Loader** node goes in between the loader and the sampler:

```
LingBot Loader (1.3B)  or  LingBot 30B MoE Loader (GGUF)  ─►  LingBot LoRA Loader  ─►  LingBot Sampler
```

- Works with **both** model families — it detects which one it received and applies the LoRA
  the right way for each.
- Reads standard LoRA `.safetensors` files (the common formats used across the community).
- You can chain more than one `LingBot LoRA Loader` node to combine multiple LoRAs.
- `strength = 0` removes the LoRA's effect entirely.

**Tested with:** [lightx2v/LightLingBot-Video](https://huggingface.co/lightx2v/LightLingBot-Video),
a 4-step distilled LoRA for the **30B-A3B MoE** model. At the time of writing, lightx2v has only
published a distill LoRA for the 30B model — not for the 1.3B dense model.

If you use that LoRA, use its recommended settings instead of the base model's defaults:

| Setting | Base 30B model | LightLingBot-Video (4-step LoRA) |
|---|---|---|
| `steps` | 40 | **4** |
| `guidance` | 3.0 | **1.0** |
| `shift` | 3.0 | 3.0 |
| LoRA `strength` | — | 1.0 |
| Resolution | 832×480 | 832×480 |

## Nodes

| Node | What it does |
|---|---|
| **LingBot Structured Prompt (JSON caption)** | Builds the caption schema the model was trained on. |
| **LingBot Text Encode (lazy Qwen3-VL)** | Encodes prompt + negative, then frees the encoder. |
| **LingBot Loader (1.3B dense)** | Builds the 1.3B DiT. |
| **LingBot 30B MoE Loader (GGUF)** | Loads the 30B-A3B MoE from a GGUF. |
| **LingBot LoRA Loader** *(new in this fork)* | Applies a LoRA to the output of either loader above. |
| **LingBot Sampler** | Denoises and decodes. Works with both loaders, with or without a LoRA attached. |

See the [original README](https://github.com/RealRebelAI/ComfyUI_Rebels_LingBot) for prompting
guidance, install steps, and the full settings/known-issues reference — none of that changed
in this fork.

## Install

Same as upstream, plus your LoRA file(s) if you use one:

1. Clone into `ComfyUI/custom_nodes/`.
2. `python_embeded\python.exe -m pip install -U diffusers transformers accelerate safetensors einops gguf`
   (for the 30B loader, also install **city96's ComfyUI-GGUF** next to this pack).
3. Download the weights into:
   - 1.3B: `models/diffusion_models/LingBot_1.3b_DiT.safetensors`
   - 30B: `models/diffusion_models/LingBot-Video-30B-A3B-Q3_K_M.gguf` (or another tier)
   - `models/vae/LingBot_vae.safetensors`
   - `models/text_encoders/LingBot_text-encoder.safetensors`
   - LoRA(s): `models/loras/*.safetensors`
4. **For the 30B loader only:** copy the 30B repo's `transformer/config.json` into
   `custom_nodes/ComfyUI_Rebels_LingBot/model_assets/transformer_config_30b.json`.
5. Restart ComfyUI.

## Credits

- **Model & pipeline:** [Robbyant / lingbot-video](https://github.com/Robbyant/lingbot-video)
  (Apache 2.0)
- **4-step distill LoRA:** [lightx2v / LightLingBot-Video](https://huggingface.co/lightx2v/LightLingBot-Video)
  (Apache 2.0)
- **Original ComfyUI nodes & GGUF quantization:** [RealRebelAI](https://github.com/RealRebelAI/ComfyUI_Rebels_LingBot)
  (MIT)
- **GGUF dequant kernels:** city96 (ComfyUI-GGUF)
- **Wan VAE:** diffusers `AutoencoderKLWan`
- **ComfyUI:** [comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- **LoRA Loader node (this fork's addition):** implemented with [Claude](https://claude.com)
  (Anthropic), directed and tested by [Carlos](https://github.com/carlos13332)

## License

Node code: **MIT** (same as upstream — see `LICENSE`). Model weights (LingBot-Video, the
GGUF quantizations, and the LightLingBot-Video LoRA) each keep their own upstream license
(Apache 2.0 at the time of writing) — check the source repos before redistribution or
commercial use. Weights are **not** bundled in this repository.
