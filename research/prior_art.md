# prior art

## the direction problem

the overwhelming majority of sign language ai research is in the **recognition** direction: video of signing → text. this is because it maps onto familiar computer vision problems (action recognition, gesture classification) and because the evaluation is straightforward (does the model produce the right english words?).

the **production** direction - text or speech → signing - is less researched, harder to evaluate, and less well-funded. glovterpreter works in the production direction. this means less prior art to build on, but also a clearer gap to fill.

---

## text-to-gloss

### statistical and rule-based approaches (early)
early text-to-sign systems used synchronous tree adjoining grammar (tag) rules to map english parse trees to asl gloss structures. the team project at university of pennsylvania (early 2000s) is a notable example. these systems worked within a limited domain (weather reports) but required hand-crafted grammar rules and did not generalize.

### llm-based text-to-gloss (recent)
recent work has used large language models fine-tuned or prompted for the text-to-gloss task. key papers:

- **gloss2text (emnlp 2024)** - uses pre-trained llms with data augmentation and a novel label-smoothing loss for the gloss-to-text direction (the reverse of what glovterpreter needs, but architecturally informative)
- **handscribe (2025/2026)** - a gloss-free framework that includes a text-to-gloss generator (gsg) module based on a fine-tuned llm. generates gloss sequences from translated sentences, allowing adaptation to new vocabularies. directly relevant to glovterpreter's interpretation layer.
- **csf: contrastive semantic features (2025)** - multilingual sign language generation via structured semantic slots. implements asl grammar ordering: modifier → time → condition → agent → location → object → event → purpose. example: "if it rains tomorrow, i stay home" → gloss: tomorrow if rain home stay.

the csf paper's explicit grammar ordering is particularly useful for glovterpreter: it shows a working approach to producing grammatically correct asl word order from english input.

### llms as sign language translators
a cvpr 2024 paper ("llms are good sign language translators") demonstrated that llms can serve as effective backends for sign language translation tasks. this supports the use of a fine-tuned llm as the interpretation layer in glovterpreter rather than a bespoke architecture.

---

## gloss-to-pose / sign language production

### pipeline approach
the standard pipeline for sign language production is: text → gloss → pose sequence → (optionally) video. glovterpreter follows this pipeline to the pose stage, then renders in 3d rather than generating video.

key papers:
- **signavatar (2024)** - 3d sign language motion reconstruction and generation using smpl-x body model format. constructs asl3dword dataset from wlasl. directly relevant to the joint-angle representation layer.
- **speak2sign3d (2025)** - multi-modal pipeline for english speech to asl animation. extracts 133 anatomical keypoints from wlasl videos using pose estimation. very close to what glovterpreter is building - validates the pipeline approach.
- **signdiff (2023)** - diffusion model for asl production, trained on how2sign. generates 3d skeleton pose sequences.
- **text-driven 3d hand motion generation (2025)** - surveys the state of 3d sign generation and notes the shift toward smpl-x body model representations across multiple concurrent 2024 works.

### key finding
the most relevant recent work is speak2sign3d (2025), which implements almost exactly the glovterpreter speech-to-asl-animation pipeline. the difference is that glovterpreter: (a) adds real interpretation rather than word-substitution, (b) supports bsl as well as asl, and (c) deploys in-browser rather than as a research demo. speak2sign3d validates the technical feasibility of the pipeline.

---

## what hasn't been done

1. **speech-to-sign with genuine interpretation** (pause-then-interpret, grammatically correct gloss, not word substitution) deployed as a usable tool rather than a research demo
2. **dual asl/bsl support** in a single system - essentially all prior work targets one language
3. **browser-native deployment** - prior art is almost entirely server-side or desktop research tools
4. **linguistically motivated pause detection** - prior work either does streaming (word-by-word, linguistically incorrect) or static (offline, not live)

these four gaps are exactly what glovterpreter addresses.