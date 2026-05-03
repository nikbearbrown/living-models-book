# Chapter 9 — The Counterfactual

*Pearl's abduction-action-prediction procedure, and why the individual-level counterfactual is the hardest and most valuable form of causal reasoning.*

---

A board chair sits in her office, six months after a decision that did not go well. The company chose, on her recommendation, to expand into a new geography. Revenue from the new market came in at a third of the projection. Two senior leaders have left. The board is asking, quite reasonably, what would have happened if the company had not expanded — if it had instead invested the same resources in deepening its existing market.

The board chair would like to answer the question. So would her CEO, her CFO, her board members, and the analysts on Wall Street. None of them can. The world in which the company did not expand does not exist. There is no data on it. There is no possibility of running the experiment. There is only the world that happened, with its disappointing revenue and its departing leaders, and the world that did not happen, about which we have only intuitions.

This is a counterfactual question. It is the kind of question every executive faces every quarter, every time a strategic decision turns out differently than expected. It is also the kind of question every patient faces when a treatment fails, every plaintiff in a wrongful-death lawsuit, every parent looking back at a difficult conversation with a teenager. *What would have happened if?*

The question seems unanswerable. The world that did not happen is, by definition, unobserved. And yet humans answer counterfactual questions all the time, with surprising consistency. We make judgments about what would have been, we hold ourselves and others accountable to those judgments, we use them as the basis of regret, attribution, and learning. The question for this chapter is whether such judgments can be made rigorous — whether we can compute counterfactuals from data and a model — and the answer, due primarily to Judea Pearl's work, is yes.

---

## Why the Lower Rungs Cannot Reach Counterfactuals

The Ladder of Causation, as we saw in Chapter 5, is sealed in one direction. Rung 1 cannot answer Rung 2 questions; Rungs 1 and 2 together cannot answer Rung 3 questions. The reason has to do with the kind of evidence each rung admits.

Rung 1 admits only observational data — what we saw happen in the world. Rung 2 admits interventional data — what happens when we manipulate variables. Rung 3 admits counterfactual data — what would have happened in a world that did not occur. We cannot collect counterfactual data directly, because the worlds in question did not happen. The only way to reach Rung 3 is to combine the data we have with a model rich enough to predict what would have happened in worlds we did not see.

<!-- → [DIAGRAM: Pearl's Ladder revisited from Chapter 1, now annotated for Chapter 9 — three rungs with their standard labels; Rung 3 highlighted; beside it, a two-column annotation: "What you need" (Rung 1: observations, Rung 2: intervention records, Rung 3: SCM + abduction) and "What you cannot get by staying below" (Rung 2: causal direction, Rung 3: case-specific noise); purpose is to show why the SCM is a strict requirement for counterfactuals, not an optional upgrade] -->

That model is the structural causal model we built in Chapter 6. The SCM is more than a diagram; it is a diagram plus the functions that determine each variable from its parents and an exogenous noise term. The functions encode the *mechanism* by which the system works. Given the mechanism, we can ask: in this specific case, with its specific circumstances, what would the outcome have been if the input had been different? That is a counterfactual question. The SCM gives us a procedure for answering it.

The procedure is called abduction-action-prediction, after its three steps. It is short, mechanical, and one of the most beautiful results in causal inference.

---

## The Three Steps

Suppose we have an SCM with variables $X$, $Y$, and others. We have observed a specific case: $X$ took value $x$, $Y$ took value $y$, and the other variables took specific values. The counterfactual question is: what would $Y$ have been in this case if $X$ had been $x'$ instead?

**Step 1: Abduction.** Use the observed data to infer the exogenous noise terms. The SCM says each variable is determined by its parents and a noise term $U$. We have observed the variables; we can therefore recover what the noise terms must have been in order to produce the observed values. This step extracts the case-specific information — the idiosyncrasies of this particular situation — that distinguishes this case from all others.

**Step 2: Action.** Modify the SCM to enforce the counterfactual condition. Replace the equation for $X$ with the assignment $X = x'$. The other equations remain unchanged. This builds the counterfactual world's model, which differs from the actual world's model only in the value of $X$.

**Step 3: Prediction.** Using the modified SCM and the noise terms recovered in Step 1, compute the value of $Y$. This is the counterfactual outcome: what $Y$ would have been in this specific case if $X$ had been $x'$ instead of $x$.

The procedure is mechanical. Given an SCM and an observation, it produces a counterfactual prediction. The prediction is not a fact — it is conditional on the SCM being correct. But the SCM is testable in the ways we covered in Chapter 6: its structural assumptions imply specific patterns in the observational data. The counterfactual inherits that empirical credibility.

---

## A Worked Example

Let me work through a business example that maps onto the kind of counterfactual the board chair is asking.

Suppose a simple model: a marketing campaign ($M$) drives leads ($L$), which drive revenue ($R$). The SCM is:

$$M = U_M$$
$$L = \alpha M + U_L$$
$$R = \beta L + U_R$$

We observe a specific case: a campaign run at $M = 100$ units of investment generated $L = 50$ leads and $R = 200$ in revenue. Estimated from past data: $\alpha = 0.4$, $\beta = 5$.

The counterfactual question: *what would revenue have been if we had run the campaign at $M = 200$ instead of $M = 100$?*

**Step 1: Abduction.** Plug the observations into the equations:

$$50 = 0.4 \times 100 + U_L \implies U_L = 10$$
$$200 = 5 \times 50 + U_R \implies U_R = -50$$

The noise terms for this specific case were $U_L = 10$ — we got 10 more leads than an average campaign at this spend produces — and $U_R = -50$ — we got \$50 less revenue per lead than the average lead produces. The campaign over-performed on volume and under-performed on conversion.

**Step 2: Action.** Set $M = 200$, keep the other equations:

$$M = 200$$
$$L = 0.4M + U_L$$
$$R = 5L + U_R$$

**Step 3: Prediction.** Using the case-specific noise terms:

$$L_{\text{cf}} = 0.4 \times 200 + 10 = 90$$
$$R_{\text{cf}} = 5 \times 90 + (-50) = 400$$

In this specific case, with this specific campaign's idiosyncrasies, doubling the investment from 100 to 200 would have doubled revenue from 200 to 400.

<!-- → [TABLE: Worked counterfactual — four columns: Step, Operation, Equation, Result; rows: (1) Abduction / recover noise / plug observations into SCM / U_L=10, U_R=−50; (2) Action / modify SCM / set M=200 / new system; (3) Prediction / run modified SCM with case-specific noise / compute L_cf and R_cf / L=90, R=400; a footer row showing the average-case prediction without abduction for contrast] -->

Notice what abduction did. In the average case, doubling investment from 100 to 200 should produce 80 leads and \$400 in revenue. In *this* case, abduction told us we got 10 extra leads and \$50 less per lead. The counterfactual carries those case-specific quirks forward into the new investment level — 90 leads (10 more than average), \$400 in revenue (90 × 5 − 50). The idiosyncrasies of the case survive into the counterfactual world, because the counterfactual is asking about *this* case, not about an average.

This is the defining feature of Rung 3. Counterfactuals are not predictions about populations; they are predictions about specific cases, with all the particularity of the case preserved. The board chair's question is exactly this kind: given the specific situation our company was in — this team, these market conditions, this competitive landscape — what would have happened under a different decision? The abduction step is what makes that question answerable.

---

## Pre-Factual vs. Retrospective Counterfactuals

Counterfactuals come in two flavors, distinguished by when they are evaluated.

**Pre-factual counterfactuals** are evaluated before an action is taken. They ask: if I were to take this action, what would happen? This resembles a Rung 2 question, but answered with Rung 3 machinery — conditioned on the specific case at hand rather than averaged over a population. A doctor evaluating whether to give a drug to a particular patient is asking a pre-factual counterfactual: given everything I know about this patient, what would happen if I prescribed this drug?

**Retrospective counterfactuals** are evaluated after an action has been taken. They ask: given that I took this action and observed this outcome, what would have happened if I had acted differently? This is the canonical counterfactual question. The board chair's question is retrospective. Most legal questions of causation are retrospective. Most regret is retrospective.

Both flavors use the same three steps. The difference is what we condition on. Pre-factual counterfactuals condition on the patient's pre-treatment characteristics, using prior distributions over the noise. Retrospective counterfactuals condition on the actual outcome as well, which is why abduction in the retrospective case can recover the noise terms more precisely — the observed outcome pins down the residual that the pre-factual case can only bound.

In practice, the retrospective form is what most decision-makers want when they ask "what would have happened if." It supports learning from experience. It also, on the second front, supports legal arguments about causation, attribution of responsibility, and the but-for test — whether the harm would have occurred in the absence of the defendant's action.

<!-- → [TABLE: Pre-factual vs. retrospective counterfactuals — rows: when evaluated, what is conditioned on, what abduction recovers, precision of noise recovery, primary use case, example from the chapter; columns: Pre-factual, Retrospective; purpose is to make the structural distinction concrete before the legal material that follows depends on it] -->

---

## The Individual-Level Counterfactual

The counterfactual question can be asked at three levels of granularity, each progressively harder.

At the **population level**: on average, what would have happened if we had not run the campaign? This is closely related to the average causal effect from Chapter 8. It uses the population distribution of noise terms rather than case-specific noise, and it is accessible via Rung 2 methods — it does not strictly require the full counterfactual apparatus.

At the **subgroup level**: among customers with these characteristics, what would have happened? This is the conditional average treatment effect, accessible via causal forests and similar methods. It is more informative than the population average but still not case-specific.

At the **individual level**: for this specific case, what would have happened? This requires abduction — recovering the case-specific noise terms from the observed data — and is the form Pearl's three-step procedure addresses directly. It is the hardest, and it is also the most valuable. Decisions are made about specific cases — this patient, this customer, this price negotiation — and case-specific reasoning is what supports the decision. Population averages are useful background; they do not answer the question the decision-maker is actually asking.

There is a caveat. The individual-level counterfactual is fully identifiable only when the SCM is completely specified and the abduction step uniquely recovers the noise terms. If the SCM has nonlinear relationships, or the noise terms are not separately identifiable from the observations, the individual-level counterfactual may be only partially identified — bounded above and below, but not pinned to a specific value. This is the discipline of causal inference: we report what we can identify, we report bounds when we cannot, and we are honest about the dependence on model assumptions.

<!-- → [DIAGRAM: Three-level counterfactual hierarchy — nested circles or stacked bars: outermost is Population (average causal effect, no abduction needed), middle is Subgroup (conditional average, partial conditioning), innermost is Individual (case-specific, full abduction); each layer labeled with the method that reaches it and what information it requires; caption: "Each inner ring requires strictly more from the model than the ring outside it"] -->

---

## Necessity and Sufficiency

Pearl's framework distinguishes between two kinds of causation, both expressible as counterfactual queries.

A is a **necessary cause** of B if, had A not occurred, B would not have occurred. The probability of necessity — $PN$ — is the probability that B would not have happened had A not happened, given that both did happen in the actual world. This is the but-for test in tort law: the standard most legal systems use to determine liability. Was the defendant's action necessary for the harm that occurred?

A is a **sufficient cause** of B if, whenever A occurs, B also occurs. The probability of sufficiency — $PS$ — is the probability that B would have occurred had A occurred, given that neither happened in the actual world. This is the standard for predictive accountability: would this action, if taken, produce this outcome?

The two probabilities are different, and the distinction matters. A drug can be necessary for a patient's recovery — the patient would not have recovered without it — but not sufficient, since taking the drug does not guarantee recovery. A flame can be necessary for a fire but not sufficient; ignition without oxygen produces no fire.

<!-- → [TABLE: Necessary vs. sufficient causation — rows: formal definition, counterfactual query, probability measure (PN vs PS), legal/ethical standard it supports, example from the chapter (drug recovery, warehouse fire), what a high value implies, what a low value implies; columns: Necessary Cause, Sufficient Cause; purpose is to make the PN/PS distinction scannable before Chapter 10's attribution application] -->

The legal and ethical implications differ accordingly. Necessary cause is the standard for assigning blame for outcomes that did occur. Sufficient cause is the standard for assigning credit for outcomes that did not occur but might have. In policy contexts, both matter: $PN$ tells you what made the difference in the past; $PS$ tells you what would make the difference in the future.

The counterfactual machinery handles both cleanly. $PN$ and $PS$ are functions of the SCM and the observed case, computable via abduction-action-prediction. We will see in Chapter 10 a substantive application — the climate change attribution problem — in which both play a role.

---

## Why Counterfactuals Frighten Some Statisticians

The counterfactual is not universally beloved. The principal complaint is that the counterfactual world cannot be observed, and a statement about an unobservable cannot, by some definitions of science, be scientific. The honest response to this objection is that the counterfactual is in exactly the same epistemic position as every other theoretical construct in science.

Atoms cannot be directly observed; they are inferred from observable patterns and the theories that explain them. Black holes cannot be seen directly; they are inferred from the behavior of surrounding matter and the theory of general relativity. The counterfactual is the same kind of construct: not directly observable, inferred from a model that is observably testable, and used to support inferences that no other apparatus supports as cleanly.

The pragmatic case is stronger still. Humans reason counterfactually all the time. The question is not whether to reason this way but whether to do it consistently and rigorously. The SCM approach makes counterfactual reasoning systematic. The alternative is to leave it implicit, intuitive, and uncheckable — which is not more scientific, only more opaque.

---

## The Living Model Connection

The counterfactual is not a theoretical luxury. It is the form of reasoning that consequential decisions actually require.

Consider what a decision-support system needs to do to be genuinely useful. It needs to say not just "here is what tends to happen" but "here is what we expect to happen in this specific situation, under this specific action, compared to this specific alternative." That is a counterfactual statement. It requires a model of mechanism, not just a correlation engine. It requires abduction — leveraging what we know about this specific case — not just population averages. And it requires the discipline to report uncertainty honestly when the model assumptions do not fully identify the counterfactual.

The Living Model architecture in Part Three is built on this requirement. A Living Model that produces a recommendation should be able to support that recommendation with counterfactual reasoning: here is what we expect under this intervention; here is what we expect under the alternative; here is the case-specific evidence that distinguishes this situation from the average. The board chair, the CFO, the patient's family — all want this kind of reasoning. The abduction-action-prediction procedure is what makes it computable.

---

## Looking Forward

Chapter 10 takes up the limits of what observational data can do for us. The counterfactual machinery in this chapter assumes the SCM is correct — that the structure of the diagram is right and the functional forms are right. In practice, models are always imperfect, and the most consequential imperfection is unmeasured confounding: a variable that affects both treatment and outcome but is not in the diagram. Chapter 10 takes up this problem honestly, including the sensitivity analysis methods that quantify how robust our conclusions are to the confounding we cannot see.

---

## Student Activities

**Problem 9.1 — Running the Procedure.** A retailer's SCM links advertising spend ($A$) to store visits ($V$) and purchases ($P$) as follows: $V = 0.3A + U_V$, $P = 0.6V + U_P$. In a specific week, $A = 50$, $V = 20$, $P = 15$. (a) Perform the abduction step: recover $U_V$ and $U_P$ for this specific week. (b) Perform the action step: write the modified SCM for the counterfactual in which advertising spend was doubled to $A = 100$. (c) Perform the prediction step: compute the counterfactual values of $V$ and $P$. (d) Compare the counterfactual prediction to what the model would predict at $A = 100$ *without* abduction — that is, using $U_V = U_P = 0$. Explain in one sentence what the difference between the two predictions represents.

**Problem 9.2 — Pre-factual vs. Retrospective.** A physician is considering two treatments for a patient with a chronic condition. Before prescribing, she asks: *what would happen to this patient under Treatment A vs. Treatment B?* After prescribing Treatment A and observing partial recovery, she asks: *would the patient have done better under Treatment B?* (a) Classify each question as pre-factual or retrospective. (b) Explain what the abduction step recovers in each case and why the retrospective version can recover it more precisely. (c) Describe one piece of observable information about the patient — not about the treatment — that would sharpen the abduction step in both cases.

**Problem 9.3 — Necessity vs. Sufficiency.** A warehouse fire is attributed to an electrical fault ($F$) in the presence of dry conditions ($D$). The fire ($B$) occurred. (a) Write the counterfactual queries corresponding to $PN(F, B)$ and $PS(F, B)$. (b) Explain in plain language what each probability captures and why they are different. (c) A plaintiff's attorney argues that the electrical fault was a necessary cause of the fire; the defendant's attorney argues it was not sufficient. Explain what each attorney must show, expressed as a counterfactual claim. (d) Identify one piece of evidence — physical or statistical — that would change your estimate of $PN$ and one that would change your estimate of $PS$.

**Problem 9.4 — Individual vs. Population.** A clinical trial estimates that a new drug increases five-year survival by 12 percentage points on average across a population. A specific patient has characteristics that place her in a subgroup where the average effect was 8 percentage points. The patient's individual-level counterfactual, computed via abduction, yields an estimated effect of 19 percentage points. (a) Explain why the three estimates differ and what each one means. (b) Which estimate is most relevant to the physician's prescription decision, and why? (c) Under what conditions would the individual-level estimate be identical to the subgroup estimate? Under what conditions would they diverge most sharply?

**Problem 9.5 — The Objection.** A statistician colleague argues: "Counterfactual claims cannot be falsified by any observation, because the counterfactual world never exists. Therefore they are not scientific." Write a structured response to this objection. Your response should (a) acknowledge the legitimate concern behind the objection, (b) explain the epistemic status of the SCM on which counterfactuals are based and how it is testable, (c) name two other constructs in science that have the same observability status as counterfactuals, and (d) make the pragmatic case for rigorous counterfactual reasoning over the alternative of leaving such reasoning implicit.

**Problem 9.6 — Design Challenge.** You are building a decision-support system for a retail operations team. The team makes weekly markdown decisions on slow-moving inventory. They want the system to support retrospective review: after each markdown decision, they want to ask "what would have happened if we had marked down more aggressively?" (a) Specify the minimum SCM structure you would need to support this counterfactual query, naming the variables and the causal relationships between them. (b) Describe the abduction step in concrete operational terms — what data would you collect, and how would it pin down the case-specific noise for a given week? (c) Identify one structural assumption in your SCM that is most likely to be wrong in practice, explain what would break if it were wrong, and describe one sensitivity analysis you would run to quantify the exposure.

---

## Key Terms

**Abduction.** The first step of Pearl's abduction-action-prediction procedure. Given an observed case, abduction uses the structural equations of the SCM to infer the values of the exogenous noise terms that produced the observed outcome. Abduction recovers the case-specific information that distinguishes this case from the population average.

**Action (in abduction-action-prediction).** The second step of the procedure. The SCM is modified by replacing the structural equation for the treatment variable with a fixed assignment $X = x'$. This modification represents the counterfactual intervention — it builds the model of the world in which a different action was taken.

**But-For Test.** The legal standard for necessary causation: the harm would not have occurred but for the defendant's action. Formally, the but-for test asks whether the probability of necessity $PN$ is sufficiently large. Courts applying this test are asking a counterfactual question.

**Counterfactual.** A statement about what would have happened in a world that did not occur. Counterfactuals are the third rung of Pearl's Ladder of Causation. They cannot be observed directly but are computable from a structural causal model via the abduction-action-prediction procedure.

**Exogenous Noise Term ($U$).** A variable in an SCM that captures all influences on a node that are not represented by other nodes in the diagram. Each node has its own noise term. The abduction step recovers the value of the noise term for a specific case from the observed data.

**Individual-Level Counterfactual.** A counterfactual prediction about a specific case, carrying forward the case-specific noise terms recovered by abduction. Distinguished from population-level and subgroup-level counterfactuals by its use of case-specific information. The hardest and most valuable form of counterfactual reasoning.

**Prediction (in abduction-action-prediction).** The third step of the procedure. The modified SCM — with the counterfactual action applied and the case-specific noise terms inserted — is used to compute the counterfactual outcome.

**Pre-factual Counterfactual.** A counterfactual question asked before an action is taken: what would happen if I were to do $X$? Conditioned on pre-action characteristics of the specific case. Closely related to Rung 2 interventional reasoning but conditioned on the individual rather than the population.

**Probability of Necessity ($PN$).** The probability that the outcome $Y$ would not have occurred had the treatment $X$ not occurred, given that both did occur in the actual world. Formalizes the but-for test. Requires a retrospective counterfactual.

**Probability of Sufficiency ($PS$).** The probability that the outcome $Y$ would have occurred had the treatment $X$ occurred, given that neither occurred in the actual world. Measures the predictive reliability of the cause. Requires a pre-factual counterfactual.

**Retrospective Counterfactual.** A counterfactual question asked after an action has been taken and an outcome has been observed: given that I did $X$ and observed $Y$, what would have happened if I had done $X'$? The canonical form of counterfactual reasoning. The abduction step uses the observed outcome to sharpen the recovery of case-specific noise.

**Structural Causal Model (SCM).** The mathematical object required for Rung 3 reasoning. An SCM is a DAG augmented with structural equations specifying the mechanism by which each variable is determined from its parents and an exogenous noise term. The abduction-action-prediction procedure runs on the SCM.

---

**What would change my mind:** A demonstration that some non-counterfactual framework can support the kinds of personalized decision-making that medicine, law, and management actually require. There are alternatives — Bayesian decision theory without explicit counterfactuals, for example. None of them, in my reading, handle the case-specific reasoning that abduction supports. I would update if shown a serious alternative.

**Still puzzling:** How to elicit the kind of structural model needed for counterfactual reasoning from domain experts who are not comfortable with formal models. Pearl's procedure works on the SCM. But getting the SCM out of an expert's head — particularly the functional forms — is hard. Chapter 16 in Part Three takes up the architecture of expert elicitation, and I expect that work to remain a frontier for years.

---

**Tags:** counterfactual reasoning, abduction action prediction Pearl, individual level counterfactual, necessary and sufficient causes, structural causal model

---

###  LLM Exercise — Chapter 9: The Counterfactual

**Project:** Build Your Own Living Model

**What you're building this chapter:** A counterfactual analysis of one historical decision in your domain — a real one your organization made, ideally one that turned out worse than expected — using Pearl's three-step abduction-action-prediction procedure on your SCM.

**Tool:** Claude Project for the reasoning. Optionally Claude Code if you want to compute the counterfactual numerically.

---

**The Prompt:**

```
Continuing my Living Model project. My DAG and structural equations are in the Project context.

This chapter teaches Pearl's three-step counterfactual procedure on a Structural Causal Model:

1. ABDUCTION — Given the observed outcome and the structural equations, infer the values of the exogenous noise terms (ε) that must have held for the observed outcome to have occurred. This locks in the case-specific facts.

2. ACTION — Modify the SCM by replacing the actual value of the intervention variable X with the counterfactual value X' (this is what the do-operator does). Hold the noise terms from abduction fixed.

3. PREDICTION — Run the modified SCM forward to compute what the outcome would have been under X'.

The result is the individual-level counterfactual: not "on average across the population, what happens at X'?" but "in this specific case, with this specific noise, what would have happened?"

Pick a HISTORICAL DECISION in my domain that I have enough information about to analyze. Ideally one that turned out worse than expected. Examples:
- A pricing change my company made that didn't move retention as predicted
- A campaign that underperformed
- A staffing decision that didn't deliver expected output
- A policy intervention with weaker than projected take-up

Walk me through the abduction-action-prediction procedure for this case:

ABDUCTION:
- What were the observed values of all variables in my DAG at the time of the decision?
- Given my structural equations, what exogenous noise terms (ε) must have held to produce the observed outcome? Be explicit — for each ε, name what real-world factors it represents (e.g., "ε_retention captures everything the model doesn't measure: macro economy, competitor moves, the specific account managers on those accounts that quarter").

ACTION:
- What is the counterfactual intervention X'? Be specific (e.g., "if we had raised price by 5% instead of 15%").
- Modify the SCM accordingly.

PREDICTION:
- With ε fixed from abduction and X replaced by X', what does the outcome become?
- Compare to the actual outcome: by how much would the decision have been better or worse under X'?
- Quantify the regret (or relief).

Then answer two synthesis questions:
- Was X a NECESSARY cause of the bad outcome? (Would the bad outcome have happened anyway under X'?)
- Was X a SUFFICIENT cause? (Was X enough on its own to produce the outcome, or did it require the specific noise that obtained?)

End with: a one-paragraph honest assessment — does this counterfactual hold up, or is the SCM too thin to support it? If too thin, what specific structural equation would need to be tightened for the counterfactual to be trustworthy?
```

---

**What this produces:** A worked individual-level counterfactual on a real historical decision, with explicit abduction (noise inference), action (do-substitution), and prediction (outcome under counterfactual). Plus a necessity/sufficiency analysis and a calibration check on your SCM.

**How to adapt this prompt:**
- *For your own project:* If you can't think of a historical decision, ask Claude to help you construct a plausible synthetic one anchored to your domain — labeled clearly as illustrative.
- *For ChatGPT / Gemini:* Works as-is.
- *For Claude Code:* Optional. If you want the counterfactual computed numerically against the Chapter 8 estimated SCM, ask Claude Code to add a `counterfactual()` function to estimate-effects.py.
- *For a Claude Project:* Recommended. The Project remembers your SCM from Chapter 6.

**Connection to previous chapters:** Chapter 6 gave you the SCM. Chapter 8 estimated some of its parameters. This chapter uses the SCM to answer the hardest class of question — what would have happened in a world that didn't occur — and reveals where your model is too thin to support strong counterfactuals.

**Preview of next chapter:** Chapter 10 audits your DAG for latent confounders — the variables you didn't include because you couldn't measure them — and computes an E-value for your strongest causal claim, giving you an honest measure of how fragile your conclusion is.
