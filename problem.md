# the problem

## captioning is not interpretation

the dominant form of accessibility tech for deaf and hard-of-hearing people across enterprise products, live events, and online platforms is live captioning: speech converted into a running text feed.

captioning is useful. it is also, on its own, an incomplete answer. it quietly assumes that the person reading it is comfortable processing english (or whichever spoken language is being captioned) as a working language. for a large portion of the deaf community, that assumption does not hold.

for someone whose first language is asl or bsl, reading english captions is not equivalent to receiving information in their first language. it is closer to a hearing english speaker being handed subtitles in a language they only partially know.

## the braille mistake

a useful comparison: braille is a representation of a spoken/written language, letter by letter, in tactile form. it exists to give blind readers access to english (or another written language) directly. sign languages are not the deaf equivalent of braille.

asl is not english performed with hands. bsl is not english performed differently. they are independent languages, with their own grammar, their own word order, their own spatial and temporal structures. in the same way that french or japanese are independent of english. treating "add captions" as equivalent to "make this accessible to deaf people" repeats the same category error as assuming braille and sign language are the same kind of thing.

## why word-by-word substitution doesn't work

current automated sign language tools, to the extent they exist, tend to treat interpretation as word substitution: take an english word, map it to a sign, move on to the next word. this is not interpretation. it is roughly equivalent to a french "translator" who replaces each english word with the nearest french word without adjusting grammar, word order, or structure. the output is technically composed of french words, but it isn't french.

asl has topic-comment sentence structure (object before subject-verb, not after). bsl has its own distinct grammar that also differs from english. neither can be correctly produced by mapping english words one-to-one to signs in sequence.

real interpretation requires waiting for a complete unit of meaning - a clause, a sentence, sometimes two, before rendering the signed equivalent. this is how human interpreters work, and it's the only linguistically valid approach for a machine to take as well.

## the gap

the result of all this: a large and growing body of "assistive technology" serves people who can read the spoken language comfortably, and does not serve people whose first language is a signed one. the problem isn't just technical, it's that the field has consistently modelled deaf accessibility on what's easiest to build (text output) rather than on what actually serves the people it claims to help.

glovterpreter exists to close that gap, starting with a single speaker, a browser tab, and the two most widely used sign languages in the english-speaking world. intentionally starting small because of the disconnect between most accessibility tech that does exist already doesn't tackle the issue we are focussing on, requiring to start from scratch and build diligently without flattening.