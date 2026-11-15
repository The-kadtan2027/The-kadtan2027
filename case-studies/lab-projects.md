# Two smaller builds

Both predate the current three and stay here because the patterns in them — degrading gracefully
when a model or a feed is unavailable, keeping a human in the loop — show up in everything I build now.

---

## Image Text Modifier (ImageEditor)

**Problem.** Replacing text baked into a photo is three separate problems: find the text, remove it
without leaving a smudge, then re-render new text that matches the original's size and colour.

**Pipeline.** FastAPI backend. PaddleOCR detects text regions and returns bounding boxes to the
client; the user edits a box; inpainting clears the old glyphs (Stable Diffusion through `diffusers`,
with an OpenCV inpainting fallback); Pillow renders the replacement, sampling font size and colour
from the original region. React 18 + TypeScript front end (MUI, Zustand, React Query, Vite), whole
stack under `docker-compose`.

**Engineering notes.**
- Detection and generation are separate steps, so a failed inpaint never discards the OCR result the
  user already reviewed.
- The OpenCV path is the reason the tool runs on a CPU-only machine; the diffusion path is the quality
  path, not a hard dependency.
- Uploads are capped at 5 MB with a content check, and every edit is undoable.

**What I'd change.** Model warm-up dominates the first request; inference belongs in a job queue with
a warm worker, not in the request path.

---

## Stock Market News Aggregator

**Problem.** Market news is spread across NewsAPI, Finnhub and RSS feeds (Economic Times, Moneycontrol,
Reuters, Bloomberg), and every source has its own shape, rate limit and failure mode.

**App.** React 18 + TypeScript + Vite + Tailwind. Per-source toggles, configurable refresh (real-time
through 1 hour), title-based deduplication, per-article summaries through Gemini or a local Ollama
model, and market widgets on Recharts. Settings persist client-side; the build deploys as a static
site on Netlify.

**Engineering notes.**
- Each source adapter fails independently — one dead feed degrades the page instead
  of blanking it.
- The summariser is optional and degrades to the raw snippet when no model is reachable, which is the
  common case behind a corporate network.
- Ollama needs `OLLAMA_ORIGINS` set for browser calls; that got its own setup document because it
  fails silently otherwise.

**What I'd change.** Dedupe by title is naive — syndicated rewrites of the same story slip through.
URL canonicalisation plus embedding similarity would do better.

---

*Both written by [Gajanan Kadtan](https://github.com/The-kadtan2027).*
