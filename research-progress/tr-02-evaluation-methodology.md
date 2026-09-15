# tr-02 evaluation methodology for english to asl/bsl gloss output

this document defines the research findings and concrete evaluation specification for evaluating English to ASL and BSL gloss output, as well as downstream 3d motion rendering for glovterpreter.

---

## open questions

1. how accurately do conventional automated text metrics (bleu, chrf) correlate with human deaf signer judgments when evaluating text-to-gloss outputs?
2. can round-trip back-translation (gloss to english) serve as a reliable automated proxy for gloss semantic accuracy without masking grammatical errors?
3. how can rule-based nlp techniques programmatically verify sign language grammatical structures (topic-comment ordering, temporal fronting, spatial referent loci) in generated glosses?
4. what 3d kinematic and distribution metrics (mpjpe, pck, fgd) best capture sign motion quality when transitioning from gloss tokens to three.js rendering keyframes?
5. what ethical and methodological protocols must govern deaf community validation for real-time sign language interpretation tools?

---

## findings

### 1. automatic metrics for sign language translation and text-to-gloss

#### bleu and its known weaknesses for gloss
bleu (bilingual evaluation understudy; papineni et al., 2002: https://aclanthology.org/P02-1040.pdf) remains the standard default metric in machine translation, measuring exact n-gram precision between candidate text and reference translations. in sign language translation, sacrebleu is recommended with internal tokenization explicitly disabled (`--tok none` or raw space-separated tokens) to prevent tokenizer artifacts from altering gloss notations such as hyphens, compound joins, or indexing markers (müller et al., acl 2023: https://aclanthology.org/2023.acl-short.60.pdf).

despite its ubiquity, bleu exhibits severe structural flaws when applied to sign language glosses:
- **exact-match vocabulary penalty:** bleu heavily penalizes valid lexical variations and glossing synonym choices (e.g. `ME` vs `IX-1`, `CAR` vs `VEHICLE-DRIVE`, `PAST` vs `BEFORE`).
- **non-manual and spatial blindness:** standard bleu treats gloss tokens as flat text strings, failing to score non-manual facial markers, spatial loci, or directional agreement (kim et al., lrec-coling 2024: https://aclanthology.org/2024.lrec-main.1289).
- **structural reordering penalty:** sign languages allow flexible topic-comment or subject-verb-object permutations. bleu's n-gram matching harshly penalizes valid topic-comment reorderings if the reference uses SVO order.
- **poor human correlation:** empirical studies confirm that bleu correlates weakly or inconsistently with deaf native signer judgments for gloss output (müller et al., acl 2023: https://aclanthology.org/2023.acl-short.60.pdf).

#### chrf and chrf++
chrf (character n-gram f-score; popović, 2015: https://aclanthology.org/W15-3049.pdf) and chrf++ (popović, 2017: https://aclanthology.org/W17-4770.pdf) evaluate overlap using character n-grams alongside optional word n-grams.
- **advantage for gloss:** chrf is significantly more robust to gloss morphological variations, compounding (e.g. `CAT+BLACK`), directional affixation (`GIVE-1-2` vs `GIVE-2-1`), and hyphenation inconsistencies. character-level sub-token matching avoids the all-or-nothing exact word match penalty of bleu (müller et al., acl 2023: https://aclanthology.org/2023.acl-short.60.pdf).

#### chronolog / temporal sequence ordering metrics
token sequence order in sign language glosses directly encodes temporal timelines and event chronology (e.g., temporal marker fronting where time indicators precede action verbs). standard string overlap metrics fail to measure sequence order preservation. sequence alignment metrics, dynamic time warping (dtw), Levenshtein edit distance, and rank correlation metrics (e.g., Kendall's tau over temporal event positions) assess whether temporal event sequences and chronological ordering are preserved across translation (mcmillan et al., acl 2004: https://aclanthology.org/C04-1108.pdf).

#### cosine embedding similarity
dense embedding metrics (e.g., sentence-bert / glossbert / mE5; reimers & gurevych, emnlp 2019: https://aclanthology.org/D19-1410.pdf) map generated gloss sequences and reference glosses into a continuous vector space and compute cosine similarity `cos(v_gen, v_ref)`.
- **advantage:** captures semantic equivalence across synonym substitutions or annotation discrepancies (e.g., mapping `STORE ME GO` and `ME GO MARKET` to adjacent embedding points), mitigating exact-match string penalties.

#### gloss-specific metrics from how2sign
the how2sign dataset benchmark (duarte et al., cvpr 2021: https://arxiv.org/html/2008.08143v2) established a multi-tier evaluation framework:
- text and gloss evaluation: bleu-1, bleu-2, bleu-3, bleu-4, rouge-l, meteor, and word error rate (wer) / position-independent error rate (per).
- visual and pose evaluation: percentage of detected keypoints (pdk) and percentage of correct keypoints (pck) using thresholds of 10% torso diameter for hand keypoints and 20% for body keypoints.

#### wmt sign language translation tasks (2022-2024)
the wmt22, wmt23, and wmt24 shared tasks on sign language translation (müller et al., wmt 2022: https://aclanthology.org/2022.wmt-1.97.pdf; de coster et al., wmt 2023: https://hal.science/hal-04287113v1/file/2023.wmt-1.4.pdf) standardized evaluation using:
- automated metrics: sacrebleu (`--tok none`), chrf++, and bleurt (sellam et al., acl 2020: https://aclanthology.org/2020.acl-main.704.pdf).
- key finding: wmt organizers noted that string-based metrics are insufficient for target sign representations, recommending neural learned metrics (bleurt/comet) alongside chrf, backed by large-scale human evaluation.

#### rwth phoenix-2014t benchmark protocols
the rwth phoenix-2014t benchmark (camgoz et al., cvpr 2018: https://openaccess.thecvf.com/content_cvpr_2018/papers/Camgoz_Neural_Sign_Language_CVPR_2018_paper.pdf; camgoz et al., ieee tpami 2020: https://arxiv.org/abs/2003.13830) uses standard dataset splits (7,096 train, 519 dev, 642 test) and evaluates two distinct protocols:
1. sign language recognition (slr; video -> gloss): evaluated via word error rate (wer), measuring insertion, deletion, and substitution errors.
2. sign language translation (slt; video -> text or gloss -> text): evaluated via bleu-1, bleu-2, bleu-3, bleu-4, and rouge-l.

---

### 2. back-translation as a check

back-translation as an evaluation check involves taking the model's generated gloss output (`gloss_gen`), passing it through a pre-trained gloss-to-english translation model (`gloss2text`), and comparing the resulting back-translated english (`text_back`) to the source english (`text_src`) using standard natural language metrics (bleurt, comet, bertscore, bleu).

#### prior usage in sign language literature
back-translation has been utilized primarily as a synthetic data augmentation strategy in sign language machine translation (fayyaz et al., emnlp 2024: https://arxiv.org/html/2407.01394v2; zhan et al., 2023: https://www.cs.jhu.edu/~xzhan138/papers/SLMT_Book_G2T.pdf). as an evaluation mechanism, round-trip back-translation allows researchers to leverage high-performing pretrained neural metrics (e.g. bleurt; sellam et al., acl 2020: https://aclanthology.org/2020.acl-main.704.pdf) that require natural spoken language inputs and cannot process raw gloss syntax directly.

#### viability and advantages
- **semantic validation in natural text space:** back-translation bypasses gloss notation discrepancies by projecting generated gloss back into english text, where neural semantic metrics (bleurt/comet) accurately assess sentence-level semantic preservation.
- **fully automated evaluation pipeline:** enables automated continuous integration testing without requiring manual gloss annotation for new test phrases.

#### pitfalls and failure modes
1. **error masking / failure cancellation:** a powerful `gloss2text` model (e.g. LLM-based translator) can fix or "guess" ungrammatical or missing gloss tokens, generating fluent English and masking severe grammatical flaws in the gloss output.
2. **compound error propagation:** if the back-translation model makes a mistake, a grammatically perfect gloss output can produce a low back-translation score, falsely penalizing the translation model.
3. **functional word hallucination asymmetry:** sign language glosses omit english function words (articles, copulas, auxiliary verbs). back-translation models must hallucinate these missing elements, leading to stylistic mismatches and false penalties under exact-string metrics.

---

### 3. grammar-specific checks & programmatic verification

grammaticality in sign language glossing cannot be captured by token overlap alone. programmatic verification requires rule-based dependency checks and syntax parsers (moryossef et al., at4ssl 2023: https://aclanthology.org/2023.at4ssl-1.3.pdf; zhao et al., 2000: https://aclanthology.org/2023.at4ssl-1.3.pdf).

#### verifiable grammatical rules in gloss output

1. **temporal marker fronting:**
   - *rule:* time expressions (e.g. `YESTERDAY`, `TOMORROW`, `PAST`, `FUTURE`, `NOW`, `NEXT-WEEK`) must be fronted to the beginning of the sentence (position 0 or 1).
   - *programmatic check:* inspect source english parse trees for temporal modifiers; verify that corresponding temporal gloss tokens appear before subject/verb tokens in the output sequence.

2. **topic-comment ordering:**
   - *rule:* in topic-comment structures, the topic (often direct object, location, or emphasized noun phrase) precedes the comment (action/verb phrase).
   - *programmatic check:* map dependency parse tags (e.g., spacy `dobj`, `pobj`) from source text; check that object gloss tokens precede main verb gloss tokens when topicalization cues exist.

3. **spatial referent locus indexing & agreement:**
   - *rule:* spatial loci assigned to entities (e.g. `IX-a` for PERSON_A, `IX-b` for PERSON_B) must remain consistent throughout the utterance, and directional verbs (e.g. `a-GIVE-b`) must preserve source-to-target spatial locus indices.
   - *programmatic check:* track index tokens (`IX-a`, `IX-1`, etc.) across the gloss token list and enforce referential consistency and directional verb matching.

4. **negation placement:**
   - *rule:* negative markers (`NOT`, `NEVER`, `NONE`) in ASL/BSL typically follow the verb or appear at the end of the clause, accompanied by non-manual headshakes.
   - *programmatic check:* verify that negation gloss tokens do not blindly copy English pre-verbal placement (`DO NOT GO` -> `GO NOT` or `GO NONE`).

---

### 4. motion-level evaluation

because glovterpreter ultimately renders 3d skeletal animation in three.js, text-level metrics must be complemented by 3d gesture and motion quality evaluation (yoon et al., icra 2020: https://arxiv.org/abs/2008.08143v2; yu et al., eccv 2024 signavatars: https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/00653.pdf).

#### fréchet distance variants
- **fréchet gesture distance (fgd):** computes the fréchet distance between feature distributions of real ground-truth motion clips and synthesized 3d keyframe sequences in a latent feature space (extracted using a motion encoder or ST-GCN network; yoon et al., icra 2020: https://arxiv.org/abs/2008.08143v2). lower fgd indicates that generated motion matches the distribution and style of natural signing.
- **fréchet inception / video distance (fid / fvd):** measures distributional similarity over rendered animation frame sequences.

#### joint and keypoint metrics
- **mean per-joint position error (mpjpe):** measures average Euclidean distance (in mm or normalized units) between predicted 3d joint locations and ground truth pose keyframes (oecd.ai pose benchmark: https://oecd.ai/en/catalogue/metrics/mean-per-joint-position-error-mpjpe).
- **percentage of correct keypoints (pck):** measures the proportion of predicted 3d joints within a specified tolerance distance (e.g. PCK@0.05 or within 10% torso diameter for finger joints; duarte et al., cvpr 2021: https://arxiv.org/html/2008.08143v2).
- **mesh vertex metrics:** procrustes-aligned mean per-joint position error (pa-mpjpe) and mean per-vertex error (pa-mpvpe) for full hand/body mesh evaluation (yu et al., eccv 2024: https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/00653.pdf).

#### physical plausibility & kinematic smoothness
- **jerk & velocity metrics:** compute the 3rd derivative of 3d joint rotations/positions over time to detect motion jitter, sudden snapping, or unnatural freezes during keyframe interpolation.

---

### 5. human evaluation protocols & deaf community validation

validating sign language technology requires rigorous, ethical, and community-centered human evaluation protocols established in sign language machine translation literature (roelofsen et al., at4ssl 2021: https://aclanthology.org/2021.mtsummit-at4ssl.9.pdf; tran & bragg, assets 2023: https://dl.acm.org/doi/fullHtml/10.1145/3597638.3614507; bragg et al., cacm 2019: https://dl.acm.org/doi/10.1145/3308827).

#### human evaluation methodology
1. **evaluator demographics:** evaluation must be conducted by fluent, native or early-deaf signers (ASL and BSL native/fluent signers for respective models). hearing non-signers or second-language learners are unsuitable for validating grammatical correctness or nuance.
2. **multi-dimensional evaluation criteria (5-point likert scale):**
   - **grammaticality:** does the gloss/rendered sign conform to ASL/BSL syntax and natural signing rules?
   - **semantic fidelity:** does the output accurately convey the full meaning of the source English audio without omission or distortion?
   - **naturalness & smooth articulation:** is the 3d skeletal motion fluid, naturally paced, and free of robotic artifacts?
   - **comprehensibility:** can a signer understand the intended message easily on first viewing?
3. **ethical and participatory principles:**
   - compensate deaf evaluators at professional consulting rates.
   - provide flexible video/animation playback controls (speed adjustment, multi-angle view, replay).
   - follow deaf-led co-design practices, involving deaf researchers in rubric creation and qualitative feedback interpretation.

---

## proposed evaluation spec for glovterpreter

based on the research findings, the following concrete evaluation specification is proposed for glovterpreter across all pipeline stages:

```
[ English Speech / Text ]
          │
          ▼
[ Stage 1: Text-to-Gloss Model ] ────► (A) Automated Metric Suite (chrF++, Cosine Sim, SacreBLEU)
          │                            (B) Programmatic Grammar Rule Checker
          ▼                            (B.1) Round-Trip Back-Translation Check
[ Stage 2: Gloss-to-Joint Angles ] ───► (C) Kinematic & Pose Metrics (MPJPE, PCK, Jerk/Smoothness)
          │
          ▼
[ Stage 3: Three.js 3D Rendering ] ──► (D) Deaf Community Human Validation (Likert 1-5 Protocol)
```

---

### (a) automatic metrics computable now on a held-out set

for offline evaluation of fine-tuned English -> ASL and English -> BSL gloss models on held-out test sets, glovterpreter will compute a multi-metric automated report:

1. **chrF++ (primary string metric):**
   - computed via sacrebleu (`chrf` with word n-grams `nw=2`).
   - selected over BLEU as primary lexical metric due to morphological awareness and robustness to gloss hyphens and compound variations.
2. **SacreBLEU (`--tok none` baseline):**
   - computed on raw whitespace-separated gloss tokens without internal English tokenization.
   - reported for backward compatibility with published SLT baselines.
3. **Cosine Embedding Similarity (semantic metric):**
   - computed using a fine-tuned Sentence-BERT / GlossBERT embedding model: `cos_sim(embed(gloss_gen), embed(gloss_ref))`.
   - measures semantic match independent of exact gloss token choices.
4. **Sequence Alignment / Edit Distance (temporal order metric):**
   - Normalized Levenshtein distance and Kendall's tau rank correlation over gloss positions to quantify sequence order preservation.

---

### (b) grammar rule checker design for asl and bsl gloss conventions

a programmatic rule-checker module (`glovterpret-eval-grammar`) will parse the source English dependency tree alongside the generated gloss token list to output a pass/fail compliance score (0.0 to 1.0) across four core grammatical rules:

#### 1. temporal fronting verifier
- *input:* source English spacy parse + output gloss token list.
- *logic:* extract temporal entities and adverbs from source (`DATE`, `TIME`, `ADV` with temporal dep). verify that corresponding gloss tokens (`YESTERDAY`, `TOMORROW`, `PAST`, `FUTURE`, `NOW`) occur within the first 2 positions of the output gloss sequence.
- *score:* 1 if temporal marker is correctly fronted; 0 if misplaced; 1 if no temporal marker present.

#### 2. topic-comment reordering verifier
- *input:* source English dependency tree (`dobj`, `pobj`) + output gloss token list.
- *logic:* if the source sentence contains a topicalized object or location modifier, verify that the object gloss token precedes the main action verb gloss token in the generated gloss.
- *score:* ratio of correctly ordered topic-comment clauses.

#### 3. spatial index consistency checker
- *input:* output gloss token list.
- *logic:* scan for indexing tokens (`IX-a`, `IX-b`, `IX-1`, `IX-2`). verify that every assigned index is referenced consistently without orphan index jumps or conflicting assignment to multiple entities within the same utterance.
- *score:* 1 - (inconsistent_indices / total_indices).

#### 4. round-trip back-translation sanity check
- *logic:* pass `gloss_gen` through a pre-trained `Gloss2Text` model to produce `text_back`. compute `BLEURT-20` score between `text_src` and `text_back`. flag any sentence scoring `BLEURT < 0.35` for manual review.

---

### (c) human / deaf community validation protocol sketch

a formal validation protocol for glovterpreter releases, designed according to Deaf-led evaluation best practices:

1. **participant cohort:** 6-10 fluent Deaf signers (3-5 native ASL signers, 3-5 native BSL signers).
2. **test dataset:** 50 benchmark utterances per language (25 everyday conversational, 15 instructional/technical, 10 complex narrative).
3. **evaluation interface & task:**
   - web-based evaluation portal displaying source English sentence (optional toggle), generated gloss, and interactive three.js 3d low-poly hand/arm animation.
   - playback controls: speed slider (0.5x, 0.75x, 1.0x), camera rotation, replay button.
4. **scoring dimensions (1 to 5 scale):**
   - *grammaticality:* "does the sign order and movement conform to proper ASL/BSL grammar?"
   - *semantic accuracy:* "does the signed output convey the exact meaning of the English sentence?"
   - *motion naturalness:* "is the 3d hand motion fluid, clear, and natural?"
   - *comprehensibility:* "how easily did you understand the signing on first playback?"
5. **qualitative feedback:** open text box for reporting specific signing errors (e.g., incorrect handshape, awkward transition, wrong sign choice).
6. **compensation & ethics:** fair hourly consulting rate compensation, full informed consent, data privacy assurance.

---

### (d) building a small held-out benchmark from how2sign (asl) and bsl corpus (bsl)

to enable reproducible evaluation for glovterpreter, we define an automated construction pipeline for a held-out benchmark set (`glovterpret-benchmark-v1`):

#### 1. ASL held-out benchmark (from How2Sign)
- **source:** How2Sign sentence-level re-aligned test partition (duarte et al., cvpr 2021: https://arxiv.org/html/2008.08143v2).
- **selection criteria:** filter for 250 high-quality sentence clips containing complete parallel English transcripts, verified gloss annotations, and 3d panoptic pose keyframes.
- **filtering rules:**
  - sequence length: 3 to 15 gloss tokens per sentence.
  - vocabulary coverage: verify that gloss tokens exist within glovterpreter's 3d lookup dictionary or fingerspelling fallback.
  - domain diversity: select 100 conversational clips, 100 instructional clips, and 50 narrative clips.
- **format:** JSON records containing `id`, `english_transcript`, `asl_gloss_reference`, and `3d_joint_keyframes`.

#### 2. BSL held-out benchmark (from BSL Corpus)
- **source:** BSL Corpus ELAN annotation files (bslcorpusproject.org; bsl linguistics research).
- **selection criteria:** extract 200 sentence-level translation pairs by aligning ELAN gloss annotation tiers with English translation tiers.
- **curation steps:**
  - standardize gloss notation: normalize regional variants to BSL SignBank citation forms.
  - clean sentence boundaries: verify complete utterance boundaries without mid-phrase cuts.
- **format:** JSON records containing `id`, `english_translation`, `bsl_gloss_reference`, and regional variant tags (e.g. London / Manchester / Bristol).

---

## open items

1. implement `glovterpret-eval-grammar` script in Python using spacy for automated CI grammar testing on held-out sets.
2. fine-tune a small lightweight `Gloss2Text` model on How2Sign / Phoenix2014T for the automated back-translation sanity check pipeline.
3. extract 3D joint angle reference keyframes for the 250 How2Sign held-out clips to enable automated offline MPJPE and PCK computation.
4. establish contact with Deaf research advisors to review and finalize the Deaf community human evaluation protocol before phase 2 user testing.
