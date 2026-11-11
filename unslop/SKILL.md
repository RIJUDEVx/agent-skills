---
name: unslop
description: Rewrite prose people will read (PR bodies, READMEs, docs, issue text) to remove AI tells. Use also for "explain simply" or "plain English".
---

# Unslop

Remove the patterns that make text read as machine written. Keep the meaning and the tone the author wanted.

## How to use this skill

1. Read the text and find the patterns in the list below.
2. Rewrite each one you find. Keep the meaning.
3. Read your version once more and ask what makes it obviously AI written. Fix whatever you find.

## Explain simply

When asked to explain simply, or in plain English, write for a reader who has to take in each sentence at once:

- Short sentences, one idea each.
- Everyday words.
- No jargon unless you define it first.

Rule 28 covers dense sentences in all writing. This section sets the bar when the user asks for simple.

## Pattern list

Rule numbers are stable ids that other skills cite by number. Keep the numbers where they are. If you delete a rule, leave its number as a gap instead of renumbering the rest.

Every rule below is one line. The full word lists and the before and after rewrites live in the reference files.

### Content

- **3. Superficial -ing phrases.** "highlighting", "ensuring", "reflecting", "showcasing", "fostering". Delete the phrase, or finish the sentence with a fact or a source.
- **5. Vague attributions.** "Experts believe", "reports suggest", "some argue". Name the source, or cut the claim.

### Language

- **7. AI vocabulary.** additionally, crucial, delve, enduring, enhance, foster, garner, interplay, intricate, landscape (when abstract), pivotal, showcase, tapestry (when abstract), testament, underscore, vibrant. Swap table in `references/word-swaps.md`.
- **8. Fancy ways to say "is".** "serves as", "stands as", "boasts", "features". Say "is" or "has".
- **9. "Not just X, but Y."** State the point directly in one sentence. Examples in `references/examples.md`.
- **10. Rule of three.** Ideas forced into groups of three. Use the number the author actually had.
- **11. Synonym cycling.** protagonist, main character, central figure, hero in one paragraph. Pick one word and repeat it.
- **12. False ranges.** "from X to Y" where X and Y are not two points on the same scale. List the topics instead.

### Style

- **13. Em dashes.** Do not use them. Split the sentence or use a comma. This covers en dashes, hyphens used as dashes, and parentheses used as dashes too.
- **14. Colons as connectors.** A colon is fine before a list or a quoted example. A colon that sets up a contrast mid sentence carries nothing, so cut it and let the point stand. Examples in `references/examples.md`.
- **15. Boldface overuse.** Do not bold every name or acronym.
- **16. Inline header lists.** A bold label and colon that restates the line, such as "**Performance:** Performance improved". Turn it into prose. A bold lead-in that ends in a period and adds new detail, such as "**Schema in TypeScript.** Tables live in one file.", is fine.
- **17. Title case headings.** Use sentence case.
- **18. Decorative emojis.** Remove them from headings and bullets.
- **19. Curly quotes.** Use straight quotes.

### Communication artifacts

- **20. Chatbot phrases.** "I hope this helps!", "Let me know if...", "Of course!", "Certainly!", "Found the smoking gun!". Delete them.
- **22. Sycophancy.** "Great question! You're absolutely right!" Answer the question.

### Filler

- **23. Filler phrases.** "in order to" becomes "to". "due to the fact that" becomes "because". "it is important to note that" gets deleted.
- **24. Hedging stacks.** "could potentially possibly be argued that it might" becomes "may".
- **25. Generic conclusions.** "The future looks bright." Name the next step, or give the number.

### Jargon

- **26. Abstract metaphor nouns.** substrate, wedge, vector, locus, vantage, nexus, primitive, harness, surface, bedrock, scaffolding, modality, paradigm, gold-plating, ratchet, evacuate, endgame, north star, flywheel. Each one hides a plain word. Swap table in `references/word-swaps.md`.

### Plain speech

- **27. Say what it does, not how it feels.** "the database stays close at hand" and "SQL you can read" describe a mood. Write the mechanism or the number instead, such as "`.toSQL()` returns the exact string sent to the database", or "renaming a column fails the build". If you cannot restate the line as an instruction, a fact, or a number, cut it. Examples in `references/examples.md`.
- **28. Dense sentences.** If the reader has to reread a sentence, split it or drop a clause. One idea per sentence.
- **29. Passive voice.** Name the actor. "queries are validated" becomes "the compiler validates queries". Keep the passive only when the actor is unknown or does not matter.
- **30. Adverbs.** Cut the adverb, or pick a stronger verb. "runs quickly" becomes "is fast", or the number. An adverb holding up a weak verb means the verb is wrong.
- **31. Fancy plain words.** utilize, leverage, facilitate, numerous, in the event that. Swap table in `references/word-swaps.md`.
- **32. Mannered prose.** Aphorisms, fragments used for effect, personified code, figurative verbs, stock framing phrases. Say the literal thing. Rule 26 covers the metaphor nouns. Examples in `references/examples.md`.
- **33. Over-compression.** Missing articles, verbless fragments, arrows, abbreviations. Write the whole sentence. Examples in `references/examples.md`.

## Reference files

Read these when you need the full list instead of the one line summary.

- `references/word-swaps.md` holds the word lists for rules 7, 26, and 31, with the plain word to use instead.
- `references/examples.md` holds before and after rewrites for the rules that are hard to spot on their own, mostly 9, 13, 14, 16, 27, 29, 32, and 33.
