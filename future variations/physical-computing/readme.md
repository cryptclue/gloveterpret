# physical computing

this document expands on [`roadmap.md`](../../roadmap.md) phase 4: the physical glove. it covers the hardware side of glovterpreter in more depth than the roadmap entry - what the device is, how it's built, and what it borrows from the phase 1 software pipeline versus what it needs new.

the core interpretation engine (speech capture → pause detection → english → gloss interpretation) does not change between the browser build and the physical build. what changes is the output stage: instead of driving a rendered 3d hand in a browser tab, the signed output drives physical actuators on a hand-shaped device.

---

## what it is

a non-wearable, glove-shaped device that sits on a surface: a lectern, a table at the front of a lecture hall, a stage at a panel, and physically signs in real time. it is not worn by anyone. it sits on a weighted base so it stays upright and stable, the same way a bust form or a paperweight does, and it signs for anyone in the room who needs it.

this is deliberately not the individual, personal accessibility model. it's institutional infrastructure procured and set up by a venue the same way a hearing loop or an induction system is, not something a deaf attendee has to bring, wear, or configure themselves.

---

## why physical, given the browser build already works

the browser build (phase 1) requires the deaf viewer to open a laptop or phone, run glovterpreter, and personally manage the tool while also trying to watch the speaker and the room. that's a real cost, and it puts the burden of accessibility on the individual attendee rather than the venue.

a physical device flips that. it sits at the front of the room, already running, and any attendee who needs it can simply look at it, the same way they'd look at an interpreter standing at the front of a hall. no device, no login, no personal setup. this is the same reasoning that motivates phase 3 (organizer-side integration for streamed events); the physical glove is the in-person equivalent of that same shift from individual tool to institutional infrastructure.

---

## design intent

- **not wearable.** the device is freestanding, not worn on a hand. no one puts it on, takes it off, or is responsible for it personally.
- **institutional, not personal.** procured and deployed by a university, conference organizer, or venue as part of the room's accessibility infrastructure,not carried by an individual attendee.
- **visible to the room, at scale.** positioned at the front of the space, sized and positioned so anyone who needs it can actually see it from where they're sitting. this is a real constraint that the browser build doesn't have: a screen a few inches wide works fine for one viewer up close, but a physical hand needs to be legible from the back of a lecture hall.
- **no cloud dependency for the core loop.** interpretation runs on-device, for two reasons: latency (a live room can't tolerate a slow round trip) and reliability (a venue's wifi shouldn't be a single point of failure for accessibility infrastructure that's meant to just work).

---

## what carries over from phase 1

the interpretation core is shared, not rebuilt:

- speech capture + utterance boundary detection (speechmatics / assemblyai)
- english → gloss interpretation (the adaptionlabs fine-tuned model from phase 1/2)
- the gloss vocabulary and motion-mapping logic, in concept, a gloss token still needs to map to a hand configuration and movement sequence, the same problem the phase 1 three.js renderer solves in software

what changes is only the last stage: instead of joint angles driving a rigged mesh in a browser, they drive physical actuators.

---

## what's new for the physical build

### actuation
each finger needs enough degrees of freedom to reproduce the handshapes in the fixed sign vocabulary - this is not full anatomical finger articulation, but enough joints per finger to distinguish the handshapes the vocabulary actually requires. servos or tendon-driven actuators per finger segment, plus wrist articulation for orientation and movement through signing space, are the baseline. the phase 1 joint-angle sequences (already computed from the mediapipe/wlasl and bsl corpus pose data) become the direct target values for the actuator controller, the same data feeding a different final stage.

### on-device compute
the interpretation model needs to run locally on whatever board drives the device, rather than round-tripping to a cloud api per utterance. this likely means a smaller/quantized version of the fine-tuned interpretation model, and, depending on the compute budget of the chosen board, an edge inference runtime to get real-time performance out of constrained hardware. this is a genuinely different engineering problem from the browser build, where inference can happen server-side or via a normal api call with no strict latency floor.

### mechanical design
a stable, weighted base; a housing that reads clearly as a hand at a room's viewing distance rather than close up; enough mechanical repeatability that the same gloss token reliably produces a recognizable handshape, sign after sign, without drift or recalibration mid-event.

### optional: visual feedback loop
a camera-based check that confirms the device's actual hand position matches the intended sign, catching mechanical drift or misformed signs before they reach the room. this is a stretch addition, not a requirement for a first working version, an open-loop device that reliably reproduces its fixed vocabulary is a complete and useful first build on its own.

---

## why this is a later phase, not the first build

this is explicitly *not* where glovterpreter starts (see [`deliberate_scope.md`](../../deliberate_scope.md)). building reliable actuation, sourcing or fabricating a hand rig with the right degrees of freedom, and getting real-time on-device inference working are all substantial hardware and embedded-systems problems layered on top of an interpretation pipeline that itself needs to be correct first. proving the interpretation logic works, that pause-then-interpret produces grammatically valid asl and bsl output, is a software problem, and it's much faster to iterate on, debug, and validate in a browser than on physical hardware. the physical build inherits that pipeline once it's proven, rather than trying to get both right at once.