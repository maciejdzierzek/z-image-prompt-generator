# Z-Image Prompt Generator - Claude Skill

> A Claude skill for writing prompts for **Z-Image**, Alibaba Tongyi-MAI's open-weights 6B image model.
> Works with Claude.ai, Claude Desktop, and Claude Code.

[![Claude Skill](https://img.shields.io/badge/Claude-Skill-orange)](https://claude.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## What is this?

Z-Image is genuinely good and genuinely confusing to prompt. Most guidance written for SDXL, Flux or Midjourney is actively wrong here, because the variant almost everyone runs - **Z-Image-Turbo** - removes two things every other diffusion workflow depends on:

- **Negative prompts do nothing.** Turbo runs with no classifier-free guidance, so the negative channel is inert, not just weak.
- **Re-rolling seeds barely changes the image.** Turbo is rated **Low diversity** by its own authors.

This skill teaches Claude the model as it actually behaves: the Turbo-vs-base fork and what each one takes, the dense-clause prompt style both official example prompts are written in, how to express exclusions as presences, the photorealism vocabulary that works, bilingual text rendering, and the prompt-enhancer workflow the authors intend you to use.

## The fork that decides everything

| | **Z-Image-Turbo** | **Z-Image** (base) |
|---|---|---|
| Steps | 8 (pass `num_inference_steps=9`) | 28-50 |
| Guidance | **`0.0` - none** | 3.0-5.0 |
| Negative prompts | ❌ inert | ✅ "strongly recommended" |
| Diversity across seeds | Low | Medium |
| Best for | fast photorealism | range, control, variety, fine-tuning |

Both ratings are Tongyi-MAI's own, from the Model Zoo table in the official repository.

Two further variants, **Z-Image-Edit** (instruction editing) and **Z-Image-Omni-Base** (fine-tuning base), were still listed "To be released" when this skill was written. The skill says so rather than promising them.

## What the skill covers

- **Variant selection** with the per-variant inference settings, including the `cfg_normalization` rule (`True` for realism, `False` for stylism)
- **The prompt style the model was tuned on** - dense, concrete noun-phrase clauses grouped by image region, no mood words, photographic terms last. Broken down from both official examples
- **Constraints without a negative prompt** - a conversion table from "no X" to a positive description
- **Photorealism** - the three-layer recipe: capture device or film stock, facial asymmetry, material texture
- **Bilingual text rendering** - quoted literal strings, typographic instructions in words, and an honest note that only English and Chinese are supported
- **The prompt-enhancer workflow** - write short, expand with an LLM, then generate. The highest-leverage habit for this model
- **Getting variety** out of a low-diversity distilled model
- **Token budget** - the 512-token demo default and `max_sequence_length=1024` locally, plus a correction of a widely-repeated misreading of that setting
- **A failure-mode table** mapping symptoms to causes to fixes
- **`references/prompt-recipes.md`** - fill-in recipes for portraits, product shots, posters with text, interiors, illustration, pseudo-consistent characters, and honest base-model negative prompts

## Installation

### Claude Code

```bash
/plugin marketplace add maciejdzierzek/claude-plugins
/plugin install z-image-prompt-generator@maciejdzierzek
```

### Claude.ai and Claude Desktop

Download the ZIP from [Releases](../../releases), then **Customize > Skills** → **"+"** → **Upload a skill**. Requires code execution enabled in **Settings > Capabilities**.

### Manual

```bash
mkdir -p ~/.claude/skills/z-image
cp -r skills/z-image/* ~/.claude/skills/z-image/
```

## Usage

```
Write me a Z-Image Turbo prompt for a photorealistic portrait of an older baker in her kitchen.
```
```
My negative prompt does nothing in Z-Image. What am I doing wrong?
```
```
I need ten genuinely different variants of this scene - Turbo keeps giving me the same picture.
```
```
How do I get a headline to render correctly on a poster in Z-Image?
```

## Model facts

Z-Image (造相) from **Alibaba Tongyi-MAI**. **Apache-2.0**, commercially usable. **6B parameters**, Scalable Single-Stream DiT (S3-DiT). Turbo fits **16 GB VRAM** and reaches sub-second latency on an H800. Recommended resolution is **512x512 to 2048x2048 as total pixel area, any aspect ratio**.

Runs via `diffusers` (`ZImagePipeline`), ComfyUI, or mflux on Apple Silicon. This skill is about **prompts**, not about any one runner - though it does flag the single most common misconfiguration: steps and guidance carried over from a different model.

## How this skill is maintained

Every factual claim is verified against Tongyi-MAI-controlled sources - the GitHub repository, the Hugging Face model cards, the project blog, and a prompting guide written by a member of the Tongyi-MAI organisation - with the verification date stated in the skill. Where advice comes from community guides rather than the vendor, the skill labels it. The skill also carries an explicit list of things it deliberately does **not** claim, including a widely-repeated "512-token attention cap" that is really a configurable demo setting.

## About

Built by [Maciej Dzierżek](https://maciejdzierzek.com). All AI tools: [maciejdzierzek.com/narzedzia](https://maciejdzierzek.com/narzedzia)

Companion skills: [nano-banana](https://github.com/maciejdzierzek/nano-banana-prompt-generator) (Gemini images) · [kling-ai](https://github.com/maciejdzierzek/kling-ai-prompt-generator) and [seedance](https://github.com/maciejdzierzek/seedance-prompt-generator) (AI video)

## License

MIT for this skill. The Z-Image model weights are Apache-2.0 from Tongyi-MAI, licensed separately.
