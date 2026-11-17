---
name: open-ai-design-agent
description: Plan and produce creative design deliverables from a plain-language brief, such as a brand kit, logo, poster, flyer, social posts, ad creatives, YouTube thumbnails, packaging or product mockups, and motion or video ads. Breaks the brief into an asset list, picks the best image or video model for each asset, writes the generation prompts, keeps a brand kit consistent, and can run the generations through the MuAPI Creative Agent API. Use when the user asks for a design kit or campaign assets, asks which AI image or video model to use for a design, or wants to run, configure or call the Open AI Design Agent project.
---

# Open AI Design Agent

Work like the design agent in this repository: **Plan → Route → Execute → Assemble**. Show the plan before generating anything, because generation costs the user's MuAPI credits.

## 1. Gather the brief

Ask only for what is missing:

- **Brand or campaign**: name, product, audience, message.
- **Brand kit**: palette (hex codes), fonts, logo, tone of voice. If none exists, propose one first.
- **Deliverables and formats**: for example IG post 1:1 or 4:5, story 9:16, A4 poster, 16:9 thumbnail, 5–15 s video.
- **References**: up to 14 images for style, product or character consistency.
- **Exact copy**: any text that must appear on an asset, word for word.

## 2. Plan the deliverables

Write the plan as a table and save it to `design_output/plan.md`:

| # | Asset | Format | Depends on | Model | Prompt summary |
|---|---|---|---|---|---|

- Order assets so that what others depend on comes first: brand kit and logo, then key visual, then layouts that reuse them, then motion.
- Record the brand kit in `design_output/brand_kit.md` and repeat its palette, fonts and tone in every prompt.

## 3. Route each asset to a model

| Asset | Recommended models | Notes |
|---|---|---|
| Logos | Recraft v3, Ideogram v3 | Vectorize afterwards for SVG |
| Posters, flyers | Ideogram v3, Flux Pro Ultra, Nano Banana 2 | Best for readable typography |
| Social posts | Seedream 5, Flux Dev, Nano Banana 2 | 1:1, 4:5, 9:16 |
| Brand identity boards | Flux Pro, GPT-Image, Nano Banana 2 Edit | Use multi-image references |
| UI mockups, app screens | Flux Dev, GPT-Image, Recraft v3 | |
| Product mockups, lifestyle | Nano Banana 2 Edit, Flux Kontext Pro | Places a product image into a scene |
| Illustrations | Flux Dev, SDXL | |
| YouTube thumbnails | Ideogram v3, Nano Banana 2 | Bold text plus faces |
| Print, 300 DPI or more | Flux Pro Ultra, Imagen 4 Ultra | Upscale with Clarity or Magnific |
| Packaging, merch | Recraft v3, Flux Pro | |
| Motion ads, social video | Kling v3, Sora 2, Veo 3, Runway, Luma Ray2 | 9:16, 1:1, 16:9 |
| Animated logos | Kling, Runway or Luma Ray2 image-to-video | Animate the final static logo |
| Talking head | Infinite Talk, LTX 2.3 Lipsync, Wan 2.2 Speech-to-Video | Portrait plus audio |

Rules of thumb:
- Any asset with on-image text goes to a text-capable model (Ideogram v3, Flux Pro Ultra, Nano Banana 2, GPT-Image). Put the exact copy in quotes in the prompt.
- With a reference image, use an edit model (Nano Banana 2 Edit, Flux Kontext, GPT-Image Edit, Seedream Edit).
- For video, generate the key frame as an image first, then animate it with image-to-video.

## 4. Write the prompts

For each asset include: subject, composition and framing, style, the brand palette as hex codes, lighting, the exact text in quotes, aspect ratio, and what to avoid. Save every prompt in `design_output/prompts.md` so results can be reproduced.

## 5. Execute through MuAPI (optional)

Only do this when the user wants the assets generated and has a MuAPI key. Read the key from the `MU_API_KEY` environment variable. Never print it, write it into files or put it in URLs. Every request sends the header `x-api-key: $MU_API_KEY`. The base URL is `https://api.muapi.ai/api/v1/creative-agent`.

1. `POST /sessions` with `{}` returns a session `id`.
2. `POST /sessions/{id}/chat` with `{"message": "<brief or one asset's prompt>", "model": "gpt-5-mini"}` returns a `job_id`.
3. Poll `GET /jobs/{job_id}/events?since=<cursor>` about every 1–2 seconds. Read `events`, keep the returned `cursor`, and stop when `done` is true.
4. If the job asks for approval, show the user what will be generated and what it will cost, and wait for their yes. Then call `POST /jobs/{job_id}/approve`, or `/reject` or `/cancel`.
5. `GET /sessions/{id}/assets` lists the generated files. Download them into `design_output/`.

Other endpoints: `GET /agent-skills` lists the agent's built-in recipes, and `POST /sessions/{id}/run-skill` with `{"skill_name": ..., "inputs": {...}}` runs one. The account balance is at `GET https://api.muapi.ai/api/v1/account/balance`.

## 6. Assemble

Organize `design_output/` by category (`logo/`, `social/`, `print/`, `video/`). Then report each asset with its file, model and prompt, list anything that failed or needs a redo, and give next steps such as vectorizing, upscaling or more format cutdowns.

## Running this repository's app

```bash
npm run install:all
npm run dev:app                     # client on http://localhost:3000
cd server && pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

Or run everything with `docker compose up --build`.

- The server needs `server/.env` (copy `server/.env.example`) with `MU_API_KEY` and `API_SUFFIX=https://api.muapi.ai`.
- `client/.env.example` sets `NEXT_PUBLIC_API_BASE`; point it at `http://localhost:8000/api/v1/creative-agent` for local development.
- The client uses Next.js 16 and React 19. Read `client/AGENTS.md` before changing client code, because its Next.js APIs differ from older versions.
