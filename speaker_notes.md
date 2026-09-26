# Speaker Notes — Capstone Defense (Project 8)

Script for the 15-minute presentation, one section per slide. It is a guide to rehearse with, not something to read aloud: the rubric asks the presenter to explain in their own words.

Total: 1750 words, about 13.5 minutes at 130 words per minute and about 15.2 minutes at 115.

## Slide 1 · Trustworthy question generation for a Bible-trivia app

*About 48 seconds.*

Good morning, and thank you for your time. I'm Arturo Ramos. Today I'll present the synthesis project of my capstone: an AI pipeline that writes multiple-choice questions for a Bible-trivia app, El Master de la Biblia, and makes sure every answer key is really correct. It brings together four of my earlier projects: the BibleData workflow, the statistical analysis, the text classifier and the question-writing agent. I'll cover the problem, the design, why I combined these projects, the main trade-offs, the ethics, and what the evaluation showed, including what did not work. I have about fifteen minutes, and then I'm happy to take any questions.

## Slide 2 · Every answer key is a small act of teaching

*About 60 seconds.*

The industry is faith-based educational technology. El Master de la Biblia is a gamified trivia app, and every question is a small act of teaching. If the key is wrong, the player who knows the text best is marked wrong. Writing hundreds of questions by hand does not scale, and a language model writing from memory misquotes verses. In Project 6 I built an agent that reads the text, checks its work and has a second model verify each answer without seeing the key. It still accepted this question: who did Ruth marry, with Boaz as the only key. But Mahlon was also an option, and Ruth 4:10 calls Ruth the wife of Mahlon. The verifier only saw the cited verse. That failure is the problem this project solves.

## Slide 3 · One model is not enough for this problem

*About 56 seconds.*

Why AI, and why more than one technique? A language model is the right tool to write varied, natural questions. But a model judges what is in front of it: if the second valid answer lives in another verse, it cannot know. Structured knowledge can, because it records relationships across the whole Bible. And I need statistics to decide whether a component is worth its cost, not just whether it looks good. There are also real constraints. I only use public data: the World English Bible, which is public domain, and BibleData, which is licensed CC BY 4.0. No player data from the production app is used, and the output is always a candidate for a human editor, never published automatically.

## Slide 4 · Screen, route, act, check, measure

*About 66 seconds.*

Here is the system. A request from an editor, a topic, a number of questions and a difficulty, is first validated and screened for instruction-like text, without any model call. The topic router, from Project 3, suggests the books where the topic is likely to be. The agent, from Project 6, searches and reads the World English Bible, and it can look people up in the knowledge layer built from BibleData in Project 1. Every question it submits goes through six deterministic checks. Then comes the new step: if the answer is a person and a distractor has the same relationship to someone in the question, the knowledge graph raises a flag, and the blind verifier re-judges the question with the evidence verses and a one-line note. Accepted questions go to a bank with their provenance, and Project 2's statistics measure the whole thing.

## Slide 5 · Four prior projects, each changed by the integration

*About 68 seconds.*

Four projects are integrated, and each one changed. From Project 1 I reused the BibleData cleaning: the same code reproduced its counts exactly. What is new is the relationship table, 5,450 family relationships with evidence verses, and entity linking, because 482 names are shared by several people. Project 2 taught me to profile every join; here that found 36 evidence references that do not resolve, mostly alternative book codes. It also gave me the evaluation methods. Project 3 gave me the classifier recipe and the lesson to split by group; the router is that recipe over 66 books, split by chapter. Project 6 gave me the agent and, most importantly, its failure cases. I made every component switchable, so the same code runs the old configuration as a baseline. Projects 4 and 5 informed the reproducibility practices and the rule that every question must quote the text verbatim.

## Slide 6 · The knowledge graph triggers re-verification; it does not reject

*About 66 seconds.*

The most important decision was how to combine the knowledge graph with the language model. My first design treated a flag as a rejection. It failed on a real case: which daughter-in-law stayed with Naomi was rejected, because Orpah is also her daughter-in-law, although only Ruth stayed. A database cannot read the word stayed. And the agent replaced Orpah with worse distractors. So the flag became a trigger: the verifier re-judges the question with the evidence verses of both relationships. A second surprise: even with Ruth 4:10 in front of it, the verifier still accepted Boaz. It found one quotation that supported the key and stopped. Only when I added an explicit one-line note, Mahlon is also a husband of Ruth, did it look for a second answer. Structured data and the model are complementary: the data is explicit, the model reads language.

## Slide 7 · Transparency first, and a cost I can measure

*About 58 seconds.*

Other decisions traded capability for transparency. The agent loop is a hand-written controller with the OpenAI SDK, not a framework, so every decision, budget and stop rule is visible in the code and in the logs, which matters when I have to defend what the system did. The router is a linear model instead of embeddings: cheap, deterministic and explainable, but weaker on paraphrased topics. The request screen is a visible list of patterns, not a second model: free and auditable, but a paraphrase gets through, and I show that in the notebook. All of this has a measurable cost: the integrated pipeline used 36 percent more tokens per accepted question. Each component also adds maintenance, so I kept only components whose benefit I could measure.

## Slide 8 · Accountability for teaching scripture wrongly

*About 64 seconds.*

The central ethical concern is accountability for teaching scripture wrongly at scale. A wrong key penalises the player who knows the text best, and no single person decided it. So the pipeline only produces candidates: a human editor approves every question, and each one carries its reference, the verbatim quotation, the verifier's reasoning and the knowledge-graph status, so it can be audited in seconds. Second, bias in the boundaries: the persona fixes the 66-book Protestant canon, but the Book of Enoch is scripture for the Ethiopian Orthodox Church. That is a legitimate product choice, but it should be stated, not presented as neutral. Third, manipulation: in Project 6 a prompt injection made the agent cite Genesis 1:1. Now it is blocked before any model call, and the code checks remain behind it. And only public data is used.

## Slide 9 · The model verifier checks only what it is shown

*About 60 seconds.*

First, the evaluation. The retro-check on Project 6's 21 accepted questions flagged exactly the Ruth question. Then I built a stress test from BibleData: 30 relationship questions with two valid answers, and 30 controls with only one. The Project 6 verifier caught 16 of 30. When the second answer was named in another verse, it caught only 1 of 13. Reading the whole chapter raised this to 20. The hybrid design caught all 30, with one false alarm in 30 controls; the paired McNemar test gives p equals 0.0001. The false alarm was instructive: BibleData itself marks that relationship as uncertain, so the verifier was right. One caveat: the labels come from the same knowledge base, so this measures what the model misses, not how good the knowledge base is.

## Slide 10 · Same quality in normal use, at a higher cost

*About 56 seconds.*

End to end, I ran both configurations on the same nine requests. Both accepted 29 of 32 questions, and neither let an ambiguous key into the bank. When the agent writes its own relationship questions, it usually makes them specific, for example the third son of David according to 2 Samuel 3:3. So in normal use the difference is not visible; the integration is insurance against a rare but damaging error, like the Ruth question, and it costs 36 percent more tokens. The one deterministic gain is the injection: without the screen, whether the agent obeys it depends on sampling; in Project 6 it did, in this run it did not. With the screen it is blocked every time, at zero cost.

## Slide 11 · Failure cases and unexpected behaviour

*About 60 seconds.*

Now what did not work, which I think is the most useful part. First, scope drift. Asked about the Book of Enoch, the Project 6 configuration declined, as its persona says. The integrated agent received book suggestions from the router and wrote three questions about Revelation 12, without telling the editor. A helpful component changed a decision about scope. Second, I told the agent to look people up before writing a person question, and it did so only three times in nine sessions. A tool the agent may use is not a safeguard; a check that always runs is. Third, the screen is a pattern list: a paraphrased injection passes. And fourth, the knowledge base itself can be uncertain, which is why a flag triggers re-verification instead of rejecting.

## Slide 12 · What would stop this from going to production tomorrow

*About 54 seconds.*

These are the limitations I would have to solve before production. The knowledge base is curated, but 54 percent of its relationships are inferred rather than quoted, and 36 evidence references do not resolve. The ambiguity check is narrow: it applied to 7 of the 29 integrated questions, those whose answer is a person related to someone in the question. The end-to-end comparison is a single random draw per configuration, with about 30 questions each, so it has little statistical power. The system works in English, while the app's players speak Spanish. And it depends on a paid API whose models can change or be retired; the recorded responses keep my results reproducible, but not future behaviour.

## Slide 13 · A pattern for any AI-generated assessment

*About 54 seconds.*

Why does this matter professionally? Any organisation that uses language models to write assessment items, certification exams, compliance training, medical education, faces the same failure: a key that is defensible from one source and wrong given another. The pattern transfers: curated structured knowledge used as a trigger for re-verification, ablations that show which component earns its cost, and provenance that makes human review fast. For me, this project shows I can combine data work, classical machine learning and agents, evaluate them honestly, and treat ethics as design. Next steps follow from the failures: a scope rule before the router, an automatic person lookup, normalised book codes, a Spanish version, and difficulty calibrated with real, anonymised player data.

## Slide 14 · Thank you. Questions are welcome.

*About 37 seconds.*

To sum up: a language model verifier missed questions with two valid answers when the second answer lived in another verse. Structured knowledge from my first project, used as a trigger for re-verification, closed that gap in a controlled test, at a measurable cost, and the evaluation also showed new risks, like scope drift, that I would fix next. The code, the notebook and the paper are in the repositories on this slide. Thank you, I'm happy to take your questions.
