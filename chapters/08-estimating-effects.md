# Chapter 8 — Estimating Effects

*From graph structure to quantified causal effects.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *Estimating Effects*
- *From Structure to Number*
- *Why the Average Effect Is Rarely the Decision-Relevant Fact*

**TL;DR.** Once we have a causal diagram, we can use it to determine which variables to adjust for in order to estimate the causal effect of one variable on another. The backdoor criterion identifies a sufficient set of adjusters. Double machine learning extends this to high-dimensional settings where standard regression fails. Causal forests, developed by Susan Athey and Stefan Wager, estimate not just the average effect but how the effect varies across subpopulations — heterogeneous treatment effects. The decision-relevant fact is rarely the average; it is what the effect would be for the specific case at hand.

---

## The Pricing Team's Problem

A pricing team at a B2B SaaS company is considering raising the price of its mid-tier plan from $99/month to $129/month. The CFO wants a number: what will this do to revenue? The analyst pulls the historical data. She has a year of customer-level pricing decisions (some accounts pay list price, some get discounts, some are on legacy plans), customer characteristics (industry, size, seat count), engagement metrics (login frequency, feature adoption, support tickets), and outcomes (renewal, expansion, churn).

She fits a regression: renewal probability as a function of price, controlling for everything else she can think of. The regression returns a coefficient. She presents it to the CFO. The CFO uses it to forecast the revenue impact of the price increase.

This is the standard workflow, and in most cases it is wrong. The regression coefficient is not the causal effect of price on renewal. It is a statistical association, conditional on the other variables in the regression, contaminated by whatever confounding the regression failed to address. Whether the coefficient is close to the causal effect or wildly different from it depends on which variables the analyst included, which she omitted, and which she should have included but is conditioning on incorrectly.

This chapter is about how to do that workflow correctly. The starting point is a causal diagram. The technical apparatus is the backdoor criterion, augmented by modern methods (double machine learning, causal forests) that handle the difficult cases.

---

## The Backdoor Criterion in Operation

We met the backdoor criterion in Chapter 4. The criterion answers a single question: given a causal diagram and a target query — what is the causal effect of X on Y? — which set of variables, if any, is sufficient to adjust for in order to estimate that effect from observational data?

The procedure is graphical:

1. Identify all backdoor paths from X to Y. A backdoor path is any path from X to Y that starts with an arrow pointing *into* X. These are the paths along which spurious correlation can flow.

2. Find a set of variables Z that *blocks* all the backdoor paths — meaning every backdoor path contains at least one variable from Z, and that variable is not a collider on the path (or a descendant of a collider). Z is then a *deconfounding set*.

3. The set Z must not contain any descendants of X. Conditioning on a descendant of X partially or completely closes the causal pathway we are trying to estimate.

If a deconfounding set Z exists, then the causal effect of X on Y can be estimated from the data by adjusting for Z. If no such set exists — if there is a backdoor path that cannot be blocked by any observable set of variables — then the effect cannot be estimated from observational data using this method, and we have to look for other approaches (front-door, instrumental variables, randomized experiment).

The pricing example. Suppose the diagram looks like this. Customer characteristics (industry, size) influence both the price the customer is offered (because the sales team gives discounts to certain industries and sizes) and the customer's renewal probability (because some industries are stickier than others). Engagement metrics are mediators — they are downstream of price and upstream of renewal. The diagram has a backdoor path: price ← customer characteristics → renewal. The backdoor path can be blocked by adjusting for customer characteristics. Engagement should *not* be adjusted for, because it is a mediator on the causal pathway from price to renewal; adjusting for it would partially close the very effect we are trying to estimate.

This diagnosis tells the analyst exactly what to do: regress renewal on price, controlling for customer characteristics, but *not* for engagement. The result is the causal effect, identified up to the model's assumptions.

The analyst's original regression — which controlled for engagement along with everything else — was not the causal effect. It was a partially closed estimate biased toward zero. The CFO's forecast was, accordingly, an underestimate of the price increase's impact on revenue. Whether the impact would have been bad or good is a separate question, but the analyst's number was the wrong number.

---

## Why Standard Regression Is Frequently Biased

The pricing example illustrates a general pattern. Standard regression is *cause-blind*. The coefficient on a predictor depends on what other predictors are in the regression — but the choice of which predictors to include is, in standard practice, made on grounds that have nothing to do with causal structure. Analysts choose predictors based on availability, statistical significance, intuition, or "what other studies controlled for." None of these criteria reliably produces a deconfounding set.

The two most common errors:

**Controlling for a mediator.** As in the pricing example: the analyst controls for engagement, which is downstream of price, and thereby partially closes the causal pathway. The result is a biased estimate of the direct effect of price, which is not the question the CFO asked. The CFO wants the *total* effect, including the indirect effect through engagement.

**Failing to control for a confounder.** The analyst omits a variable that affects both X and Y, leaving a backdoor path open. The result is an estimate biased by the magnitude of the confounding. Classical example: omitting age from a study of medication efficacy. Older patients are more likely to be prescribed the medication and more likely to die from the underlying condition. Without controlling for age, the medication appears harmful when it is actually beneficial.

The backdoor criterion resolves both errors. It tells the analyst *exactly* which variables to control for. The variables to control for are the ones that block all backdoor paths and are not on the causal pathway. The variables to *not* control for are mediators and colliders.

This last category — *do not control for colliders* — is the one most analysts have not internalized. We saw collider bias in Chapter 4 (M-bias). The standard advice to "control for everything you can measure" is not just inefficient; it is actively harmful when one of the things you control for is a collider. Conditioning on a collider opens a path that was previously closed and introduces a new bias. The backdoor criterion identifies this in advance.

---

## Double Machine Learning

The backdoor criterion is the structural answer to the question of which variables to adjust for. The remaining question is how to do the adjustment in practice, particularly when the deconfounding set is high-dimensional. If we have hundreds of customer characteristics, we cannot stratify the data by every combination of them; we run out of data in every stratum. We need a method that can handle the dimensionality.

The method developed by Victor Chernozhukov and colleagues is called *double machine learning* (DML). The idea is to use modern machine learning — random forests, gradient boosting, neural networks — to model the relationships between the deconfounders and both the treatment and the outcome, then use the residuals to estimate the causal effect.

The procedure, in outline:

1. Use a machine learning method to predict the treatment X from the deconfounders Z. Compute the residuals: *X − E[X | Z]*. These are the parts of X not explained by the deconfounders — the "exogenous" variation in X.

2. Use a machine learning method to predict the outcome Y from the deconfounders Z. Compute the residuals: *Y − E[Y | Z]*. These are the parts of Y not explained by the deconfounders.

3. Regress the residuals from step 2 on the residuals from step 1. The coefficient is the causal effect of X on Y.

The method has two important properties. First, it allows arbitrarily flexible machine learning models for the relationship between Z and the treatment and outcome. The structural assumption — that Z is a sufficient deconfounding set — comes from the causal diagram, not from the regression model. Second, the method is *orthogonal* to small errors in the machine learning models. If the model for E[X | Z] is slightly off, the resulting bias in the causal effect estimate is second-order. The method tolerates the kinds of errors machine learning typically makes, in a way that ordinary regression does not.

Double machine learning is one of the workhorses of modern causal inference. It enables credible causal estimation in high-dimensional settings where ordinary regression fails. It is, however, only as good as the deconfounding set it adjusts for. If the diagram is wrong — if there is an unmeasured confounder — DML will produce a precise estimate of the wrong quantity, with the same false confidence as any other method.

---

## Why the Average Effect Is Rarely the Answer

The backdoor criterion and double machine learning, when applied correctly, give us the *average causal effect* of X on Y across the population. This is often what the textbook says we should want. It is rarely what the decision actually requires.

Consider the pricing question again. The CFO wants to know: if we raise the price from $99 to $129, what happens to revenue? The average effect tells him the expected change averaged across all customers. But the customers are not all the same. Some will absorb the price increase without complaint. Some will churn. Some will downgrade. Some will use the price increase as a trigger to consolidate their software stack and may churn even if the price had not gone up. The average effect averages all these responses. It tells the CFO what to expect *on average*. It does not tell him which customers are at risk.

A more useful answer would tell the CFO not just the average but the *distribution* — which customers respond positively, which negatively, and what characterizes each group. This is the *heterogeneous treatment effect* (HTE), and estimating it is the subject of a rapidly developing literature.

The most prominent method is *causal forests*, developed by Susan Athey and Stefan Wager (Athey is at Stanford, Wager at Stanford; both have been central figures in the causal inference revival in economics). Causal forests adapt the machinery of random forests — an ensemble method from machine learning — to estimate not the average effect but the conditional average treatment effect (CATE): the expected treatment effect for a customer with specific characteristics x. The method partitions the population into subgroups defined by the deconfounders, estimates the treatment effect within each subgroup, and aggregates the estimates with proper accounting for sample size and variance.

The output of a causal forest is, for each customer, a personalized estimate of how the treatment would affect that customer. For the CFO, this means: instead of a single number for "the effect of the price increase," he receives a distribution. He can see which customers are most at risk of churning, segment his rollout strategy accordingly, perhaps offer grandfathered pricing to the most price-sensitive segment.

This is what most consequential business decisions actually require. The decision is not "should we change the price for everyone?" The decision is "for which customers should we change the price, and how should we communicate the change?" The heterogeneous treatment effect, properly estimated, supports this decision in a way the average effect cannot.

---

## A Worked Comparison

Imagine two customer segments: a price-insensitive enterprise segment, where a $30/month price increase has effectively zero impact on renewal, and a price-sensitive small-business segment, where a $30/month price increase reduces renewal probability by 15 percentage points. Suppose enterprise accounts are 30% of the customer base and small businesses are 70%.

The average causal effect of the price increase on renewal is:

> (0.30 × 0%) + (0.70 × −15%) = −10.5%

Interpreted as an aggregate, this looks bad. The CFO might decide against the price increase because of the 10.5 percentage point drop in renewal.

The heterogeneous causal effect tells a different story. The enterprise segment can be raised to $129 with no churn impact and immediate revenue gain. The small business segment cannot. The right rollout is to raise prices for enterprise customers and either grandfather the small businesses, offer them an alternative tier, or accept the renewal impact in exchange for the revenue gain — but to make that trade-off explicitly, on a segment-by-segment basis, rather than averaging it away.

The average effect would have led to a worse decision than the heterogeneous effect. This is the rule, not the exception. Average effects are useful for population-level reasoning (what will overall revenue do?) but they are not what most decisions require. Decisions require knowing which subpopulations respond which way, which is exactly what the HTE provides.

---

## The Targeted Maximum Likelihood Estimator and Other Modern Methods

I have focused on the backdoor criterion, double machine learning, and causal forests because they are the most widely used. The literature contains many other methods worth knowing about, each with its own strengths.

**Targeted Maximum Likelihood Estimation (TMLE)**, developed by Mark van der Laan and colleagues, combines machine learning with a targeted updating step that achieves the same kind of orthogonality DML does. It is the basis of much of the modern epidemiological literature.

**Inverse Probability Weighting (IPW)** reweights the sample by the inverse of the propensity score (the probability of receiving the treatment given the deconfounders). The method has a long history, going back to Horvitz and Thompson in the 1950s, but it has been refined and combined with machine learning in recent decades.

**Matching methods** (propensity score matching, exact matching, coarsened exact matching) construct a pseudo-randomized experiment by matching treated units to controls with similar deconfounders. The method has the virtue of transparency — the analyst can examine the matched pairs and verify that they look comparable. It has the drawback of throwing away unmatched data, which can be inefficient.

**G-methods**, developed by Jamie Robbins, handle time-varying treatments and time-varying confounders — the kind of structure that arises in long-term medical treatment, sequential business decisions, and longitudinal studies. We will see G-methods more in Chapter 11.

The choice among these methods is partly stylistic, partly determined by the specifics of the problem (linear vs. nonlinear, high-dimensional vs. low-dimensional, time-varying vs. cross-sectional). The structural commitment — the causal diagram and the backdoor criterion identifying the deconfounding set — is the same across all of them. The methods differ in how they perform the adjustment, not in what they are adjusting for.

---

## When the Effect Is Not Identifiable

Sometimes the diagram does not allow the backdoor criterion to be satisfied. There is a backdoor path that cannot be blocked by any set of observable variables. In that case, the effect is not identifiable from observational data using the backdoor approach.

Three responses are possible.

**Look for other identification strategies.** The front-door criterion (Chapter 7 of Part Three discusses this in detail in Pearl's framework) sometimes works when the backdoor does not. Instrumental variables — covered in Chapter 11 — sometimes work when neither does.

**Conduct a randomized experiment.** A randomized intervention on X, by definition, severs the backdoor paths. If the experiment is feasible and ethical, it produces unbiased estimates regardless of the deconfounding structure.

**Conduct sensitivity analysis.** If neither of the above is possible, the analyst can report bounds on the effect under varying assumptions about the unmeasured confounder. If the effect is robust to plausible amounts of confounding, the conclusion is supported. If the effect could plausibly disappear under modest unmeasured confounding, the conclusion is not supported. The discipline of sensitivity analysis — making the dependence on assumptions visible — is what distinguishes credible causal inference from speculation.

---

## Looking Forward

Chapter 9 takes us to the third rung of the ladder — counterfactuals. The methods we have just discussed all operate on Rung 2: they estimate the average or heterogeneous causal effect of an intervention. They do not, however, answer the question of what would have happened in a specific case if a different decision had been made. That is a Rung 3 question, and it requires the structural causal model in full force, with the abduction-action-prediction procedure that lets us reason about worlds that did not happen.

---

**What would change my mind:** A class of practical problems where standard regression, applied without causal reasoning, reliably produces causal estimates as good as those from properly identified methods. There are stylized cases — randomized data, no confounding — where this happens trivially. I do not know of practical observational cases where it does. I would update if shown one.

**Still puzzling:** The relationship between estimation precision and structural assumptions. Methods like causal forests give very precise estimates of heterogeneous effects, but the precision is contingent on the deconfounding assumption being correct. A tighter confidence interval is not necessarily a more credible estimate; it is a more confident one, conditional on assumptions that may or may not hold. Communicating this distinction to decision-makers is hard.

---

**Tags:** backdoor criterion estimation, double machine learning, causal forests Athey Wager, heterogeneous treatment effects, why average effect is wrong question
