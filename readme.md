# glovterpreter

> **interpretation, not transcription.**

glovterpreter is a browser-based accessibility tool that listens to a single english speaker, waits for a semantically complete utterance, and interprets it into asl or bsl that's rendered live as a procedurally animated 3d hand model.

it is not a captioning tool. it does not scroll english words. it treats speech-to-sign as what it actually is: translation between two independent languages.

---

## docs

| file | what it covers |
|---|---|
| [`about.md`](./about.md) | what glovterpreter is, who it's for, and the linguistic premise |
| [`problem.md`](./problem.md) | why captioning ≠ interpretation, and the gap this fills |
| [`solution.md`](./solution.md) | the pipeline, tech stack, and how it works |
| [`idea.md`](./idea.md) | origin of the concept and design philosophy |
| [`deliberate_scope.md`](./deliberate_scope.md) | what's in and out of scope for this build, and why |
| [`roadmap.md`](./roadmap.md) | where this goes after the hackathon |
| [`research/`](./research/) | deep research on asl/bsl linguistics, datasets, and prior art |

---

## one-line pitch

existing assistive tech interprets speech into text. glovterpreter interprets speech into a different language entirely.

---

## tech stack

| layer | tool |
|---|---|
| speech capture + pause detection | speechmatics / assemblyai |
| research sweep | firecrawl, tavily, browserbase |
| interpretation model (english → gloss) | llm fine-tuned via adaptionlabs adaptive data + autoscientist |
| 3d hand rendering | low-poly rigged hand model (three.js / webgl, browser-native) |
| deployment | browser-based, no install required |

---

## initial build scope

- software only. 
- one english speaker (without multi speaker support)
- dual output: asl and bsl. runs in the browser. 
- the 3d "hand" is low-poly rigged model 
    - procedurally driven 
    - clarity of interpretation over visual realism.

see [`deliberate_scope.md`](./deliberate_scope.md) for the full rationale.