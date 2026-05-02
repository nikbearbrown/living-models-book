# Chapter 10 — Confounders, Colliders, and the Limits of Observational Data

*The unconfoundedness assumption, latent confounders, collider bias, and the honest account of what causal inference cannot do.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *The Honest Account*
- *Confounders, Colliders, and What Observational Data Cannot Do*
- *The Latent Confounder Problem*

**TL;DR.** All causal inference from observational data rests on the assumption that we have measured a sufficient set of confounders to block every backdoor path. This assumption — *unconfoundedness*, or *no unmeasured confounding* — cannot be tested from data alone. When it fails, our estimates are biased, sometimes dramatically. Sensitivity analysis quantifies how robust our conclusions are to plausible amounts of unmeasured confounding. Collider bias, which arises when we condition on a variable that is jointly affected by two other variables, is a separate failure mode that is easy to introduce and easy to overlook. The honest account of causal inference acknowledges these limits and supplies tools for working within them.

---

## The Story That Reversed Itself

In the early 2000s, a series of large observational studies appeared to show that hormone replacement therapy (HRT) reduced the risk of coronary heart disease in postmenopausal women. The studies were well designed by the standards of the time. They controlled for age, smoking, diet, exercise, and a long list of other plausible confounders. The reported effect was substantial — a 40-50% reduction in risk. Doctors began routinely prescribing HRT for cardiovascular protection.

Then, in 2002, the Women's Health Initiative published the results of a large randomized controlled trial. It found exactly the opposite. HRT *increased* the risk of coronary heart disease. The randomized trial was stopped early because of harm to the participants. The observational studies had been thoroughly misleading.

What had gone wrong? The most plausible explanation, developed in the years that followed, is that the observational studies failed to control for socioeconomic status and health-seeking behavior. Women who took HRT in the observational studies were, on average, wealthier, better educated, and more attentive to their health than women who did not. They had better diets, exercised more, saw their doctors more often, and were generally at lower risk for cardiovascular disease for reasons that had nothing to do with HRT itself. The studies controlled for some of these factors but missed enough that the residual confounding produced the spurious protective effect.

The HRT story is a cautionary one, and it is one of the cleanest examples in the medical literature of how observational studies can mislead. The methodology was not careless. The analysis was not naive. The investigators controlled for the confounders they could think of. They could not control for the confounders they could not measure, and that gap was enough to flip the sign of the conclusion.

This chapter is about that gap — the limits of what observational data can do, and what we can and cannot do about those limits.

---

## The Unconfoundedness Assumption

The methods we have developed in Chapters 6 through 9 — backdoor adjustment, double machine learning, causal forests, counterfactual reasoning — all share a common foundation. They assume that the causal diagram is correct, and in particular that we have measured a sufficient set of confounders to block all backdoor paths from treatment to outcome.

This assumption goes by several names. *Unconfoundedness*. *Conditional ignorability*. *Selection on observables*. The names differ; the content is the same. Treatment assignment, conditional on the measured covariates, is independent of the potential outcomes. In plain language: once we have controlled for the variables in our diagram, any remaining association between treatment and outcome is a real causal effect, not a residual confounding artifact.

The unconfoundedness assumption cannot be tested from observational data. This is a sharp, unpleasant fact. We can run all the diagnostic checks we like, and they will not reveal whether an unmeasured confounder is biasing our estimate. The diagnostics tell us whether our model fits the data we have; they cannot tell us about the data we don't have.

This is the principal limit of observational causal inference. We work within it by being explicit about the assumption, by gathering data on as many potential confounders as we can, by using domain knowledge to identify confounders that might not be obvious, and by performing sensitivity analysis to quantify how robust our conclusions are. We do not pretend the assumption is testable. We do not pretend the assumption is always satisfied. We work in a state of acknowledged uncertainty, and we communicate that uncertainty to decision-makers.

---

## Latent Confounders in the Wild

Latent (unmeasured) confounders are not a hypothetical concern. They are present in every observational study. The question is whether they are large enough to alter the conclusions.

Some examples of how latent confounders show up:

**The competitor's internal meeting.** A pharmaceutical company is studying the effect of its drug on patient outcomes. Two months into the study, a competitor announces a recall of a similar drug for safety reasons. Doctors, hearing the news, switch some of their patients to the company's drug. The patients who switch are systematically different from the patients who started on the drug from the beginning — they are sicker, more concerned, more risk-averse. None of this is captured in the company's data. The estimated effect is biased.

**The macro-sentiment shift.** A B2B SaaS company is studying the effect of a new pricing tier on customer behavior. During the study period, a recession hits, and customers across the industry start cutting software spend. The pricing change interacts with the recession in ways the company cannot disentangle. The recession is a confounder, but it is not a variable in any of the company's databases.

**The organizational change.** An HR analytics team is studying the effect of a new training program on employee retention. Six months into the study, the company announces a major reorganization. Employees who go through the training disproportionately end up in the units that survived the reorganization, while those who did not go through the training disproportionately end up in the units that were eliminated. The reorganization is a confounder, and it is largely outside the data.

In each case, the confounder is structural, large, and specific to the context. The standard checklist of confounders — age, sex, income, education, prior outcome — would not have caught any of them. They are the confounders that domain experts identify, if anyone does. They are the reason expert input is mathematically necessary, not merely useful.

---

## Collider Bias, Revisited

The other major bias in observational studies is collider bias. We met it in Chapter 4 (the M-graph) and again in Chapter 6 (the smoking and birth weight paradox). Unlike confounding bias, which is introduced by *failing* to control for a variable, collider bias is introduced by *controlling for* a variable we should have left alone.

A collider, recall, is a variable that is the joint effect of two other variables: *X → Z ← Y*. Conditional on the collider Z, X and Y become correlated even if they were independent in the unconditioned data. This is the explain-away effect: knowing Z provides information about both X and Y, and knowing one tells us about the other through the lens of Z.

Collider bias is easy to introduce in practice. The default advice in statistics — *control for everything you can measure* — actively promotes it. Any time we condition on a variable that is downstream of both treatment and outcome, we are conditioning on a collider, and we are introducing bias. Hospital-based studies condition on hospitalization, a collider for many disease pairs. Subscriber-based studies condition on subscription, a collider for many engagement-outcome pairs. Studies of survivors condition on survival, the most consequential collider of all.

The Birkson paradox we touched on earlier — the apparent negative correlation between two diseases among hospitalized patients, even though the diseases are independent in the population — is a textbook example. The smoking and birth weight paradox is another. The collider in those cases is hospitalization or birth weight, and conditioning on it produces a correlation that is not a causal relationship.

The discipline against collider bias is the same as the discipline against confounder bias: the causal diagram. The diagram tells us which variables are colliders. The backdoor criterion — properly applied — tells us which variables to condition on and which to leave alone. The naive practice of "control for everything" violates this discipline and introduces bias whenever the controlled-for set includes a collider.

---

## Sensitivity Analysis: The Honest Response

If we cannot test the unconfoundedness assumption from data, what can we do? The answer is sensitivity analysis. We accept that an unmeasured confounder might exist; we ask how strong it would have to be to alter our conclusions; and we report the answer.

The classical example, due to Jerome Cornfield in 1959, addressed the smoking-cancer debate. Cornfield asked: how strong would an unmeasured confounder have to be — how much would it have to increase both the probability of smoking and the probability of cancer — to explain the entire observed association between smoking and cancer? The answer, given the magnitude of the association (smokers had 9 times the cancer risk of non-smokers), was that the confounder would have to increase the probability of smoking by at least 9-fold. No such confounder is biologically plausible. The smoking-cancer association is too large to be plausibly explained by unmeasured confounding.

Cornfield's argument is the prototype of sensitivity analysis. The general procedure: for a given observed effect, compute the strength of unmeasured confounding that would be required to explain the effect away. Compare that strength to plausible benchmarks (the strength of confounders we have measured, the strength of confounders we know about in similar settings). Report the comparison.

Modern sensitivity analysis is more sophisticated than Cornfield's, but the structure is the same. Methods like the E-value (Tyler VanderWeele and colleagues) provide a single number summarizing how strong an unmeasured confounder would have to be to explain away the observed effect. Methods like Rosenbaum's bounds quantify the impact of departures from unconfoundedness on the conclusions of matched observational studies. The general principle: the more confounding required to overturn the result, the more credible the result.

Sensitivity analysis does not prove the absence of unmeasured confounding. It tells us how robust we are to it. Sometimes the result is robust — the observed effect is so large, or the data are so strong, that no plausible confounder could overturn it. Sometimes the result is fragile — modest unmeasured confounding could erase the effect, and the conclusion is therefore tentative. Reporting which is which is part of the discipline of honest causal inference.

---

## What Sensitivity Analysis Is Not

Sensitivity analysis is sometimes confused with bound-tightening, which it is not. The bounds it provides are wide when our knowledge is limited, and they remain wide regardless of how much data we collect. More data can sharpen the *point estimate* but cannot reduce the *bias bound* due to unmeasured confounding. The bias bound is set by structural assumptions, not sample size.

This is one of the deepest pedagogical confusions in applied statistics. The standard error of an estimate decreases with sample size; the bias does not. With infinite data and unmeasured confounding, we converge to the wrong answer with arbitrary precision. The sensitivity analysis reminds us that the precision is not the credibility.

This is why the discipline of causal inference is partly a discipline of communication. The estimate has a confidence interval; the conclusion has a *plausibility range* governed by the sensitivity analysis. The two are different. A 95% confidence interval that does not include zero is not, on its own, evidence of a causal effect. It is evidence of a robust statistical association, conditional on the model being correct. The causal conclusion requires the additional assumption that the model is correct, and the sensitivity analysis tells us how robust the conclusion is to violations of that assumption.

---

## When Observational Inference Should Not Be Trusted

There are settings in which observational inference, even with the best methodology, should be treated with extreme caution.

**Strong selection effects.** When the population in the study is dramatically self-selected — volunteers for a behavioral intervention, users of a particular service, patients of a particular hospital — the selection itself is a source of confounding that is hard to address. The HRT studies suffered from this; the women who took HRT differed systematically from those who did not in ways that confounded the comparison.

**Highly heterogeneous populations.** When the treatment effect varies dramatically across subpopulations and the subpopulation membership is correlated with treatment, the estimated average effect can be misleading. A drug might benefit one subgroup, harm another, and produce a near-zero average effect in observational data even when it has substantial effects in both directions.

**Endogenous treatment selection.** When the treatment is chosen by an actor with information that affects the outcome — a doctor choosing which patient to treat, a customer choosing whether to subscribe — the selection process itself creates structure that is hard to model. This is the classical reason economists are suspicious of observational estimates of policy effects.

**Long causal chains.** When the treatment affects the outcome through many intermediate steps, each of which has its own confounding structure, the cumulative bias can be substantial. The bias does not always go in a predictable direction.

In each of these settings, the appropriate response is not despair but *triangulation*. Combine observational evidence with mechanism (does the proposed effect have a plausible biological or causal mechanism?), with experimental evidence (is there any related intervention that has been studied in a randomized trial?), with cross-context comparison (does the effect appear in different settings, populations, time periods?). The HRT story did not unwind from a single observational study; it unwound from the combination of observational studies, biological mechanism, and the eventual randomized trial. The triangulation, not any one method, is the source of credibility.

---

## A Worked Example of Sensitivity Analysis

Suppose we have an observational study showing that customers who attend a quarterly business review (QBR) renew at a 95% rate, while customers who do not attend renew at 80%. The simple observation suggests QBRs are highly effective. We adjust for measurable covariates — account size, contract length, industry, current health score — and the effect remains: a 10 percentage-point lift in renewal probability associated with QBR attendance.

Is this a causal effect? An unmeasured confounder is a likely concern. Customers who attend QBRs probably differ from customers who don't in ways we have not measured: they may have more engaged executive sponsors, they may have more strategic value to derive from the relationship, they may simply be customers whose business is going well and who are therefore happy to engage. These confounders might explain the entire observed association.

Sensitivity analysis: how strong would the unmeasured confounder have to be to explain a 10 percentage-point effect? Using the E-value framework, we can compute that the confounder would need to be associated with both QBR attendance and renewal at a relative risk of approximately 2 — that is, customers with the confounder would need to be twice as likely to attend QBRs and twice as likely to renew, beyond what is explained by the measured covariates.

Is a confounder that strong plausible? In this case, probably yes. Executive sponsorship, strategic value, account health — these are all things that could plausibly produce a 2x relative risk. The observed effect is therefore *not robust* to plausible unmeasured confounding. The right reporting is: "We observe a 10 percentage-point association between QBR attendance and renewal after adjusting for measured covariates. The association is not robust to plausible unmeasured confounding. To establish a causal effect, we would need either a randomized experiment in QBR scheduling or stronger control over the variables we suspect are confounding."

This is the honest account. It is less satisfying than "QBRs cause a 10 percentage-point lift in renewal." It is more truthful, and it leads to better next steps.

---

## What This Means for the Living Model

A Living Model that produces causal estimates from observational data inherits all the limitations of observational inference. The estimates are conditional on the diagram being correct and on a sufficient set of confounders being measured. The Living Model's job is to make these conditions visible to the decision-maker, to perform sensitivity analysis on the principal estimates, and to report when the estimates are not robust.

This is why the Causal Brain Executive Report (Chapter 19 in Part Three) has an *assumptions* section as a first-class element. The assumptions are not buried in technical appendices; they are presented alongside the recommendation, as part of what the decision-maker needs to evaluate. The robustness of the conclusion to unmeasured confounding is part of what the report communicates.

A Living Model that does not communicate its limits is, in effect, fabricating confidence. The decision-maker, lacking the apparatus to evaluate the assumptions, takes the recommendation on the strength of the model's apparent precision. When the assumptions fail — when an unmeasured confounder turns out to dominate — the model produces a recommendation that looks well-supported and is not. This is the failure mode the discipline of sensitivity analysis is designed to prevent.

---

## Looking Forward

Chapter 11 takes up the gold standard of causal inference: the randomized controlled trial. We will see why randomization works, why it remains the strongest single tool for establishing causation, and why the translation from clinical "treatment" to organizational "intervention" is harder than it looks. We will also see when randomization is infeasible — for ethical, practical, or scientific reasons — and what we can do in those cases. The Stable Unit Treatment Value Assumption (SUTVA), which most introductory accounts of randomized experiments brush past, will turn out to be the most consequential and the most often-violated assumption in field experiments.

---

**What would change my mind:** A robust empirical demonstration that, in some specific class of observational studies, unmeasured confounding can be ruled out by methodological means short of randomization. There are claimants — natural experiments, regression discontinuity designs, instrumental variables — and each has merit in specific settings. None of them, in my reading, fully replaces the randomized experiment as the strongest inferential tool. I would update if shown a class where the replacement is genuinely complete.

**Still puzzling:** How to communicate the difference between statistical precision and causal credibility to non-technical audiences. The decision-maker sees a confidence interval and assumes it bounds the answer. The interval bounds the answer *conditional on the assumptions*; the assumptions themselves carry uncertainty that the interval does not represent. I do not yet have a clean way to communicate this without either confusing the audience or appearing to evade the question.

---

**Tags:** unmeasured confounding, sensitivity analysis Cornfield E-value, hormone replacement therapy reversal, collider bias, limits of observational inference
