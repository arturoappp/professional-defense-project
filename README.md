# Professional Industry Defense: Capstone Presentation Materials

Materials for the live defense of the capstone, a 30-minute session with a Udacity mentor: 15 minutes of presentation in English and 15 minutes of discussion. The defense is based entirely on the Project 7 synthesis, **Trustworthy Question Generation for a Bible-Trivia App: An Integrated AI Pipeline**. It introduces no new technical work.

## Contents

| File | What it is |
|---|---|
| `Capstone_Defense_Slides.pdf` | The 14-slide deck (16:9), covering the seven required sections in order. |
| `figures/architecture_simple.png` | The simplified architecture diagram of slide 4, readable when the screen is shared. The detailed diagram is Figure 1 of the Project 7 notebook. |
| `speaker_notes.md` | What to say on each slide, with an estimated time per slide, for rehearsal. It is not meant to be read aloud. |
| `qa_preparation.md` | Likely mentor questions on design, integration, ethics, limitations, deployment, evaluation and authorship, with answers grounded in the Project 7 results. |

## Structure and Timing (15 minutes)

| Required section | Slides | Time at 120 words per minute |
|---|---|---|
| 1. Industry context and problem definition | 1-3 | 2:48 |
| 2. Integrated AI system overview | 4 | 1:06 |
| 3. Integration of prior capstone projects | 5 | 0:56 |
| 4. Key technical decisions and trade-offs | 6-7 | 2:12 |
| 5. Ethical considerations and responsible AI | 8 | 1:09 |
| 6. Evaluation, limitations and risks | 9-12 | 3:56 |
| 7. Professional relevance and next steps | 13-14 | 1:42 |
| **Total** | 14 | **13:50** |

These times come from the word count of the speaker notes (1,660 words), and `speaker_notes.md` gives the time of each slide. The margin to 15 minutes is small, so there is no room to read slides aloud. Rehearse with a timer: if a run passes 14:30, shorten slide 5 or slide 12.

## The Work Being Defended

| Project | Repository |
|---|---|
| **7. Industry synthesis (defended here)** | https://github.com/arturoappp/industry-synthesis-project |
| 1. AI programming foundations | https://github.com/arturoappp/ai-programming-foundations-project |
| 2. Statistical analysis | https://github.com/arturoappp/statistical-analysis-project |
| 3. Applied machine learning | https://github.com/arturoappp/applied-machine-learning-project |
| 4. Deep learning systems | https://github.com/arturoappp/deep-learning-systems-project |
| 5. Generative AI applications | https://github.com/arturoappp/generative-ai-applications-project |
| 6. Agentic workflows | https://github.com/arturoappp/agentic-workflows-project |

Project 7 integrates Projects 1, 2, 3 and 6.

## Key Numbers to Remember

- **The app** (aggregate counts from its content files, not player data):
  - 53,831 questions in Spanish, English and Portuguese, generated with an LLM and screened by LLM judges;
  - 3,178 corrections and 571 withdrawals since publication.
  - The pipeline is not in production yet: it is what I plan to put in front of the app's question workflow.
- **Problem:** the Project 6 agent accepted *"Who did Ruth marry?"* with Boaz as the only key, although Ruth 4:10 calls Ruth "the wife of Mahlon".
- **Stress test** (30 questions with two valid answers and 30 controls):

  | Design | Caught |
  |---|---|
  | Project 6 verifier | 16 of 30 (1 of 13 when the second answer is in another verse) |
  | Whole chapter | 20 of 30 |
  | Hybrid design | 30 of 30, with 1 false alarm in 30 controls (exact McNemar p = 0.0001) |

- **End to end** (9 requests):
  - both configurations accepted 29 of 32 questions, with no ambiguous key;
  - the integrated pipeline costs 36 % more tokens per accepted question;
  - it blocks the known prompt injection before any model call.
- **Unexpected:** asked about the Book of Enoch, the integrated agent followed the router's hints and wrote Revelation 12 questions instead of declining.

## Scheduling

Udacity emails a Calendly link to book the session with a mentor. Book it promptly, test the screen share and the PDF beforehand, and have the Project 7 repository open to show the notebook, a trace file and `mdlb/agent.py` if asked.
