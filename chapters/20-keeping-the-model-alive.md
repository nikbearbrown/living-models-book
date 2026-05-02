# Chapter 20 — Keeping the Model Alive

*Bayesian updating of edge parameters, structural change detection, DecisionOps vs. MLOps, and the minimum viable feedback loop that closes the orchestration loop introduced in Chapter 13.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *Keeping the Model Alive*
- *The Minimum Viable Feedback Loop*
- *Why DecisionOps Is Not MLOps*

**TL;DR.** A Living Model that does not maintain itself becomes a fossil within a year or two. *Keeping it alive* requires four operational disciplines: Bayesian updating of edge parameters as new evidence arrives; structural change detection that recognizes when the diagram itself, not just its parameters, has become wrong; an operational philosophy — *DecisionOps* — that differs from MLOps in what it monitors, optimizes, and protects against; and a minimum viable feedback loop that closes the recommendation-action-outcome cycle so the model can learn from its own deployments. Without these four, the four properties of Chapter 13 — causal, counterfactual, continually updated, treatment-oriented — degrade to mere description, and the executive report of Chapter 19 starts to lie about a world that has moved past it.

---

## Eighteen Months In

A logistics company has been running a Living Model in production for eighteen months. The model recommends weekly adjustments to capacity allocation across nine distribution centers, and the executive team has been acting on its recommendations consistently. The model's projections have, until recently, been close enough to actual outcomes that the operations leadership has come to trust the system.

For the past three weeks, that trust has been fraying. The drift detector has been firing intermittent alerts. Weekly fulfillment has been running about four percent below the model's projection — not catastrophically, but systematically. Three weeks is enough to be noise; three weeks is also enough to be the leading edge of a shift the model has not yet absorbed. The team is meeting on Thursday to decide what to do.

Their options divide into three categories. They can update the model's parameters — recompute the path coefficients on the edges affected by the recent data, and let the existing diagram absorb the new information. They can examine whether the diagram itself has become wrong — perhaps a new mechanism is operating that the diagram does not represent, or a previously dominant mechanism has weakened. Or they can do nothing, on the assumption that the four-percent gap is noise and the model will catch up on its own. Each of these is the right move in some circumstances. Picking the wrong one has costs. Updating the parameters when the structure has changed will produce a model that is wrong in a worse way than it was before. Restructuring the diagram when the gap is just noise will introduce instability without justification. Doing nothing when the world has actually shifted will leave the model recommending into a context that no longer applies.

The team needs a diagnostic that tells them which mode of correction is appropriate. They also need a system that has been doing the routine maintenance all along, so that this Thursday meeting is not a crisis but a moment of escalation in a process that runs continuously. The disciplines that produce both — the routine maintenance and the diagnostic for when more is needed — are the subject of this chapter, and they are what determine whether the model remains alive at month nineteen, year three, year five.

This is the last chapter of the architectural arc. The Living Model has been defined, the expert has been brought into the room, the elicitation has produced a graph, the algorithms have refined it, the recommendation has been generated, the executive report has communicated it, the executive has acted on it. What remains is the discipline of maintaining the system across time as the world changes, the data accumulates, the experts move on, and the model's commitments either hold or fail. Without these maintenance disciplines, every prior chapter is undermined. The model that produced a defensible recommendation in month one produces an indefensible recommendation in month nineteen, because it has not been kept alive.

---

## Three Modes of Decay

A Living Model can decay in three structurally different ways, and the maintenance disciplines distinguish among them.

**Parametric drift** is the slowest and most familiar mode. The structure of the diagram remains correct, but the magnitudes of the causal effects shift. A price-elasticity coefficient that was estimated at -1.4 last year is closer to -1.7 this year. A retention-rate parameter that was 92% has become 88%. The variables, the arrows, and the mechanisms are unchanged; the strengths of the relationships have moved. Parametric drift is what Bayesian updating is designed to handle. As new data arrives, the parameters refresh, and the model continues to operate without disruption.

**Structural drift** is more consequential. A new mechanism becomes operative that was not in the original diagram, or a previously important mechanism becomes irrelevant. A competitor enters the market and changes the price-demand relationship. A regulatory shift makes a previously unconstrained variable suddenly binding. A technology adoption curve crosses a threshold and an effect that was negligible becomes dominant. Structural drift cannot be handled by parameter updates alone, because the parameters of a wrong structure cannot be made right. The diagram itself has to change.

**Mechanism failure** is the most disruptive. The very causal mechanism the model was built around stops functioning the way it did. A pandemic disrupts the supply-chain dynamics that every operations model was calibrated on. A geopolitical event reorders the trade flows that financial models were built on. The 2020 collapse of restaurant demand, the post-2022 reordering of energy markets, the AI-driven shift in software development costs — each of these was a moment when many models died because their fundamental causal commitments had become inappropriate to the world they were operating in.

The maintenance architecture has to handle all three. Bayesian parameter updating handles parametric drift; structural change detection handles structural drift; deeper interventions — re-elicitation, full model rebuild, strategic decisions about what the model is even modeling — handle mechanism failure. The diagnostic that distinguishes among the three is what determines which response is appropriate, and it is the first piece of the maintenance discipline.

---

## Bayesian Updating of Edge Parameters

The most routine maintenance discipline is the continuous refresh of edge parameters as new data arrives. Modern causal modeling treats each parameter not as a fixed estimate but as a posterior distribution that updates with new evidence. The mechanism is Bayesian, and its operational form is straightforward.

Each path coefficient in the diagram has a prior — initially the estimate produced by Chapter 18's parameterization. As new data arrives, the prior is multiplied by the likelihood of the new data given the parameter, and the result is renormalized to produce the posterior. The posterior then becomes the prior for the next round of updating. Over time, parameters that are consistent across data refresh into tight posteriors with high confidence. Parameters that are inconsistent — that the data keeps revising — develop wide posteriors that signal the underlying instability.

This sounds heavy mathematically, but in production it is implemented through standard infrastructure. Probabilistic programming libraries (PyMC, Stan, Pyro) handle the updating natively. For models that operate on streaming data, sequential Monte Carlo or particle-filtering methods do the same job in real time, refreshing the posterior with each new observation rather than batching the updates. The engineering cost of routine Bayesian updating is, in 2026, comparable to the cost of routine model retraining in conventional ML pipelines — it is more sophisticated mathematically but no harder operationally.

Three implementation considerations matter for keeping the parameter updates honest.

The first is *uncertainty propagation*. The parameter posterior carries uncertainty, and the uncertainty has to flow into the recommendations. A parameter with a tight posterior produces a confident projection; a parameter with a wide posterior produces a hedged projection. Chapter 19's executive report depends on this — the confidence intervals it communicates are derived from the parameter posteriors. A maintenance discipline that updates point estimates without updating uncertainties produces reports that look more confident over time even when the underlying evidence is becoming weaker. This is a quiet failure mode that organizations notice only when a high-confidence recommendation turns out to be wrong.

The second is *informative-data weighting*. Not every observation deserves equal weight in the update. Observations from anomalous periods — a flash sale that distorted normal pricing dynamics, a holiday week that reorganized labor patterns, a one-off promotion that confused the response data — should be down-weighted or excluded. The discipline of distinguishing informative observations from anomalous ones is part of the data-quality work that the maintenance architecture has to handle, and it is more domain-specific than the underlying mathematics suggests. A mature deployment includes data-quality gates that flag anomalous periods and adjust the parameter updates accordingly.

The third is *interventional data*, when it can be obtained. Observational data refines parameters within the constraints of the diagram. Interventional data — randomized experiments on specific variables — can do more, including occasionally resolving structural ambiguities that observational data cannot. When the organization runs an A/B test, a price experiment, or any other randomized intervention on a variable in the model, that data is gold for the maintenance system. It deserves a privileged role in the parameter updates, because it carries causal information that pure observation does not. Modern Bayesian causal discovery frameworks — including the Interventional Bayesian Causal Discovery (IBCD) approach the source material describes — are explicitly designed to leverage this kind of data when it is available.

---

## Structural Change Detection

Bayesian parameter updating handles drift within a stable structure. It does not handle changes to the structure itself. For that, a separate set of disciplines is needed, collectively called *structural change detection*.

The diagnostic question is whether the gap between the model's projections and the actual outcomes is consistent with parameter drift or with structural drift. The two have different signatures. Parameter drift produces gaps that are roughly proportional across the model's predictions — when the parameters shift, the projections shift consistently. Structural drift produces gaps that are concentrated. Some predictions are unaffected; others are dramatically wrong. The pattern of the residuals tells the diagnostic.

Several specific algorithms operate on this principle. The *Causal Mechanism Shift Identification* (CMSI) approach examines the conditional distributions of each variable given its parents in the current diagram, and tests whether those conditionals have shifted between the period the model was calibrated on and the current period. A shift in a specific conditional points to a structural problem in the mechanism that produces that variable, while leaving the rest of the diagram potentially intact. The *Dynamic Online Causal Learning* (DOCL) framework operates on streaming data, maintaining sufficient statistics for the underlying covariance structure and triggering structural alerts when the Mahalanobis distance between current observations and the model's expectations exceeds a calibrated threshold. The *CaSCo* approach uses graph neural networks to encode causal graphs as embeddings and compare them across time windows, detecting structural drift through changes in the embedding space.

The technical details vary; the operational pattern is consistent. The maintenance system runs structural change detection on a continuous or periodic basis. When the detection flags a possible structural shift, the system raises the flag to the team operating the model. The team's response is a graduated escalation. Minor flags trigger a diagnostic review of the affected mechanism — *is there a plausible reason this conditional distribution would have shifted?* Significant flags trigger a re-elicitation of the affected portion of the graph — typically a focused expert session on the mechanism in question, similar to but smaller than the full Chapter 16 elicitation. Major flags trigger a more substantial intervention, possibly including a full re-elicitation of major portions of the graph.

The cost of structural change detection — the engineering effort to build the detectors, the analyst time to triage flags, the expert time to re-elicit when warranted — is substantial. It is also unavoidable. A Living Model without structural change detection will, over time, accumulate structural errors that the parameter updates cannot fix, and the recommendations will degrade. The investment in detection is not optional; it is what allows the model to remain trustworthy over years rather than months.

I want to flag one specific operational point. Structural change detection is more sensitive than the maintenance team typically wants. The natural response to a flag is to treat it as a false alarm and continue operating; the right response is usually to take the flag seriously and investigate. Many real structural shifts produce ambiguous signals before they produce decisive ones, and the maintenance discipline is to act on the ambiguous signals rather than waiting for the decisive ones. The discipline is uncomfortable because most flags do turn out to be noise, and most investigations confirm that no action is needed. But the few that do reveal structural drift are the ones that protect the model from going stale, and the cost of investigating false alarms is typically much smaller than the cost of missing a real shift.

---

## DecisionOps Is Not MLOps

The operational practices that have grown up around traditional machine learning — collectively, *MLOps* — are well understood and increasingly mature. Model training, validation, deployment, monitoring, and retraining have established tooling and patterns. A Living Model deployment cannot rely on these patterns alone, because what a Living Model produces is structurally different from what a predictive model produces. The maintenance disciplines for a Living Model belong to a different operational category, which I will call *DecisionOps*.

The principal differences are four.

**MLOps optimizes prediction; DecisionOps optimizes decision.** A predictive model is judged on the gap between predicted outcomes and actual outcomes — accuracy, RMSE, AUC, depending on the problem. A Living Model is judged on whether its recommendations produce better outcomes than the alternatives would have. The metric is not prediction accuracy; it is decision quality. A Living Model could be moderately less accurate at predicting outcomes than a comparable predictive model and still produce dramatically better decisions, because the recommendations are based on causal mechanism rather than correlation, and the recommendations survive distributional shifts that wreck the predictive model. The MLOps practice of monitoring prediction accuracy as the principal health metric is therefore the wrong default for a Living Model. The right default is to monitor decision quality, which is harder to measure but more relevant to the system's value.

**MLOps assumes the data-generating process is stable; DecisionOps assumes it is contested.** The conventional MLOps assumption is that the world the model was trained on is the world the model will be deployed in. When the assumption holds, the model continues to perform; when it fails, the model needs retraining. A Living Model assumes the world is in motion — that interventions reshape the world, that competitors respond, that regulations shift, that the data-generating process the model is meant to describe is itself partly the product of the model's actions. The maintenance discipline is therefore not "retrain when the data shifts" but "monitor for structural change continuously, and update the structure when warranted."

**MLOps treats the model as the artifact; DecisionOps treats the recommendation pipeline as the artifact.** The MLOps system is organized around the model — its training, its weights, its versioning. The DecisionOps system is organized around the pipeline that produces recommendations — the elicitation, the parameterization, the counterfactual engine, the EV ranking, the constrained-knapsack optimization, the executive report. Each of these has its own lifecycle, its own update cadence, its own maintenance requirements. The model's parameters update on one cadence; the elicitation refreshes on another; the executive report's narration discipline is reviewed on a third. The DecisionOps system has to coordinate all of these, and the coordination is what produces a coherent system rather than a collection of disconnected updates.

**MLOps can usually accept human review at the end; DecisionOps requires it throughout.** A predictive model can produce predictions that are checked by a human only at the point of consumption — the analyst reviews the dashboard, the customer success manager reviews the churn score. A Living Model has to integrate human review at multiple points: the recommendation goes to a named human owner who decides whether to execute it; structural change flags go to a domain expert who decides whether to investigate; the audit record is reviewed periodically by people with the authority to challenge the system's commitments. The DecisionOps practice has to support all of these review points, with the appropriate tooling, audit trails, and escalation paths. This is more demanding than the typical MLOps pipeline, and the operational team has to be staffed and trained accordingly.

The transition from MLOps to DecisionOps is not, in most organizations, a clean break. The two coexist. Predictive models continue to operate alongside Living Models, and the operational team often runs both. The architectural commitment is to *recognize the difference* — to staff and tool the two operations differently, to set different metrics and different escalation paths, to expect different failure modes and different remediation cadences. Organizations that try to operate a Living Model with MLOps practices will find that the model degrades in ways the practices were not designed to detect, and the maintenance work the model needs will not get done.

---

## The Minimum Viable Feedback Loop

The deepest commitment of the maintenance architecture is the closure of the recommendation-action-outcome loop. This is what makes the model *living* in the strongest sense — the system is not just updated periodically by a team; it learns continuously from the consequences of its own actions.

I have called this loop the *minimum viable feedback loop*, and the discipline is to specify what its components have to be.

The loop has four stages, and each has to be operational for the loop to close.

*Recommendation* is the first stage. The Living Model produces a recommendation, structured as the four-part executive report of Chapter 19. The recommendation is specific, sized, owned, and accompanied by evidence, assumptions, and counterfactual.

*Action* is the second stage. The named human owner — the one whose name appears on the audit record — decides whether to execute the recommendation, modify it, or override it. The decision is recorded with the rationale. The action, if taken, is logged in a way that connects it back to the recommendation that produced it. (This linkage is important and easy to omit. Many organizations have separate systems for analytical recommendations and for operational actions, and the link between them gets lost.)

*Outcome* is the third stage. The world responds. Time passes. The outcome — the actual revenue, the actual retention, the actual fulfillment, whatever the recommendation was projecting — arrives. The outcome is recorded in a way that the system can connect it back to the action and the recommendation.

*Update* is the fourth stage, and the one that closes the loop. The system compares the projected outcome (from the recommendation) to the actual outcome (from the world). The gap, if any, is decomposed into the contributions of various model components. Bayesian parameter updates apply the gap to the relevant edge parameters. Structural change detection considers whether the gap signals a structural problem. The audit record captures the comparison so future reviews can examine it.

When all four stages operate together, the model learns from each cycle. The recommendation that was made last month becomes evidence for or against the model's structural commitments this month. The model that produced this month's recommendation is, in a literal sense, a refined version of the model that produced last month's. The orchestration concept I introduced in Chapter 13 — that a Living Model is more than the sum of its four properties because of the closed loop — is operationalized through this minimum viable feedback loop.

When any stage fails, the loop breaks. A model whose recommendations are not connected to actions cannot learn from outcomes. A model whose actions are not recorded with rationale cannot distinguish between recommendation failures and execution failures. A model whose outcomes are not connected back to recommendations cannot tell whether its projections were right. A model that does not feed the comparison back into the parameters and structure cannot improve over time. Each broken stage is a failure mode, and each is a common one in real deployments.

The minimum viable feedback loop is the smallest version of the architecture that produces a system that can learn. Larger and more sophisticated loops — with experimental interventions, with active learning, with multi-stakeholder review at each stage — are valuable but not strictly necessary. The minimum is what every Living Model deployment must have. Without it, the system is a sophisticated piece of analysis that happens to refresh periodically, and the *living* qualifier is unearned.

---

## Closing the Architectural Arc

This chapter closes Part Three of the book and the architectural arc that began with Chapter 13's definition of the Living Model. The architecture is now described in full. The Living Model has been defined, the elicitation has been specified, the data-driven refinement has been laid out, the operational pipeline from graph to decision has been built, the executive report has been structured, and now the maintenance disciplines that keep the system alive over time have been described.

A reader who has worked through Part Three has, I hope, a clear picture of what it would mean to deploy a Living Model in their organization. Not as a research project, not as an analytical curiosity, but as operational infrastructure that supports consequential decisions over years. The architecture is feasible. Every component has been demonstrated. The integration is the engineering work, and the engineering is no harder than the engineering required for any modern data infrastructure.

The remaining parts of the book — *The Frameworks* (Christensen, Damodaran, and the theory-guided Living Model), *The Cases* (worked examples of Living Models applied to real strategic decisions), and *The Frontier* (what Living Models cannot yet do, and what comes next) — apply and extend the architecture I have just described. They do not introduce new architectural commitments; they exercise the architecture in specific contexts. A reader who wants to see the architecture in action should continue. A reader who has the architecture and wants to start building should be in a position to do so from the foundation Part Three has laid.

I want to close with one observation about the trajectory of this kind of work. In 1999, drawing a causal diagram in a corporate strategy meeting would have been considered eccentric at best; the language of structural causal models, do-operators, and counterfactual reasoning was confined to academic departments. In 2026, the language has spread far enough that boards expect their analytics teams to engage with it. By 2030 — perhaps sooner — the architectural patterns I have described in Part Three will be conventional, and the organizations that have not adopted them will be at a competitive disadvantage they will struggle to identify, because they will be unable to ask the questions the architecture is designed to answer. The window for early adoption is open now. It will close. The organizations that build Living Models in 2026 and 2027 will, in 2032, have years of operational experience with the architecture that their followers will spend years trying to acquire.

The more important closing observation is the human one. Every chapter of this book has, at one level or another, been about the relationship between human judgment and computational apparatus. The apparatus does not replace the judgment. The expert in the room remains mathematically necessary; the named human owner remains accountable; the executive remains the decision-maker. What the apparatus does is *augment* the judgment — by making the assumptions explicit, the evidence auditable, the alternatives concrete, the consequences traceable. A Living Model is, in the end, a structured way for humans to make better decisions about the world they are trying to change. The architecture is in service of that. The architecture is, on its own, only an artifact. The decisions it supports — to expand a hospital's surgical program, to restructure a supply chain, to change a pricing model, to reform a public benefit program — are what the architecture is for, and the quality of those decisions is what the architecture will ultimately be judged on.

---

## Looking Forward

Part Four takes up the integration of theoretical frameworks — Christensen's disruption theory, Damodaran's corporate life cycle — with the Living Model architecture. The frameworks are not models in the sense Part Three has used the term; they are inputs to models, sources of structural priors and Bayesian beliefs that the elicitation step can leverage. The chapters in Part Four show how to use them well, and where they break.

---

**What would change my mind:** A demonstration that a Living Model deployed without one of the four maintenance disciplines — Bayesian updating, structural change detection, DecisionOps, the minimum viable feedback loop — produces decision quality comparable to a model with all four, sustained over a multi-year deployment. There are organizations that have shipped causal models without the full maintenance architecture, typically because the architecture was not yet articulated when they shipped. Their experience is the natural test. I believe the four disciplines are necessary; if shown a serious counter-example, I would update toward whichever subset turned out to be sufficient.

**Still puzzling:** How to handle the case where the maintenance system itself becomes the limiting factor — where the team operating the Living Model spends so much time maintaining it that the model's recommendations become marginal in the cost-benefit. The architecture I have described is substantial, and operating it well is not free. There is a real risk that the maintenance overhead crowds out the analytical work the model is supposed to support. I do not yet have a clean way to size the maintenance investment relative to the model's value, and I suspect this will be one of the practical lessons learned by the first generation of organizations that operate Living Models at scale.

---

**Tags:** Bayesian updating edge parameters, structural change detection CMSI DOCL CaSCo, DecisionOps versus MLOps operational philosophy, minimum viable feedback loop, Living Model maintenance discipline
