# Engineering Documentation

> **Documentation where the limits are written down next to the claims.**
>
> Repository name: `build-notes` — the engineering documentation
> collection for projects I build, starting with AFTERTONE. Code is not
> stored here.

## Documentation map

| If you are looking for… | Go to |
|---|---|
| A project's documentation index | `<project>/README.md` — starting with [AFTERTONE](aftertone/README.md) |
| Architecture and lifecycle diagrams | `<project>/architecture/` |
| Security model and limitations | `<project>/security/` |
| Public policies — Terms, Privacy, Security, FAQ | `<project>/policies/` |
| Test plans and manual checklists | `<project>/testing/` |
| Status reports and verification records | `<project>/reports/` |
| Deployment and operations runbooks | `<project>/operations/` |
| Demo and development videos | [media-index.md](media-index.md) |

## Demo — watch it here

This video plays right here in the README. It is not just a link to
another page.

<video src="assets/aftertone-demo.mp4" controls width="100%">
  Your browser does not support the video tag. The demo file is
  <a href="assets/aftertone-demo.mp4">assets/aftertone-demo.mp4</a>.
</video>

> AFTERTONE demo — the retro terminal interface, pairing with a private
> invite code, and a message going from typed text to encrypted relay to
> printed delivery. No database. No message history.
>
> Note: this demo clip shows the interface and the message path. It does
> not show the fingerprint VERIFY step — see
> [aftertone/policies/FAQ.md](aftertone/policies/FAQ.md) and
> [aftertone/security/SECURITY.md](aftertone/security/SECURITY.md) for
> why that step matters before messaging is enabled.

## Projects

| Project | What it is | Documentation | Source code |
|---|---|---|---|
| AFTERTONE | Real-time, two-participant messaging with a retro CRT interface. Two devices pair with a private invite code, messages are encrypted in the browser, the relay sees only ciphertext. | [aftertone/README.md](aftertone/README.md) | Private, separate repository — not here, by design |

## AFTERTONE at a glance

| | |
|---|---|
| Live site | https://aftertone-u6rr.onrender.com/ |
| Tagline | Two devices. One private line. |
| Pairing | Private invite code (`MB88-XXXX-XXXX`) |
| Encryption | P-256 ECDH, HKDF-SHA256, AES-256-GCM, in the browser |
| Relay sees | Only ciphertext (`{ iv, ciphertext }`) |
| Database / message history | None, by design |
| Verification | Both sides must compare the fingerprint and press VERIFY before messaging is enabled |
| Production | On hold — the runbook is not a launch approval |
| Device testing | Real Safari, physical iOS, physical Android — not tested yet |
| Security audit | Not independently audited — status: **UNVERIFIED** |
| Policies | Terms placeholders (operator, contact, governing law) still to complete |

## What this repository is — and is not

| This repo is | This repo is not |
|---|---|
| Architecture, security, testing, operations, and public policies, in one place | A source-code repository |
| A paper trail: what was built, how it works, what was tested, what is still unknown | A launch approval, a security audit, or a production guarantee |
| Snapshots, preserved as received, with corrections recorded in the project index | A place for secrets or internal credentials |

## Folder structure

| Path | Contents |
|---|---|
| `README.md` | You are here — the map, then the demo |
| `assets/aftertone-demo.mp4` | The demo video embedded above, so it plays in the README |
| `media-index.md` | Other videos, indexed not committed |
| `aftertone/README.md` | Project index + known inconsistencies |
| `aftertone/architecture/` | Archify exports + commentary |
| `aftertone/security/` | Protocol + trust model |
| `aftertone/policies/` | Terms, Privacy, Security Notice, FAQ |
| `aftertone/testing/` | Device checklist |
| `aftertone/reports/` | Master checklist + test records |
| `aftertone/operations/` | Deployment + runbook |
| `aftertone/project-readme-snapshot.md` | Source repo README, as received |

## Three rules that keep it honest

| Rule | What it means |
|---|---|
| One document, one purpose | A document explains, records evidence, or gives instructions — not all three |
| Evidence apart from conclusions | Raw results (checklists, test counts) stay separate from the summaries that interpret them |
| Snapshots stay snapshots | Files copied from a source repository are preserved as received. Corrections go in the project index, not into the copy |

## Architecture and visual documentation

Diagrams are exported from [Archify](https://github.com/tt-a1i/archify) as
self-contained HTML files. They are kept unmodified, with their original
names, in `<project>/architecture/archify/`. Human-written explanation
lives beside them in `<project>/architecture/README.md`, so the exports
stay as source material and the commentary stays editable.

GitHub does not render these HTML pages — download the file and open it
in a browser. The diagrams are not embedded in this README.

## Media and demonstrations

Most videos are not committed to Git — [media-index.md](media-index.md)
lists each video with its project, type, description, and where it is
held. A video with no public link is listed as not hosted, and no URL is
given until one exists.

The one exception is the demo above: `assets/aftertone-demo.mp4` is
committed on purpose, so the README can play it inline instead of
linking out to another page.

## Adding to this repository

| To add… | Do this |
|---|---|
| A new project | Create a top-level folder named after the project with a `README.md` index. Add subfolders only when you have a file for them. Link the project from the tables above. |
| A new diagram | Put the original export in `<project>/architecture/archify/` and add a row in `<project>/architecture/README.md`. |
| A new report or test record | Put it in `reports/` or `testing/`, and note the date and the commit or version it describes. |
| A new policy page | Put it in `<project>/policies/` and link it from the project index. Note any placeholders that must be completed before publication. |
| A new video | Add a row to [media-index.md](media-index.md). Do not commit the video file — unless it is meant to play in a README, like the demo above. |
| Learning notes | Create a top-level `learning/` folder with an index when the first reusable note exists. |

## Repository status

| Item | Status |
|---|---|
| Projects documented | AFTERTONE only |
| Learning area | Does not exist yet — no content for it |
| AFTERTONE snapshots | Documents written on or around 2026-10-25, not re-checked against the source code from here |
| Physical-device testing | Not done |
| Production preparation | On hold |
| Cryptographic design | Not independently audited |
| Demo videos indexed | One, with no public link yet — plus the embedded demo above |
