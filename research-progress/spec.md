# research progress spec

this folder is the working space for the in-depth research phase of glovterpreter. everything here is original research output that builds on the existing `research/` folder in the repo root. the repo docs define the problem; this folder closes the open questions they raise.

---

## ground rules

1. **every factual claim gets a source url.** unsourced claims are marked clearly as hypothesis or open question, never presented as fact.
2. **file structure per track:** open questions → findings (sourced) → implications for the build → open items for the next pass.
3. **contradictions are flagged, not hidden.** if research here contradicts something in `research/`, the track file says so explicitly.
4. **all lowercase, no em dashes.** project convention.
5. **sources log:** every source actually used gets appended to [`sources.md`](./sources.md) with a one-line description and access date.
6. **status:** [`status.md`](./status.md) tracks every track: queued / in progress / done. update it as work lands.

---

## tracks

| id | track | question it answers |
|---|---|---|
| tr-01 | prior art deep dive | what exactly did speak2sign3d, signdiff, signavatar, handscribe, csf, gloss2text etc actually do, and what can we reuse? is any of our "four gaps" already claimed by someone newer? |
| tr-02 | evaluation methodology | how do we know if our english → gloss output is any good? automatic metrics, back-translation checks, deaf community validation protocol, motion quality metrics |
| tr-03 | bsl parallel corpus construction | how do we build english → bsl gloss training pairs from the bsl corpus, bsl signbank, and the bsl syntax project? what's downloadable, licensed how, in what format? |
| tr-04 | utterance boundary + semantic completeness | how do we reliably know an utterance is semantically complete? assemblyai vs speechmatics exact behavior, end-of-turn prediction literature, completeness classifier design |
| tr-05 | asl dataset access | how do we actually get how2sign / wlasl / asl3dword data, and how do we convert sign video into joint angle lookup tables? |
| tr-06 | inference layer | what small model + serving setup makes sense for english → gloss in a browser demo? adaptionlabs capabilities, onnx/webllm in-browser options, latency budget |
| tr-07 | 3d hand rigging + three.js rendering (queued, next batch) | rigged low-poly hand sourcing, skinnedmesh + bones, keyframe interpolation approach |
| tr-08 | asta / allenai scientific corpus sweep (queued, needs browserbase session with logged-in asta account) | citation traversal starting from the prior art papers |

---

## tooling map

| tool | access | role |
|---|---|---|
| tavily | `$TAVILY_API_KEY` in bash | discovery searches, finding papers, datasets, docs |
| firecrawl | `$FIRECRAWL_API_KEY` in bash | full-content scraping of paper pages, corpus docs, tool docs |
| browserbase | `$BROWSERBASE_API_KEY` + native browser tools | asta.allen.ai (logged-in account), js-heavy sites, anything needing interaction |
| adaptionlabs | `$ADAPTIONLABS_API_KEY` | dataset upload + augmentation + fine-tuning. **phase 2 only**: research docs first, no credit spend until the seed dataset exists and tess approves |

notes: all of these are https apis, so they work from the sandbox. never print full key values anywhere, including in research files.

---

## working loop

1. each track is researched in depth: search → scrape sources → extract → write the track file → log sources → update status.
2. when all current tracks land, findings get reviewed against the repo docs, and a findings summary (`findings.md`) is written for the repo owners: what changed, what's confirmed, what's newly known.
3. new tracks get added here as they're identified, never worked off-book.

---

## repo state note

this folder lives in a local clone of `github.com/kqrla/gloveterpret`. fork + push setup is pending github credentials from tess; until then, all work is local and committed on a branch (`research-progress`).
