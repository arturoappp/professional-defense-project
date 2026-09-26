# Speaker Notes — Capstone Defense (Project 8)

Script for the 15-minute presentation, one section per slide. It is a guide to rehearse with, not something to read aloud: the rubric asks the presenter to explain in their own words.

Total: 1692 words, about 14:06 minutes at 120 words per minute.

## Slide 1 · Trustworthy question generation for a Bible-trivia app

*About 34 seconds at 120 words per minute.*

Hello, and thank you for your time. I'm Arturo Ramos. Today I'll present the synthesis project of my capstone: a pipeline that writes multiple-choice questions for a Bible-trivia app and makes sure every answer key is really correct. It brings together four of my earlier projects. I'll cover the problem, the design, the integration, the trade-offs, the ethics, and what the evaluation showed, including what did not work.

## Slide 2 · Every answer key is a small act of teaching

*About 90 seconds at 120 words per minute.*

I build and run El Master de la Biblia, a Bible-trivia app I published on Google Play in 2013. It has more than half a million downloads and a 4.4 rating from about thirty thousand reviews, so real people learn from it every day. Its catalog has almost fifty-four thousand questions in Spanish, English and Portuguese. We generate them with a language model and screen them with language-model judges. Players are scored on every question, so a wrong key teaches something false about scripture. And errors do reach the catalog: since publication it has received more than three thousand corrections and 571 withdrawals. In Project 6 I rebuilt that design, a writer and a blind verifier, and it failed on this question: who did Ruth marry, with Boaz as the only key. But Ruth 4:10 calls Ruth the wife of Mahlon, and the verifier only saw the cited verse. This project is the pipeline I plan to put in front of our question workflow. It is not in production yet, and it covers English, one of the app's three languages.

## Slide 3 · One model is not enough for this problem

*About 60 seconds at 120 words per minute.*

Why AI, and why more than one technique? A language model is the right tool to write varied, natural questions. But a model judges what is in front of it: if the second valid answer lives in another verse, it cannot know. Structured knowledge can, because it records relationships across the whole Bible. And I need statistics to decide whether a component is worth its cost, not just whether it looks good. There are also real constraints. I only use public data: the World English Bible, which is public domain, and BibleData, which is licensed CC BY 4.0. No player data from the production app is used, and the output is always a candidate for a human editor, never published automatically.

## Slide 4 · Screen, route, act, check, measure

*About 66 seconds at 120 words per minute.*

Here is the system. An editor's request is first screened for instruction-like text, without any model call. The topic router, from Project 3, suggests the books to search. The agent, from Project 6, reads the Bible and can look people up in the knowledge graph built from BibleData in Project 1. Every question it submits passes six deterministic checks. Then the new step: if a distractor has the same relationship to someone in the question as the answer, the graph raises a flag, and the blind verifier re-judges the question with the evidence verses and a one-line note. Accepted questions go to a human editor with their provenance, and Project 2's statistics measure the whole pipeline. The detailed diagram is Figure 1 of the Project 7 notebook, if you'd like to see it.

## Slide 5 · Four prior projects, each changed by the integration

*About 56 seconds at 120 words per minute.*

Four projects are integrated, and each one changed. From Project 1 I reused the BibleData cleaning, which reproduced its counts exactly; what is new is the relationship table, 5,450 relationships with evidence verses, and entity linking, because 482 names are shared by several people. Project 2 taught me to profile every join; here that found 36 evidence references that do not resolve. It also gave me the evaluation methods. Project 3 gave me the classifier recipe and the lesson to split by group. Project 6 gave me the agent and, most importantly, its failure cases. Every component can be switched off, so the same code runs the old configuration as a baseline.

## Slide 6 · The knowledge graph triggers re-verification; it does not reject

*About 69 seconds at 120 words per minute.*

The key decision was how to combine the knowledge graph with the language model. Why not simply reject every flagged question? Because a database records relationships but cannot read the question. In my end-to-end runs the graph flagged seven questions the agents had accepted, and with the evidence in front of it the verifier judged all seven correct. For example: which daughter-in-law kissed Naomi and then left her? Ruth is also a daughter-in-law, but only Orpah left. A hard rule would have thrown away all seven. So the flag is a trigger for re-verification. The second finding is in the notebook too: with the evidence verses alone, the verifier still accepted Boaz, quoting Ruth 4:13 and stopping. Only the explicit note made it look for a second answer: 24 of 30 without it, 30 of 30 with it.

## Slide 7 · Transparency first, and a cost I can measure

*About 63 seconds at 120 words per minute.*

Other decisions traded capability for transparency. The agent loop is a hand-written controller with the OpenAI SDK, not a framework, so every decision, budget and stop rule is visible in the code and in the logs, which matters when I have to defend what the system did. The router is a linear model instead of embeddings: cheap, deterministic and explainable, but weaker on paraphrased topics. The request screen is a visible list of patterns, not a second model: free and auditable, but a paraphrase gets through, and I show that in the notebook. All of this has a measurable cost: the integrated pipeline used 36 percent more tokens per accepted question. Each component also adds maintenance, so I kept only components whose benefit I could measure.

## Slide 8 · Accountability for teaching scripture wrongly

*About 69 seconds at 120 words per minute.*

The central ethical concern is accountability for teaching scripture wrongly at scale. A wrong key penalises the player who knows the text best, and no single person decided it. So the pipeline only produces candidates: a human editor approves every question, and each one carries its reference, the verbatim quotation, the verifier's reasoning and the knowledge-graph status, so it can be audited in seconds. Second, bias in the boundaries: the persona fixes the 66-book Protestant canon, but the Book of Enoch is scripture for the Ethiopian Orthodox Church. That is a legitimate product choice, but it should be stated, not presented as neutral. Third, manipulation: in Project 6 a prompt injection made the agent cite Genesis 1:1. Now it is blocked before any model call, and the code checks remain behind it. And only public data is used.

## Slide 9 · The model verifier checks only what it is shown

*About 65 seconds at 120 words per minute.*

First, the evaluation. The retro-check on Project 6's 21 accepted questions flagged exactly the Ruth question. Then I built a stress test from BibleData: 30 relationship questions with two valid answers, and 30 controls with only one. The Project 6 verifier caught 16 of 30. When the second answer was named in another verse, it caught only 1 of 13. Reading the whole chapter raised this to 20. The hybrid design caught all 30, with one false alarm in 30 controls; the paired McNemar test gives p equals 0.0001. The false alarm was instructive: BibleData itself marks that relationship as uncertain, so the verifier was right. One caveat: the labels come from the same knowledge base, so this measures what the model misses, not how good the knowledge base is.

## Slide 10 · Same quality in normal use, at a higher cost

*About 56 seconds at 120 words per minute.*

End to end, I ran both configurations on the same nine requests. Both accepted 29 of 32 questions, and neither let an ambiguous key into the bank: when the agent writes its own relationship questions, it usually makes them specific, like the third son of David according to 2 Samuel 3:3. So in normal use the difference is not visible. The integration is insurance against a rare but damaging error, like the Ruth question, and it costs 36 percent more tokens. The injection is the one deterministic gain: without the screen, whether the agent obeys depends on sampling. And the last row is a new failure, which I'll come back to next.

## Slide 11 · Failure cases and unexpected behaviour

*About 64 seconds at 120 words per minute.*

Now what did not work, which I think is the most useful part. First, scope drift. Asked about the Book of Enoch, the Project 6 configuration declined, as its persona says. The integrated agent received book suggestions from the router and wrote three questions about Revelation 12, without telling the editor. A helpful component changed a decision about scope. Second, I told the agent to look people up before writing a person question, and it did so only three times in nine sessions. A tool the agent may use is not a safeguard; a check that always runs is. Third, the screen is a pattern list: a paraphrased injection passes. And fourth, the knowledge base itself can be uncertain, which is why a flag triggers re-verification instead of rejecting.

## Slide 12 · What would stop this from going to production tomorrow

*About 50 seconds at 120 words per minute.*

These are the limitations I would have to solve before production. The knowledge base is curated, but 54 percent of its relationships are inferred rather than quoted, and 36 evidence references do not resolve. The ambiguity check is narrow: it applied to 7 of the 29 integrated questions. The end-to-end comparison is one random draw per configuration, with about 30 questions each, so it has little statistical power. The app publishes in three languages and the pipeline covers only English. And it depends on a paid API whose models can change; the recorded responses keep my results reproducible, not future behaviour.

## Slide 13 · A pattern for any AI-generated assessment

*About 61 seconds at 120 words per minute.*

Why does this matter professionally? Any organisation that uses language models to write assessment items, certification exams, compliance training, medical education, faces the same failure: a key that is defensible from one source and wrong given another. The pattern transfers: curated structured knowledge used as a trigger for re-verification, ablations that show which component earns its cost, and provenance that makes human review fast. For me, this project shows I can combine data work, classical machine learning and agents, evaluate them honestly, and treat ethics as design. Next steps follow from the failures: a scope rule before the router, an automatic person lookup, normalised book codes, and, for my app, running this stress test against our production judges and adding Spanish and Portuguese.

## Slide 14 · Thank you. Questions are welcome.

*About 40 seconds at 120 words per minute.*

To sum up: a language model verifier missed questions with two valid answers when the second answer lived in another verse. Structured knowledge from my first project, used as a trigger for re-verification, closed that gap in a controlled test, at a measurable cost, and the evaluation also showed new risks, like scope drift, that I would fix next. The code, the notebook and the paper are in the repositories on this slide. Thank you, I'm happy to take your questions.
