# sign language datasets

## the core problem with existing datasets

most sign language datasets were built for **recognition** - the task of identifying what a signer is saying from video. glovterpreter needs **production** data, like examples of english text or speech paired with correct signed output. these are not the same thing, and the production direction is dramatically under-resourced.

additionally, gloss annotation is expensive. the how2sign dataset notes that gloss annotation took on average one hour per 90 seconds of video. this has kept gloss-annotated datasets small relative to the raw video data available.

---

## asl datasets

### wlasl (word-level american sign language)
- largest publicly available word-level asl dataset
- ~2,000 glosses (individual signs), 7–40 video samples per sign
- used for isolated sign recognition, not continuous signing
- useful for glovterpreter: building the vocabulary lookup table; extracting 3d joint angles per sign from video using pose estimation

**limitation:** word-level only. does not include sentence-level or discourse-level signing. distribution is uneven - some signs have very few samples.

### how2sign
- large-scale multimodal continuous asl dataset
- 80+ hours of sign language video with parallel speech, english transcripts, and depth data
- subset of ~34,000 sentence-level clips; 3-hour subset with full 3d panoptic studio pose data
- gloss annotation available for part of the dataset (annotation is expensive; not all clips are glossed)
- used in signdiff, signavatar, and other production-direction research

**useful for glovterpreter:** the parallel english transcript → signing video pairs are directly relevant to training an english → asl interpretation model. the 3d pose data enables procedural motion generation from joint angles.

### phoenix-2014t
- german sign language (dgs) weather forecast dataset
- widely used in sign language nlp research as a benchmark
- not asl or bsl, but the methodology and model architectures developed on this dataset transfer directly
- notable because it has paired text (german weather forecasts) + gloss + signing video - the kind of parallel data glovterpreter needs, but for german sl

### asl3dword (signavatar)
- derived from wlasl using smpl-x pose estimation
- 3d skeletal motion data for isolated asl words
- useful for: building joint-angle lookup tables for specific signs

---

## bsl datasets

### bsl corpus
- 249 bsl deaf signers from 8 uk locations
- annotated with elan: gloss-level and linguistic annotations
- primarily sociolinguistic (variation, change, frequency) rather than translation-paired
- video data accessible online via bslcorpusproject.org
- bsl signbank: 50,000 signs from 4 regions, accessible as online dictionary

**limitation for glovterpreter:** not a parallel corpus (english text → bsl). it is a documentation corpus. to build an english-to-bsl interpretation model, training pairs need to be constructed from this and supplementary sources.

### bsl signbank
- online lexical database derived from bsl corpus
- covers signs from bristol, birmingham, london, and manchester
- shows regional variation
- useful for: bsl vocabulary coverage and understanding sign variation by region

---

## what's missing (and how to address it)

the main gap is **parallel english → gloss training data** for both asl and bsl at sentence level. this is where adaptionlabs' adaptive data platform is directly relevant:

1. seed the dataset with known english–gloss pairs from how2sign (asl) and bsl corpus annotations
2. use adaptive data to generate additional synthetic training pairs, expanding coverage beyond the seed set
3. use autoscientist to optimize the fine-tuning of a base llm on this dataset for the english → gloss task

this approach - small real dataset + synthetic expansion → fine-tuned model is the realistic path to a working interpretation model given the data gap in this field.

---

## notation systems (useful for structured data)

### hamnosys (hamburg notation system)
a detailed phonological notation system for sign languages. encodes handshape, location, movement, and orientation systematically. more complete than gloss but harder to work with directly. useful as a reference for building the joint-angle mapping layer.

### signwriting
a visual notation system that writes signs using symbols representing handshape, movement, and location. not widely used in computational work but provides an intuitive check on whether a sign representation is correct.