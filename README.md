# FreshSight AI — Core

**Predicting hidden food spoilage before it happens.**
Team **Veyrix** · Everseen Computer Vision AI Hackathon (E-Cell IIT Bombay) · Problem Statement 4: *Predicting the Unseen: Pre-Visual Food Waste Mitigation*

> **Status: Round 1 proposal.** This repository holds the planned architecture and the Round 2 build plan. Items marked ✅ are done, 🔲 are planned. Performance numbers in our deck are **targets**, not yet measured results.

---

## The problem

One-third of all food produced is wasted. Fresh produce decays from the inside, so by the time a store clerk or a standard RGB camera sees mould, slime or browning, the product is already unsellable. Retail inventory still runs on blind FIFO and static "Best Before" dates that ignore each batch's real condition.

## The idea

FreshSight AI turns images of fresh produce into a **per-batch shelf-life forecast with uncertainty**, plus a recommended action for store staff:

| Risk level | Recommended action |
|---|---|
| High | Dynamic markdown (-20%) |
| Medium | Proactive reroute (e.g. in-store deli or kitchen) |
| Low | Hold and monitor |

It is a **software-first, hardware-agnostic** engine: the MVP ingests standard image/video from devices a store already has (a manager's phone, existing cameras) and is architected to accept multi-spectral inputs (NIR / thermal) later.

---

## System architecture

```
Image / video  ──►  Ingest API  ──►  Vision backbone  ──►  Fusion  ──►  Survival model  ──►  Risk score + action
 (phone, camera)    (FastAPI)        (CNN, PyTorch)        (storage     (Weibull:            (store dashboard)
                                                            context)     shelf-life + CI)
```

| Stage | What it does | Planned tech |
|---|---|---|
| 01 Ingest | Accepts standard image/video via REST upload (MVP); input schema reserved for multi-spectral bands | FastAPI, Pydantic |
| 02 Perceive | Extracts decay-related visual features | PyTorch CNN backbone (lightweight, edge-friendly) |
| 03 Fuse | Aligns visual features with storage context (temperature, humidity, time in storage) | Cross-attention Transformer (planned) |
| 04 Predict | Probabilistic remaining shelf-life with confidence intervals | Weibull survival regression (`lifelines` / PyTorch) |
| 05 Act | Maps risk score to markdown / reroute / hold | Rules engine |
| Dashboard | Per-batch view: shelf-life, risk, recommended action | Next.js + Tailwind |

**Supporting stack:** Python 3.11, PyTorch, scikit-learn, FastAPI, Next.js, Docker, GitHub Actions.

---

## Data

Prototype data: public **strawberry and avocado ripening dataset** (four stages: unripe, partially ripe, ripe, rotten), Mendeley Data, DOI `10.17632/zysvgmxcyz.1`. Check the dataset's licence before redistribution; we do not include the data in this repo.

Known limitations we are designing around:
- Labels are **stages, not days**. We map stages to approximate days-to-spoilage; this mapping is our own assumption and will be documented and stress-tested.
- Most images in the dataset are **augmented copies** of a small number of originals. We split train/validation/test by **original image** to avoid leakage.
- The dataset is RGB. NIR / thermal sensing is a **future input**, not part of the MVP.

---

## Round 2 plan

| Phase | Goal | Deliverable | Status |
|---|---|---|---|
| 0 | Repo, environment, data download scripts | Reproducible setup (`make setup`) | 🔲 |
| 1 | Baseline classifier on ripening stages (leak-free split) | Notebook + metrics report | 🔲 |
| 2 | Stage → days-to-spoilage mapping and Weibull survival model | Shelf-life + confidence interval per sample | 🔲 |
| 3 | FastAPI service: `POST /predict` (image in, shelf-life + risk + action out) | Running API + OpenAPI docs | 🔲 |
| 4 | Next.js store dashboard consuming the API | Demo UI: batch card, risk gauge, markdown button | 🔲 |
| 5 | Contextual fusion with simulated storage telemetry (labelled as simulated) | Cross-attention module + ablation | 🔲 |
| 6 | Evaluation and honest reporting | MAE (days), high-risk recall, calibration of intervals | 🔲 |

**Stretch goals:** video input with frame aggregation; second produce type; multi-spectral input adapter; handheld scanner / ERP integration stubs.

### Evaluation targets (to be confirmed, not yet achieved)
- Shelf-life MAE ≈ ±0.35 days
- High-risk class recall ≈ 91%, flagging spoilage ~48 hours ahead

---

## Proposed repository layout

```
FreshSight-AI-Core/
├── README.md
├── data/                # scripts only, no raw data committed
├── notebooks/           # exploration and baselines
├── src/
│   ├── ingest/          # FastAPI app
│   ├── models/          # backbone, fusion, survival model
│   ├── actions/         # risk → markdown / reroute / hold
│   └── eval/            # metrics and plots
├── dashboard/           # Next.js app
├── docker-compose.yml
└── .github/workflows/   # lint + tests
```

---

## Team

**Team Veyrix** — Everseen Computer Vision AI Hackathon, E-Cell IIT Bombay.

## Licence

To be decided (MIT planned for our code). Third-party datasets keep their own licences.
