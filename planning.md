TakeMeter — Planning

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

My rule when I hit one during annotation: it's analysis only if the reasoning points to something specific and checkable about the play (a tactical pattern, a sequence, a stat, a named situation). A vague causal claim ("he's just not clutch," "they always bottle it") stays hot_take. I'll log every one of these in the notes column with what I decided and why, so the boundary stays consistent across all 200.

The secondary edge case is reaction vs. hot_take: emotional comments that smuggle in a claim ("absolute DISGRACE of a defense"). Rule: if you can extract a debatable proposition, it's hot_take; if removing the emotion leaves nothing assertable, it's reaction.

Data Collection Plan


Source: public r/soccer comments from 2026 World Cup match threads, post-match threads, and a few tactical/discussion posts. Public content only, copy-pasted manually into a spreadsheet so I stay close to the data.
Target: ~200+ examples, aiming for a rough balance of about 65–70 per label. reaction will be the easiest to over-collect (match threads are full of it), so I'll deliberately hunt post-match and tactical threads for analysis, which is the scarcest.
If a label is underrepresented after 200: go back to the threads most likely to contain it — post-match and dedicated tactical posts for analysis, heated live threads for hot_take — and collect more until no label is below ~25% of the set. I will not let any single label exceed 70%.


Evaluation Metrics

Accuracy alone is misleading here because the classes won't be perfectly balanced and the errors aren't equally costly. I'll use:


Per-class precision, recall, and F1 — the whole point is whether the model can find analysis (the rare, valuable class), so recall on analysis matters more than overall accuracy.
Macro-averaged F1 — weights each class equally, so a model that nails reaction but can't tell hot_take from analysis won't look good just because reactions are common.
Confusion matrix — to confirm my hypothesis that the model's mistakes concentrate on the hot_take↔analysis boundary rather than being spread randomly. The shape of the errors is the actual deliverable for the failure-analysis section.


Definition of Success

For a real community tool (e.g., surfacing high-quality takes or filtering noise), I'd want:


Macro-F1 ≥ 0.70, and specifically
analysis recall ≥ 0.65 — if it misses most of the substantive comments, it's useless for surfacing them.
reaction precision ≥ 0.85 — false-flagging good comments as noise is the costly error, so I want to be confident when it calls something a reaction.


"Good enough to deploy" = it reliably separates the bottom tier (reaction) from the top tier (analysis), even if it stays fuzzy on the middle boundary, because that middle fuzziness mirrors genuine human disagreement.

AI Tool Plan

Label stress-testing. Before annotating, I'll give an LLM my three definitions and the hot_take↔analysis edge case and ask for 8–10 comments that deliberately sit on that boundary. If I can't cleanly classify what it generates, my definitions are too loose and I'll tighten them first.

Annotation assistance. I may pre-label a batch with an LLM using my exact definitions from this doc, then review and correct every single one myself — pre-labeled rows will be marked in the notes column (e.g., prelabeled:corrected or prelabeled:kept) so the AI usage is fully disclosed and auditable. Skimming defeats the purpose, so I read every post regardless.

Failure analysis. After the baseline and fine-tuned runs, I'll hand the AI my list of wrong predictions (text + true label + predicted label) and ask it to spot patterns — e.g., whether errors cluster on short comments, sarcasm, or the thin-reasoning boundary. I'll then verify each proposed pattern by hand against the actual rows before writing it up, since the model can hallucinate a tidy story.