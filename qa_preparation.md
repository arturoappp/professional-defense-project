# Defense Q&A Preparation (Project 8)

These are the likely mentor questions for the 15-minute discussion. The instructions name five areas: design choices, integration strategy, ethical reasoning, limitations and risks, and real-world deployment. Two more are added here: evaluation, and how the work was built.

Each answer is short, grounded in the Project 7 notebook and paper, and meant to be said in my own words, not read. The numbers come from `integrated_pipeline.ipynb` in https://github.com/arturoappp/industry-synthesis-project.

A good answer has three parts:
1. The decision.
2. The evidence.
3. The trade-off or limitation.

When I do not know something, I say so, and I say how I would find out.

---

## 1. Design choices

**Why a single agent and not a multi-agent system?**
The job is one editorial task: find material, write, revise after feedback. Splitting it across agents adds coordination cost and new failure modes. The only role that must be independent is verification, so the verifier is a separate model call that never sees the answer key and cannot act. It returns a verdict.

**Why doesn't the verifier see the answer key?**
A model that sees the proposed answer tends to agree with it. The verifier has to solve the question itself, with the options shuffled, so it cannot rely on position either. In Project 6 the agent put the correct answer in option A 19 times out of 21, so the shuffle matters.

**Why didn't the knowledge-graph flag simply reject the question?**
Because a knowledge base records relationships but cannot read the question. In the end-to-end runs the graph flagged 7 questions that the agents had accepted. With the evidence in front of it, the verifier judged all 7 correct, and inspection agrees. One was *"Which daughter-in-law kissed Naomi and then left her?"*: Ruth is also Naomi's daughter-in-law, but only Orpah left. A hard rule would have discarded all 7. So the flag is a trigger: the verifier re-reads the question with the evidence verses and a one-line note. The notebook shows this in Section 7.3, below the table of flagged questions.

**Why the one-line note? Aren't the verses enough?**
They were not. With Ruth 4:10 in the prompt, the verifier still accepted Boaz, quoting Ruth 4:13 and stopping. It satisfices: it finds one quotation that supports the key. Adding the explicit relationship raised detection from 24 to 30 of 30. The paired difference is significant, with exact McNemar p = 0.031. The notebook prints the 6 items missed without the note, with the verifier's own reasoning, in Section 7.2.

**Why TF-IDF and logistic regression for the router, and not embeddings?**
It is the method I validated in Project 3. It is cheap, deterministic and explainable, and it is good enough for topics: 33 of 36 editorial topics have an expected book in the top three. It is weaker on paraphrased topics. For example, *"The feeding of the five thousand"* went to Numbers because of census vocabulary. The router only suggests books, and the agent can still search anywhere.

**Why write the controller by hand instead of using a framework?**
Transparency. Every decision, budget and stop rule is plain Python and appears in the trace logs, which is what I need to defend the system's behaviour. The cost is more code.

**What alternatives did you consider?**
- For verification: reading the whole chapter, which caught 20 of 30, and a hard knowledge-graph rejection, which would have discarded 7 correct questions.
- For routing: the knowledge layer alone, which put the right book in the top three for only 21 of 36 topics.
- For the screen: a second model as a classifier. I rejected it for cost and opacity.

## 2. Integration strategy

**Which projects did you integrate, and why these?**
- **Project 1:** the BibleData knowledge layer.
- **Project 2:** data-quality checks and statistics.
- **Project 3:** the topic router.
- **Project 6:** the agent.

I chose them because the problem Project 6 exposed needs exactly these capabilities: structured knowledge across verses, a way to narrow the search, and a way to measure whether the fix is worth it.

**Why not Projects 4 and 5?**
They did not solve a problem in this pipeline. Project 4's language identification would matter for a Spanish version. Project 5's generator writes text that imitates scripture, which is the opposite of what a question bank needs. Their lessons are in the design: deterministic, reproducible runs from Project 4, and verbatim quotation of the text from Project 5's misattribution risk.

**How did the integration improve the solution? Give a number.**
In the stress test, the Project 6 verifier caught 16 of 30 questions with two valid answers, and only 1 of 13 when the second answer was in another verse. The integrated design caught 30 of 30, with exact McNemar p = 0.0001. Retroactively, it flags exactly the Ruth question that Project 6 accepted.

**Is this new work or a resubmission?**
It is new work:
- the relationship table was not used in Project 1;
- entity linking, the lookup tool, the ambiguity flag, hybrid verification, the router, the request screen and the stress test are new;
- the Project 6 agent was refactored so that every component can be switched on or off.

## 3. Ethical reasoning

**What is the main ethical risk?**
Teaching scripture wrongly at scale. A wrong key penalises the player who knows the text best, and no single person decided it. The mitigations:
- the pipeline only produces candidates;
- a human editor approves every item;
- each item carries its reference, a verbatim quotation, the verifier's reasoning and the knowledge-graph status.

**Is the system biased?**
Yes, in its boundaries, and that should be stated. It uses the 66-book Protestant canon, but the Book of Enoch is scripture for the Ethiopian Orthodox Church. It is a legitimate product choice, but not a neutral fact. The agent also tends to ask about the best-known moments of each story.

**Who is accountable if a wrong question reaches players?**
The editor who approved it and the team that owns the pipeline, never "the model". That is why human approval is part of the design and why every question keeps its provenance. It follows the NIST AI Risk Management Framework's idea of human oversight proportionate to impact.

**What about misuse and manipulation?**
In Project 6, an instruction hidden in the topic made the agent cite Genesis 1:1. The new screen blocks that text before any model call, deterministically. But it is a list of patterns and a paraphrase passes, so the code-level checks remain the real control. Only public data is used, and no player data.

## 4. Limitations and risks

**What is the weakest part of the system?**
The knowledge base:
- 54 % of its relationships are inferred rather than quoted;
- 36 of 5,484 evidence references do not resolve, mostly because of alternative book codes such as `JHN` instead of `JOH`;
- the one false alarm came from an uncertain entry, which BibleData itself describes as "second wife of Machir [or second sister of the wife?]".

This is why the knowledge base triggers re-verification instead of deciding.

**What surprised you?**
Scope drift. Asked about the Book of Enoch, the Project 6 configuration declined, but the integrated agent followed the router's suggestions and wrote questions about Revelation 12. A helpful component changed a decision about scope. The fix is a scope rule that runs before the router.

**Did the agent use the lookup tool?**
Only 3 times in 9 sessions, although the prompt asks it to. A tool the agent may use is not a safeguard. A check that always runs is. The next version should run the lookup automatically.

**How general is the ambiguity check?**
It is narrow on purpose. It applied to 7 of the 29 integrated questions: those whose answer is a single person related to someone named in the question. It is insurance for a frequent and damaging question type, not a general quality score.

**Is the end-to-end comparison statistically meaningful?**
Not very. It is one random run per configuration with about 30 questions each, and both configurations accepted 29 of 32 with no ambiguous keys (Fisher p = 1.0). I report it honestly as no measurable difference in normal use. The controlled stress test is where the effect is measurable.

**Isn't the stress test circular?**
Partly, and I say so in the notebook. The labels come from the same knowledge base, so the knowledge-graph flag alone scores 30 of 30 by construction. The test is not meant to grade the knowledge base. It measures how often a model verifier misses a second valid answer, and every label can be checked by reading the evidence verses.

## 5. Real-world deployment

**Is this pipeline running in your app today?**
No. It is the pipeline I plan to put in front of the app's question workflow. The Project 7 results come from public data and a controlled evaluation, not from production.

**How are questions created in the app today?**
The app generates questions with a language model and screens them with language-model judges. This is the same writer-plus-verifier design I rebuilt in Project 6, and its blind spot is what Project 7 measures. I have not yet run the stress test against the production judges, so I do not claim they have the same miss rate. That test is my first next step.

**How widely is the app used?**
It has been on Google Play since April 2013, with more than 500,000 downloads and a 4.4-star rating from about 30,600 reviews, according to the public store listing. That is why a wrong key matters: many people learn from these questions every day.

**How big is the problem in your app?**
- **Catalog:** 53,831 questions in three languages (16,287 in Spanish, 18,534 in English and 19,010 in Portuguese).
- **After publication:** it has received 3,178 corrections and 571 withdrawals.

These are aggregate counts from the app's content files. They are not player data, and the pipeline does not use them.

**Have players reported wrong answers?**
Not through a formal channel. The app's report feature is used for community posts, and it holds no question reports. Errors have been found through internal review, which is exactly the slow, manual step this pipeline is meant to shorten: an editor receives each candidate with its verse, the verbatim quotation and the verifier's reason.

**The app's questions have three options, and yours have four. Does that matter?**
No. The number of options is a parameter of the schema and the checks. The ambiguity check compares every distractor with the answer, so it works the same with two distractors.

**What would you need before production?**
- a scope rule before the router;
- an automatic person lookup;
- normalised book codes;
- a Spanish version with a public-domain Spanish Bible;
- monitoring of flags and rejections;
- repeated live runs to measure variability;
- an editor review interface built around the provenance fields.

**What does it cost?**
The integrated pipeline used 36 % more tokens per accepted question than the Project 6 configuration: about 10,200 against 7,500, with gpt-4.1-mini writing and gpt-4.1 verifying. The extra verification call only happens when the knowledge graph raises a flag.

**What if the model provider changes or retires the model?**
The model versions are pinned, and every response of the documented run is recorded, so the results are reproducible. Future behaviour is not guaranteed. A model change would require re-running the stress test before release, and the stress test is cheap to repeat.

**Would this work outside Bible trivia?**
Yes. The pattern fits any AI-generated assessment, such as certification exams, compliance training or medical education: curated structured knowledge used as a trigger for re-verification, ablations to justify each component, and provenance for fast human review.

## 6. Evaluation and statistics

**Why Wilson intervals?**
The samples are small, for example 30 items or 36 topics, and the normal approximation behaves badly near 0 or 1. Wilson intervals stay inside [0, 1] and have better coverage at these sizes.

**Why McNemar and not a chi-square between methods?**
The methods judge the same items, so the data are paired. McNemar uses only the discordant items: those caught by one method and missed by the other. The exact version suits small counts.

**What does the difficulty analysis show?**
Hard questions had answers mentioned in a median of 3 verses, against 22 for medium ones. The correlation has the expected sign (rho = -0.39) but is not significant (p = 0.163, n = 14). Prominence is a useful signal, but alone it cannot label difficulty.

## 7. How the work was built (authorship)

Be ready to walk through the notebook live:
- the knowledge-layer quality table;
- the stress-test table;
- one trace file, for example the integrated Enoch session in `runs/traces/`;
- `mdlb/agent.py`, where the flag triggers the second verifier call.

Know where each number in the paper comes from, and explain the replay cache: why the notebook reproduces the documented run without an API key, and what changes in a live run.
