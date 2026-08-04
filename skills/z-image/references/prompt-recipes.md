# Z-Image prompt recipes

Ready-to-adapt prompts in the dense-clause style the model was tuned on. All written for **Z-Image-Turbo** unless noted (`steps=9`, `guidance=0.0`, no negative prompt). Replace bracketed parts.

A reminder that governs every recipe here: **no exclusions.** Everything you want absent must be phrased as something present.

---

## 1. Photorealistic portrait (anti-plastic)

```
[Age] [nationality] [gender], [hair detail], [facial irregularity 1], [facial irregularity 2],
wearing [garment with material and wear state]. [Posture and what the hands are doing].
[One or two supporting props, placed relative to the subject].
[Light source] from [direction], [shadow quality].
Point-and-shoot film camera, mild grain, medium depth of field, face in sharp focus.
[Two- or three-colour palette], candid unstaged photograph, visible pores, real fabric weave.
```

Worked:
```
Mid-fifties Polish woman, grey hair pulled back loosely, deep laugh lines, one eyebrow slightly
higher, wearing a faded denim apron over a linen shirt. Stands at a wooden counter, both hands
resting flat on the surface. Ceramic jug and a scattering of flour beside her left hand.
Late-afternoon window light from the left, long soft shadows. Point-and-shoot film camera,
mild grain, medium depth of field, face in sharp focus. Muted warm ochre and off-white palette,
candid unstaged photograph, visible pores, real fabric weave.
```

## 2. Product shot on a clean field

```
[Product] in [material and finish], [orientation], centred, occupying the middle third of the frame.
[Surface it stands on]. Bare surroundings, wide empty margins, one continuous [colour] backdrop.
Soft even diffuse light from above and slightly front, gentle shadow falloff, small contact shadow.
Studio product photograph, medium-format look, every edge crisp, uniform sharp focus.
[Palette]. Clean unbroken background.
```

The last two clauses are doing negative-prompt work positively: "wide empty margins" replaces "no clutter", "clean unbroken background" replaces "no text or watermark", "uniform sharp focus" replaces "no blur".

## 3. Poster or card with rendered text

```
Poster, [orientation] format. Headline "[EXACT TEXT]" across the [position], [weight] geometric
sans-serif, [colour] on [colour]. Below it, smaller line "[EXACT TEXT]", light weight, generous
letter-spacing. [Corner] a small monospace block "[EXACT TEXT]".
Flat solid [colour] field, no imagery. Crisp vector-clean edges, high contrast, print-quality typography.
```

Rules: quote every literal string, give position and relative size, name weight and family in words, keep it to three strings. English or Chinese only - other languages need a proof render, otherwise generate clean and set type in post.

## 4. Interior / environment

```
[Room type] interior, [architectural feature], [flooring], [wall treatment].
[Furniture item 1] at [position], [furniture item 2] at [position]. [One lived-in detail].
[Time of day] light entering through [aperture] from [direction], [shadow behaviour],
[secondary light source and its colour].
Wide shot, [lens character], deep focus. [Palette]. Architectural interior photograph.
```

## 5. Stylised illustration (single style family)

```
[Style family] illustration. [Subject with 2-3 identifying specifics]. [Action or pose].
[Setting reduced to two or three elements]. [Light direction and quality].
[Line and shading character: line weight, flat fills or gradients, texture].
[Named palette of two or three colours]. Even flat lighting, clean [background treatment].
```

One style family only. "Photoreal anime cel-shaded oil painting" produces mush.

## 6. Consistent-ish character across images

Turbo has no reference-image conditioning and is rated Low diversity, which cuts both ways: it will not reproduce a character on demand, but it does repeat itself. Best available approach:

1. Write one **canonical description block** - 6-10 clauses covering face, hair, build, wardrobe, in fixed wording.
2. Paste that block verbatim, first, in every prompt.
3. Vary only what follows it: pose, setting, light, camera.
4. Keep the seed fixed as a weak extra anchor.

```
[CANONICAL BLOCK - identical every time, 6-10 clauses]
[New pose and action]. [New setting, two elements]. [New light]. [Same camera and palette clauses].
```

Expect drift. If identity must hold, this is the wrong tool - use a model with reference-image conditioning, or wait for Z-Image-Edit / fine-tune the base model.

## 7. Batch of genuine variants

Do not re-roll seeds. Write N prompts differing on **one axis each**, and put the varying clause **early**:

| Axis | Variants |
|---|---|
| Camera | point-and-shoot film / medium-format studio / handheld phone snapshot |
| Light | overcast diffuse / hard low sun from the side / single warm lamp in a dark room |
| Time | dawn blue hour / flat midday / late golden hour / night |
| Framing | tight close-up / medium waist-up / wide environmental |
| Palette | muted warm greys / high-saturation primaries / cool teal and slate |

---

## Base-model recipes (`Z-Image`, not Turbo)

Base takes `steps=28-50`, `guidance=3.0-5.0`, a **real negative prompt**, and `cfg_normalization` (`True` for realism, `False` for stylism). Use it when Turbo's two limits bite.

**Negative prompt starting point** - only on base, and only listing things you actually keep seeing:
```
blurry, low quality, distorted anatomy, extra fingers, watermark, signature, text,
oversaturated, harsh flash, plastic skin
```

Do not paste a giant boilerplate negative. Each term costs you; add them as symptoms appear.

**When to use base over Turbo:**

| Need | Variant |
|---|---|
| One good photorealistic image, fast | Turbo |
| A specific artefact suppressed and positive phrasing failed | Base |
| Ten genuinely different takes | Base |
| Artistic style range | Base |
| Fine-tuning or a LoRA | Base (or Omni-Base when released) |
| Tight VRAM (16 GB) | Turbo |
