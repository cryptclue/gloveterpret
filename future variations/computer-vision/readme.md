 # computer vision

the computer vision branch explores the **reverse side of glovterpreter**: interpreting signed communication back into spoken-language communication.

the current glovterpreter pipeline is primarily one-directional:

```text
english speech
      ↓
speech recognition
      ↓
utterance boundary detection
      ↓
linguistic interpretation
      ↓
asl / bsl gloss
      ↓
sign motion
      ↓
3d signing hand
```

the computer vision branch asks what happens when that direction is reversed:

```text
signed communication
      ↓
camera
      ↓
visual perception
      ↓
sign representation
      ↓
linguistic interpretation
      ↓
english
      ↓
speech
```

the long-term goal is therefore not simply sign recognition. it is to investigate whether the two directions can eventually form a **bidirectional interpretation system**.

---

## why this exists

most of the current prototype is concerned with the problem:

> how can an english-speaking person communicate something in sign language without manually knowing how to sign it?

the computer vision branch introduces the corresponding question:

> how can signed communication be understood by an english-speaking person without requiring them to already know the sign language?

this creates a dual-sided system:

```text
                 glovterpreter
                      │
          ┌───────────┴───────────┐
          ↓                       ↑
     english → sign          sign → english
          ↓                       ↑
       3d hand                 computer vision
```

the two directions are deliberately treated as **interpretation problems**, rather than as word-for-word conversion.

a sign language is not a manually encoded form of english. consequently, the reverse pipeline cannot simply identify individual hand poses and substitute english words for them.

the system needs to account for the fact that the observed signing is itself a linguistic sequence.

---

## the dual-sided model

the eventual system can be thought of as two independent but connected interpreters.

### side a: speech to sign

this is the existing glovterpreter direction.

```text
microphone
    ↓
speech capture
    ↓
utterance boundary detection
    ↓
english linguistic interpretation
    ↓
asl / bsl representation
    ↓
motion representation
    ↓
3d hand
```

the output is a visual signing representation intended to communicate the interpreted meaning to a signer.

### side b: sign to speech

the computer vision branch investigates the reverse:

```text
camera
    ↓
visual capture
    ↓
hand / body tracking
    ↓
sign recognition
    ↓
linguistic representation
    ↓
english interpretation
    ↓
speech synthesis
```

the output is spoken english for a hearing participant.

these are not necessarily mirror-image implementations.

the forward direction starts with an already linguistic spoken signal.

the reverse direction starts with a **visual signal containing linguistic information**, meaning perception and language understanding become tightly coupled.

---

# computer vision is not the interpreter

a major design principle of this branch is separating **visual perception** from **linguistic interpretation**.

computer vision should answer questions such as:

* where are the hands?
* what are their shapes?
* where are they located?
* how are they moving?
* what is the orientation of the hands?
* what is the movement of the arms and upper body?
* what temporal sequence occurred?
* what relevant visual information is present?

it should not by itself be responsible for deciding:

> “this entire sequence means this english sentence.”

that belongs to the linguistic layer.

conceptually:

```text
camera
  ↓
visual perception
  ↓
structured sign representation
  ↓
linguistic interpretation
  ↓
english
```

this separation makes it possible to experiment with different vision models without rebuilding the linguistic interpretation layer each time.

---

# why opencv

opencv is relevant primarily as a **computer-vision infrastructure layer**, rather than as the entire solution.

it provides the machinery needed to work with camera input and perform conventional image-processing operations.

potential responsibilities include:

* camera capture
* frame processing
* image transformations
* cropping and normalization
* background / foreground processing
* temporal frame handling
* visualization and debugging
* integration with pose / landmark estimation systems
* preprocessing for downstream models

opencv therefore sits toward the beginning of the pipeline.

```text
camera
  ↓
opencv
  ↓
landmarks / pose / visual features
  ↓
sign-language model
```

opencv is not expected to magically solve sign-language translation by itself.

---

# what needs to be observed

a sign is not adequately represented by a single hand image.

the visual representation may need to capture several simultaneous dimensions.

## handshape

the configuration of the fingers and hand.

this can include:

* finger positions
* finger flexion
* thumb position
* palm configuration
* relationship between the fingers

## orientation

the direction the hand is facing.

orientation can distinguish otherwise similar configurations.

## location

where the hands occur relative to:

* the body
* the head
* the torso
* previously established signing-space locations

## movement

signs can encode information through movement over time.

therefore the system needs temporal information rather than independent frame classification.

```text
frame 1 → frame 2 → frame 3 → frame 4 → ...
```

is more meaningful than:

```text
frame 1
```

alone.

## two-handed interaction

some signs involve relationships between both hands.

the model therefore cannot assume that each hand is an independent object.

it may need to represent:

```text
left hand
    ↕
right hand
```

including relative position, contact, movement, and timing.

## body and head movement

signing is not necessarily confined to the hands.

future research may need to consider:

* arm position
* shoulder movement
* head movement
* body lean
* facial expression
* mouth movements
* other non-manual markers

these are particularly important because non-manual information can contribute grammatical meaning.

the initial computer-vision experiments may therefore deliberately restrict the representation before expanding it.

---

# temporal modeling

one of the central problems is that signing is inherently temporal.

a useful representation is not:

```text
sign a
sign b
sign c
```

but something closer to:

```text
t0 ───── t1 ───── t2 ───── t3

handshape
orientation
location
movement
body state
```

the system needs to determine where one meaningful signing unit ends and another begins.

this is analogous to the utterance-boundary problem in the speech pipeline, but it is not necessarily solved using the same mechanisms.

for speech:

```text
audio
 ↓
pause / prosody / semantic completion
 ↓
utterance boundary
```

for signing:

```text
visual sequence
 ↓
motion / transition / linguistic structure
 ↓
sign boundary
```

the research question is therefore not simply:

> “what sign is this frame?”

but:

> “what linguistic event is occurring across this sequence of frames?”

---

# representation before translation

the vision branch should ideally produce an intermediate representation before attempting english generation.

a conceptual representation might contain:

```text
{
  handshape,
  orientation,
  location,
  movement,
  handedness,
  spatial_reference,
  temporal_sequence,
  non_manual_features
}
```

this representation is intentionally not equivalent to english.

it functions as a bridge:

```text
video
 ↓
visual / articulatory representation
 ↓
sign-language representation
 ↓
english interpretation
```

this also creates a useful research boundary.

the vision system can be evaluated separately from the language model.

---

# sign recognition vs sign interpretation

these terms should not be treated as interchangeable.

### sign recognition

asks:

> what sign or sequence is being produced?

this can be useful for:

* isolated-sign recognition
* vocabulary lookup
* dataset benchmarking
* controlled experiments

### sign interpretation

asks:

> what is the signer communicating?

this requires substantially more context.

for example, recognizing a sequence of lexical signs does not necessarily tell the system how those signs should be interpreted in english.

spatial references, grammatical structure, discourse context, non-manual information, and other linguistic properties can affect the final meaning.

therefore:

```text
recognition
    ≠
translation
    ≠
interpretation
```

the branch is ultimately interested in the latter two.

---

# connection to the existing gloss layer

the existing forward pipeline already uses an intermediate gloss representation.

the reverse pipeline can potentially converge on a compatible structured representation:

```text
english
  ↓
asl / bsl gloss
  ↓
motion
  ↓
3d hand
```

and:

```text
camera
  ↓
motion / landmarks
  ↓
asl / bsl gloss
  ↓
english
```

this creates a potentially useful shared abstraction:

```text
             sign-language representation
                    ↙          ↘
                 motion       english
```

however, this should not be interpreted as saying that gloss is a complete representation of a sign language.

gloss is an intermediate research representation.

it does not fully encode:

* phonological detail
* spatial structure
* prosody
* non-manual features
* all morphological information
* all discourse context

the computer vision branch therefore should not assume that producing a gloss sequence means the translation problem is solved.

---

# asl and bsl

the dual-sided system should preserve the distinction between ASL and BSL.

the computer vision pipeline should not assume:

```text
sign language = universal sign language
```

nor should it assume that visual similarity implies linguistic equivalence.

the eventual architecture should allow the language-specific interpretation layer to remain separate:

```text
                 visual input
                     ↓
              visual features
                     ↓
             sign representation
                ↙          ↘
              ASL          BSL
               ↓            ↓
             english      english
```

this becomes particularly important for:

* vocabulary
* grammar
* regional variation
* two-handed vs one-handed forms
* non-manual features
* spatial organization

the goal is not to create one universal “sign classifier.”

---

# datasets

the computer vision branch has a different data requirement from the forward speech-to-sign system.

recognition datasets can provide:

* videos
* isolated signs
* gloss labels
* hand/body landmarks
* signer variation

continuous sign-language datasets are particularly valuable because the eventual system needs to move beyond isolated vocabulary.

potential sources include research datasets already investigated elsewhere in this repository.

the important distinction is:

```text
recognition data
video → label

production data
meaning / text → signing

bidirectional research
video ↔ linguistic representation ↔ english
```

a dataset being large does not automatically make it suitable for the task.

annotation structure matters as much as raw video quantity.

---

# signer variation

the model must eventually deal with variation between people.

signing can vary according to:

* signer
* region
* age
* community
* signing style
* speed
* camera position
* lighting
* clothing
* background
* dominant hand
* individual articulation

a system trained on one signer should therefore not be assumed to generalize to an entire sign-language community.

this makes signer-independent evaluation an important future research requirement.

---

# camera constraints

the first experiments can intentionally use constrained conditions.

for example:

```text
single signer
front-facing camera
upper-body framing
controlled lighting
limited background movement
```

this is not intended to represent the final environment.

it provides a controlled environment for determining whether the linguistic pipeline works before introducing additional visual complexity.

future experiments can progressively introduce:

```text
controlled
   ↓
variable lighting
   ↓
variable backgrounds
   ↓
different cameras
   ↓
different signers
   ↓
different signing speeds
   ↓
real-world environments
```

---

# non-manual features

non-manual features are a major research consideration.

the hands alone do not necessarily contain all of the linguistic information being expressed.

future visual modeling may therefore incorporate:

```text
hands
+
arms
+
head
+
face
+
mouth
+
body
```

rather than treating sign language as purely hand movement.

this is especially important when moving from isolated sign recognition toward actual interpretation.

the initial implementation may omit these features deliberately for tractability.

that omission should be understood as a **scope decision**, not a claim that they are linguistically unimportant.

---

# possible technical stack

the exact implementation is intentionally experimental.

a possible stack is:

```text
camera
  ↓
opencv
  ↓
hand / pose estimation
  ↓
landmark normalization
  ↓
temporal representation
  ↓
sign recognition / sequence model
  ↓
linguistic interpretation
  ↓
english text
  ↓
speech synthesis
```

potential components can be swapped independently.

for example:

```text
opencv
   +
mediapipe
   +
custom temporal model
```

could later become:

```text
opencv
   +
another pose estimator
   +
transformer / sequence model
```

without changing the conceptual architecture.

---

# research questions

the branch is intended to investigate questions such as:

1. how much linguistic information can be recovered from hand and body landmarks alone?

2. when do non-manual features become necessary for reliable interpretation?

3. how should continuous signing be segmented into meaningful units?

4. what intermediate representation is most useful between visual perception and language interpretation?

5. can a model generalize across signers without memorizing individual signing styles?

6. how much does regional variation affect recognition and interpretation?

7. can ASL and BSL share a visual perception layer while maintaining separate linguistic interpretation layers?

8. where does isolated-sign recognition stop being useful and continuous interpretation become necessary?

9. can the reverse pipeline use the same structured linguistic representations as the forward speech-to-sign pipeline?

10. how should the system communicate uncertainty when it does not confidently understand a signing sequence?

---

# uncertainty

a production interpreter should not silently invent an interpretation when the visual evidence is ambiguous.

the system should eventually be able to distinguish between:

```text
high confidence
 ↓
interpret normally
```

and:

```text
low confidence
 ↓
request clarification / wait for more context
```

this mirrors the broader design philosophy of glovterpreter:

**an interpretation should be committed when there is enough information to make it meaningful, rather than merely because a prediction is available.**

---

# evaluation

evaluation should not rely on a single accuracy number.

possible evaluation layers include:

### visual layer

* landmark accuracy
* hand tracking stability
* pose estimation quality
* robustness to occlusion

### recognition layer

* isolated sign accuracy
* continuous sequence recognition
* signer-independent performance
* regional generalization

### linguistic layer

* gloss accuracy
* grammatical ordering
* spatial-reference preservation
* non-manual information
* contextual interpretation

### translation layer

* semantic preservation
* adequacy
* fluency of english output
* human evaluation by fluent signers

the final question is not:

> “did the model identify the correct hand pose?”

it is:

> **“did the resulting interpretation preserve what the signer actually communicated?”**

---

# relationship to the physical glove branch

the computer-vision branch and the physical-glove branch are separate future variations.

the physical-glove branch explores a **hardware embodiment** of the system.

the computer-vision branch explores a **visual input channel**.

they may eventually intersect, but neither depends on the other.

```text
                 glovterpreter
                       │
             future variations
                  ╱         ╲
                 ╱           ╲
        physical glove     computer vision
             │                  │
          hardware             camera
             │                  │
        tactile /              visual
        physical              perception
        interface               │
                                ↓
                           sign → english
```

the current browser-based 3d hand therefore remains the primary implementation.

these branches describe possible future research directions rather than features that are already implemented.

---

# current status

this branch is a **research direction**, not a completed computer-vision implementation.

the folder exists to document the technical and linguistic questions that would need to be answered before building the reverse channel.

initial work should prioritize:

1. defining the visual representation
2. selecting appropriate datasets
3. establishing a reproducible camera pipeline
4. testing landmark extraction
5. testing temporal segmentation
6. evaluating isolated vs continuous recognition
7. connecting recognized signing to a linguistic representation
8. evaluating interpretation separately from perception

implementation should follow the research rather than assuming that an off-the-shelf sign-recognition model is equivalent to an interpreter.

---

# long-term architecture

the eventual goal is a system capable of supporting both directions:

```text
                         ┌──────────────────┐
                         │  sign language   │
                         │   representation │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    ↓                           ↑
             sign → english               english → sign
                    ↓                           ↑
              text / speech                gloss / motion
                    ↑                           ↓
                 hearing                    3d hand
                 speaker                    rendering
```

with computer vision providing the input side for signed communication and the existing speech pipeline providing the input side for spoken communication.

the ultimate research question is therefore larger than sign recognition:

> **can a browser-native system perform meaningful bidirectional interpretation between spoken english and a signed language while respecting the fact that both are independent languages?**

that is the problem this branch exists to investigate.
