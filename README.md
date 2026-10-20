# MiniMax H3 Ref2VA Workflow for ComfyUI

A ComfyUI workflow designed for **MiniMax H3 Reference-to-Video** generation with strong reference fidelity across image, video, and audio inputs. It combines the Ref2VA pipeline with an 8-step Turbo LoRA, latent upscaling, RIFE frame interpolation, sharpening, and automatic RAM cleanup.

> [!NOTE]
> This workflow was created and tested on an **NVIDIA GeForce RTX 3060 with 12 GB VRAM and 64 GB RAM**. It is intended especially for systems with similar or better hardware.

## Features

- Up to **9 image references** for people, objects, and scenes
- Optional video and audio references
- **16:9** and **9:16** aspect-ratio selection
- Configurable duration with an automatically calculated frame count
- 8-step Turbo LoRA and MiniMax H3 optimizations for limited VRAM
- **1.5x** latent upscaling
- RIFE interpolation to **48 FPS** and RCAS sharpening
- Video export to `output\video\ComfyUI`
- RAM cleanup after export

## Tested Environment

| Component | Tested configuration |
| --- | --- |
| Operating system | Windows 11 Pro |
| GPU | NVIDIA GeForce RTX 3060, 12 GB GDDR6 |
| CPU | AMD Ryzen 7 2700X |
| System RAM | 64 GB DDR4 |
| ComfyUI | 0.34.0 |
| Python | 3.13.12 |
| PyTorch | 2.12.1 + CUDA 13.0 |
| NVIDIA driver | 616.92 |
| Launch flags | `--windows-standalone-build --fast fp16_accumulation --use-ck-attention` |

The `MiniMaxLowVRAMAttention` configuration (`head_chunks: 4`), chunked feed-forward processing, and TeaCache are tuned for 12 GB of VRAM. On GPUs with less memory, reduce the resolution and duration, and adjust chunk settings if necessary.

## Requirements

1. Install a current **ComfyUI** release and **ComfyUI-Manager**.
2. Import [`minimax-h3-ref2va-workflow.json`](minimax-h3-ref2va-workflow.json) into ComfyUI by dragging and dropping it onto the interface.
3. Let ComfyUI-Manager install all missing custom nodes, then restart ComfyUI.

The workflow uses nodes from, among others, the following packages:

| Package | Usage in the workflow |
| --- | --- |
| `comfyui-easy-use` | Connection nodes and logic |
| `comfyui-fizgig-h3-tweaks` | MiniMax H3 detail and prompt tweaks |
| `comfyui-frame-interpolation` | RIFE interpolation |
| `comfyui-kjnodes` | Preview, sharpening, and utility nodes |
| `comfyui-memory-cleanup` | RAM cleanup |
| `comfyui-rf-nodes` | Prompt and text utility nodes |
| `comfyui-spectrum-minimax-h3` | Spectrum model patch |
| `rgthree-comfy` | Seed and LoRA loader |
| `Comfyui_Minimax_h3_latent_Upscaler` | 3D latent upscaling |

> [!TIP]
> If a node is highlighted in red after import, install its package through **Manager → Install Missing Custom Nodes**. The list above is a reference in case ComfyUI-Manager cannot resolve a package automatically.

## Models

Download models only from official or otherwise trusted sources and place them in the appropriate ComfyUI model folders. File names and subdirectories must match the following table for the workflow to load them without changes.

| Type | Expected path relative to `ComfyUI\models` |
| --- | --- |
| Diffusion model / UNet | `diffusion_models\minimax h3 ref2va pruned int8 convrot\minimax_h3_ref2va_pruned_int8_convrot.safetensors` |
| Text encoder | `text_encoders\qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` |
| Video VAE | `vae\minimax h3 video vae fp16\minimax_h3_video_vae_fp16.safetensors` |
| Audio VAE | `vae\minimax h3 audio vae fp32\minimax_h3_audio_vae_fp32.safetensors` |
| Turbo LoRA | `loras\MiniMax H3\minimax_h3_ref2v_turbo_8step_v1.0_768p_comfyui_bf16.safetensors` |
| 3D latent upscaler | The latent-upscaler custom node's `models` folder: `minimax_h3_latent_upscaler_3d_fp16.safetensors` |
| Tiny VAE for previews | The folder expected by KJNodes: `taeh3.safetensors` |
| RIFE | The folder expected by the frame-interpolation package: `rife49.pth` |

The selected core models are visible in the **Load Models** group. If your installation uses different model folders, select the files manually in their respective nodes once, then save the workflow.

## Using the Workflow

1. **Check models:** Open the **Load Models** group and confirm that every dropdown points to an available file.
2. **Choose format and duration:** Set the aspect ratio and megapixels in the **Input** group. The default is 16:9 at **0.98 MP**, or **1344 × 768 pixels**. Set the desired length in the `Duration` node; the workflow calculates a compatible frame count for 24-FPS generation.
3. **Replace references:** Assign your images in **Load References**:
   - **Image #1–#5:** References for subject 1
   - **Image #6–#8:** References for subject 2
   - **Image #9:** Scene or environment reference
   - Optional: Video and audio references in the same area
4. **Update the prompt:** Edit the text nodes in the **Prompt** group and the preservation rules in **Prompt Fixes**. The `$DURATION$`, `$ORIENTATION$`, and `$PRSV#…$` placeholders are replaced automatically.
5. **Generate:** Set a seed if needed, then queue the workflow. It generates at 24 FPS, upscales the latent by 1.5x, interpolates to 48 FPS, and sharpens the output.
6. **Find the result:** By default, the completed video is saved under `ComfyUI\output\video\ComfyUI`.

## Important Default Values

| Setting | Default |
| --- | --- |
| Resolution | 1344 × 768 (16:9, 0.98 MP) |
| Duration | 5 seconds |
| Sampling | `res_multistep`, 20 steps |
| Turbo LoRA | Enabled, strength `0.85` |
| Sigma shift | Video `12`, audio `3` |
| Latent upscaling | 1.5x |
| RIFE | `rife49.pth`, 2x, 48 FPS |
| Sharpening | RCAS, strength `0.4` |

For shorter test runs, start at 0.5 MP. In this workflow, that corresponds to **960 × 544 pixels** at 16:9. All resolutions should be divisible by 32.

## Runtime and Memory

Sampling and VAE decoding are the most memory-intensive stages. The workflow runs `RAMCleanup` automatically after saving. Still, close other GPU- and RAM-intensive applications before queuing long or high-resolution jobs.

## Reference Material and Publishing

- Use only images, videos, voices, and audio for which you have the necessary rights and consent.
- The example content embedded in the workflow is a replaceable template. Replace it, together with any example references and wording, before your own use.
- Also comply with the license terms of the models and custom nodes, as well as applicable platform and local requirements.

## License

No license has been specified for this repository yet. Add an appropriate license file before allowing third-party reuse or distribution.
