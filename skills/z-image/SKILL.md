---
name: z-image
description: Write and improve prompts for Z-Image, Alibaba Tongyi-MAI's open-weights 6B image model, and pick the right variant. Covers the Turbo vs. base fork that decides whether negative prompts work at all, the dense-clause prompt style the model was tuned on, photorealism vocabulary, bilingual English/Chinese text rendering, the prompt-enhancer workflow, getting variety out of a low-diversity distilled model, and the correct inference settings per variant.
when_to_use: Use for anything involving Z-Image, Z-Image-Turbo, Z-Image-Edit, Z-Image-Omni-Base, Tongyi-MAI image generation, or running these weights through diffusers, ComfyUI or mflux. Trigger phrases include "Z-Image", "Z-Image-Turbo", "zimage", "Tongyi image model", "mflux-generate-z-image-turbo". Also use it whenever a question about negative prompts, guidance scale, step counts, seed variety, plastic-looking output or in-image text mentions Z-Image or a Z-Image runner - for example "my negative prompt does nothing", "same image on every seed", "which guidance for Z-Image", "Turbo or base".
---

# Z-Image Prompting Guide

Z-Image (造相) is an open-weights image generation family from **Alibaba's Tongyi-MAI**, Apache-2.0, **6B parameters**, built on a Scalable Single-Stream DiT (S3-DiT) architecture. It runs locally: the Turbo variant fits in **16 GB VRAM** and reaches sub-second latency on an H800.

Facts here are verified against Tongyi-MAI's own sources (GitHub README, Hugging Face model card, project blog) on **2026-08-05**. Community-sourced claims are labelled as such. See "Verification & Sources".

---

## Read this first: the Turbo / base fork decides how you prompt

This is the single most important thing about Z-Image and the most common source of frustration. **Turbo and the base model are prompted differently, and Turbo removes two things people assume every diffusion model has.**

| | **Z-Image-Turbo** | **Z-Image** (base) |
|---|---|---|
| Steps | **8** (pass `num_inference_steps=9`) | **28-50** |
| CFG / guidance | **None.** `guidance_scale=0.0` | `guidance_scale` **3.0-5.0** |
| **Negative prompts** | ❌ **Do not work at all** | ✅ **"Strongly recommended for better control"** |
| Diversity across seeds | **Low** (vendor's own rating) | Medium |
| Visual quality rating | Very High | High |
| Post-training | SFT + **RL** | SFT |
| Fine-tunable | N/A | Easy |
| Best for | Fast, photorealistic, deterministic output | Artistic range, control, variety, fine-tuning |

Both ratings above are Tongyi-MAI's own, from the Model Zoo table in the [official README](https://github.com/Tongyi-MAI/Z-Image).

**Two consequences you must design prompts around when using Turbo:**

1. **There is nowhere to put exclusions.** No CFG means the negative-prompt channel is not merely discouraged, it is **inert** - anything you pass is ignored. Every constraint has to be expressed as something *present* in the positive prompt. "No text" does not work; "clean unbroken surface" does.
2. **Re-rolling seeds barely helps.** Turbo was RL-post-trained toward a preferred output and is rated **Low diversity**. If you want three different takes, you must write three different prompts. Changing the seed on a fixed prompt will mostly give you the same picture again. This is a property of the model, not a mistake you are making.

**If either of those is blocking you, the fix is to switch variant, not to fight the prompt.** Base Z-Image gives you negative prompts, real seed variety and 3.0-5.0 guidance, at the cost of ~4-6x the steps.

### The other two variants

- **Z-Image-Edit** - fine-tuned from base for **instruction-based image editing** (image-to-image from natural-language instructions, bilingual instruction understanding). Status on the official Model Zoo: *"To be released"* as of 2026-08-05. If a workflow needs true instruction editing, check whether it has shipped before planning around it.
- **Z-Image-Omni-Base** - raw foundation checkpoint for **both generation and editing**, released for community fine-tuning. Also *"To be released"*. Medium quality, High diversity, easy to fine-tune - the base to build a LoRA on, not to generate with.

Do not promise Edit or Omni-Base capabilities as available. Check the Model Zoo table first.

---

## Quick start: the settings that matter

**Turbo:**
```python
image = pipe(
    prompt=prompt,
    height=1024, width=1024,
    num_inference_steps=9,   # yields 8 DiT forwards
    guidance_scale=0.0,      # MUST be 0 for Turbo
).images[0]
```

**Base:**
```python
image = pipe(
    prompt=prompt,
    negative_prompt=negative_prompt,   # actually works here
    height=1280, width=720,
    num_inference_steps=50,
    guidance_scale=4,
    cfg_normalization=False,   # False for general stylism, True for realism
).images[0]
```

**Resolution:** the official recommendation is **512x512 to 2048x2048 as total pixel area, any aspect ratio** (stated for the base model; Turbo examples use 1024x1024). So aspect ratio is free - you pick dimensions, not a preset ratio - but keep the pixel *count* in that band.

**`cfg_normalization`** is base-only and the guidance is one line, worth memorising: **`False` for general stylism, `True` for realism.**

**Prompt token budget:** the online demo caps text at **512 tokens**; locally you can raise it to **1024** with `max_sequence_length=1024`. Roughly, 600-1000 words lands around 800-1333 tokens, so a very long prompt genuinely can be truncated - raise the cap or trim. (Source: prompting guide by `Cxxs` of the Tongyi-MAI org on the model's HF discussions.) Note: a claim circulating in third-party guides that "effective attention caps at 512 tokens" appears to be a misreading of this **demo setting** - it is a configurable sequence length, not a documented attention limit.

---

## Language of prompts

Write prompts in **English or Chinese**. Those are the two languages Z-Image was trained on, and both official example prompts are one or the other.

For a Polish-speaking user: talk to them in Polish, write the prompt in English. Community reports are consistent that **English prompts outperform other non-Chinese languages** for both detail and text accuracy. Do not prompt in Polish.

Text you want **rendered inside the image** is a separate matter - see Text Rendering below.

---

## The prompt style this model was actually tuned on

Z-Image does not want tag soup, and it does not want flowing literary prose either. Both official examples are the same distinctive shape: **a dense stack of short, concrete noun phrases, comma- and period-separated, walking through the image from subject outward to background, ending with photographic and colour-tone terms.**

Official Turbo example, verbatim:

> Young Chinese woman in red Hanfu, intricate embroidery. Impeccable makeup, red floral forehead pattern. Elaborate high bun, golden phoenix headdress, red flowers, beads. Holds round folding fan with lady, trees, bird. Neon lightning-bolt lamp (⚡️), bright yellow glow, above extended left palm. Soft-lit outdoor night background, silhouetted tiered pagoda (西安大雁塔), blurred colorful distant lights.

Read what that prompt is doing:

- **Zero mood words.** No "beautiful", "stunning", "masterpiece", "8k". Every clause carries visual information.
- **Grouped by region.** Garment, then face, then hair/headdress, then hands and held objects, then background. One subject area per sentence.
- **Spatial anchoring is explicit** - "above extended left palm", not "holding a lamp".
- **Named specifics over categories** - "red Hanfu", "golden phoenix headdress", a named landmark in Chinese characters.
- **Photographic terms land at the end** - lighting quality, background treatment, depth cues.

The official base-model example (Chinese) has the identical shape and closes with a run of exactly these terms: photograph, natural light, soft shadows, a named neutral palette, casual fashion photography, medium depth of field, face and upper body in sharp focus, indoor, solid-colour background.

### Working formula

```
[Subject: who/what + 2-3 identifying specifics]
[Subject detail, grouped by region: garment / face / hair / hands and held objects]
[Spatial relations: what is where, relative to what]
[Setting: location + 1-2 supporting elements, no more]
[Light: direction, quality, colour temperature, time of day]
[Camera and rendering: shot size, depth of field, lens or film stock]
[Colour palette and medium]
```

Order matters less than coverage - but front-load whatever must be right. Detail early in the prompt gets more reliable treatment than detail buried at the end.

### How long?

First-party guidance says the model "performs best with long and detailed prompts". Third-party guides often say the opposite - cap yourself at 3-5 visual concepts. **Both are describing something real and they do not actually conflict:**

- **Long in specifics = good.** More concrete detail about one coherent scene is what the official examples do, and what the model rewards.
- **Long in modifiers = bad.** Stacking 40 quality adjectives, or two competing style families ("photoreal anime cel-shaded oil painting"), dilutes attention and produces mush.

So: **one style family, one scene, as much concrete visual detail as you can supply.** If you cannot say what a word changes in the picture, cut it.

### The prompt-enhancer workflow (this is the intended way to use it)

Tongyi-MAI ships a **Prompt Enhancer** and describes it as giving the model reasoning: it uses "a structured reasoning chain to inject logic and common sense", which is how the blog's demos answer things like a maths word problem or a classical poem as an image. First-party prompting advice is explicit about the workflow:

1. Write a short prompt yourself, capturing intent and the non-negotiables.
2. Hand it to an LLM to expand into the dense-clause style above.
3. Generate from the expanded version.

**This is the single highest-leverage habit for this model**, and doing step 2 by hand is why short prompts feel weak. When acting on a user's request, do step 2 for them: take their one-line idea and return the expanded dense-clause prompt.

---

## Photorealism

Z-Image-Turbo's headline strength is photorealism, and it responds to **photographic vocabulary rather than emotional modifiers** (community consensus, consistent with the official examples ending on photographic terms).

The recipe that works, in three layers:

1. **Name the capture device or film stock.** "Point-and-shoot film camera", "handheld phone snapshot", "medium-format", "Kodak Portra 400 tones", "anamorphic lens". Each maps to a visual fingerprint the model has learned; "photorealistic, 4K" does not.
2. **Break the symmetry of the face and body.** Real people are irregular: "three-day stubble", "slightly crooked nose", "visible pores", "ordinary everyday appearance, not a model". Absence of this is the main cause of the plastic look.
3. **Name the material texture.** "Visible pores", "fabric weave", "film grain", "brush strokes". Skin and cloth without a texture word render as plastic.

Weak: `Realistic photo of an average middle-aged French man in a cafe.`

Strong: `Middle-aged French man, thinning hair, three-day stubble, deep smile lines, worn grey wool coat. Sits at a small marble cafe table, elbow on the surface, holding a small espresso cup. Half-empty carafe and folded newspaper beside him. Overcast midday light through a large window, soft directional shadows from the left. Point-and-shoot film camera, mild grain, medium depth of field, face in sharp focus, muted warm-grey palette, candid unstaged photograph.`

*(The weak/strong contrast pattern is from a community guide; the strong prompt above is written to the official clause style.)*

---

## Constraints without a negative prompt (Turbo)

On Turbo you cannot say what to leave out. Convert every exclusion into a positive description of the state you want.

| Instead of "no ..." | Write |
|---|---|
| no text, no watermark | clean unbroken surface, plain untouched background |
| no extra people | a single figure alone in an empty room |
| no blur | every edge crisp, uniform sharp focus across the frame |
| no clutter | bare tabletop, one object centred, wide empty margins |
| not cartoonish | photograph, natural skin texture, visible pores, real fabric weave |
| no harsh shadows | soft even diffuse light, gentle shadow falloff |
| no modern objects | period-accurate 1930s setting, wooden and brass fittings only |

If a specific artefact keeps appearing and no positive phrasing suppresses it, that is a legitimate reason to move the job to **base Z-Image** and use a real negative prompt.

---

## Bilingual text rendering

Accurate **English and Chinese** text rendering is one of Z-Image-Turbo's advertised strengths, with the project blog noting "strong compositional skills and a good sense of typography" and workable small font sizes.

**Put the exact string in quotes.** The official base-model example does this inline (a hoodie printed with `"Tun the tables"` above `"New ideas"`), so quoting is supported by first-party example, not just community lore.

```
Poster, portrait format. Massive headline "FUTURE STACK 2026" across the upper third,
heavy geometric sans-serif, warm white on deep navy. Below it, smaller line
"Tickets now open", light weight, generous letter-spacing. Lower right corner, small
monospace block "05.09 - 07.09 / Warsaw". Flat solid navy field, no imagery.
Crisp vector-clean edges, high contrast, print-quality typography.
```

Rules that hold up:

- **Quote the literal text**, and say where it sits and at what relative size.
- **Specify typographic weight and family in words** - heavy geometric sans, light serif, monospace.
- **Ask for less text.** Three short strings render far more reliably than a paragraph.
- **Only English and Chinese are supported languages.** Polish and other diacritic-bearing Latin text is outside what the vendor claims. It often works because the glyphs are Latin, but treat it as needing a proof render, and fall back to generating the image clean and setting the type in post.
- **Prefer common characters**; rare glyphs and special symbols degrade.

---

## Getting variety out of a low-diversity model

Turbo is rated **Low diversity** by its own authors. Practical tactics, in order of effectiveness:

1. **Vary the prompt, not the seed.** Write three prompts differing in one axis each - camera type, light direction, wardrobe, time of day. This is what actually produces different pictures.
2. **Move the varying element early in the prompt.** Detail near the front has more influence.
3. **Change the aspect ratio.** Recomposing forces a different layout, not just a different render of the same layout.
4. **Switch to base Z-Image** when genuine variety is the requirement - it is rated Medium diversity, accepts negative prompts, and has real guidance to modulate.

Note for batch runs: some UIs support wildcard syntax like `{a|b|c}` to randomise a slot across a batch. That is a **feature of the front-end** (ComfyUI-style), not of the model. Whether it works depends entirely on your runner.

---

## Running the weights (brief - this skill is about prompts)

Z-Image is open weights, so "how you run it" is separate from "how you prompt it". Common routes:

- **diffusers** - `ZImagePipeline`, the reference path. Requires diffusers from source at the time the model card was written (two PRs were merged upstream). bfloat16, optional Flash Attention backend, `pipe.transformer.compile()`, and `enable_model_cpu_offload()` for tight memory.
- **ComfyUI** - node ecosystem; a community node exists supplying the official Z-Image resolutions as latents.
- **mflux** (Apple Silicon, MLX) - the practical local route on a Mac; exposes Turbo as its own generate command with `--seed` accepting multiple values for a batch in one process.
- **Hosted demos** - Hugging Face Spaces and ModelScope, both first-party, for prompt iteration before committing to local runs.

Whatever the runner, the two settings that decide whether output looks right are the ones above: **steps and guidance must match the variant**. A Turbo checkpoint run at guidance 4 with 50 steps, or a base checkpoint run at guidance 0 with 8 steps, produces garbage - and it is a very common misconfiguration when a runner's defaults were written for a different model.

---

## Failure modes

| Symptom | Cause | Fix |
|---|---|---|
| Negative prompt has no effect | Turbo has no CFG - the channel is inert | Rewrite the exclusion as a positive description, or switch to base Z-Image |
| Every seed gives nearly the same image | Turbo is rated Low diversity by design | Vary the prompt, not the seed; or use base |
| Plastic, airbrushed faces | No texture or asymmetry words | Add "visible pores", specific facial irregularities, a film stock or camera |
| Output looks washed out or muddy | Guidance/steps mismatched to the variant | Turbo: `steps=9, guidance=0.0`. Base: `steps=28-50, guidance=3.0-5.0` |
| Realism still off on base model | `cfg_normalization` wrong way round | `True` for realism, `False` for general stylism |
| Style is mush | Two competing style families in one prompt | One style family per prompt |
| End of a long prompt appears ignored | Token cap - demo default is 512 | `max_sequence_length=1024` locally, or shorten; move must-haves earlier |
| Short prompt gives a weak generic image | The model expects dense specifics | Run the prompt-enhancer workflow: expand to dense clauses before generating |
| Rendered text garbled | Too much text, or an unsupported language/rare glyphs | Fewer, shorter quoted strings; English or Chinese; common characters. Otherwise set type in post |
| Composition ignores your layout | Spatial relations left implicit | State position explicitly, relative to a named element |
| Machine hangs or OOMs on a local batch | Two model instances at once - each copy holds the full weights | One process at a time; batch by passing multiple seeds to a single process |

---

## What NOT to do

- **Passing a negative prompt to Turbo** and assuming it did something.
- **Quality-word stuffing** - "masterpiece, 8k, trending, ultra detailed, award winning". The model was not trained to reward them; they cost you attention budget.
- **Tag soup.** Comma-separated *phrases* yes; bare keyword lists no.
- **Mixing style families** in one prompt.
- **Prompting in Polish or any language other than English or Chinese.**
- **Promising Z-Image-Edit or Omni-Base** - both were still unreleased on the official Model Zoo as of 2026-08-05.
- **Re-rolling seeds to fix a prompt problem** on Turbo. It is the least effective lever available.
- **Copying SDXL or Midjourney settings** - a different guidance regime entirely, and Turbo has no guidance at all.

---

## Verification & Sources

Verified **2026-08-05** against Tongyi-MAI-controlled sources:

- **GitHub repository (most complete):** https://github.com/Tongyi-MAI/Z-Image - Model Zoo table, per-variant recommended parameters, both official example prompts
- **Hugging Face model card:** https://huggingface.co/Tongyi-MAI/Z-Image-Turbo
- **Base model card:** https://huggingface.co/Tongyi-MAI/Z-Image
- **Project blog:** https://tongyi-mai.github.io/Z-Image-blog/ - text rendering and Prompt Enhancer claims
- **Technical report:** https://arxiv.org/abs/2511.22699 · Decoupled-DMD: https://arxiv.org/abs/2511.22677 · DMDR: https://arxiv.org/abs/2511.13649
- **First-party prompting guide** by `Cxxs` (Tongyi-MAI org member), model HF discussions: long-detailed-prompt guidance, `max_sequence_length` 512/1024, confirmation that negative prompts are not used at all
- **Official demos:** HF Space and ModelScope Space, both linked from the model card
- **License:** Apache-2.0 (per the model card frontmatter) - commercially usable

**Community-sourced, labelled as such in the text above** (useful, not vendor-confirmed): the camera/film-stock vocabulary approach, the anti-plastic face recipe, the three-to-five-concepts guidance, and the observation that English outprompts other non-Chinese languages.

**Deliberately not claimed here:**
- Any per-variant benchmark number beyond the vendor's own qualitative High/Medium/Low ratings.
- An attention-limit figure. What is documented is a configurable `max_sequence_length` (512 in the demo, 1024 available), not an attention cap.
- Availability of Z-Image-Edit or Z-Image-Omni-Base - both listed "To be released".
- Any wall-clock speed for consumer hardware. The vendor's figures are sub-second on an H800 and a 16 GB VRAM fit; anything about a specific laptop or Mac is a local measurement, not a model property.
- Support for languages other than English and Chinese, in prompts or in rendered text.

If you find a claim above is wrong, fix it here and in the body section, with the source URL and the date you checked.
