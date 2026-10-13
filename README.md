# MediGuard

**On-device medication safety. Reads your pill bottles, flags dangerous interactions, and shows the real 3D molecule of every medicine.**

MediGuard is a single-page web app that helps someone on several medications see, in plain language, how their drugs interact — and inspect the actual molecular structure of each one. Everything runs **in the browser**: label reading (OCR), the interaction check, and the molecular rendering. Nothing about your medicines is uploaded.

---

## Features

| Feature | What it does | Free (per day) | Pro |
|---|---|---|---|
| **Photo capture** | Photograph a pill bottle/strip → on-device OCR reads the drug name | 3 scans | Unlimited |
| **Type to add** | Add a medicine by name (autocomplete over the reference set) | Unlimited | Unlimited |
| **Interaction check** | Every pair checked against a built-in reference set, with a 1–5 severity scale | Unlimited | Unlimited |
| **Inspect + 3D molecule** | Drug monograph plus the real 3D structure, rotatable | 5 inspections | Unlimited |
| **Daily schedule** | Groups medicines into morning/afternoon/evening/night | Unlimited | Unlimited |

Everything is present in the free tier; Pro only removes the daily caps.

---

## The 3D molecule viewer

The inspect view renders each medicine's **real structure** in WebGL (Three.js):

- On the **deployed site**, it fetches the exact 3D structure from **PubChem** (the public NIH chemistry database) by name or CID — free, no API key. Atoms are coloured by the standard CPK scheme (carbon grey, oxygen red, nitrogen blue, hydrogen white).
- **Aspirin** ships as an **offline** structure so the viewer always has something real to show without internet.
- Privacy note: the PubChem lookup sends only the **drug name/CID** — never your personal regimen. It needs internet; the offline library covers the bundled example.

---

## Tech stack

- **Plain HTML / CSS / JS** — one file, no build step.
- **[Three.js](https://threejs.org/)** (r128) — real WebGL 3D molecule rendering.
- **[Tesseract.js](https://tesseract.projectnaptha.com/)** — on-device OCR for pill-bottle labels.
- **[PubChem PUG REST](https://pubchem.ncbi.nlm.nih.gov/)** — real 3D structures (public, free, no key).
- **Google Fonts** — Sora + Space Mono.

No backend, no API keys, no cost.

---

## Project structure

```
mediguard/
├── index.html    # the entire app (landing hero + app)
└── README.md
```

---

## Deploying to Netlify

1. Push the contents of this folder to a GitHub repo.
2. Netlify: **Add new site → Import an existing project → GitHub** → pick the repo.
3. Leave build command and publish directory **empty** (static site).
4. Deploy. Your site is live at `https://<your-site>.netlify.app/`.

Or drag the folder onto Netlify's manual-deploy drop zone.

---

## Configuration

Near the top of the `<script>` in `index.html`:

```js
var CONFIG = { PRO_CODE:"MEDIGUARD-PRO-2026", LIMITS:{scan:3, inspect:5} };
```

- **Pro unlock code** → `PRO_CODE`. Enter it via the **Unlock Pro** button (case/space/dash-insensitive).
- **Daily limits** → `LIMITS`.
- **Add medicines / interactions** → extend the `MEDS` object and the `IX` array.

---

## Browser support

- WebGL (the 3D viewer) and OCR work in any modern desktop browser and most phones.
- The PubChem structure fetch needs internet; without it, bundled structures still render.

---

## Important disclaimer

MediGuard is a **demo**. Its interaction data is a curated reference set of common, clinically significant interactions — it is **not exhaustive** and is **not medical advice**. Always confirm with a pharmacist or doctor, and never start or stop a medicine based on this tool.

---

*Built to run on your device. Your medicine data never leaves your machine.*
