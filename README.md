# PDEOncology — Tumor Drug Penetration Simulator

[![Live Demo](https://img.shields.io/badge/demo-pdeoncology.com-4d9eff?style=flat-square)](https://pdeoncology.com)
[![Version](https://img.shields.io/badge/version-v0.4-3de383?style=flat-square)](https://pdeoncology.com)
[![License](https://img.shields.io/badge/license-MIT-f0a54a?style=flat-square)](LICENSE)
[![Built with](https://img.shields.io/badge/built%20with-vanilla%20JS-f05b5b?style=flat-square)]()

A browser-based platform for simulating tumor drug penetration using **reaction-diffusion partial differential equations (PDEs)**. All computation runs client-side in JavaScript — no server, no installation required.

🔗 **Live site:** [pdeoncology.com](https://pdeoncology.com)

---

## Overview

Drug resistance and treatment failure in solid tumors are often not purely pharmacological — they are **physical**. Even potent drugs can fail to reach the tumor core due to:

- Elevated interstitial fluid pressure (IFP)
- Dense extracellular matrix (ECM)
- Poor vascularisation

PDEOncology provides an interactive simulation environment to visualise these phenomena, enabling researchers and educators to explore how molecular weight, metabolic stability, and receptor expression shape drug distribution — without requiring a computational background.

---

## The PDE Model

Drug concentration C(x,y,t) evolves according to the **reaction-diffusion equation**:

```
∂C/∂t = ∇·(D(x,y)∇C) − λC − k·ρ(x,y)·C
```

| Symbol | Meaning | Typical range |
|--------|---------|---------------|
| `D(x,y)` | Spatially-varying diffusion coefficient | 0.005 – 0.25 |
| `λ` | Drug degradation / metabolic clearance rate | 0.001 – 0.05 |
| `k` | Cellular uptake rate | 0.01 – 0.15 |
| `ρ(x,y)` | Cell density field (radially graded) | 0.05 – 1.0 |
| `r` | Tumor radius in grid units | 10 – 38 px |

**Numerical method:** Explicit finite difference (FTCS scheme) on a uniform 80×80 grid, representing a 1 cm × 1 cm tissue cross-section. Stability enforced via CFL condition: `dt ≤ dx² / (4D)`.

---

## Features

### Simulation
- Real-time PDE solver running entirely in the browser
- Magma colormap heatmap with tumor boundary overlay
- Radial penetration curve with gradient rendering
- 4 key metrics: tumor average concentration, peak C, penetration depth, coverage %
- 3 delivery modes: IV infusion, vascular ring, intratumoral injection

### Diffusion Animation
- 20-frame animation player showing drug spread over time
- Play / Pause / Step / Scrub controls
- Adjustable playback speed (1× – 10×)

### AI Drug Input
- Describe a drug + tumor in plain English
- Claude API extracts biophysically grounded PDE parameters (D, λ, k)
- Automatic fallback to local database when no API key provided

### Compare Mode
- Simulate two drugs on the same tumor simultaneously
- Side-by-side heatmaps with individual metrics
- Overlay penetration curves (Drug A vs Drug B)
- Auto-generated comparison summary
- Export: CSV data, heatmap PNGs, curve PNG

### Drug Database
- 21 built-in drug × tumor combinations
- Covers: Doxorubicin, Paclitaxel, Cisplatin, Gemcitabine, Trastuzumab, 5-Fluorouracil, Temozolomide, Dacarbazine
- Tumor types: breast, pancreatic, glioma, melanoma, colorectal
- Searchable and filterable

### Results & Export
- Auto-generated report summary
- Export radial curve data as CSV
- Export concentration heatmap as PNG
- Print report

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML / CSS / JavaScript |
| Fonts | Space Mono + Inter (Google Fonts) |
| PDE solver | Custom FTCS finite difference (JS) |
| Visualisation | HTML5 Canvas API |
| AI integration | Anthropic Claude API (claude-sonnet) |
| CORS proxy | Cloudflare Workers (free tier) |
| Hosting | GitHub Pages |
| Domain | pdeoncology.com (IONOS) |

No frameworks. No build tools. One single `index.html` file.

---

## Project Structure

```
tumor-drug-penetration-model/
├── index.html          # Entire application (HTML + CSS + JS)
├── CNAME               # Custom domain config for GitHub Pages
├── README.md           # This file
├── requirements.txt    # Python dependencies (for Colab notebook)
├── Tumor_Drug_Simulation_Core.ipynb   # Original Colab prototype
└── Drug_Diffusion_Model_test01.ipynb  # Early experiments
```

---

## Getting Started

### Run locally
```bash
# No build step needed — just open the file
open index.html
# or serve with any static server:
python -m http.server 8000
```

Then visit `http://localhost:8000`

### Deploy to GitHub Pages
1. Fork this repository
2. Go to Settings → Pages
3. Set Source: Deploy from branch → main → / (root)
4. Your site will be live at `https://yourusername.github.io/tumor-drug-penetration-model`

### Use AI Drug Input
1. Get an Anthropic API key from [console.anthropic.com](https://console.anthropic.com)
2. Set up a CORS proxy (see below) or use the built-in fallback
3. Enter your key in the AI Drug Input tab

### CORS Proxy Setup (Cloudflare Workers)
The Claude API cannot be called directly from the browser due to CORS restrictions. Deploy this worker:

```javascript
export default {
  async fetch(request) {
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type, x-api-key, anthropic-version',
        }
      });
    }
    if (request.method !== 'POST') {
      return new Response('PDEOncology API Proxy — OK', {
        headers: { 'Access-Control-Allow-Origin': '*' }
      });
    }
    const body = await request.json();
    const apiKey = request.headers.get('x-api-key');
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': apiKey,
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify(body)
    });
    const data = await res.json();
    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
      }
    });
  }
}
```

Then update the fetch URL in `index.html` to point to your worker.

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v0.4 | Mar 2026 | 20-frame diffusion animation, Compare tab, favicon, About tab with references and team |
| v0.3 | Mar 2026 | Full UI overhaul — Space Mono/Inter fonts, dark academic theme, mobile responsive |
| v0.2 | Mar 2026 | Cloudflare Worker, Claude API integration, AI fallback, drug database (21 entries), export |
| v0.1 | Mar 2026 | Initial release — PDE solver, heatmap, radial curve, 3 delivery modes |

---

## References

1. Jain RK. Transport of molecules in the tumor interstitium. *Cancer Research*, 47(12):3039–3051, 1987.
2. Nugent LJ, Jain RK. Extravascular diffusion in normal and neoplastic tissues. *Cancer Research*, 44(1):238–244, 1984.
3. Thurber GM, Schmidt MM, Wittrup KD. Antibody tumor penetration. *Advanced Drug Delivery Reviews*, 60(12):1421–1434, 2008.
4. Chauhan VP et al. Delivery of molecular and nanoscale medicine to tumors. *Annual Review of Chemical and Biomolecular Engineering*, 2:281–298, 2011.
5. Tannock IF et al. Limited penetration of anticancer drugs through tumor tissue. *Clinical Cancer Research*, 8(3):878–884, 2002.
6. Minchinton AI, Tannock IF. Drug penetration in solid tumours. *Nature Reviews Cancer*, 6(8):583–592, 2006.

---

## Team

**Genius** — Technical Development, Website, UI Engineering  
*NSFZ · IB*

**Baichi** — Research, Chatting, FaceTime  
*Wycombe Abbey · UK*

> 祝我们都能去自己想上的学校！ @MIT @Cambridge

---

## Disclaimer

For research and educational use only. Results are model approximations and should not be used for clinical decision-making.

---

## License

MIT License — feel free to fork, modify, and build upon this project.
