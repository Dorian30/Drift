# Research Notes

Source check for the claims behind Drift. Checked on 2026-09-10.

Each claim below has a verdict:

- **Solid** means a peer-reviewed study reports the number, and the study design fits knowledge work.
- **Partial** means the research is real, but the popular version of the claim overstates it.
- **Unsupported** means no primary source contains the claim.

Read the "What this means for Drift" line under each claim. That line is the reason the claim is here.

---

## 1. "It takes 23 minutes and 15 seconds to return to an interrupted task"

**Verdict: Unsupported as a citation. The underlying effect is real, but this number is not.**

**Sources:** [Interruptions cost 23 minutes 15 seconds, right?](https://blog.oberien.de/2023/11/05/23-minutes-15-seconds.html) (the source check) and [Too Many Interruptions at Work?](https://news.gallup.com/businessjournal/23146/too-many-interruptions-work.aspx) (Gallup, the earliest traceable appearance).

An independent source check examined five of the papers most often cited for this figure. None of them contains the number. The earliest traceable appearance is a 2006 Gallup interview with Gloria Mark about her fieldwork, not a published paper.

The most-cited paper, Mark, Gudith and Klocke (CHI 2008), never uses the number 23. That paper reports something different, covered in claim 2 below.

**What this means for Drift:** Do not print "23 minutes" anywhere in the product. Use the figures from claim 3, which come from a published field study.

---

## 2. Interruption makes work faster and more stressful

**Verdict: Solid, and it contradicts the popular reading.**

**Source:** Mark, G., Gudith, D., Klocke, U. (2008). The Cost of Interrupted Work: More Speed and Stress. CHI 2008.
[Free PDF](https://www.ics.uci.edu/~gmark/chi08-mark.pdf) - [ACM](https://dl.acm.org/doi/10.1145/1357054.1357072) - [ResearchGate](https://www.researchgate.net/publication/221518077_The_cost_of_interrupted_work_More_speed_and_stress)

From the abstract:

> We found that context does not make a difference but surprisingly, people completed interrupted tasks in less time with no difference in quality. Our data suggests that people compensate for interruptions by working faster, but this comes at a price: experiencing more stress, higher frustration, time pressure and effort.

The authors explain the result this way:

> When people are constantly interrupted, they develop a mode of working faster (and writing less) to compensate for the time they know they will lose by being interrupted.

Design: a controlled lab experiment with an email task, built on earlier field observation. Workload was measured with a modified NASA Task Load Index.

**What this means for Drift:** The cost of context switching is not lost minutes. Measured in minutes, interrupted work sometimes looks better. The cost is workload, stress and effort. Drift measures minutes because minutes are what a Mac can observe. State this limit in the product, and never claim the minute count is a measure of output.

---

## 3. Returning to a suspended task costs 10 to 16 minutes

**Verdict: Solid. This is the number to build on.**

**Source:** Iqbal, S. T., Horvitz, E. (2007). Disruption and Recovery of Computing Tasks: Field Study, Analysis, and Directions. CHI 2007.
[Free PDF](http://erichorvitz.com/CHI_2007_Iqbal_Horvitz.pdf) - [ACM](https://dl.acm.org/doi/10.1145/1240624.1240730) - [Academia.edu](https://www.academia.edu/12906299/Disruption_and_recovery_of_computing_tasks)

A field study of 27 users over two weeks, using a logging tool that recorded application and window switches plus incoming email and instant-message alerts.

From the summary:

> We found that participants spent on average nearly 10 minutes on switches caused by alerts, and spent on average another 10 to 15 minutes (depending on the type of interruption) before returning to focused activity on the disrupted task.

The detailed figures:

| Measure | Email alert | Instant-message alert |
|---|---|---|
| Time on the diversion | 9 min 33 s (S.D. 13 m 15 s) | 8 min 0 s (S.D. 11 m 32 s) |
| Resumption phase, immediate response | 16 min 33 s (S.D. 27 m 20 s) | 10 min 58 s (S.D. 14 m 16 s) |
| Resumption phase, delayed response | 15 min 50 s (S.D. 25 m 5 s) | 12 min 2 s (S.D. 14 m 58 s) |

Three further findings from the same paper matter for the design:

1. **The cost tracks the destination, not the delay.** Resumption time did not differ significantly between immediate and delayed responses. It did differ between email and instant messaging. Where you went predicts the cost better than how fast you went.
2. **Short blocks get abandoned.** Windows where users spent 5 to 30 minutes before suspension were typically resumed within 5 to 15 minutes. If users spent less than 5 minutes on a task before suspension, they had a 10 percent probability of not resuming it within 2 hours.
3. **App switching is constant.** The maximum time spent on an application before switching averaged just above 4 minutes, and the average was below one minute. Alerts arrived at 3.74 per hour.

**Caveat:** every standard deviation here is as large as the mean or larger. These are averages across people and days, not a reliable per-day predictor. Also, the study measured alert-driven interruptions. Drift applies these costs to self-driven switches too, which the study does not cover.

**What this means for Drift:** Set the default re-entry cost per destination app, not per gap length. Finding 1 is the direct evidence for the change you asked for. Finding 3 confirms that frontmost-app tracking sees a real signal at a useful rate.

---

## 4. Attention residue

**Verdict: Solid. This is the mechanism behind the whole project.**

**Source:** Leroy, S. (2009). Why is it so hard to do my work? The challenge of attention residue when switching between work tasks. *Organizational Behavior and Human Decision Processes* 109(2), 168-181. Paywalled.
[RePEc listing](https://ideas.repec.org/a/eee/jobhdp/v109y2009i2p168-181.html) - [Semantic Scholar](https://www.semanticscholar.org/paper/Why-is-it-so-hard-to-do-my-work-The-challenge-of-Leroy/58a602c378da63993ab19b514e1bd57817bc18e5) - [ResearchGate](https://www.researchgate.net/publication/46489122_Why_is_it_so_Hard_to_do_My_Work_The_Challenge_of_Attention_Residue_when_Switching_Between_Work_Tasks)

Leroy found that part of your attention stays on the previous task after you switch, and that this residue lowers performance on the next task. Participants who left a task unfinished performed worse on the following task than participants who finished it. The effect is strongest when the previous task was unfinished, time-pressured or emotionally engaging.

**What this means for Drift:** This is the honest one-line answer to "why does switching cost anything". It also predicts that leaving work unfinished costs more than finishing it, which is a future feature, not an MVP feature.

---

## 5. Four hours of deep work per day

**Verdict: Partial. The real number is 3.5 hours, and it comes from violinists.**

**Sources:** Ericsson, K. A., Krampe, R. T., Tesch-Römer, C. (1993). The Role of Deliberate Practice in the Acquisition of Expert Performance. *Psychological Review* 100(3), 363-406. Read here through the 2019 replication paper that restates its design and figures:
[Macnamara & Maitra (2019), Royal Society Open Science](https://royalsocietypublishing.org/rsos/article/6/8/190327/68523/The-role-of-deliberate-practice-in-expert) - [PMC full text](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6731745/) - [Free PDF](https://hhs.purdue.edu/skill-learning-and-performance-lab/wp-content/uploads/sites/43/2024/08/macnamara-maitra-2019-the-role-of-deliberate-practice-in-expert-performance-revisiting-ericsson-krampe-tesch-romer.pdf)

Ericsson, Krampe and Tesch-Römer (1993) studied three groups of 10 violin students at the Music Academy of West Berlin, grouped by skill level. The two best groups practiced alone about 3.5 hours per day. The music-teacher group practiced 1.3 hours per day. The best group had accumulated over 10,000 hours by age 20.

Cal Newport's *Deep Work* (2016) generalized this into a roughly four-hour daily ceiling for cognitively demanding work. *Deep Work* is a trade book, not a study.

**What this means for Drift:** 4 hours is a reasonable default. It is not a measured constant of software work. Make the budget editable, and label it in the interface as a setting, not as a fact.

---

## 6. "It takes 15 to 20 minutes to get into flow"

**Verdict: Unsupported. Treat it as folklore.**

**Sources:** none found. That is the finding. For the flow research this claim is usually attached to, see Csikszentmihalyi's work as summarized in [Investigating the "Flow" Experience: Key Conceptual and Operational Issues](https://pmc.ncbi.nlm.nih.gov/articles/PMC7033418/), which describes the conditions for flow and gives no warm-up time.

Searching for a primary source returns blog posts and productivity sites that cite each other. The estimates range from 10 to 30 minutes with no consistent origin. Csikszentmihalyi's flow research describes the conditions for flow. It does not establish a warm-up constant.

**What this means for Drift:** Drift keeps a warm-up setting anyway, and this is a deliberate
choice rather than an oversight. The absence of a study is not evidence that ramp-in time does not
exist. It only means nobody has measured it, so Drift cannot cite a number and must not present one
as fact.

The rules that follow from that:

- Warm-up is labelled in the interface as your working assumption, not as a research finding.
- Warm-up is a toggle. Switch it off and a block counts from its first minute, so you can compare
  your own numbers both ways.
- The default value comes from claim 3, which is measured, rather than from the folklore figure.

---

## 7. Focus as a battery that drains

**Verdict: Unsupported. Do not build the model on it.**

**Source:** Hagger, M. S., Chatzisarantis, N. L. D., et al. (2016). A Multilab Preregistered Replication of the Ego-Depletion Effect. *Perspectives on Psychological Science* 11(4), 546-573.
[PubMed](https://pubmed.ncbi.nlm.nih.gov/27474142/) - [Free PDF](https://statmodeling.stat.columbia.edu/wp-content/uploads/2017/11/Hagger_0-407863.pdf) - [Commentary, PMC](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4971805/)

Ego depletion is the theory that willpower is a limited resource that a day of self-control uses up. Hagger and Chatzisarantis coordinated a preregistered replication across 23 labs with 2,141 participants (published 2016). The effect was close to zero and not significantly different from zero. The earlier meta-analysis had reported d = 0.62.

Time-on-task decline in sustained attention is a separate and better-established effect. It does not give a unit of "focus energy" that you can subtract per switch.

**What this means for Drift:** Do not draw a battery. Do not say "depleted". Drift tracks a budget the user sets and spends it against observed minutes. Every coefficient stays visible and editable, so the app never claims to measure the user's brain.

---

## Also referenced

**Mark, G., González, V., Harris, J. (2005). No Task Left Behind? Examining the Nature of Fragmented Work. CHI 2005.**
[Free PDF](https://ics.uci.edu/~gmark/CHI2005.pdf) - [ACM](https://dl.acm.org/doi/10.1145/1054972.1055017)

Shadowed managers, financial analysts and software developers at an outsourcing company. People
averaged about 3 minutes on a task and about 12 minutes in a "working sphere" before switching.
Removing interruptions shorter than 2 minutes as non-significant still left 12 minutes 18 seconds
per working sphere. Drift takes its 2-minute companion grace period from that cutoff.

**Mark, G. (2023). *Attention Span*. Hanover Square Press.**
[Author page](https://gloriamark.com/attention-span/) - [APA interview](https://www.apa.org/news/podcasts/speaking-of-psychology/attention-spans)

Reports average attention on a screen falling from 2.5 minutes in 2004, to 75 seconds in 2012, to
47 seconds in recent measurements, with a median of 40 seconds. Trade book, not a paper.

---

## Summary table

| Claim | Verdict | Use in Drift |
|---|---|---|
| 23 minutes to return | Unsupported | Do not use |
| Interruption raises stress, not duration | Solid | Frames what the minute count cannot show |
| 10 to 16 minute resumption phase | Solid | Default re-entry cost |
| Attention residue | Solid | The stated mechanism |
| 4 hours of deep work | Partial | Editable default, labelled as a setting |
| 15 minutes to reach flow | Unsupported | Rename to block threshold |
| Focus battery / ego depletion | Unsupported | Rejected. Use a budget instead |

---

## Sources

- Mark, G., Gudith, D., Klocke, U. (2008). [The Cost of Interrupted Work: More Speed and Stress](https://www.ics.uci.edu/~gmark/chi08-mark.pdf). CHI 2008.
- Iqbal, S. T., Horvitz, E. (2007). [Disruption and Recovery of Computing Tasks: Field Study, Analysis, and Directions](http://erichorvitz.com/CHI_2007_Iqbal_Horvitz.pdf). CHI 2007.
- Mark, G., González, V., Harris, J. (2005). [No Task Left Behind? Examining the Nature of Fragmented Work](https://ics.uci.edu/~gmark/CHI2005.pdf). CHI 2005.
- Leroy, S. (2009). [Why is it so hard to do my work? The challenge of attention residue when switching between work tasks](https://ideas.repec.org/a/eee/jobhdp/v109y2009i2p168-181.html). Organizational Behavior and Human Decision Processes 109(2), 168-181.
- Hagger, M. S., et al. (2016). [A Multilab Preregistered Replication of the Ego-Depletion Effect](https://pubmed.ncbi.nlm.nih.gov/27474142/). Perspectives on Psychological Science.
- Macnamara, B., Maitra, M. (2019). [The role of deliberate practice in expert performance: revisiting Ericsson, Krampe & Tesch-Römer (1993)](https://royalsocietypublishing.org/rsos/article/6/8/190327/68523/The-role-of-deliberate-practice-in-expert). Royal Society Open Science.
- [Interruptions cost 23 minutes 15 seconds, right?](https://blog.oberien.de/2023/11/05/23-minutes-15-seconds.html) - the source check that traced the 23-minute figure through five papers.
- Mark, G. (2023). *Attention Span*. Reports average attention on a screen falling from 2.5 minutes in 2004 to 47 seconds, with a median of 40 seconds.

## Method note

The 2007 and 2008 papers were read directly as PDFs, and the quoted passages come from those files. The Leroy, Hagger and Ericsson findings come from published abstracts and secondary summaries, not from the full texts, because those are behind paywalls. Verify them against the full papers before you rely on an exact number.
