# deliberate scope

this document records what glovterpreter does and does not do in its current build, and the reasoning behind each constraint. these are not oversights but rather are decisions made to keep the core working correctly rather than spreading thin.

---

## in scope

### one speaker at a time
glovterpreter listens to a single speaker's audio stream. it does not attempt to separate, diarize, or interleave multiple speakers. the target use case includes a lecture, a webinar, a streamed talk, an instructional video and is inherently single-speaker. 

multi-party conversation introduces a separate set of problems (turn detection, referent switching, speaker attribution) that are solvable but not for this build.

### english input only
the current model interprets english speech into asl or bsl. spanish, french, mandarin, and other spoken languages are not supported. this is not because those languages don't matter. 

it's because the english-to-asl and english-to-bsl training pipelines are the best-resourced starting point, and getting those right first produces something that actually works rather than something that sort-of-works in five languages.

### asl and bsl output
two sign languages, not one, and not a catch-all "sign language." asl and bsl are distinct languages with different grammar, different vocabulary, different manual alphabets in some contexts. supporting both from the start reflects the linguistic reality that they are not interchangeable.

### browser-based, user-side
the current build runs in the user's browser tab. it is for the individual viewer who wants interpreted access to content they are already watching. it requires no action from the event organizer or speaker.

### fixed sign vocabulary (hackathon build)
the gloss-to-motion mapping layer operates on a pre-built vocabulary of signs. this is a deliberate scope limit for the hackathon build: a lookup table of well-formed signs is faster to build, easier to debug, and more reliable in a demo than a generative motion synthesis system.

### low-poly procedural hand rendering
the output is a simplified 3d hand, not a photorealistic avatar or a video of a human interpreter. this is a design choice, not a compromise. see [`idea.md`](./idea.md) for the reasoning.

---

## out of scope (this build)

### multi-party conversation
interpreting a conversation between two or more people, including back-and-forth dialogue, q&a, or panel discussions. this requires speaker diarization, rapid context switching, and utterance attribution; a separate and significantly harder problem.

### non-english input languages
any language other than english as the speech input. roadmap item, not current scope.

### full open-vocabulary sign generation
generative motion synthesis that can produce any sign from scratch rather than retrieving from a fixed set. this is a research-level problem. the current build uses a fixed vocabulary; expansion to open vocabulary is a longer-term goal.

### physical glove hardware
the physical glove aka a non-wearable device that sits on a table and physically signs in a large hall is a roadmap item. see [`roadmap.md`](./roadmap.md).

### event organizer integration
a mode where organizers toggle on sign language interpretation for all viewers simultaneously, without individual users having to install or activate anything. this is a roadmap item requiring platform partnerships.

### non-manual markers (facial expressions, mouthing, etc.)
asl and bsl both use non-manual markers aka facial expressions, eyebrow position, mouthing as grammatically meaningful elements of the language. the current build does not attempt to render these. the hand model signs. non-manual markers are a future layer.

---

## the reasoning

every item listed as out of scope is there for the same reason: doing fewer things correctly is more valuable than doing more things approximately. the core linguistic argument is **that interpretation is different from transcription** is only persuasive if the interpretation the system produces is actually correct. that requires focus.