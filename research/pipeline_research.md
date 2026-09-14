# pipeline research

this document covers the research basis for each stage of the glovterpreter pipeline.

---

## stage 1: utterance boundary detection

the hardest single problem in the pipeline is knowing when to commit to interpretation. the system must not pass a partial utterance downstream - because asl/bsl grammar depends on the full structure of the utterance - but it also must not hold so long that the signed output falls hopelessly behind the speaker.

### what the speech apis provide

both speechmatics and assemblyai offer real-time transcription with **utterance-level boundary signals** - not just word-level streaming output. these signals detect:
- acoustic pause (silence above a threshold)
- end-of-utterance prosodic cues (falling intonation, lengthening of final syllable)
- turn completion signals based on voice activity detection

these are the primary signals glovterpreter uses. a pause alone is insufficient - speakers pause mid-sentence frequently. the combination of pause duration + prosodic completion signal is more reliable.

### semantic completeness check

for utterances where the boundary signal fires but the utterance is grammatically incomplete (ends on a conjunction, subordinate clause, or unresolved topic), the system holds and waits for the next boundary event before committing.

detection approach: the completed utterance buffer is passed to a lightweight classifier that checks for dangling grammatical structures (e.g. utterance ending with "but", "because", "if", "and"). if detected, the system waits for one more utterance before committing.

### hard timeout

a maximum hold window (suggested: 2 sentences / ~10–15 seconds) ensures the system never falls so far behind that the signed output is no longer useful. at timeout, the system commits with the best available interpretation.

---

## stage 2: english → gloss interpretation

### why not prompt a general-purpose llm directly

general-purpose llms (gpt-4, claude, etc.) have some ability to produce asl gloss, but their output is unreliable on grammatical features - especially topic-comment structure, spatial referent assignment, and temporal marking. they have been trained on english-heavy corpora and their sign language knowledge is shallow and inconsistent.

a fine-tuned model on english → asl gloss paired data consistently outperforms prompted general-purpose models on grammatical accuracy. the gloss2text paper (emnlp 2024) and related work confirm that fine-tuned llms on domain-specific sign language data significantly outperform out-of-the-box prompting.

### the adaptionlabs approach

adaptionlabs (adaptionlabs.ai) provides two relevant tools:

**adaptive data**: a synthetic data augmentation platform. starting from a seed set of english–gloss pairs (sourced from how2sign annotations and bsl corpus annotations), adaptive data expands the training set by generating additional synthetic examples. this addresses the data gap in this field - the seed set is small, but the augmented dataset can be made large enough to fine-tune on.

**autoscientist**: automated hyperparameter and training recipe optimization. given the target task (english → asl gloss, english → bsl gloss), autoscientist searches over training configurations (lora vs full fine-tune, learning rate, batch size, etc.) to find the recipe that maximizes task performance.

the two-step process: adaptive data to build the dataset → autoscientist to find the optimal training recipe → fine-tuned model to deploy.

### separate models for asl and bsl

asl and bsl have different grammar, different word order conventions, and different gloss notation practices. they require separate fine-tuned models, not a single multilingual model. sharing a model would require the system to learn two conflicting grammar systems simultaneously - a harder problem than training two specialized models.

---

## stage 3: gloss → joint angle mapping

### fixed vocabulary approach (hackathon build)

for the hackathon, each gloss token maps to a pre-defined joint angle sequence stored in a lookup table. joint angles are derived by running pose estimation (mediapipe holistic or smpl-x) on videos from wlasl (asl) and bsl corpus (bsl) for each sign in the vocabulary.

mediapipe holistic extracts:
- 21 landmarks per hand (x, y, z per landmark = 63 values per hand)
- 8 pose landmarks for upper body
- facial landmarks (not used in current build)

from landmark sequences, joint angles are computed and stored as keyframe sequences per sign. the renderer interpolates between keyframes.

### future: generative motion synthesis

beyond the fixed vocabulary, the production direction research suggests several approaches to generative sign generation:
- diffusion models (signdiff approach) for generating novel sign motion sequences from gloss tokens
- smpl-x body model generation (signavatar approach) for more complete body representation
- pose stitching (posestitch-slt, emnlp 2025) for combining known poses to generate new signs

these are future-phase approaches requiring more training data and compute than the hackathon build.

---

## stage 4: 3d hand rendering in-browser

### three.js for browser rendering

three.js is the standard webgl library for 3d browser rendering. a rigged hand model with a mesh with bones corresponding to finger joints, knuckles, wrist - can be driven by joint rotation data from the mapping layer.

key concepts:
- **rigged mesh**: a 3d hand mesh with a skeleton (armature) of bones. each bone corresponds to a joint.
- **inverse kinematics (ik)**: optionally, instead of setting joint angles directly, an ik solver computes joint angles from target fingertip positions. for sign language, direct joint angle setting is simpler and sufficient.
- **morph targets**: pre-computed mesh deformations for common handshapes, blended at runtime. useful for achieving handshapes that are difficult to approximate purely with skeletal animation.

### low-poly model sourcing

low-poly rigged hand models are available from:
- grabcad (engineering models, some with rigs)
- sketchfab (creative commons licensed 3d models)
- mixamo (rigged humanoid models; hand can be isolated)
- custom: blender can be used to create a simple low-poly hand and rig it with an armature

the key requirement is a rig with bones for each finger segment (3 per finger, 2 for thumb) plus wrist. this gives 16 bones minimum for the hand.