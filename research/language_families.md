# sign language families: it's a cobweb, not a tree

one of the most important things to understand about asl and bsl, and about why they need to be treated as entirely separate languages in glovterpreter, is that sign language families have almost nothing to do with spoken language families.

the geography and politics that shaped spoken languages? largely irrelevant. the colonial and educational histories that spread specific sign languages? completely decisive. 

the result? a family tree so counterintuitive it reads more like a cobweb.

---

## the central jumpscare: english ≠ english

america and britain share a spoken language. you would be forgiven for assuming their sign languages are related. they are not. asl and bsl are mutually unintelligible. a fluent asl signer in london cannot understand bsl and vice versa.

they come from entirely different lineages.

---

## the lsf cluster: french got around

the single most influential sign language in the world is **french sign language (lsf - langue des signes française)**.

> for reference: in 1817, laurent clerc, a deaf french educator, travelled to america with thomas hopkins gallaudet and co-founded the american school for the deaf in hartford, connecticut. he brought lsf with him. asl developed from the contact between lsf and local signing varieties already present in the us. asl is, in family terms, a daughter of lsf.

the jumpscare that follows from this: **irish sign language (isl) is also lsf-family.** ireland, despite being geographically next to britain and sharing english as a spoken language, has a sign language far more closely related to asl than to bsl. this is because irish deaf education was historically influenced by french catholic models rather than british ones.

so: britain and america share a spoken language → their sign languages are unrelated. ireland and britain share a spoken language and a landmass → irish sl is closer to american sl than british sl. language families don't care about geography.

lsf also influenced **russian sign language** and has descendants across multiple continents.

---

## the banzsl cluster: the one time british colonialism tracks

**banzsl** aka british, australian, and new zealand sign language, is the one cluster where the spoken language family intuition accidentally works. bsl expanded through british colonial educational infrastructure to produce **auslan** (australian sign language) and **nzsl** (new zealand sign language). these three are closely related and have significant mutual intelligibility.

but don't get comfortable. this is the exception, not the rule.

---

## the swedish jumpscare: portuguese is not spanish

here's one that gets people every time. spanish and portuguese are romance languages. they're mutually partially intelligible in spoken form. you would assume portuguese sign language (lgp - língua gestual portuguesa) is related to spanish sign language (lse).

**it is not.** lgp is in the **swedish sign language family**.

portuguese deaf education in the 19th century was influenced by swedish methods and educators rather than spanish ones. the spoken language family is completely irrelevant. lgp and lse are no more related to each other than either is to asl.

swedish sign language (ssl) has an unusually wide reach for a small country's language: ssl influences finnish sl, lgp, and has connections to several other national sign languages.

---

## the german cluster: central europe + israel

**german sign language (dgs - deutsche gebärdensprache)** has influenced **polish sign language (pjm)** and **israeli sign language (isl)** - the latter through the large number of deaf jewish immigrants who came to israel from german-speaking countries and brought dgs with them. israeli sign language is accordingly a dgs-influenced language rather than being related to the sign languages of arabic-speaking neighbours.

(note: isl is the abbreviation for both irish sign language and israeli sign language. linguists use context to disambiguate. it is indeed as confusing as it sounds.)

---

## the full cobweb (simplified)

```
lsf (french sl)
  ├── asl (american sl)
  │     └── (various influences on other american sign languages)
  ├── isl (irish sl)      ← jumpscare: not related to bsl
  └── russian sl

bsl (british sl)  ← not related to asl
  ├── auslan (australian sl)
  └── nzsl (new zealand sl)

swedish sl
  ├── finnish sl
  └── lgp (portuguese sl)  ← jumpscare: not spanish sl family

dgs (german sl)
  ├── pjm (polish sl)
  └── isl (israeli sl)     ← jumpscare: not related to arabic sls
```

and then there are genuinely isolate sign languages - like **kata kolok**, which developed independently in a single village in bali, and **al-sayyid bedouin sign language**, which emerged in an isolated bedouin community in israel. these have no known relatives.

---

## why this matters for glovterpreter

two things follow directly from all of this:

**1. asl and bsl require completely separate models.** they are not dialects. they are not variants. they do not share vocabulary, grammar, or family heritage. building one model for "sign language" and expecting it to produce both asl and bsl correctly is like building one model for "european language" and expecting it to translate french and finnish.

**2. the spoken language of the input does not predict the sign language of the output.** a future version of glovterpreter that supports spanish speech input should not assume spanish sl (lse) output is appropriate for a portuguese deaf viewer - because lgp and lse are unrelated. the sign language a user needs is a product of their community and educational history, not their country's spoken language.

the cobweb is the reality. the tool has to be built to match it.