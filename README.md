# Professional Industry Defense: Capstone Presentation Materials

Materials for the live defense of the capstone, a 30-minute session with a Udacity mentor: 15 minutes of presentation in English and 15 minutes of discussion. The defense is based entirely on the Project 7 synthesis, **Trustworthy Question Generation for a Bible-Trivia App: An Integrated AI Pipeline**. It introduces no new technical work.

## Contents

| File | What it is |
|---|---|
| `Capstone_Defense_Slides.pdf` | The 14-slide deck (16:9), covering the seven required sections in order. |
| `speaker_notes.md` | What to say on each slide, with an estimated time per slide, for rehearsal. It is not meant to be read aloud. |
| `qa_preparation.md` | Likely mentor questions on design, integration, ethics, limitations, deployment, evaluation and authorship, with answers grounded in the Project 7 results. |

## Structure and Timing (15 minutes)

| Required section | Slides | Target time |
|---|---|---|
| 1. Industry context and problem definition | 1-3 | 3:00 |
| 2. Integrated AI system overview | 4 | 1:15 |
| 3. Integration of prior capstone projects | 5 | 1:15 |
| 4. Key technical decisions and trade-offs | 6-7 | 2:15 |
| 5. Ethical considerations and responsible AI | 8 | 1:15 |
| 6. Evaluation, limitations and risks | 9-12 | 4:15 |
| 7. Professional relevance and next steps | 13-14 | 1:30 |

The speaker notes add up to about 1,750 words. That is about 13.5 minutes at 130 words per minute and about 15 minutes at a calmer 115, which leaves no room to read slides aloud. Rehearse with a timer.

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
