Planning · MDCopyTakeMeter — Planning

Community

I chose r/soccer, collected during the 2026 World Cup. I picked it because match threads, post-match threads, and tactical posts all run at once, so the same comment section contains pure emotional venting, low-effort hot takes, and genuine tactical breakdowns sitting side by side. That spread is exactly what makes the discourse-quality distinction real here: a regular reader instantly knows the difference between someone yelling "ROBBED" and someone explaining why a team's high line got punished, and that recognition is what I'm trying to teach a model. World Cup timing also guarantees volume — any active match thread produces hundreds of comments per hour.

Labels

I'm measuring discourse quality along a claim-and-support axis: does a comment just feel something, assert something, or argue something?

reaction

A comment that expresses emotion or an in-the-moment response without making a substantive claim about the game.


"NOOOO not like this again 😭😭"
"I have never been so stressed in my life lmaooo"


hot_take

A comment that makes a debatable claim or judgment but offers little or no reasoning to back it up.


"Mbappé is honestly overrated, France play better without him."
"This is the worst refereeing performance of the entire tournament, embarrassing."


analysis

A comment that makes a claim and supports it with reasoning, evidence, or specific observation about play.


"Brazil's midfield got overrun because they pushed both 8s forward and left Casemiro isolated — every transition came straight through that gap."
"The penalty was soft but the bigger issue was the defending before it: the fullback stepped up with no cover, so the cross was always going to find space at the back post."


Hard Edge Cases

The genuinely ambiguous boundary is hot_take vs. analysis: a comment that makes a strong claim and gestures at a reason, but the reason is thin or unsupported ("Mbappé is overrated, he disappears in big games"). It has the shape of analysis — claim plus justification — but the justification is just another assertion, not evidence or specific observation.

My rule: it's analysis only if the reasoning points to something specific and checkable about the play (a tactical pattern, a sequence, a stat, a named situation). A vague causal claim ("he's just not clutch," "they always bottle it") stays hot_take. I logged every borderline call in the notes column with the decision and a one-line rationale so the boundary stays consistent across all 200.

The secondary edge case is reaction vs. hot_take: emotional comments that smuggle in a claim ("absolute DISGRACE of a defense"). Rule: if you can extract a debatable proposition, it's hot_take; if removing the emotion leaves nothing assertable, it's reaction.

During annotation, two recurring patterns turned out to be the real test of these rules. First, tactics-flavored verdicts that name a problem but don't explain it ("Brazil need a midfield, their transitioning was so poor") — these read like analysis but stay hot_take, because naming a weakness is not the same as showing the mechanism. Second, comments that marshal real data for a banter jab rather than an argument (a list of teams beaten used to mock a rival) — these stay hot_take/reaction by intent, since the data isn't doing analytical work.

Data Collection Plan


Source: public r/soccer comments from 2026 World Cup match threads, post-match threads, and tactical/discussion threads. Public content only, copy-pasted manually so I stayed close to the data.
Target: 200 examples, roughly balanced across the three labels.
What I actually collected: 200 comments across four threads — a daily World Cup discussion thread, an anthems/banter thread, and the Brazil–Morocco and France–Senegal post-match threads. The reaction-heavy banter threads and the analysis-heavy post-match threads balanced each other out.
Handling underrepresentation: as predicted, analysis was the scarcest class and live/banter threads barely produced it, so I deliberately pulled the post-match threads to bring it up. Final balance: hot_take 70 (35%), analysis 66 (33%), reaction 64 (32%) — no class below 25% or above 70%.


Evaluation Metrics

Accuracy alone is misleading here because the classes aren't perfectly balanced and the errors aren't equally costly. I'll use:


Per-class precision, recall, and F1 — the whole point is whether the model can find analysis (the rare, valuable class), so recall on analysis matters more than overall accuracy.
Macro-averaged F1 — weights each class equally, so a model that nails reaction but can't tell hot_take from analysis won't look good just because reactions are common.
Confusion matrix — to test my hypothesis that the model's mistakes concentrate on the hot_take↔analysis boundary rather than being spread randomly. The shape of the errors is the actual deliverable for the failure-analysis section.


Definition of Success

For a real community tool (e.g., surfacing high-quality takes or filtering noise), I'd want:


Macro-F1 ≥ 0.70, and specifically
analysis recall ≥ 0.65 — if it misses most of the substantive comments, it's useless for surfacing them.
reaction precision ≥ 0.85 — false-flagging good comments as noise is the costly error, so I want to be confident when it calls something a reaction.


"Good enough to deploy" = it reliably separates the bottom tier (reaction) from the top tier (analysis), even if it stays fuzzy on the middle boundary, because that middle fuzziness mirrors genuine human disagreement.

AI Tool Plan

Label stress-testing. Before finalizing, I used my three definitions and the hot_take↔analysis edge case to surface boundary comments and confirm I could classify them cleanly; where I couldn't, I tightened the rule (the "specific and checkable about the play" test above is the result).

Annotation assistance. I used an LLM to pre-label the collected comments against my exact definitions, then reviewed and corrected every row myself. Pre-labeled rows are marked in the notes column so the AI usage is auditable. Skimming defeats the purpose, so I read every comment.

Failure analysis. After the baseline and fine-tuned runs, I'll hand the AI my list of wrong predictions (text + true label + predicted label) and ask it to spot patterns — e.g., whether errors cluster on short comments, sarcasm, or the thin-reasoning boundary. I'll verify each proposed pattern by hand against the actual rows before writing it up, since the model can hallucinate a tidy story.

AI Usage Disclosure


Tool: Claude, used as an annotation assistant and writing aid.
What it did: helped resolve borderline cases by applying the "specific and checkable about the play" rule consistently.
What I did: collected all 200 comments myself from public r/soccer threads, reviewed every one. The comments are real public posts, not generated.
