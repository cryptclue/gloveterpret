# research index

this folder contains the research foundation for glovterpreter. each file covers a distinct area. read in this order if starting from scratch:

1. [`asl_linguistics.md`](./asl_linguistics.md) for asl grammar, gloss notation, and why it's not english
2. [`bsl_linguistics.md`](./bsl_linguistics.md) for bsl grammar and how it differs from both asl and english
3. [`datasets.md`](./datasets.md) for existing sign language datasets and what they're useful for
4. [`prior_art.md`](./prior_art.md) for what's been attempted in text-to-sign and speech-to-sign, and where the gaps are
5. [`pipeline_research.md`](./pipeline_research.md) for research on each stage of the glovterpreter pipeline
6. [`tooling.md`](./tooling.md) for research on the specific tools in the stack: adaptionlabs, speechmatics, assemblyai, three.js

---

the key finding across all of this: sign language production (text/speech → signing) is dramatically under-researched compared to sign language recognition (signing → text). most existing datasets are built for recognition. most existing models work in the recognition direction. glovterpreter is working against the grain of where the field has focused - which is both the hard part and the reason it needs to exist.