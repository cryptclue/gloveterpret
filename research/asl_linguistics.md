# asl linguistics

## asl is a language, not a code

american sign language is a full, natural human language. it was formally recognized as such starting with william stokoe's 1960 linguistic analysis, which was the first work to treat asl as having genuine grammatical structure rather than being a simplified gesture system. before stokoe, the prevailing view - including within deaf education - was that asl was not a real language. that view was wrong.

asl has its own phonology (the sub-lexical structure of signs, described in terms of handshape, location, movement, and orientation rather than sound), its own morphology, its own syntax, and its own pragmatics. it is not derived from english and is not a visual encoding of english.

## grammar: how asl differs from english

### word order

english uses subject-verb-object (svo) order as its default: "i eat apples."
asl most commonly uses **topic-comment** structure, or object-subject-verb (osv): "apple, i eat" or "apple me eat."

this is not a stylistic variation - it is a grammatical feature of the language. an automated system that maps english words to asl signs in english word order is not producing asl. it is producing english glossed with hand shapes.

### spatial grammar

asl uses the signing space (the area in front of the signer's body) grammatically. referents - people, objects, ideas - are assigned to locations in space and then referred back to by pointing at those locations. verbs can move between spatial referents to indicate subject and object without separate pronoun signs.

example: if "john" is assigned to the left and "mary" to the right, a verb moving from left to right means "john [verbs] mary" and from right to left means "mary [verbs] john." this is called **verb agreement** and it has no equivalent in english grammar.

### tense and aspect

asl does not mark tense with verb inflections the way english does (walk → walked). time is established at the beginning of a clause using time signs (yesterday, tomorrow, future, past) and then remains in force until changed. aspect; whether an action is completed, ongoing, or repeated - is marked by modifying the movement of the sign (slower, repeated, tense).

### non-manual markers

facial expressions, eyebrow position, mouth shape, head tilt, and body lean are all grammatically meaningful in asl. these are called **non-manual markers (nmms)**. they are not emotional signals but rather, they are part of the grammar.

examples:
- raised eyebrows mark yes/no questions
- furrowed brows mark wh-questions (who, what, where)
- head shake is grammatical negation (separate from the sign not)
- mouth shapes can modify meaning of signs

non-manual markers are out of scope for the current glovterpreter build but are a required future layer for linguistically accurate output.

## gloss notation

asl gloss is a written notation system for representing asl utterances in english text, used by linguists and researchers. key conventions:

- signs are written in **uppercase**: book, eat, go
- classifiers use prefix notation: cl:b (flat hand classifier)
- pointing/indexing: ix (index to a spatial referent)
- spatial assignment: john(rt) assigns john to right space
- non-manual markers in parentheses: (q) for question nmm
- repeated movement: ++  (eat++ = eat repeatedly)
- verb movement between referents: john(lt)→give→(rt)mary

**gloss is not asl.** it is a notational approximation that captures structure but omits phonological detail, prosody, and much of the non-manual information. for glovterpreter, gloss serves as the intermediate representation between english input and sign output i.e. a structured form that the motion mapping layer can work from.

example:
- english: "are you busy two weeks from now?"
- asl gloss: two-week-future saturday ix-you busy?

the word order, the temporal placement, and the question marker are all different from the english source.

## key sources

- stokoe, w.c. (1960). *sign language structure*. studies in linguistics, occasional papers 8.
- baker, c. & cokely, d. (1980). *american sign language: a teacher's resource text on grammar and culture.*
- valli, c. & lucas, c. (2000). *linguistics of american sign language.*
- lifeprint.com (dr. bill vicars) - practical asl gloss conventions reference
- kent state university, dept. of modern & classical language studies - asl linguistics program