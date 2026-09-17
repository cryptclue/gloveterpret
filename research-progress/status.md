# research progress status

updated: 2026-09-15

| track | status | notes |
|---|---|---|
| tr-01 prior art deep dive | in progress | researching |
| tr-02 evaluation methodology | done | completed findings and concrete evaluation spec in tr-02-evaluation-methodology.md |
| tr-03 bsl parallel corpus | done | completed parallel corpus research and pipeline spec in tr-03-bsl-parallel-corpus.md |
| tr-04 boundary detection | done | completed findings, comparison table, and recommended architecture in tr-04-boundary-detection.md |
| tr-05 asl dataset access | in progress | researching |
| tr-06 inference layer | done | completed findings, latency budget, structured output, and recommended setup in tr-06-inference-layer.md |
| tr-07 3d hand + three.js | done | completed model sourcing, three.js renderer architecture, and two-hands verdict in tr-07-3d-hand-rendering.md |
| tr-08 asta sweep | queued | route changed: asta cli (github.com/allenai/asta-plugins, installed in sandbox) via device-code login, replaces browserbase scraping |
| tr-09 adaptionlabs features | done | complete feature inventory + call sequence + cost estimate in tr-09-adaptionlabs-features.md |
| tr-11 bsl corpus extraction | in progress | worker stalled twice; relaunching with strict execute orders |
| tr-10 seed dataset v0 | done | completed seed-asl.jsonl (135 pairs), seed-bsl.jsonl (135 pairs), and seed-conventions.md |

## changelog

- 2026-09-17: tr-09 report recovered from worker result and written to disk (worker hit tool limit before saving). tr-08 re-planned around the asta cli.
- 2026-09-17: worker hygiene note: tr-01/tr-05 continuations ran 2 days without output; stopping and relaunching.
- 2026-09-15: tr-10 seed dataset v0 constructed and validated.
- 2026-09-15: tr-07 3d hand rigging and three.js rendering research completed.
- 2026-09-15: tr-06 inference layer research completed.
- 2026-09-15: tr-03 bsl parallel corpus research completed.
- 2026-09-15: tr-02 evaluation methodology research completed.
- 2026-09-15: tr-04 boundary detection research completed.
- 2026-09-15: review note: 81/135 seed pairs have identical asl and bsl gloss; needs a divergence revision pass before adaptionlabs upload.
- 2026-09-15: spec sheet written, tracks tr-01 through tr-06 launched.
