---
name: client-project-pitch
description: Turn rough project materials (a sketch, render, Rhino/SketchUp screenshot, or floor plan) plus basic project facts into a polished client-facing PDF pitch / commercial offer. Generates a geometry-locked photorealistic hero render and several consistent angles with the Nim MCP, then assembles them with the project's text and data into a branded PDF. Trigger when an architect, visualizer, or real-estate user wants a presentation, pitch, commercial offer, or listing document for a building or space.
---

# Client Project Pitch → PDF

A single orchestrator skill for architects, visualizers and real-estate users. From rough project inputs it produces a finished **PDF commercial offer / pitch**:

`project inputs + user-supplied facts → geometry-locked renders (Nim) → user approves renders → PDF with project text & data`

It directly addresses the recurring user-research needs: *fast buyer-facing materials before a building is finished* (Domantas), *project-level consistency and locked geometry* (Boldizsár, Andrey, Gustav), and *one place that turns weak inputs into impressive sales assets* (Spyros, Carlos).

Runs on the **Nim MCP** for image generation and the **pdf** skill for document assembly. The user is `nim.video`; always use the connected Nim tools and never invent external links.

## Defaults (override on request)
- **Output:** a single PDF commercial offer.
- **Visuals:** one hero render + 2–3 consistent additional angles, plus a project-data section.
- **Image style:** photorealistic architectural visualization, geometry preserved.
- **Aspect ratio:** 16:9 for hero/exterior, 4:3 for detail/angle shots (keep consistent within one pitch).

## Step 1 — Intake (gather everything before generating)

### 1a. Visual input(s)
Find at least one sketch / render / Rhino or SketchUp screenshot / floor plan — in the user's folder or attached. Multiple angles are better: pass them all so the building stays consistent. If the folder has none, ask the user to add or attach one before continuing. Run `media_upload` to get Nim file URLs for each.

### 1b. Project facts — ALWAYS ASK, never silently placeholder
The text/data section needs real project facts. **Do not quietly fill the PDF with `[ ... ]` placeholders and move on.** First look in the folder / attachments / file names for any facts; then **explicitly ask the user (use the AskUserQuestion tool) for everything still missing** before assembling the PDF. The selling points especially must come from the user — invented marketing claims are the most common failure of this skill.

Ask, in one or two grouped questions, for whatever is missing:
- Project / building name
- Location
- Type (residential, office, mixed, villa…)
- Area (m² / sq ft), floors, number of units
- Price / price range (if a sales offer)
- Key materials & finishes
- **3–5 selling points** (location perks, layout, sustainability, view…) — ask the user to provide these; offer a few drafted suggestions based on the visuals they can accept or edit, but let them decide
- Contact / company info for the closing page

Only after the user has answered (or explicitly said "use placeholders / I'll fill them later") do you proceed. If they opt for placeholders, mark those fields clearly as `[ ... ]` in the PDF — but never present invented numbers or selling points as if they were real.

### 1c. Style or brand
Any preferred look (photoreal default, golden hour, night, minimal white-model) and brand color/logo if available.

## Step 2 — Build the project brief (locked constraints)
Before any generation, write a short internal **brief** that every image prompt reuses. This is what keeps outputs consistent and on-spec — the single biggest ask across the research.

The brief contains:
- A positive base: building type, materials, surroundings, light/mood, camera language.
- A **locked / negative block** appended to every prompt:

  > Preserve the exact massing, proportions, floor count, window positions, rooflines and façade layout from the reference. Do NOT change the building's geometry, do NOT move or resize windows, do NOT add or remove structures, do NOT alter the layout. Only refine materials, lighting, surroundings and realism.

## Step 3 — Generate the hero render (Nim)
Use a structure-preserving image-edit model.

- **Primary:** `Nano Banana Pro Edit` — `model_id: 3c1b1c5b-c1d6-44a8-b986-3068820f4927` (best geometry preservation).
- **Richer aesthetic alt:** `Seedream 4.5 Edit` — `model_id: c553d974-a7eb-4113-ac3c-60cf73ea9db7`.

Run `models_explore action=get` on the chosen `model_id` first, then call `generate_image`:
- `fileInputs`: all reference views
- `requestedAspectRatio`: `16:9`, `resolution`: `2K`
- `prompt`: brief base + the locked block. Hero template:

  > Photorealistic architectural visualization of {project name}, {type} in {location}. Hero exterior three-quarter view, realistic {materials}, soft natural daylight, landscaped context, professional real-estate render quality, ultra-detailed 8k. {locked/negative block}

Poll `get_generation_status` until done; capture the output image URL.

## Step 4 — Generate consistent angles
Reuse the **same brief and reference images** so the angles read as the same building. Generate 2–3 of:
- front elevation / entrance approach
- side or rear three-quarter
- aerial / bird's-eye context
- (real estate) a key interior, if interior inputs were given

For each, swap only the camera line in the prompt (e.g. `straight-on front elevation at eye level`, `low aerial bird's-eye showing the plot and surroundings`) and keep the locked block. Use `batchSize` for quick variants and pick the best. Capture each output URL.

## Step 5 — Collect images locally
Download each finished image (from its Nim output URL via `get_generation_status`) into the working folder so they can be embedded. Verify each file opens.

## Step 5.5 — User review & approval of renders (REQUIRED before the PDF)
Image models miss sometimes — one render in a set is often weak (wrong geometry, distorted windows, an invented wing). **Never assemble the PDF from un-reviewed images.**

1. **Show the user every generated render** (present the image files) labelled by role: hero, front elevation, aerial, etc.
2. **Ask the user to approve or reject each one** with the AskUserQuestion tool — e.g. "Which renders should go in the pitch?" / "Regenerate any of these?" Make it easy to say *approve all*, *drop one*, or *regenerate #N*.
3. **Regenerate the rejected ones** — reuse the same brief, reference images and locked block, vary the seed or refine the camera/prompt line, and use `batchSize` 2–4 so the user has options to pick from. Loop back to this review step until the user is happy.
4. Only the **user-approved** images proceed into the PDF.

Do not skip this even if the renders look fine to you — the user is the judge of which visuals represent their project.

## Step 6 — Assemble the PDF (pdf skill)
**Read the `pdf` skill's SKILL.md now** and build the document from the approved images and the facts the user supplied in Step 1b. Do not start the PDF before images are approved.

Suggested structure of the commercial offer:
1. **Cover** — hero render full-bleed, project name, location, one-line tagline, logo.
2. **Project overview** — 2–3 sentence description + the 3–5 selling points.
3. **Gallery** — the consistent angles, 1–2 per page, captioned.
4. **Key data** — table: area, floors, units, materials, price / price range, delivery date.
5. **Closing / contact** — CTA and company contact info.

Keep typography clean and consistent; use the brand color if provided, otherwise a neutral architectural palette (charcoal + warm grey + one accent). Mark any missing facts as `[ ... ]` placeholders rather than inventing numbers.

## Step 7 — Deliver
Save the PDF to the user's folder and present it. Offer follow-ups: regenerate any angle, add a day/night variant, or export an extra format. If the user later wants motion, hand the hero render to the `sketch-to-drone-flyover` skill for a flyover video to accompany the pitch.

## Guardrails
- **Always ask the user for missing facts and selling points (Step 1b)** — never silently placeholder or invent prices, areas, dates, or marketing claims. For real estate, inaccurate claims are also a legal issue (misrepresenting a property can void a deal).
- **Always get user approval of the renders (Step 5.5) before building the PDF** — regenerate rejected images rather than shipping a weak one.
- Preserve geometry — the value is "the real project, presented beautifully," not a redesign.
- Use only image URLs returned by Nim; never fabricate generation links.
- Confirm the chosen models and credit cost with the user if they care about spend (`get_credit_balance`).
