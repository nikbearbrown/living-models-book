# Chapter 18 — From Graph to Decision

*Parameterizing the graph, running the counterfactual, ranking interventions by Expected Value, and the constrained-knapsack from rankings to portfolio decisions.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *From Graph to Decision*
- *Turning Structure Into a Recommendation*
- *The Counterfactual, the Ranking, and the Knapsack*

**TL;DR.** A validated causal graph is not yet a recommendation. Four operational steps convert it into one. *Parameterization* assigns functional mechanisms and quantitative coefficients to the edges, turning structure into something that produces numbers. The *counterfactual engine* runs each candidate intervention through the parameterized model and computes the predicted change in outcome relative to inaction. *Expected Value ranking* combines the predicted effect with the probability of success and the heterogeneity of response across the population, producing a ranked list of interventions. The *constrained-knapsack optimization* selects the subset of interventions that maximizes total expected value subject to budget, capacity, dependency, and exclusion constraints. The output of the four steps is the recommendation Chapter 19's executive report will communicate.

---

## Thirty Interventions, Six Slots

The vice president of operations at a consumer-goods manufacturer is reading the Friday output from her Living Model. The graph behind the model has been validated through the protocols of Chapters 14 through 17. The company runs eleven production lines across three plants, manufacturing about a hundred and fifty distinct SKUs that share fillers, labor, and packaging materials. Demand has been turbulent for the past two quarters; one product has gone unexpectedly viral and another has collapsed. The board has asked her to recommend, by Monday, a set of operational interventions to stabilize the next quarter.

The model has produced a ranked list of thirty-one candidate interventions. Each intervention is specified at the level the team can actually execute — *increase output of SKU 4421 by twenty percent for the next four weeks, reallocating shared labor from line two; renegotiate the packaging contract on SKU 1108 to allow same-day delivery from the third-party supplier; defer the planned changeover at line five by two weeks*. For each intervention, the model has produced an expected change in the company's primary outcome, a confidence interval around the change, and a list of which plants and lines would be affected.

The list cannot be acted on as a list. The interventions interact. Some require labor that other interventions also need. Some are mutually exclusive — you cannot both surge SKU 4421 and reallocate the same labor to SKU 1108. Some have prerequisites — the contract renegotiation has to land before the packaging change can be implemented. The total cost of doing all thirty-one is far beyond what the operations budget will support; even if it were not, the operational complexity of changing thirty-one things at once would create more disruption than the interventions are designed to address.

She needs a portfolio. She needs the model to tell her not just which interventions are good but which six or so should be done together, given that the budget is finite, the labor is finite, the calendar is finite, and the interventions interact. This chapter is about how the model produces that portfolio. The graph from Chapter 17 is the starting point. The recommendation that goes into the board meeting on Monday is the destination. Four operational steps separate the two.

---

## Parameterizing the Graph

The graph specifies which variables cause which. Parameterization specifies *how strongly* and *how* — the functional form and the magnitudes of each causal relationship. A graph without parameters is a structure; a graph with parameters is a generative model that can be queried for predictions.

The simplest case is a linear parameterization. For each variable Y in the graph, we write its value as a linear combination of its parents, plus a noise term:

> Y = β<sub>1</sub> X<sub>1</sub> + β<sub>2</sub> X<sub>2</sub> + ... + β<sub>k</sub> X<sub>k</sub> + ε<sub>Y</sub>

The coefficients β<sub>i</sub> are the *path coefficients* we met in Chapter 6. They are estimated from the data, given the structure of the graph. Standard regression methods, with the deconfounding sets identified by the backdoor criterion of Chapter 8, produce unbiased estimates when the linear assumption holds. The result is a fully specified linear structural equation model — a graph annotated with numbers, ready to be queried.

The linear parameterization is a starting point, not an end. Many real systems are nonlinear. The relationship between price and demand is rarely linear over more than a small range; a doubling of price does not produce twice the elasticity response. The relationship between marketing spend and acquisition follows diminishing returns. The relationship between staffing and throughput hits saturation. A model that forces these relationships into linear form will be wrong in the regions the team most cares about — the regions where the recommendation might be aggressive enough to push the system into a new operating regime.

For nonlinear cases, the parameterization uses a richer functional family. Generalized linear models extend the linear case to handle binary, count, and bounded outcomes. Spline-based methods allow flexible curves without committing to a specific functional form. Modern *Deep Structural Causal Models* (DSCMs) use neural networks to learn arbitrary mapping functions between parents and children while preserving the ability to perform abduction on the latent noise terms. The trade-off is that as the functional family grows richer, the estimation becomes more demanding and the diagnostic checks become harder to perform. A linear model is easy to verify; a deep model often is not.

The choice of parameterization is, in my view, primarily a question of how nonlinear the underlying mechanisms actually are in the regions the recommendation will operate in. For decisions near the operating point — *should we increase output by ten percent?* — linear local approximations are usually sufficient. For decisions far from the operating point — *should we increase output by two hundred percent?* — the linear approximation breaks down and richer parameterizations are needed. A mature deployment of the architecture will use linear parameterizations for most variables and richer ones where the science of the domain demands it.

Two implementation considerations matter for the rest of the architecture.

The first is *modularity*. Each parameterized mechanism in the graph should be a self-contained piece of the model, replaceable without affecting the others. This is the autonomy-of-causal-mechanisms principle from Chapter 6, cashed in as an architectural commitment. When a single mechanism's parameters need to be updated — because new data has arrived, because the world has shifted, because the original estimate was wrong — the update should be local. The other mechanisms should not need to be touched. Modularity is what makes the continual updating property of Chapter 13 operational.

The second is *uncertainty representation*. Every parameter in the model has uncertainty around it. Some parameters are estimated from large samples and have tight confidence intervals; others are estimated from thin data or expert input and have wide intervals. The parameterization has to carry the uncertainty forward. When the model produces a recommendation, the recommendation should come with bounds, not just a point estimate, and the bounds should reflect the underlying parameter uncertainty. Chapter 19's executive report depends on this; without uncertainty propagated through the pipeline, the report cannot honestly communicate confidence to the decision-maker.

---

## Running the Counterfactual

With the graph parameterized, the model can be queried for predictions. Three kinds of query matter, and they correspond to the three rungs of Pearl's Ladder from Chapter 5.

A *Rung 1* query asks the model to predict an outcome given observed evidence. *Given that production at line two is currently running below target, what is the predicted fulfillment rate next week?* This query uses the parameterized model to propagate from observed inputs to predicted outputs. It is the kind of query a predictive model handles, and the parameterized graph handles it as well.

A *Rung 2* query asks the model to predict the consequences of an intervention. *If we increase production at line two by fifteen percent, what is the predicted fulfillment rate?* This query uses the do-operator. The model deletes the upstream causes of *line two output*, sets the variable to its intervened value, and propagates the consequences forward. The result is a predicted outcome under the intervention, which is the direct input to ranking.

A *Rung 3* query asks the model to predict what would have happened in a specific case under a different intervention. *Given that we ran line two at its current rate and observed last week's fulfillment, what would last week's fulfillment have been if we had run line two fifteen percent higher?* This query uses Pearl's three-step abduction-action-prediction procedure from Chapter 9.

Step one: *abduction*. Take the observed evidence — the actual production rates, the actual fulfillment, the actual values of every variable in the model — and use the parameterized model to back out the case-specific noise terms. These noise terms are what distinguish this specific week from the average week the model was estimated on. Whatever was idiosyncratic about the demand pattern, the supplier reliability, the workforce attendance, gets captured in the noise terms.

Step two: *action*. Modify the model to enforce the counterfactual condition — line two at fifteen percent higher than actual — by deleting the upstream causes of the line-two output and setting it to the counterfactual value.

Step three: *prediction*. Run the modified model with the case-specific noise terms recovered in step one, producing a counterfactual outcome for the week. The counterfactual carries forward the idiosyncrasies of the actual week into the alternative-action prediction.

For the operations head's question — *what should we do this quarter?* — the relevant query is mostly Rung 2. She is choosing among interventions that have not yet been taken. The Rung 3 machinery becomes relevant later, when the chosen interventions have been deployed and the team is reviewing what happened: *given that we did this and got this outcome, would we have done better with a different choice?* The audit-ability that Chapter 17 emphasized depends on Rung 3 queries; the immediate recommendation depends on Rung 2.

The implementation move that turns the parameterized graph into a counterfactual engine is operationally straightforward. Modern causal inference libraries — DoWhy, EconML, CausalML — support the do-operator and the three-step procedure as primary operations. Given a parameterized SCM, the libraries can compute interventional and counterfactual distributions with a few lines of code. The intellectual work is in the graph and the parameters; the engineering work of running the queries is comparatively light.

One important nuance is *heterogeneity*. The model has produced a parameterized graph that represents the average mechanism across the population. But individual units — individual customers, individual production lines, individual time periods — vary in how they respond to interventions. The same intervention applied to two customers with different characteristics may produce dramatically different effects. This is the conditional average treatment effect (CATE) we met in Chapter 8, and it is the reason average effects are rarely the decision-relevant fact.

For the operations example, heterogeneity means that an intervention's effect on overall fulfillment depends on which production line it targets, which SKUs are running, which suppliers are involved, and which customers are downstream. The model's counterfactual queries can be run at any level of granularity: average over the whole network, conditional on a specific plant, conditional on a specific SKU, conditional on a specific customer segment. The right level of granularity depends on what the intervention can be targeted at. If the intervention will be applied uniformly across the network, the average is the right answer. If it can be targeted at specific lines or SKUs, the conditional answers matter, and the model should produce them.

---

## Ranking by Expected Value

The counterfactual engine produces, for each candidate intervention, a predicted change in the outcome relative to inaction. This is the *effect size* of the intervention. The recommendation, however, is not based on effect size alone. It is based on *Expected Value* — the effect size weighted by the probability that the intervention actually produces the predicted effect.

The Expected Value of an intervention can be written as:

> EV<sub>i</sub> = p<sub>i</sub> × m<sub>i</sub>

where p<sub>i</sub> is the probability that the intervention produces a positive outcome, and m<sub>i</sub> is the magnitude of the outcome conditional on its occurring. The two factors are conceptually different. The probability captures the reliability of the intervention — how often, in the population the intervention is targeting, does it work? The magnitude captures the size of the effect when it does work. An intervention with a high probability and a small magnitude is reliable but unimpressive. An intervention with a low probability and a large magnitude is a long shot. The EV summarizes both into a single number that can be ranked.

Two refinements matter for the operational ranking.

The first is *uncertainty bounds*. The model's estimate of EV is itself uncertain — the parameter uncertainties from the parameterization step propagate through the counterfactual engine and into the EV. A mature ranking does not just produce point estimates; it produces ranges. *Intervention A has an expected value of $2.1M with a 90% interval from $1.4M to $2.9M. Intervention B has an expected value of $1.9M with a 90% interval from $0.3M to $3.5M.* The point estimates suggest A is better; the intervals show that B has both more upside and more downside. The right ranking depends on the decision-maker's risk tolerance and on whether the question is about the average outcome or the worst-case outcome. The architecture has to surface this distinction rather than collapsing it into a single number that hides it. (The risk-decomposition logic from Chapter 4 — *risk is two numbers, not one* — applies directly to the EV ranking.)

The second is *heterogeneous response*. The same intervention applied to two different units may have different probabilities and different magnitudes. In customer-facing interventions, the literature has converged on a four-way segmentation that captures the typical response patterns. *Persuadables* are the units for whom the intervention causes the desired behavior; they are the units worth targeting. *Sure things* are units that exhibit the desired behavior with or without the intervention; targeting them wastes resources. *Lost causes* are units that fail to exhibit the desired behavior regardless of intervention; targeting them is similarly wasteful. *Do-not-disturb* units actually respond negatively to the intervention; targeting them is actively harmful. The ranking, properly done, identifies the persuadables and prioritizes them, while explicitly avoiding the do-not-disturb cases.

For the operations example, the heterogeneity question is less about customer segments and more about which lines, SKUs, and time windows are most responsive to each intervention. The same logic applies. The model's CATE estimates allow the ranking to be done at the level of granularity at which the intervention will be applied, and the ranking explicitly distinguishes the high-response cases from the low-response cases.

I want to underline a methodological point that is easy to underweight. The EV ranking depends on the parameterized model being right. If the model overestimates the effect size of an intervention, the ranking will overrate it. If the probability estimates are miscalibrated — typically toward overconfidence, given the cognitive biases of Chapter 15 — the ranking will be miscalibrated in the same direction. Sensitivity analysis is the discipline that addresses this. For each intervention in the top of the ranking, the model should be queried with perturbed parameters to see how robust the ranking is. Interventions whose EV estimates are robust to plausible parameter perturbations are more credible; interventions whose EV depends critically on a specific parameter value are less credible. The executive report in Chapter 19 will surface this distinction explicitly.

---

## The Constrained-Knapsack Portfolio

A ranked list of interventions is not yet a recommendation. The recommendation has to take into account the constraints under which the interventions will be implemented — budget, labor, time, dependencies, and exclusions. The mathematical apparatus that handles this is the *constrained knapsack problem*, and its variants.

The simplest case is the *0-1 knapsack*. There are n candidate interventions; each has an Expected Value v<sub>i</sub> and a cost c<sub>i</sub>; there is a total budget B. The problem is to select a subset of interventions that maximizes total Expected Value subject to the budget:

> Maximize: Σ v<sub>i</sub> x<sub>i</sub>
> Subject to: Σ c<sub>i</sub> x<sub>i</sub> ≤ B
> With: x<sub>i</sub> ∈ {0, 1}

The variable x<sub>i</sub> is one if intervention i is selected and zero otherwise. The problem is well-studied in operations research; for problems of moderate size, modern solvers find optimal solutions in seconds. For very large problems, approximation algorithms with provable bounds are available.

The 0-1 knapsack is rarely sufficient for actual portfolio selection. Real interventions face multiple constraints simultaneously, interact with each other, and come with prerequisites. Four extensions of the basic formulation matter for the architecture.

**Multidimensional knapsack** handles multiple independent constraints. The operations example has at least four: budget (dollars), labor (person-hours by skill category), filler-time (machine hours on shared equipment), and time-to-launch (calendar days before each intervention can be deployed). Each constraint defines its own capacity. The selected portfolio must satisfy all of them simultaneously. The problem is harder than the 0-1 case but still tractable for typical sizes.

**Logical-dependency knapsack** handles prerequisites and exclusions. *Intervention A can only be done if Intervention B is also done* becomes the constraint x<sub>A</sub> ≤ x<sub>B</sub>. *Interventions C and D cannot both be selected* becomes x<sub>C</sub> + x<sub>D</sub> ≤ 1. *At most two of the price-change interventions can be selected* becomes Σ x<sub>price</sub> ≤ 2. These linear constraints encode the operational realities of how interventions actually fit together. A portfolio that violates them is not implementable, regardless of how attractive it looks on the unconstrained ranking.

**Multiple-choice knapsack** handles mutually exclusive groups. When one of the candidate interventions is *raise prices on SKU X by 5%*, another is *raise prices on SKU X by 10%*, and a third is *raise prices on SKU X by 15%*, the team can select at most one of the three. The multiple-choice formulation enforces this directly: the group of mutually exclusive interventions is treated as a single decision variable that takes one of the candidate values, including the value *do nothing on SKU X*.

**Chance-constrained knapsack** handles uncertainty in the costs and values. When the cost of an intervention is itself uncertain — labor costs may run over, contractor rates may shift, scope may expand — the constraint cannot be enforced as a strict inequality with point estimates. The chance-constrained version specifies a confidence level: *the budget must be respected with at least 95% probability*. The optimization then accounts for the joint distribution of intervention costs and selects a portfolio whose expected cost is below the budget with the specified confidence. The chance-constrained formulation is particularly important when the EV ranking already includes uncertainty bounds, because the optimization can then propagate the uncertainty through to the portfolio recommendation.

The four extensions can be combined. The full operational formulation for a typical Living Model recommendation is a multidimensional, logical-dependency, multiple-choice, chance-constrained knapsack. This sounds intimidating; in practice, modern mixed-integer linear programming solvers handle problems of this complexity with hundreds of variables in seconds. The intellectual work is in formulating the constraints correctly. The computational work is comparatively light.

The output of the optimization is a *portfolio* — a subset of interventions, satisfying all the constraints, that maximizes total Expected Value. For the operations head, the portfolio specifies which six or seven of the thirty-one candidate interventions to do, in what order, on which lines, with what budget allocation. The portfolio comes with its own EV bounds (the sum of the bounds on its constituent interventions, with appropriate covariance corrections), its own sensitivity analysis (which interventions in the portfolio are most sensitive to parameter perturbations), and its own counterfactual statement (the predicted outcome relative to *doing nothing* and to the *next-best portfolio*).

This last point is operationally important. The recommendation is not just *do this portfolio*. It is *do this portfolio because, relative to the alternatives, it produces this expected gain, with these bounds, under these assumptions*. The counterfactual framing makes the recommendation auditable. When the operations head presents the recommendation to the board, she can answer the inevitable questions about the alternatives. *What if we did this other portfolio instead?* The model has the answer; the counterfactual machinery generates it on demand.

---

## The Handoff to Chapter 19

What the four operational steps have produced, by the end of Chapter 18, is a structured package ready for executive consumption.

The package contains, at minimum: the recommended portfolio of interventions; the predicted change in the outcome of interest, with uncertainty bounds; the alternative portfolios considered, ranked by their Expected Values; the constraints that bind on the optimal portfolio (which budget, which labor category, which time window is the limiting factor); the assumptions on which the recommendation depends (which parameters were estimated from data versus from expert input, which had wide uncertainty intervals); the sensitivity of the recommendation to plausible perturbations of those assumptions; and the counterfactual statement of what happens if the recommendation is not taken.

This is more information than the executive can consume directly. The Living Model has produced a complete causal-and-decision artifact; what the executive needs is a compact, prioritized, action-oriented presentation of the artifact's most important content. Chapter 19 takes up the design of that presentation — what we call the Causal Brain Executive Report — and the principles that govern what such a report must do, what it must avoid, and how it differs from the dashboard or summary slide that has historically served the same role.

I want to flag one structural point about the architecture before moving on. Chapter 18's pipeline — parameterize, simulate counterfactuals, rank by EV, optimize subject to constraints — is not specific to any particular domain. The same four operational steps work for operations, marketing, clinical care, public policy, supply chain, and most other domains where Living Models will be deployed. What changes from domain to domain is the content of the graph, the parameterization choices, the relevant constraints, and the structure of the response heterogeneity. The pipeline itself transfers. This is part of why the Living Model architecture is general-purpose: the same operational machinery serves a wide range of decision problems, and the domain-specific work is concentrated in the elicitation and parameterization steps that the architecture has already addressed.

---

## Looking Forward

Chapter 19 takes up the executive report. The package described above contains far more information than any executive can use, and the executive needs not the full artifact but the actionable subset of it: the recommendation, the evidence behind it, the assumptions that could break it, and the counterfactual that justifies acting now rather than waiting. The chapter describes the structure of the report, the principles that determine what it must include and exclude, the role of LLM narration in producing the report from the artifact's raw outputs, the visualization choices that support rather than obscure the recommendation, and — finally — the question of accountability when a recommendation is followed and the outcome is not what was predicted.

---

**What would change my mind:** A demonstration that an end-to-end black-box system — no explicit parameterization, no counterfactual machinery, no constrained optimization, just a learned mapping from observed state to recommended action — produces portfolio recommendations of comparable quality to the structured pipeline in this chapter. There are recent attempts in this direction, including some applications of reinforcement learning. The structural disciplines I have described impose costs (engineering, governance, computation), and if the costs do not buy improved recommendations, the case for the structure is weaker. I would update if shown a head-to-head comparison where the black-box approach matched or exceeded the structured one in a non-trivial domain.

**Still puzzling:** How to handle the case where the optimal portfolio depends critically on a parameter that the team is not confident about. The sensitivity analysis surfaces the dependence, but it does not tell the team what to do about it. The clean answer is *gather more data on the contested parameter before committing to the portfolio*, but the team often does not have time. The unclean answer is *commit to a less optimal portfolio that is more robust to the parameter uncertainty*. The trade-off between expected value and robustness is real, and I do not yet have a confident framework for how decision-makers should make it. The discussion in Chapter 19 about communicating uncertainty to executives bears on this, but it does not resolve it.

---

**Tags:** parameterizing causal graph structural equations, counterfactual engine abduction action prediction, Expected Value of Intervention ranking, heterogeneous treatment effects persuadables, constrained-knapsack portfolio optimization
