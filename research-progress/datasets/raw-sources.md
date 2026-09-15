# raw data sources: what's fetched, what exists, what's blocked

last updated: 2026-09-15

this documents the actual data acquisition state for the english→gloss training sets. raw files live in this folder but are excluded from git via `.gitignore` (they're large); the urls and licenses here are the source of truth for re-fetching.

---

## fetched and on disk (2026-09-15)

### aslg-pc12 (english → asl gloss parallel corpus, second release)

- **what**: `raw/corpus_0001.clean.en.txt` + `raw/corpus_0001.clean.asl.txt`, 1,060,672 line-aligned pairs
- **source**: author's page https://achrafothman.net/site/english-asl-gloss-parallel-corpus-2012-aslg-pc12/, direct download: https://www.dropbox.com/s/egdxi7gwhtricir/corpus_0001.clean.zip?dl=1 (shard 1 of 16)
- **total available**: 100m+ pairs across 16 shards, all direct dropbox links on the author's page
- **license**: released free online to promote asl processing research; cite: achraf othman and zouhour tmar, "english-asl gloss parallel corpus 2012: aslg-pc12, the second release", ICTA'13
- **gloss notation observed in the data**: pronouns as PRO-1(we), PRO-3(he); possessives POSS-3; question markers wh-q(when); negation as BENOT-, contract-style token fusion (SELFEBEPROUD-)
- **caveats (important)**:
  1. the pairs are **rule-generated**, not human-signer-verified. the glosses are systematic transformations of english (this is the known weakness of aslg-pc12). usable as large-scale seed material, but not as ground truth for grammaticality.
  2. the english side is **project gutenberg literary text** (shakespeare, verne, etc). our target domain is spoken lecture english. this is a domain mismatch that adaptionlabs adaptive data augmentation should be explicitly directed to fix (modern conversational + lecture register).
  3. the fused token style (e.g. SELFEBEPROUD-POSS-3) may need normalization into cleaner gloss conventions before fine-tuning.

### wlasl metadata (word-level asl video index)

- **what**: `raw/wlasl_v0.3.json`, 2,000 glosses, 21,083 video instances
- **source**: https://github.com/dxli94/WLASL, direct: https://raw.githubusercontent.com/dxli94/WLASL/master/start_kit/WLASL_v0.3.json
- **license**: dataset released for asl research (check repo readme for current terms); videos are youtube urls so attrition is expected
- **use**: vocabulary backbone for the gloss→motion lookup table; not parallel training data

---

## known but blocked / not yet fetched

### how2sign gloss annotations

- **status: not publicly released.** the repo's `research/datasets.md` says "gloss annotation available for part of the dataset". the actual situation per the maintainers (github issue how2sign/how2sign.github.io#5, last checked 2026-09-15): a gloss-annotated subset was promised in 2022, but an october 2022 maintainer comment says full-corpus gloss annotation is **suspended** due to lack of qualified annotators, and users were still asking for the glosses without answer as of 2025-07.
- **what IS available**: rgb/depth video, english sentence-level transcripts and csv metadata via https://how2sign.github.io
- **implication**: the asl seed strategy should NOT depend on how2sign glosses. english transcripts remain useful for input-domain data. **this contradicts the repo docs and needs flagging in the findings summary.**

### bsl corpus (open access elan annotations)

- **status: plan ready (see tr-03), bulk .eaf download not yet executed.** open-access .eaf files with free translation + idgloss tiers live on the ucl repository collection. next step: bulk download + pympi-eling extraction. cc by-nc 3.0 / cc by-sa 4.0.

### bobsl (bbc bsl)

- **status: requires form.** access requires emailing the terms of use agreement to bobsl.dataset@bbc.co.uk (see tr-03). action item for tess/her friend to send the form.

### bsl signbank

- **status: online dictionary, csv/export to investigate** (https://bslsignbank.ucl.ac.uk/). next step in tr-03's plan.
