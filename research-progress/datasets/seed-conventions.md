# seed dataset v0 conventions documentation

this document details the linguistic grounding, structural conventions, notation rules, and known limitations applied during the construction of the glovterpreter seed dataset v0 (track tr-10). the dataset consists of two parallel files: `seed-asl.jsonl` (135 pairs) and `seed-bsl.jsonl` (135 pairs), targeting single-speaker lecture and academic speech interpretation.

---

## 1. gloss token format and conventions

gloss notation provides a symbolic text interface between english speech transcription and 3d motion synthesis. signs are represented as uppercase tokens representing distinct lexical concepts:

- **core sign tokens**: written in uppercase ascii characters (e.g. `STUDY`, `COMPUTER`, `BOOK`, `TEACHER`, `SCIENCE`).
- **compound signs**: hyphenated uppercase tokens representing single signed concepts formed from compound gestures or unified ideas (e.g. `NEXT-TO`, `TURN-OFF`, `NOT-YET`, `HOW-MANY`).
- **indexing and pronouns (`ix-`)**: spatial pointing gestures and pronominal indexing are prefixed with `ix-` (e.g. `ix-me` for first-person singular, `ix-you` for second-person, `ix-we` for first-person plural, `ix-he` / `ix-she` for third-person, `ix-our` for possessive, `ix-located` for spatial predicate indexing).
- **fingerspelling (`fs:`)**: manual alphabet spelling of proper nouns, acronyms, or non-lexicalized technical terms.
  - **asl (one-handed alphabet)**: prefixed with `fs:` (e.g. `fs:smith`, `fs:ai`, `fs:glovterpreter`).
  - **bsl (two-handed alphabet)**: prefixed with `fs:2h-` (e.g. `fs:2h-smith`, `fs:2h-ai`, `fs:2h-glovterpreter`) to reflect the two-handed manual configuration required by the 3d motion layer.
- **classifiers (`cl:`)**: handshapes representing physical object categories, spatial locations, and motion trajectories:
  - `cl:1`: upright human index finger trajectory (e.g. `cl:1(move-right)` for person walking).
  - `cl:b`: flat hand representing flat surfaces, papers, screens, or shelves (e.g. `cl:b(side-by-side)`, `cl:b(fall-down)`).
  - `cl:c`: curved handshape representing cylindrical objects such as cups, microphones, or containers (e.g. `cl:c(on-top)`, `cl:c(move-close)`).
  - `cl:3`: three-finger handshape representing vehicles, robots, or moving mechanical objects (e.g. `cl:3(move-forward)`, `cl:3(orbit-around)`).
  - `cl:v`: bent-v handshape representing seated human figures or legs (e.g. `cl:v(sit)`).
- **non-manual markers (nmms)**: enclosed in parentheses to indicate required facial or head gestures:
  - `(whq)`: wh-question marker (furrowed eyebrows, slight head tilt).
  - `(ynq)`: yes/no question marker (raised eyebrows, slight forward lean).

---

## 2. ordering rules per language

### american sign language (asl)
asl follows topic-comment ordering and time-fronting. structure adheres to the contrastive semantic features (csf) slot model:

$$\text{modifier} \rightarrow \text{time} \rightarrow \text{condition} \rightarrow \text{agent} \rightarrow \text{location} \rightarrow \text{object} \rightarrow \text{event} \rightarrow \text{purpose}$$

key asl rules applied:
1. **time fronting**: time expressions (`yesterday`, `tomorrow`, `future week`, `past month`) appear at the start of the clause.
2. **conditional fronting**: condition clauses begin with `if` and precede the main consequence clause (e.g. `if computer break, data all lost`).
3. **wh-question end-placement**: wh-words (`what`, `where`, `who`, `when`, `why`, `how`, `which`) appear in clause-final position accompanied by `(whq)`.
4. **topic-comment / osv**: object or topic is fronted before agent and action verb (e.g. `book ix-me read`).
5. **numeral post-position**: numerical quantifiers follow the noun (e.g. `book three`, `class five`).

### british sign language (bsl)
bsl is an independent sign language from a distinct language family (banzsl). bsl grammar differs systematically from asl in clause structure and sign ordering:

1. **clause-final main verbs and modals**: bsl strongly prefers placing main action verbs and modals at the end of the clause or sentence (e.g. `today machine learn ix-we talk`, `finals before student study hard must`).
2. **conditional clause-final `if`**: in conditional sentences, bsl frequently places the conditional sign `if` at the end of the condition clause rather than at the front (e.g. `computer break if, data all lost`).
3. **question markers and time expressions**: bsl uses `time what` in place of `when` for temporal wh-questions (e.g. `work due time what (whq)`).
4. **object-verb / topic-comment ordering**: objects routinely precede verbs (e.g. `this class book three need`).
5. **lexical items**: bsl utilizes distinct lexical items where asl differs (e.g. `fortnight future` for two weeks in future, `fs:2h-` for two-handed manual spelling).

---

## 3. verb agreement, negation, and spatial notation

- **verb agreement / directional verbs**: verbs that move between spatial referents indicate subject and object via trajectory. spatial locations are assigned to referents using spatial markers `(lt)` for left space and `(rt)` for right space. directional movement is represented as `(lt)→give→(rt)` (e.g. `john(lt) paper (lt)→give→(rt) mary(rt)`).
- **negation**:
  - in asl, negative markers (`not`, `never`, `cannot`, `none`) are placed immediately before or after the verb/predicate, or in sentence-final position.
  - in bsl, negation is overwhelmingly clause-final (e.g. `new idea right not`). bsl also employs `impossible` for strict prohibition or capability negation in formal academic contexts (e.g. `decide impossible`).
  - absence of items is glossed using `none` (e.g. `eye glasses none`).
  - uncompleted actions use `not-yet` (e.g. `first test ix-we finish not-yet`).

---

## 4. handling english idioms, filler words, and passive structures

- **english idioms**: mapped conceptually to meaning-based sign sequences rather than literal word substitutions:
  - "easy as pie" / "piece of cake" -> `easy very`
  - "hit the books" -> `study hard`
  - "on the fence" -> `decide not-yet`
  - "break a leg" -> `good luck`
  - "spill the beans" -> `secret tell`
- **filler words, articles, and copulas**: english articles (`a`, `an`, `the`), auxiliary verbs (`is`, `are`, `was`, `were`, `do`, `does`, `did`), and prepositions that lack signed equivalents are dropped during translation.
- **passive voice structures**: english passive sentences are converted into active agent-verb-object (or topic-agent-verb) structures (e.g. "the experiment was conducted by the graduate students" -> asl: `high student do test` / bsl: `high student test do`).
- **complex / dangling structures**: sentences where english word order is impossible in target sign language (e.g. "what are the key findings of this study?") are restructured so the topic is fronted and the wh-question marker is clause-final (asl: `this study main result what (whq)` / bsl: `this study result main what (whq)`).

---

## 5. known limitations and future passes

1. **lack of non-manual facial pose rendering**: v0 glosses include non-manual annotations `(whq)` and `(ynq)`, but current 3d hand pose rendering layers only execute skeletal arm/hand poses. facial non-manual channels will be integrated in future rendering iterations.
2. **loss of spatial-temporal nuance in text gloss**: written gloss tokens approximate 3d spatial positions but cannot fully capture simultaneous non-manual expressions, lip patterns (mouthing in bsl), or continuous body lean.
3. **restricted vocabulary size**: the seed dataset operates within a small, highly consolidated core vocabulary (~280 distinct tokens) to keep the 3d pose lookup table tractable for in-browser evaluation.
4. **regional bsl variation**: bsl possesses documented regional variation across the UK (documented in the BSL Corpus). the v0 seed dataset adopts standardized bsl sign bank forms without regional dialectal branching.
