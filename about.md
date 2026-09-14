# about glovterpreter

## what it is

glovterpreter at it's initial stage is a browser-run tool that sits in a user's browser tab and provides live sign language interpretation of a single english speaker's speech.

it listens. it waits. when the speaker completes a full, semantically coherent thought. one that can be properly interpreted without needing to hear what comes next. glovterpreter renders the interpretation as live hand movement on a low-poly 3d hand model. 

the user can toggle between outputs in
- asl (american sign language) 
- bsl (british sign language)

## who it's for

deaf and hard-of-hearing users whose first language is asl or bsl, attending:

- online lectures, webinars, or courses
- live-streamed talks or panels
- virtual events and conferences
- any single-speaker scenario delivered in english

## the linguistic premise

asl and bsl are not hand-encoded english. they are independent languages with their own grammar, syntax, spatial structure, and word order. a fluent asl signer reading english captions is not receiving information in their first language, any more than a french speaker would be if handed german subtitles.

the implication for software is significant: you cannot interpret sign language word-by-word as speech streams in, because the signed grammar of the output depends on the full structure of the utterance. you have to wait for a complete semantic unit before you can produce a grammatically correct sign sequence.

this is why glovterpreter is built around a pause-then-interpret model rather than a streaming word-substitution one. the pause isn't a limitation at all that needs correcting or fine tuning, it's the linguistically correct behavior.

## why a 3d hand, not text

the output of glovterpreter is hand movement, not a gloss transcript or a text feed. this is intentional. the point of the tool is to deliver meaning in asl or bsl only, not to produce an english-adjacent representation of that meaning. 

a signing hand is the medium of those languages. rendering it visually, even in a simplified low-poly form, is a more faithful delivery of the interpreted content than any text output would be.

the low-poly aesthetic is also a deliberate choice: it signals that this is a functional interpretation tool, not an attempt to simulate a human interpreter. the hand signs correctly. that's what matters.