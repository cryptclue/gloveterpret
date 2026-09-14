# bsl linguistics

## bsl is not asl

british sign language and american sign language are distinct languages. they are not dialects of the same thing. they do not share a common ancestor in the way that, say, spanish and portuguese do. a fluent asl signer visiting the uk is not able to communicate with bsl signers without learning bsl - the two languages are mutually unintelligible.

this matters directly for glovterpreter: asl and bsl require separate interpretation models, separate gloss conventions, and separate motion vocabularies. treating them as variants of a single "sign language" would produce incorrect output in both.

## origins

bsl developed largely independently of asl. it draws on a tradition of british deaf education and community signing that predates the influence of french sign language - which is the common ancestor that asl and french sign language share. bsl's grammar and vocabulary are thus quite different from asl's, even where both differ from english.

## grammar: how bsl differs from english (and from asl)

### word order

bsl uses a topic-comment structure broadly similar to asl, but the specifics differ. bsl has more flexibility in word order than english and also uses spatial grammar, but the rules governing how space is used and how referents are assigned differ from asl conventions.

### finger spelling

bsl uses a **two-handed manual alphabet**, which is entirely different from the one-handed manual alphabet used in asl. this has practical implications for the motion model: finger-spelled words require different hand configurations in bsl than they would in asl.

### mouthing

bsl makes significant use of mouthing - silently forming spoken english words with the lips while signing. this is a grammatically integrated feature of bsl, more prominent than in asl. mouthing can change or specify the meaning of a sign. it is out of scope for the current build but is a more significant omission for bsl than for asl.

### regional variation

bsl has notable regional variation across england, scotland, wales, and northern ireland. signs for the same concept can vary by region. the bsl corpus project (led by university college london and heriot-watt university) has documented this variation across 249 deaf signers from 8 uk sites.

## the bsl corpus

the bsl corpus project is the primary large-scale research dataset for bsl. key facts:

- collected from 249 bsl deaf signers across england, scotland, wales, and northern ireland
- annotated using elan software with gloss-level and linguistic annotations
- available online via the bsl corpus project website (bslcorpusproject.org)
- focus on sociolinguistic variation, language change, and lexical frequency
- led by university college london in collaboration with heriot-watt university and others
- the bsl signbank (lexical database of 50,000 signs from 4 regions) was derived from this corpus

the bsl corpus is not as large as the largest asl datasets, and gloss annotation is still incomplete. this is one of the key research challenges for the bsl side of glovterpreter.

## bsl syntax project

a follow-on to the bsl corpus, the bsl syntax project (bslcorpusproject.org/projects/bsl-syntax-project/) conducted the first large-scale study of bsl's grammatical system, combining experimental and corpus-based approaches. it investigated clause structure, word order, and non-manual features in declaratives, questions, and negation.

this project's findings are directly relevant to building a correct bsl interpretation model - not just a vocabulary lookup, but an understanding of how bsl sentences are actually structured.

## key sources

- bsl corpus project: bslcorpusproject.org
- bsl signbank: online lexical database derived from the bsl corpus
- schembri, a. et al. (various) - bsl corpus project publications
- esrc research grant: res-062-23-0825 - british sign language corpus project
- woll, b. - ucl deafness, cognition and language research centre