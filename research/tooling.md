# tooling research

## speech capture

### speechmatics
- real-time streaming transcription api
- provides utterance-level boundary detection (not just word-level streaming)
- strong accuracy on english, multiple accent variants
- relevant feature: turn-end detection with configurable sensitivity
- docs: speechmatics.com/docs

### assemblyai
- real-time transcription with streaming websocket api
- provides utterance-end events with configurable silence threshold
- universal-2 model is strong on english accuracy
- relevant feature: `utterance_end_ms` threshold for utterance boundary detection
- docs: assemblyai.com/docs

**for glovterpreter:** either can serve as the speech capture layer. speechmatics' utterance boundary detection is slightly more configurable; assemblyai has more straightforward browser websocket integration. worth testing both with the actual pause-detection logic to see which produces cleaner boundaries.

---

## research sweep tooling

### firecrawl
- web scraping and crawling api
- use for: pulling full content from academic paper pages, sign language research sites, corpus documentation
- particularly useful for deep-crawling the bsl corpus project site and related university research pages

### tavily
- ai-optimized web search api
- use for: finding relevant papers, datasets, and tools with targeted queries
- better than raw google search for research queries because it returns structured, relevant results

### browserbase
- headless browser automation
- use for: accessing sites that require javascript rendering, logging into corpus portals, navigating paginated research databases
- useful if bsl corpus or wlasl access requires interaction that firecrawl can't handle statically

---

## dataset and model tooling

### adaptionlabs (adaptionlabs.ai)
adaptionlabs provides two core tools relevant to glovterpreter:

**adaptive data**
- synthetic dataset augmentation platform
- can ingest a seed dataset (e.g., csv of english → gloss pairs from how2sign annotations) and generate additional synthetic training examples
- supports language expansion across iso codes - useful if we later want to expand input languages
- sdk: `pip install adaption`; `client.datasets.upload_file()` to ingest, `client.datasets.run()` to augment
- billing is per row added

**autoscientist**
- automated model training and hyperparameter optimization
- takes a dataset + training goal and searches over hyperparameter space (lora, sft, dpo configurations) to find the optimal training recipe
- returns trained model weights for deployment
- useful for: fine-tuning a base llm on the english → gloss task without manually iterating on training configurations

**workflow for glovterpreter:**
1. build seed dataset: english utterance → asl gloss pairs (from how2sign) + english utterance → bsl gloss pairs (from bsl corpus annotations)
2. upload to adaptionlabs via `client.datasets.upload_file()`
3. run adaptive data to augment to a viable training size
4. use autoscientist to fine-tune base llm (e.g., a small instruction-tuned model) on the augmented dataset
5. download resulting weights; deploy for inference in the pipeline

### allenai.org 

---

## 3d rendering

### three.js
- standard webgl library for browser 3d
- mature, well-documented, large ecosystem
- cdn-accessible: no build step required for a hackathon demo
- key modules: `three.skinnedmesh` (for rigged models), `three.bone`, `three.animationmixer` (for keyframe sequences)

### mediapipe hands (for pose extraction, not rendering)
- google's hand landmark detection
- 21 3d landmarks per hand
- can run in-browser (mediapipe js) or as a python library
- use for: extracting joint angle data from wlasl and bsl corpus sign videos to build the lookup table

### blender (for model preparation)
- free and open-source 3d software
- use for: rigging a low-poly hand model with a proper bone structure if an off-the-shelf rigged model doesn't have the right rig
- can export to gltf format which three.js loads natively

---

## model / inference

for the interpretation layer (english → gloss), a small fine-tuned llm is preferred over api calls to a large general-purpose model because:
- latency: local or edge inference is faster than a round-trip api call per utterance
- cost: no per-token api cost for a deployed fine-tuned model
- accuracy: fine-tuned on domain-specific data outperforms prompted general-purpose models on asl/bsl grammar

for the hackathon demo specifically, running a small model (e.g., 1b–7b parameter class) via api (with adaptionlabs providing the fine-tuned weights) or via a browser-compatible onnx export is the most practical approach.