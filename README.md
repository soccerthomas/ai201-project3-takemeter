# ai201-project3-takemeter
Fine-tuned discourse quality classifier for AI201 Project 3
TakeMeter — A Discourse-Quality Classifier for r/soccer

TakeMeter classifies football-forum comments by discourse quality: whether a comment is just an emotional reaction, an unsupported hot take, or a reasoned piece of analysis. It compares a zero-shot LLM baseline (Groq, Llama 3.1 8B) against a fine-tuned DistilBERT model, trained and evaluated on 200 hand-reviewed comments collected during the 2026 World Cup.

Community Choice and Reasoning

I chose r/soccer, collected during the 2026 World Cup. The same comment sections run pure emotional venting, low-effort hot takes, and genuine tactical breakdowns side by side, which is exactly the variance a discourse-quality classifier needs. A regular reader instantly knows the difference between someone yelling "ROBBED" and someone explaining why a high line got punished — that recognition is what I wanted to teach a model. World Cup timing also guaranteed volume: any active match thread produces hundreds of comments per hour.

Label Taxonomy

I measure discourse quality on a claim-and-support axis: does a comment just feel something, assert something, or argue something?

LabelDefinitionExamplesreactionExpresses emotion or an in-the-moment response without a substantive claim about the game."NOOOO not like this again 😭😭" / "I have never been so stressed in my life lmaooo"hot_takeMakes a debatable claim or judgment but offers little or no reasoning."Mbappé is honestly overrated, France play better without him." / "Worst refereeing performance of the entire tournament."analysisMakes a claim and supports it with reasoning, evidence, or specific observation about play."Brazil's midfield got overrun because both 8s pushed forward and left Casemiro isolated — every transition came through that gap." / "The penalty was soft but the real issue was the fullback stepping up with no cover behind."

Boundary rule: a claim is analysis only if the reasoning points to something specific and checkable about the play (a tactical pattern, a sequence, a stat, a named situation). A vague causal claim ("he's not clutch", "they always bottle it") stays hot_take. Emotion with no extractable claim is reaction.

Data Collection


Source: public r/soccer comments from 2026 World Cup threads — a daily discussion thread, an anthems/banter thread, and the Brazil–Morocco and France–Senegal post-match threads. Public content only, collected manually.
Labeling process: I pre-labeled comments against the definitions above, then reviewed and corrected every row by hand. Borderline cases were resolved with the "specific and checkable about the play" rule and logged in a notes column with a one-line rationale for each decision. (See AI Usage for disclosure of pre-labeling assistance.)
Label distribution (200 total): hot_take 70 (35%), analysis 66 (33%), reaction 64 (32%). No class exceeds 70% or falls below 25%.


Three difficult-to-label examples and my decisions


"Brazil need a midfield. Their transitioning was so poor." → labeled hot_take, not analysis. It names a problem but never explains the mechanism. Naming a weakness is not the same as showing why it happened, so it fails the "specific and checkable" test.
"Notable teams England have beaten since Euro 2016: [list]. Scotland: Haiti." → labeled hot_take. It marshals real data, which looks analytical, but the intent is a banter jab, not an argument about play. Data used for a joke is not analysis.
"They have a striker who had 30 g/a last season and won POTS at a PL club..." → labeled analysis. A bare-looking comment, but the claim (Brazil already has a striker) is backed by a specific, checkable stat and achievement, so it clears the bar.


Fine-Tuning Approach


Base model: distilbert-base-uncased with a 3-class sequence-classification head.
Training setup: 70/15/15 split (140 train / 30 val / 30 test), stratified by label. Tokenized with the DistilBERT tokenizer.
Hyperparameter decision: the notebook defaults are 3 epochs, learning rate 2e-5, batch size 16. I ran with these defaults. In hindsight this was the key weakness: training loss stayed essentially flat across all three epochs (1.098 → 1.106 → 1.097), which is the signature of underfitting — the model never converged. With only 140 training examples, 3 epochs was too few for the model to learn a real decision boundary. A clear next step would be to raise num_train_epochs to ~10 to let the loss actually drop. (See Reflection.)


Baseline Description


Model: Groq llama-3.1-8b-instant, zero-shot.
Prompt: the model was given the three label definitions verbatim, the "specific and checkable about the play" boundary rule, one example per label, and an instruction to respond with only the label name (no punctuation or explanation) so the notebook's parser could read it cleanly.
Collection: every comment in the locked 30-example test set was classified by Groq; predictions were parsed against the label map and scored with the same metrics as the fine-tuned model. All 30 responses parsed successfully after the prompt was finalized.


Evaluation Report

Overall accuracy

ModelAccuracyZero-shot baseline (Groq Llama 3.1 8B)0.633Fine-tuned DistilBERT0.433

Fine-tuning regressed performance by 0.20 versus simply prompting an off-the-shelf LLM.

Per-class metrics

Zero-shot baseline (Groq):

LabelPrecisionRecallF1Supportreaction0.900.900.9010hot_take~0.40~0.40~0.4010analysis~0.57~0.57~0.5710macro avg~0.6230

Fine-tuned DistilBERT:

LabelPrecisionRecallF1Supportreaction0.000.000.0010hot_take0.410.700.5210analysis0.460.600.5210macro avg0.290.430.3530


Replace the ~ baseline values with the exact figures from outputs/evaluation_results.json when you finalize.



Confusion matrix — Fine-tuned model (rows = true, columns = predicted)

true ↓ / pred →reactionhot_takeanalysisreaction064hot_take073analysis046

The entire reaction column is zero: the fine-tuned model never predicted reaction once. Errors are not spread evenly — they concentrate in (a) the total collapse of the reaction class and (b) the hot_take↔analysis boundary I flagged as hard in planning.

Three wrong predictions, analyzed

1. reaction → hot_take — "bouaddi is 18. a child. he looks like zidane out there at times." (conf 0.34)
A piece of hyperbolic praise — pure reaction. The model saw "Zidane" and the player reference and treated football proper nouns as a signal of substance. The failure is lexical: the model learned to associate football vocabulary with the claim-bearing classes, but reaction comments are defined by the absence of a claim, not by topic. This is the single most common error type in the set.

2. analysis → hot_take — "He's the BO winner, that's why he's being played. Yesterday he was a ghost and should have been subbed way earlier..." (conf 0.35)
This is real analysis: it gives a specific selection critique with reasoning (who should have come off, when, why). The model demoted it to hot_take. The hot_take↔analysis boundary depends on whether reasoning is specific and checkable, and 140 examples wasn't enough for the model to learn that distinction — it can tell football-argument from joke roughly, but not supported-argument from unsupported-argument.

3. hot_take → analysis — "Notable teams England have beaten since Euro 2016: [long list]. Scotland: Haiti." (conf 0.34)
The reverse error. This is a banter jab that looks analytical because it lists real data and makes a comparison. I labeled it hot_take on intent (it's mockery, not an argument about play). The model can't read intent — it sees data + comparison and calls it analysis. This is the "topic signals one label, structure signals another" case, and it's genuinely hard even for a human annotator.

Is it a labeling or a data/prompt problem? Primarily data. The borderline-resolution pass kept my labels internally consistent, yet the model still failed — and crucially, every wrong prediction landed at 0.34–0.36 confidence, barely above the 0.33 random baseline for three classes. The model wasn't confidently wrong; it was near-guessing. That points squarely at underfitting from too little data and too few epochs, not annotation noise. The reaction collapse specifically is a signal problem: reaction comments are lexically diverse (jokes, questions, anecdotes share no keywords), so they are the hardest class to learn from few examples, while hot_take/analysis share argumentative vocabulary.

What would fix it: (1) far more than 200 examples; (2) more epochs so the training loss actually drops; (3) specifically more diverse reaction examples so the model learns reaction = no claim, independent of topic.

Sample classifications (fine-tuned model)

CommentPredictedConfidence"Atrocious. Didn't even try to gamble on Endrick when Vini gassed in last 20 min..."hot_take0.34"Notable teams England have beaten since Euro 2016: [list]. Scotland: Haiti."analysis0.34"Sad days for Brazil's NT when Morocco's midfield is overwhelmingly more talented and in control."analysis0.35"He played flute as a kid so James Corden asked him to do that as a celebration if he scored."hot_take0.35"Brazil need a midfield. Their transitioning was so poor."hot_take(correct)

A reasonable correct prediction: "Brazil need a midfield. Their transitioning was so poor." was correctly predicted hot_take. This is reasonable: the comment makes a clear judgment ("need a midfield") with only a one-line, non-specific reason ("transitioning was so poor") and no checkable detail — which is precisely the definition of a hot take. Even an underfit model picked up that short, verdict-shaped football claims tend to be hot takes.


Confidence values above are from the fine-tuned run; the uniformly low (~0.34) scores are themselves a finding — see the analysis above.



Reflection: What the Model Learned vs. What I Intended

I intended the model to learn discourse function: is the author feeling, asserting, or arguing? What it actually learned was a much shallower, mostly lexical boundary — "does this contain football substance vocabulary?" — and even that only weakly.

The clearest evidence is the reaction collapse. My definition treats reaction as a structural category (no extractable claim), which can attach to any topic — a joke about Kane, a complaint about cleaning posts, a question about a celebration. Because those comments share no common vocabulary, the model had nothing to latch onto and never learned the class at all, folding every reaction into the two classes that do share argumentative words. The model overfit to surface topic signal and missed the thing I actually cared about: the presence or absence of a supported claim.

The hot_take↔analysis errors tell the same story from the other side. My boundary hinges on quality of support (specific and checkable vs. asserted), which is a subtle, almost rhetorical distinction. The model never approached it — its near-random confidence shows it wasn't drawing that boundary at all. The gap between intended and learned behavior here is large, and it's a data-scale gap: 200 examples is enough to define a nuanced discourse axis but nowhere near enough to teach one to a fine-tuned transformer. The honest finding of this project is that, at this data scale, a well-prompted general LLM captured my intended distinction better than fine-tuning did.

Spec Reflection

One way the spec helped: the requirement to write definitions plus 2 clear examples and 1 uncertain example per label, before annotating, forced me to confront the hot_take↔analysis boundary up front. Pre-committing to the "specific and checkable about the play" rule meant I labeled all 200 consistently, which is exactly why I can now attribute model errors to data scale rather than to messy labels — the consistency check paid off in the failure analysis.

One way my implementation diverged: the spec frames fine-tuning as the main event and the baseline as a point of comparison. In practice the baseline outperformed the fine-tuned model, so my implementation's center of gravity shifted to explaining why fine-tuning regressed rather than celebrating it. I leaned into that as the actual result instead of forcing the fine-tune to look good — which I think is the spec's deeper intent ("honestly assess where it falls apart"), even if it diverges from the surface framing.

AI Usage


Pre-labeling: I used an LLM to review and correct every row 
Failure-pattern analysis: I pasted your wrong predictions into an LLM to surface error patterns, then re-read the examples yourself to confirm or discard them. 


Repository Structure

ai201-project3-takemeter/
├── README.md
├── planning.md
├── data/
│   └── takemeter_labeled.csv
└── outputs/
    ├── evaluation_results.json
    └── confusion_matrix.png