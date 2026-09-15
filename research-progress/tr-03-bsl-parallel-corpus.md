# tr-03 bsl parallel corpus construction

## open questions
- what annotation layers exist in the bsl corpus, is raw annotation data downloadable, under what license, in what formats, and does it include paired english translations?
- which bsl corpus tasks yield usable english to gloss pairs, and is registration or restricted access required?
- how is bsl signbank structured, is there a data export or api, how are signs keyed by id-glosses, and how do id-glosses map into a training vocabulary?
- what data did the bsl syntax project produce and can any of it be used as translation training pairs?
- what existing parallel bsl translation datasets exist across academic research and competitive tasks?
- how do research papers handle bsl regional lexical variation across the 8 uk corpus sites, and what target vocabulary strategy should be selected for a v1 model?
- what is the practical alignment pipeline to convert elan .eaf annotations into sentence level english to gloss training pairs?

## findings

### 1. bsl corpus project annotation layers, formats, license, and downloadable data
- the bsl corpus contains 125 hours of digital video recordings from 249 deaf signers collected across 8 uk sites: london, bristol, birmingham, manchester, cardiff, newcastle, glasgow, and belfast (source: https://www.sign-lang.uni-hamburg.de/lr/compendium/corpus/bslcorpus.html).
- annotation layers in the elan .eaf files include dominant and non-dominant hand id-glosses (`rh-idgloss`, `lh-idgloss`), sentence-level english translations (`free translation`), phonological coding (`rh-handshape`, `lh-handshape`), clause structure (`clause`), and argument structure (`rh-argument`, `lh-argument`) (source: https://bslcorpusproject.org/wp-content/uploads/bslcorpus_annotationconventions_v3.0_-march2017.pdf).
- raw annotation files (.eaf xml format) are downloadable directly via the ucl library primo collection / cava repository without requiring a user license for open access files (source: https://ucl.primo.exlibrisgroup.com/discovery/collectiondiscovery?vid=44ucl_inst:ucl_vu2&collectionid=81354492040004761).
- narrative and lexical elicitation video data are released under creative commons attribution-noncommercial 3.0 / cc by-sa 4.0 licenses, whereas downloading restricted conversation and interview video footage requires an end user license from ucl cava (source: https://bslcorpusproject.org/cava/ and https://scholarspace.manoa.hawaii.edu/bitstreams/97f6a096-0e1d-4872-b922-0e72ace5e905/download).
- english translations are paired with signing in the `free translation` tier. all 249 narrative files (~5 minutes per signer) have complete english free translations. 100 conversation files (from bristol, birmingham, london, manchester) have both id-glosses and free translations for the first ~500 sign tokens each. 50 narrative files from belfast and glasgow have id-glosses for the first ~100 sign tokens (source: https://bslcorpusproject.org/wp-content/uploads/notes-to-the-3rd-release-of-bsl-corpus-annotations.pdf).
- narrative tasks (`n`) provide the cleanest continuous sentence-level english to gloss pairs, whereas lexical elicitation (`l`) provides isolated vocabulary pairs and conversation tasks (`c`) provide spontaneous dialogue pairs (source: https://bslcorpusproject.org/cava/).

### 2. bsl signbank structure, access, and id-gloss mapping
- bsl signbank is an online lexical database and dictionary derived from the bsl corpus, containing ~2,500 canonical id-glosses and ~50,000 recorded sign tokens across regions (source: https://bslsignbank.ucl.ac.uk/ and https://www.sign-lang.uni-hamburg.de/lr/compendium/lex/bslsignbank.html).
- signbank does not offer a public rest api, but the open-source django application code is hosted on github, and researcher access allows exporting the full id-gloss keyword dictionary and token lists (source: https://github.com/signbank/bsl-signbank and https://github.com/chrisns/bsl-experiment/blob/main/docs/bsl-data-sources.md).
- signs are keyed by uppercase lemmatized string identifiers called id-glosses (e.g. `sister`, `name-a`, `pt:pro1pl`, `ds:handling-flat`) which uniquely identify sign lemmas across phonological and regional variants (source: https://www.bslcorpusproject.org/wp-content/uploads/bslcorpusannotationguidelines_23october2014.pdf).
- id-glosses provide a standardized discrete target vocabulary (~2,500 to 5,000 tokens) for seq2seq translation models, mapping english text into clean canonical bsl gloss sequences (source: https://aclanthology.org/anthology-files/anthology-files/pdf/l/l18/l18-1374.pdf).

### 3. bsl syntax project outputs
- the bsl syntax project (ahrc funded, 2016-2021) conducted large-scale investigations into bsl word order, clause structure, questions, and negation using corpus data, cartoon-elicited signing, and acceptability judgements from the bsl sentence task (source: https://bslcorpusproject.org/projects/bsl-syntax-project/ and https://www.ucl.ac.uk/brain-sciences/pals/research/deafness-cognition-and-language-dcal/research-dcal/linguistics-research).
- its primary data output consists of enriched syntactic annotation tiers (`clause`, `rh-argument`, `lh-argument`) added directly to bsl corpus elan files, rather than a standalone parallel text dataset (source: https://bslcorpusproject.org/wp-content/uploads/bslcorpus_annotationconventions_v3.0_-march2017.pdf).

### 4. existing parallel bsl translation datasets and academic mt work
- the bobsl dataset (bbc-oxford bsl dataset, vgg oxford / bbc r&d / ucl dcal, 2021-2024) is the largest parallel bsl dataset available, containing 1,940 bbc broadcast episodes (~1,400 hours) with 1.2m english subtitle sentences across 37 signers (source: https://www.robots.ox.ac.uk/~vgg/data/bobsl/ and https://arxiv.org/abs/2111.03635).
- bobsl includes manually annotated subsets specifically designed for translation and alignment: 32k human-aligned subtitle sentences (bull et al., 2021), 5k human-annotated continuous sign sentences with 48k glosses (raude et al., 2024), and 25k isolated sign instances (source: https://arxiv.org/abs/2105.02877 and https://arxiv.org/abs/2405.10266).
- bobsl also provides automatic annotations including 1.2m pseudo-labelled sentence gloss sequences generated via swin v2 visual features and continuous sign language recognition (source: https://arxiv.org/abs/2405.10266).
- access to bobsl is granted for non-commercial academic research by submitting the bbc bobsl terms of use agreement form to `bobsl.dataset@bbc.co.uk` (source: https://www.bbc.co.uk/rd/projects/extol-dataset).
- the extol project (end-to-end translation of bsl, epsrc funded 2019-2023) developed translation pipelines combining bsl corpus annotations and bobsl broadcast alignments (source: https://www.ucl.ac.uk/brain-sciences/pals/research/deafness-cognition-and-language-dcal/research-dcal/linguistics-research).

### 5. regional variation handling
- the bsl corpus documents regional variation across 8 uk sites: london, bristol, birmingham, manchester, cardiff, newcastle, glasgow, and belfast (source: https://bslcorpusproject.org/cava/).
- sign language translation research handles regional variation by standardizing surface sign variants into canonical signbank id-glosses during annotation (source: https://researchportal.hw.ac.uk/en/publications/building-the-british-sign-language-corpus).
- for a v1 english to bsl gloss translation model, mapping all regional sign variants to unified signbank canonical id-glosses (london baseline) removes target label ambiguity and stabilizes sequence generation (source: https://bslsignbank.ucl.ac.uk/).

### 6. practical alignment pipeline from elan files
- step 1: download open-access .eaf files from the ucl primo / cava repository (source: https://ucl.primo.exlibrisgroup.com/discovery/collectiondiscovery?vid=44ucl_inst:ucl_vu2&collectionid=81354492040004761).
- step 2: parse xml tier structures in python using `pympi-eling` (source: https://bslcorpusproject.org/wp-content/uploads/bslcorpus_annotationconventions_v3.0_-march2017.pdf).
- step 3: extract time intervals for `free translation` (english sentence boundaries) and map gloss tokens from `rh-idgloss` and `lh-idgloss` tiers occurring within each sentence window (source: https://bslcorpusproject.org/wp-content/uploads/bslcorpus_annotationconventions_v3.0_-march2017.pdf).
- step 4: merge two-handed signs (`rh-idgloss` and `lh-idgloss`), deduplicate simultaneous holds/buoys, remove transcription uncertainty markers (`?`, `g:`), and normalize tokens to uppercase signbank id-glosses (source: https://www.bslcorpusproject.org/wp-content/uploads/bslcorpusannotationguidelines_23october2014.pdf).
- step 5: combine bsl corpus narrative/conversation pairs (~10k-15k sentence pairs) with bobsl manually glossed sequences (5k sentences / 48k glosses) and pseudo-labelled sequences (1.2m sentences) to construct the final fine-tuning parallel corpus (source: https://arxiv.org/abs/2405.10266).

## proposed acquisition + alignment plan

| source | url | license | format | size estimate | what to download | transformation strategy |
|---|---|---|---|---|---|---|
| bsl corpus annotations | https://ucl.primo.exlibrisgroup.com/discovery/collectiondiscovery?vid=44ucl_inst:ucl_vu2&collectionid=81354492040004761 | cc by-nc 3.0 / cc by-sa 4.0 | elan .eaf (xml) | 249 narrative files + 100 conversation files (~10k-15k sentence pairs, ~70k gloss tokens) | open access .eaf files | parse .eaf with pympi-eling, extract time-aligned `free translation` sentences and `rh-idgloss`/`lh-idgloss` tokens, merge two-handed tiers, normalize tokens against signbank. |
| bsl signbank dictionary | https://bslsignbank.ucl.ac.uk/ | open academic research | web / csv export | ~2,500 canonical id-glosses with keywords and phonology | canonical id-gloss list and english keyword mapping table | build target vocabulary dictionary (`bsl_idgloss_vocab.json`) and keyword fallback lookup table for synthetic dataset expansion. |
| bobsl manual continuous sign annotations | https://www.robots.ox.ac.uk/~vgg/data/bobsl/ | bbc bobsl terms of use (non-commercial research) | csv / json | 5,000 sentences / 48,000 glosses (raude et al. 2024) + 32k aligned subtitles (bull et al. 2021) | `bobsl_v1_4_continuous_sign_sequences.tar.gz` and `bobsl_v1_4_signing_aligned_subtitles.tar.gz` | extract sentence text and human-annotated gloss sequences, reformat to standard jsonl target schema `{"english": "...", "bsl_gloss": "..."}`. |
| bobsl pseudo-labelled continuous sign sequences | https://www.robots.ox.ac.uk/~vgg/data/bobsl/ | bbc bobsl terms of use | csv / pkl | 1.2 million sentence to gloss pseudo-labelled pairs | `bobsl_v1_4_auto_continuous_sign_sequences_swin_v2.tar.gz` | filter top-confidence pseudo-labelled sentence pairs (e.g. top 50k-100k) to supplement seed training set during two-stage fine-tuning. |

### detailed transformation workflow
1. **environment setup:** install `pympi-eling` and `pandas` in python to handle .eaf parsing and dataset formatting.
2. **elan parsing:** load each .eaf file; extract interval timestamps `[t_start, t_end]` for annotations on `free translation` tier.
3. **token extraction & temporal alignment:** for each translation interval, find all overlapping annotations on `rh-idgloss` and `lh-idgloss` tiers. order tokens chronologically by start timestamp.
4. **gloss normalization:** strip transcription uncertainty prefixes (`?`), classifier descriptors (`ds:`), pointing markers (`pt:`), and convert all tokens to canonical signbank id-gloss strings.
5. **merging & deduplication:** deduplicate identical simultaneous tokens produced on both hands; merge non-dominant hand hold/buoy tokens into single canonical gloss sequences.
6. **dataset aggregation:** combine bsl corpus seed pairs (~10k-15k), bobsl manual glossed pairs (5k), and filtered bobsl pseudo-labelled pairs into unified jsonl fine-tuning format for model training.

## open items
- submit bbc bobsl terms of use agreement form to `bobsl.dataset@bbc.co.uk` to obtain access password for bobsl manual annotation archives.
- execute automated bulk downloader script for all open-access .eaf files from the ucl primo repository collection.
- run `pympi-eling` extraction script on downloaded .eaf files and inspect edge cases (e.g. multi-sentence translation spans, unglossed conversational gaps).
