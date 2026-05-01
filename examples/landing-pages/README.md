# Landing Page Examples

Full-page landing-page screenshots produced by the scarfold-line assembly's **Simple E2E Proof** station, one per project under owner `poulkjeldagercom1`. Each PNG was captured by a Playwright E2E test against the freshly-scaffolded landing page and embedded as a base64 `dataUri` in the project's `delivery` artifact (`data.screenshots[0]`).

## Inventory

| Project | File | Caption (from delivery artifact) |
|---|---|---|
| [agent-inbox](agent-inbox/landing-page.png) | `landing-page.png` (381 KB) | Agent Inbox landing page — full-page E2E screenshot |
| [agent-inbox-v5](agent-inbox-v5/landing-page.png) | `landing-page.png` (213 KB) | Full-page screenshot at 1440×900, signature move triggered (transcript + JSON envelope visible in hero pane) |
| [agent-inbox-v6](agent-inbox-v6/landing-page.png) | `landing-page.png` (376 KB) | Full-page screenshot from E2E run |
| [consent-service-v3](consent-service-v3/landing-page.png) | `landing-page.png` (617 KB) | Consent.io landing page — full-page E2E screenshot |
| [consent-service-v4](consent-service-v4/landing-page.png) | `landing-page.png` (445 KB) | Full-page screenshot captured during the scroll-video E2E test |
| [consent-service-v5](consent-service-v5/landing-page.png) | `landing-page.png` (855 KB) | Full-page E2E screenshot (The Standing Order) |
| [idealab](idealab/landing-page.png) | `landing-page.png` (902 KB) | IdeaLab landing page — full-page E2E screenshot (1280×5469) with hero, signature stamp, §1–§8 dossier sections |

## Not included

`agent-inbox-2` and `consent-service` are missing — neither project completed the E2E Proof station, so no `artifacts.json` (and therefore no screenshot) was produced.

## Source

Decoded from `user-data/owners/poulkjeldagercom1/projects/<project>/artifacts.json` → `data.screenshots[0].dataUri` on 2026-04-26. Each delivery artifact also carries `data.media[]` references to launch-video MP4s served at `/api/lives/media/<id>` — those are not included here as they live outside the artifact JSON.
