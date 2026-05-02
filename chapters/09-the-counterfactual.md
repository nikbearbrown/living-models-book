# Chapter 9 — The Counterfactual

*Pearl's abduction-action-prediction procedure, and why the individual-level counterfactual is the hardest and most valuable form of causal reasoning.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *The Counterfactual*
- *Abduction, Action, Prediction*
- *Worlds That Never Happened*

**TL;DR.** Counterfactuals are statements about worlds that did not occur — *if I had done X, Y would have happened*. They cannot be verified by experiment, since the world that happened is the only one we have. But they are mathematically tractable in a structural causal model, via the abduction-action-prediction procedure. Counterfactuals are the upper rung of the Ladder of Causation, the level at which regret, attribution, blame, credit, and personalized intervention live. The Living Model architecture in Part Three depends on them.

---

## The Decision That Was Not Taken

A board chair sits in her office, six months after a decision that did not go well. The company chose, on her recommendation, to expand into a new geography. Revenue from the new market has come in at a third of the projection. Two senior leaders have left. The board is asking, quite reasonably, what would have happened if the company had not expanded — if it had instead invested the same resources in deepening its existing market.

The board chair would like to answer the question. So would her CEO, her CFO, her board members, and the analysts on Wall Street. None of them can. The world in which the company did not expand does not exist. There is no data on it. There is no possibility of running the experiment. There is only the world that happened, with its disappointing revenue and its departing leaders, and the world that did not happen, about which we have only intuitions.

This is a counterfactual question. It is the kind of question every executive faces every quarter, every time a strategic decision turns out to be other than expected. It is also the kind of question every patient faces when a treatment fails, every plaintiff in a wrongful-death lawsuit, every parent looking back at a difficult conversation with a teenager. *What would have happened if?*

The question seems unanswerable. The world that did not happen is, by definition, unobserved. And yet, humans answer counterfactual questions all the time, with surprising consistency. We make judgments about what would have been, we hold ourselves and others accountable to those judgments, we use them as the basis of regret, attribution, and learning. The question for this chapter is whether such judgments can be made rigorous — whether we can compute counterfactuals from data and a model — and the answer, due primarily to Judea Pearl's work, is yes.

---

## Why the Lower Rungs Cannot Reach Counterfactuals

The Ladder of Causation, as we saw in Chapter 5, is sealed in one direction. Rung 1 cannot answer Rung 2 questions; Rungs 1 and 2 together cannot answer Rung 3 questions. The reason has to do with the kind of evidence each rung admits.

Rung 1 admits only observational data — what we observed about the world. Rung 2 admits interventional data — what happens when we manipulate variables. Rung 3 admits *counterfactual* data — what would have happened in a world that did not occur. We cannot collect counterfactual data directly, because the worlds in question did not happen. The only way to get to Rung 3 is to combine the data we have with a model rich enough to predict what would have happened in the worlds we did not see.

The model, in this case, is the structural causal model (SCM) we met in Chapter 6. The SCM is more than a diagram; it is a diagram plus the functions that determine each variable from its parents and an exogenous noise term. The functions encode the *mechanism* by which the system works. Given the mechanism, we can ask: in this specific case, what would the outcome have been if the input had been different? That is a counterfactual question. The SCM gives us a procedure for answering it.

The procedure is called *abduction-action-prediction*, after the three steps it involves. It is short, mechanical, and one of the most beautiful results in causal inference.

---

## The Three Steps

Suppose we have a structural causal model M with variables X, Y, and other variables. We have observed a specific case: X took value x, Y took value y, the other variables took specific values. The counterfactual question is: what would Y have been in this case if X had been x′ instead?

**Step 1: Abduction.** Use the observed data to update our knowledge of the exogenous noise terms. The SCM says that each variable is determined by its parents and a noise term U. We have observed the variables; we can therefore infer (in many cases) what the noise terms must have been to produce the observed values. This step recovers the case-specific information that distinguishes this case from others.

**Step 2: Action.** Modify the SCM to enforce the counterfactual condition. Replace the equation for X with the assignment X = x′. The other equations remain. This step builds the counterfactual world's model, which differs from the actual world's model only in the value of X.

**Step 3: Prediction.** Using the modified SCM and the noise terms recovered in Step 1, compute the value of Y. This is the counterfactual outcome — what Y would have been in this case if X had been x′ instead of x.

The procedure is mechanical. Given an SCM and an observation, it produces a counterfactual prediction. The prediction is not a fact; it is conditional on the SCM being correct. But the SCM is testable in the ways we discussed in Chapter 6 — its structural assumptions imply specific patterns in the observational data — and the counterfactual it produces inherits the SCM's empirical credibility.

---

## A Worked Example

Pearl's classic worked example is the firing squad, which I will not repeat here. Let me instead work through a business example that maps onto the kind of counterfactual the board chair is asking.

Suppose we have a simple model: a marketing campaign (M) drives leads (L), which drive revenue (R). The SCM is:

> M = U<sub>M</sub>
> L = α M + U<sub>L</sub>
> R = β L + U<sub>R</sub>

We observe a specific case: a campaign was run with M = 100 (units of investment), it generated L = 50 leads, and revenue came in at R = 200. We have estimated the parameters from past data: α = 0.4, β = 5.

The counterfactual question: *what would revenue have been if we had run the campaign at M = 200 instead of M = 100?*

**Step 1: Abduction.** The actual data: M = 100, L = 50, R = 200. Plugging into the equations:

> 50 = 0.4 × 100 + U<sub>L</sub>, so U<sub>L</sub> = 10
> 200 = 5 × 50 + U<sub>R</sub>, so U<sub>R</sub> = −50

The noise terms in this specific case were U<sub>L</sub> = 10 (we got 10 more leads than the average campaign at this spend produces) and U<sub>R</sub> = −50 (we got $50 less revenue per lead than the average lead produces).

**Step 2: Action.** Modify the SCM by setting M = 200. The other equations stay:

> M = 200
> L = 0.4 M + U<sub>L</sub>
> R = 5 L + U<sub>R</sub>

**Step 3: Prediction.** Using the noise terms from Step 1:

> L<sub>counterfactual</sub> = 0.4 × 200 + 10 = 90
> R<sub>counterfactual</sub> = 5 × 90 + (−50) = 400

So in this specific case, with this specific campaign's idiosyncrasies, doubling the investment from 100 to 200 would have doubled revenue from 200 to 400. The counterfactual answer is $400 in revenue, given the case-specific noise.

Notice what abduction did. In the average case, doubling investment from 100 to 200 should produce 80 leads (0.4 × 200) and $400 in revenue (5 × 80). In this specific case, abduction told us that we got 10 extra leads and $50 less per lead. The counterfactual carries those case-specific quirks forward into the new investment level. The result is 90 leads (10 more than average) and $400 in revenue (90 × 5 − 50). The case-specific noise survives the counterfactual because the counterfactual is asking about *this* case, not about the average.

This is the key feature of counterfactuals. They are not predictions about averages; they are predictions about specific cases, with all the idiosyncrasies of the case preserved. The board chair's question is exactly this kind of question: *given the specific situation our company was in, what would have happened if we had made a different decision?* The counterfactual procedure carries forward everything that was specific to the company's situation — the team, the market conditions, the competitive landscape — and asks what the outcome would have been under a different action.

---

## Pre-Factual vs. Retrospective Counterfactuals

Counterfactuals come in two flavors, distinguished by when they are evaluated.

**Pre-factual counterfactuals** are evaluated before an action is taken. They ask: *if I were to take this action, what would happen?* This is, in essence, a Rung 2 question — an interventional question — being answered with the apparatus of Rung 3. The reason to use Rung 3 machinery instead of Rung 2 is that pre-factual counterfactuals can be conditioned on the specific case at hand, whereas Rung 2 estimates are typically averaged over a population. A doctor evaluating whether to give a particular drug to a particular patient is asking a pre-factual counterfactual: *given everything I know about this patient, what would happen if I prescribed this drug?*

**Retrospective counterfactuals** are evaluated after an action has been taken. They ask: *given that I took this action and observed this outcome, what would have happened if I had taken a different action?* This is the canonical counterfactual question. The board chair's question is retrospective. Most legal questions of causation are retrospective. Most regret is retrospective.

The two flavors require the same machinery. Both use abduction-action-prediction. The difference is what we are conditioning on. Pre-factual counterfactuals condition on the patient's pre-treatment characteristics. Retrospective counterfactuals condition on the actual outcome as well. The retrospective version is harder because it requires us to update our model of the noise terms based on the actual observation, whereas the pre-factual version uses prior distributions over the noise.

In practice, the retrospective form is what most decision-makers want when they ask "what would have happened if?" It is the form that supports learning from experience. It is also, on the second front, the form that supports legal arguments about causation, attribution of responsibility, and assessment of necessity (the "but-for" test in tort law).

---

## The Individual-Level Counterfactual

The counterfactual question can be asked at three levels of granularity, and they get progressively harder.

**Population-level counterfactual.** *On average, what would have happened if we had not run the campaign?* This is closely related to the average causal effect we estimated in Chapter 8. It uses the population distribution of noise terms rather than case-specific noise.

**Subgroup counterfactual.** *Among customers with these characteristics, what would have happened if we had not run the campaign?* This is the conditional average treatment effect, accessible via causal forests.

**Individual-level counterfactual.** *For this specific case, what would have happened if we had not run the campaign?* This requires abduction — recovering the case-specific noise terms from the observed data — and is the form Pearl's three-step procedure addresses directly.

The individual-level counterfactual is the hardest. It is also the most valuable. Decisions are made about specific cases — this patient, this customer, this price negotiation — and the case-specific reasoning is what supports the decision. Population averages are useful as background; they do not address the question the decision-maker is actually asking.

There is, however, a caveat. The individual-level counterfactual is identifiable only when the SCM is fully specified and the abduction step uniquely determines the noise terms. If the SCM has nonlinear relationships, or the noise terms are not separately identifiable from the observations, the individual-level counterfactual may be only partially identified — bounded above and below, but not pinned down to a specific value. This is, again, the discipline of causal inference: we report what we can identify, we report bounds when we cannot, and we are honest about the dependence on model assumptions.

---

## Counterfactuals in Clinical Reasoning

The most familiar setting for counterfactual reasoning is clinical medicine. A patient presents with symptoms. The doctor decides on a treatment. The patient improves, or does not. The natural questions are: *did the treatment cause the improvement? would the patient have improved anyway? would a different treatment have worked better?* These are all counterfactual questions, and they are part of how clinical reasoning works.

The structural causal model approach formalizes the clinician's reasoning. Given a model of the disease — what causes the symptoms, how the treatment affects the mechanism, what other factors moderate the response — the clinician can reason explicitly about counterfactuals. Pre-factually, before deciding on treatment: *what is the probability of recovery under treatment A vs. treatment B for this patient?* Retrospectively, after the outcome: *given that the patient recovered under treatment A, would she have recovered without treatment? would she have recovered faster with treatment B?*

The clinical literature has been quietly using counterfactual reasoning for decades, often without naming it as such. The recent shift toward causal inference frameworks — driven in part by work like that of Miguel Hernán and Jamie Robbins at Harvard — has made the counterfactual structure explicit and given it computational teeth. The result has been substantial improvements in observational studies of treatment effects, particularly in long-term and chronic conditions where randomized trials are impractical.

The Living Model architecture inherits this discipline from clinical reasoning. A Living Model that produces a recommendation should be able to support the recommendation with counterfactual reasoning: *here is what we expect under this intervention; here is what we expect under the alternative; here is the case-specific evidence that distinguishes this situation from the average.* The board chair, the CFO, the patient's family — all want this kind of reasoning, and the SCM apparatus is what produces it.

---

## The Necessity and Sufficiency of Causes

Pearl's framework distinguishes between two kinds of causation, both of which can be expressed as counterfactual queries.

**Necessary cause.** A is a necessary cause of B if, *had A not occurred, B would not have occurred*. The probability of necessity (PN) is the probability that B would not have occurred had A not occurred, given that A and B both did occur. This is the "but-for" test in tort law — the standard most legal systems use to determine liability.

**Sufficient cause.** A is a sufficient cause of B if, *whenever A occurs, B also occurs*. The probability of sufficiency (PS) is the probability that B would occur had A occurred, given that A and B both did not occur in the actual world. This is the standard for predictive accountability — would this action, if taken, produce this outcome?

These two probabilities are different. A drug can be necessary for a patient's recovery (the patient would not have recovered without it) but not sufficient (taking the drug does not guarantee recovery). A flame can be necessary for a fire (no fire without ignition) but not sufficient (ignition without oxygen produces no fire).

The legal and ethical implications differ. Necessary cause is the standard for assigning blame for outcomes that did occur. Sufficient cause is the standard for assigning credit for outcomes that did not occur but might have. In policy contexts, both matter: necessary cause tells you what made the difference; sufficient cause tells you what would have made the difference.

The counterfactual machinery handles both cleanly. PN and PS are both functions of the SCM and the observed case, computable via the abduction-action-prediction procedure. We will see in Chapter 10 a substantive application — the climate change attribution problem — in which both PN and PS play a role.

---

## Why Counterfactuals Frighten Some Statisticians

The counterfactual is not universally beloved among statisticians. The principal complaint is that the counterfactual world cannot be observed, and a statement about an unobservable cannot, by some definitions of science, be scientific. Karl Popper would have wanted to know what observation could falsify a counterfactual claim, and the honest answer is: no direct observation can. We can only test the SCM that produces the counterfactual, and the test is on its observable implications.

The defense of counterfactual reasoning is the same as the defense of any unobservable construct in science. Atoms cannot be directly observed; they are inferred from observable patterns and the theories that explain them. Black holes cannot be directly observed in the strict sense; they are inferred from the behavior of the surrounding matter and the theory that explains the behavior. The counterfactual is the same kind of construct: not directly observable, but inferred from a model that is observably testable, and used to support inferences and decisions that no other apparatus supports as cleanly.

The pragmatic case is stronger still. Humans reason counterfactually all the time. The question is not whether to reason this way but whether to do it consistently and rigorously. The structural causal model approach makes counterfactual reasoning systematic; the alternative is to leave it implicit, intuitive, and uncheckable.

---

## Looking Forward

Chapter 10 takes up the limits of what observational data can do for us. The counterfactual machinery in this chapter assumes the SCM is correct — that the structure of the diagram is right and the functional forms are right. In practice, models are always imperfect, and the most consequential imperfection is unmeasured confounding: a variable that affects both treatment and outcome but is not in our diagram. Chapter 10 takes up this problem honestly, including the methods for sensitivity analysis that quantify how robust our conclusions are to unmeasured confounding.

---

**What would change my mind:** A demonstration that some non-counterfactual framework can support the kinds of personalized decision-making that medicine, law, and management actually require. There are alternatives — Bayesian decision theory without explicit counterfactuals, for example. None of them, in my reading, handle the case-specific reasoning that abduction supports. I would update if shown a serious alternative.

**Still puzzling:** How to elicit the kind of structural model needed for counterfactual reasoning from domain experts who are not comfortable with formal models. Pearl's procedure works on the SCM. But getting the SCM out of an expert's head — particularly the functional forms — is hard. Chapter 16 in Part Three takes up the architecture of expert elicitation, and I expect that work to remain a frontier for years.

---

**Tags:** counterfactual reasoning, abduction action prediction Pearl, individual level counterfactual, necessary and sufficient causes, structural causal model
