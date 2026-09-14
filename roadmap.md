# roadmap

this document describes where glovterpreter goes after the hackathon build. the current scope is deliberately narrow. each phase below is an expansion that preserves the core: interpretation, not transcription.

---

## phase 1 [current  build]

**user-facing browser tool. one english speaker. asl and bsl output. fixed vocabulary. low-poly 3d hand.**

the user opens glovterpreter in their browser tab while watching a lecture, webinar, or stream. it listens to the speaker, waits for complete utterances, and renders the signed interpretation on a 3d hand model. no install, no organizer action required.

goal: prove the pipeline works end-to-end. interpretation-first, linguistically correct, running in a browser.

---

## phase 2 [open vocabulary and model improvement]

- expand the sign vocabulary beyond the fixed hackathon set using adaptionlabs adaptive data to generate larger english-to-gloss training sets
- fine-tune separate asl and bsl models with autoscientist for better grammatical accuracy
- add non-manual markers to the 3d hand renderer (eyebrow position, facial expression cues) - these are grammatically meaningful in both asl and bsl and are currently out of scope
- expand to additional spoken language inputs (spanish, french, etc.) as training data allows

---

## phase 3 [event organizer integration]

rather than requiring each viewer to run glovterpreter themselves, event organizers can toggle on sign language interpretation as an accessibility channel for all viewers simultaneously.

this means a signing window embedded directly into the event platform alongside the video, as a layered overlay, or in a separate accessibility sidebar, without any action required from individual viewers.

target platforms: zoom, youtube live, eventbrite, and similar virtual event stacks. implementation via browser extension, embed sdk, or platform-level api integration depending on what each platform supports.

this phase moves the access point from individual viewer → institutional organizer, which is how accessibility at scale has to work. the viewer shouldn't have to set up their own tools to receive access that should have been built in.

---

## phase 4 [physical glove (institutional)]

the physical form of glovterpreter: a non-wearable glove-shaped device that sits on a surface like a lectern, a table at the front of a hall, a panel stage, and physically signs in real time, visible to anyone in the room who needs it.

design intent:
- **not wearable.** the device is freestanding. it sits on a weighted base (a paperweight or bust form) to keep it stable and upright. no one has to put it on.
- **institutional, not personal.** the glove is procured and deployed by the venue or institution like a university, a conference organizer, a government department  & not carried by the individual deaf attendee. it's part of the room's accessibility infrastructure, like a hearing loop.
- **visible to the room.** positioned at the front of the space so that any attendee who needs it can see it. not a screen. not a phone. a physical signing hand, in the room, at scale.
- **no cloud dependency for the core loop.** on-device interpretation to avoid latency and to allow deployment in environments with unreliable connectivity.

this phase requires hardware design, actuator engineering, and institutional partnerships. it is not a software problem. it follows, not leads, the software phase.

---

## phase 5 [multi-speaker and conversational interpretation]

extend the pipeline to handle multi-party conversation: panel discussions, q&a, back-and-forth dialogue.

this requires speaker diarization (identifying who is speaking), rapid context switching between referents, and utterance attribution in the signed output. it is a meaningfully harder problem than single-speaker interpretation and is deliberately deferred until the single-speaker pipeline is mature and well-tested.

---

## long-term

- community-driven vocabulary expansion in partnership with deaf-led organizations, to ensure sign choices reflect actual community usage rather than researcher assumptions
- integration with hamnosys or signwriting notation systems to allow linguists and interpreters to validate and correct gloss outputs
- open-sourcing the english-to-gloss models and training data to support the broader research community working on sign language production