# Chapter 8 — Estimating Effects

*From graph structure to quantified causal effects.*

---

A pricing team at a B2B SaaS company is considering raising the price of its mid-tier plan from \$99 per month to \$129 per month. The CFO wants a number: what will this do to revenue? The analyst pulls the historical data. She has a year of customer-level pricing decisions — some accounts pay list price, some get discounts, some are on legacy plans — customer characteristics like industry and company size, engagement metrics like login frequency and feature adoption, and outcomes like renewal, expansion, and churn.

She fits a regression: renewal probability as a function of price, controlling for everything else she can think of. The regression returns a coefficient. She presents it to the CFO. The CFO uses it to forecast the revenue impact of the price increase.

This is the standard workflow, and in most cases it is wrong. The regression coefficient is not the causal effect of price on renewal. It is a statistical association, conditional on the other variables in the regression, contaminated by whatever confounding the regression failed to address. Whether the coefficient is close to the causal effect or wildly different from it depends on which variables the analyst included, which she omitted, and which she should have included but is conditioning on incorrectly.

This chapter is about how to do that workflow correctly. The starting point is a causal diagram. The technical apparatus is the backdoor criterion, augmented by modern methods — double machine learning and causal forests — that handle the difficult cases the criterion identifies.

---

## The Backdoor Criterion in Operation

We met the backdoor criterion in Chapter 4. The criterion answers a single question: given a causal diagram and a target query — what is the causal effect of $X$ on $Y$? — which set of variables, if any, is sufficient to adjust for in order to estimate that effect from observational data?

The procedure is graphical. First, identify all backdoor paths from $X$ to $Y$. A backdoor path is any path from $X$ to $Y$ that starts with an arrow pointing *into* $X$ — these are the paths along which spurious correlation can flow. Second, find a set of variables $Z$ that blocks all the backdoor paths: every backdoor path must contain at least one variable from $Z$, and that variable must not be a collider on the path or a descendant of a collider. $Z$ is then a deconfounding set. Third, the set $Z$ must not contain any descendants of $X$; conditioning on a descendant of $X$ partially or completely closes the causal pathway we are trying to estimate.

If a deconfounding set $Z$ exists, the causal effect of $X$ on $Y$ can be estimated from the data by adjusting for $Z$. If no such set exists — if there is a backdoor path that cannot be blocked by any observable variables — then the effect cannot be estimated from observational data using this method, and we have to look for other approaches: the front-door criterion, instrumental variables, or a randomized experiment.

Now apply this to the pricing problem. Suppose the diagram has the following structure: customer characteristics — industry, company size — influence both the price the customer is offered (because the sales team gives discounts to certain industries and sizes) and the customer's renewal probability (because some industries are stickier than others). Engagement metrics are mediators — they are downstream of price and upstream of renewal. The diagram has a backdoor path: price ← customer characteristics → renewal. This path can be blocked by adjusting for customer characteristics. Engagement should *not* be adjusted for, because it is a mediator on the causal pathway from price to renewal; adjusting for it would partially close the very effect we are trying to estimate.

<!-- → [DIAGRAM: Pricing causal diagram — nodes: Customer Characteristics (C), Price (P), Engagement (E), Renewal (R); arrows: C → P, C → R, P → E, E → R, P → R (direct); a red-highlighted path showing the backdoor path C → P and C → R; a label "block this" pointing to C with a box around it; a label "do NOT condition on this" pointing to E; caption: "The backdoor criterion identifies what to adjust for and what to leave alone"] -->

![Figure 8.1 — Pricing causal diagram](images/08-estimating-effects-fig-01.jpg)


This diagnosis tells the analyst exactly what to do: regress renewal on price, controlling for customer characteristics, but *not* for engagement. The result is the causal effect of price on renewal, identified up to the model's assumptions.

The analyst's original regression — which controlled for engagement along with everything else — was not the causal effect. It was a partially closed estimate biased toward zero, because conditioning on a mediator blocks some of the causal path. The CFO's forecast was accordingly an underestimate. Whether the actual impact would have been good or bad is a separate question, but the analyst's number was the wrong number.

---

## Why Standard Regression Is Frequently Biased

The pricing example illustrates a general pattern. Standard regression is cause-blind. The coefficient on a predictor depends on what other predictors are in the regression — but the choice of which predictors to include is, in standard practice, made on grounds that have nothing to do with causal structure. Analysts choose predictors based on availability, statistical significance, intuition, or "what other studies controlled for." None of these criteria reliably produces a deconfounding set.

The two most common errors follow a predictable structure.

**Controlling for a mediator** is the first. As in the pricing example: the analyst controls for engagement, which is downstream of price, and thereby partially closes the causal pathway. The result is a biased estimate of the direct effect of price, which is not the question the CFO asked. The CFO wants the *total* effect of price — the effect operating through engagement included. Controlling for engagement strips out exactly that indirect path.

**Failing to control for a confounder** is the second. The analyst omits a variable that affects both $X$ and $Y$, leaving a backdoor path open. The result is an estimate biased by the magnitude of the confounding. The classical example: omitting age from a study of medication efficacy. Older patients are more likely to be prescribed the medication and more likely to die from the underlying condition. Without controlling for age, the medication appears harmful when it is actually beneficial.

The backdoor criterion resolves both errors simultaneously. It identifies which variables to control for and which to leave alone. Variables to control for are those that block all backdoor paths and are not on the causal pathway. Variables to *not* control for are mediators and colliders. The last category — do not control for colliders — is the one most analysts have not internalized. Conditioning on a collider opens a path that was previously closed and introduces a new bias. The criterion identifies this in advance.

| Error | Variable's causal role | What the analyst usually believes they are doing | Direction of resulting bias | What the backdoor criterion says instead |
|---|---|---|---|---|
| **Controlling for a mediator** | $M$ lies on the causal path $X \to M \to Y$ | "I'm controlling for everything that varies with $X$" — assumed to make the estimate cleaner | Toward zero — the indirect effect is partialled out | Mediators must *not* be conditioned on when estimating the total effect; condition on them only when isolating direct effects |
| **Omitting a confounder** | $Z$ is a common cause of $X$ and $Y$ | "I'm running the simplest model that fits" | Direction depends on the sign of the confounding; magnitude can dwarf the true effect | The backdoor criterion identifies a sufficient $Z$; estimate is identifiable only if all backdoor paths are blocked |
| **Controlling for a collider** | $C$ is caused by both $X$ and $Y$ (or their descendants) | "I'm adjusting for as much as possible" — adding more controls assumed to be safer | Opens a backdoor path that was previously closed; *introduces* spurious association | Colliders must *not* be conditioned on; doing so makes the estimate worse, not better |

*Figure 8.2*

| | **Property** | **Value** |
|---|---|---|
| **Controlling for a Mediator** | _fill in_ | _fill in_ |
| **Omitting a Confounder** | _fill in_ | _fill in_ |
| **Controlling for a Collider** | _fill in_ | _fill in_ |

: {.comparison-table}


---

## Double Machine Learning

The backdoor criterion is the structural answer to the question of which variables to adjust for. The remaining question is how to do the adjustment when the deconfounding set is high-dimensional. If there are hundreds of customer characteristics, we cannot stratify the data by every combination; we run out of data in every stratum. We need a method that can handle the dimensionality without sacrificing credibility.

The method developed by Victor Chernozhukov and colleagues is called double machine learning (DML). The idea is to use modern machine learning — random forests, gradient boosting, neural networks — to model the relationships between the deconfounders and both the treatment and the outcome, then use the residuals to estimate the causal effect.

The procedure works in three steps. First, use a machine learning model to predict the treatment $X$ from the deconfounders $Z$. Compute the residuals: $\tilde{X} = X - \hat{E}[X \mid Z]$. These are the parts of $X$ not explained by the deconfounders — the variation in $X$ that is effectively exogenous. Second, use a machine learning model to predict the outcome $Y$ from the deconfounders $Z$. Compute the residuals: $\tilde{Y} = Y - \hat{E}[Y \mid Z]$. Third, regress $\tilde{Y}$ on $\tilde{X}$. The coefficient is the estimated causal effect of $X$ on $Y$.

The method has two important properties. First, it allows arbitrarily flexible machine learning models for the relationship between $Z$ and the treatment and outcome; the structural assumption — that $Z$ is a sufficient deconfounding set — comes from the causal diagram, not from the regression model. Second, the method is *orthogonal* to small errors in the machine learning models. If the model for $\hat{E}[X \mid Z]$ is slightly off, the resulting bias in the causal effect estimate is second-order — small relative to the error in the model. The method tolerates the kinds of imprecision machine learning typically produces, in a way that ordinary regression does not.

Double machine learning is one of the workhorses of modern causal inference in practice. It enables credible causal estimation in high-dimensional settings where ordinary regression fails. It is, however, only as good as the deconfounding set it adjusts for. If the diagram is wrong — if there is an unmeasured confounder — DML will produce a precise estimate of the wrong quantity, with the same false confidence as any other method.

<!-- → [DIAGRAM: DML procedure as a three-step flow — left box: "Regress X on Z using ML → residual X̃"; middle box: "Regress Y on Z using ML → residual Ỹ"; right box: "Regress Ỹ on X̃ → causal effect τ̂"; arrows between boxes; below the flow, a note: "Z is determined by the causal diagram (backdoor criterion), not by the ML step"; caption: "DML separates the structural question (which Z to adjust for) from the statistical question (how to adjust for high-dimensional Z)"] -->

![Figure 8.3 — DML procedure as a three-step flow](images/08-estimating-effects-fig-03.jpg)


---

## Why the Average Effect Is Rarely the Answer

The backdoor criterion and double machine learning, when applied correctly, give the *average causal effect* of $X$ on $Y$ across the population. This is often what the textbook says we should want. It is rarely what the decision actually requires.

Return to the pricing question. The CFO wants to know: if we raise the price from \$99 to \$129, what happens to revenue? The average effect tells him the expected change averaged across all customers. But the customers are not all the same. Some will absorb the price increase without complaint. Some will churn. Some will downgrade. Some will use the price increase as a trigger to consolidate their software stack and churn even if the price had not gone up. The average effect averages all these responses — it tells the CFO what to expect on average, but it does not tell him which customers are at risk.

A more useful answer would tell the CFO not just the average but the distribution: which customers respond positively, which negatively, and what characterizes each group. This is the heterogeneous treatment effect (HTE), and estimating it is the subject of a rapidly developing literature.

The most prominent method is *causal forests*, developed by Susan Athey and Stefan Wager at Stanford. Causal forests adapt the machinery of random forests — an ensemble method from machine learning — to estimate not the average effect but the conditional average treatment effect (CATE): the expected treatment effect for a customer with specific observed characteristics $x$. The method partitions the population into subgroups defined by the deconfounders, estimates the treatment effect within each subgroup, and aggregates the estimates with proper accounting for sample size and variance. The result is, for each customer, a personalized estimate of how the treatment would affect that customer.

For the CFO, this means: instead of a single number for "the effect of the price increase," he receives a distribution. He can see which customers are most at risk of churning, segment the rollout strategy accordingly, perhaps offer grandfathered pricing to the most price-sensitive segment. This is what most consequential business decisions actually require. The decision is not "should we change the price for everyone?" It is "for which customers should we change the price, and how should we communicate the change?" The heterogeneous treatment effect supports this decision in a way the average effect cannot.

---

## A Worked Comparison

Suppose two customer segments: a price-insensitive enterprise segment, where a \$30 per month price increase has effectively zero impact on renewal, and a price-sensitive small-business segment, where a \$30 per month price increase reduces renewal probability by 15 percentage points. Suppose enterprise accounts are 30% of the customer base and small businesses are 70%.

The average causal effect of the price increase on renewal is:

$$(0.30 \times 0\%) + (0.70 \times -15\%) = -10.5\%$$

Interpreted as an aggregate, this looks bad. The CFO might decide against the price increase because of the 10.5 percentage point drop in renewal.

The heterogeneous causal effect tells a different story. The enterprise segment can be raised to \$129 with no churn impact and immediate revenue gain. The small-business segment cannot. The right rollout is to raise prices for enterprise customers and either grandfather the small businesses, offer them an alternative tier, or accept the renewal impact in exchange for the revenue gain — but to make that trade-off explicitly, on a segment-by-segment basis, rather than averaging it away into a single number that obscures the decision.

The average effect would have led to a worse decision than the heterogeneous effect. This is the rule, not the exception. Average effects are useful for population-level reasoning — what will overall renewal rates do? — but they are not what most decisions require. Decisions require knowing which subpopulations respond which way, which is exactly what the HTE provides.

<!-- → [CHART: Heterogeneous treatment effect — a distribution plot or two-panel bar chart; left panel shows the average effect (−10.5%) as a single bar; right panel shows the CATE by segment: enterprise at 0%, small business at −15%, with segment sizes shown; a third element showing the resulting decision: "raise for enterprise, grandfather small business"; caption: "The average hides the segmentation that makes the decision tractable"] -->

![Figure 8.4 — Heterogeneous treatment effect](images/08-estimating-effects-fig-04.jpg)


---

## Other Methods Worth Knowing

The backdoor criterion, double machine learning, and causal forests cover most practical cases in cross-sectional observational data. The literature contains other methods each with strengths the above lack.

Targeted Maximum Likelihood Estimation (TMLE), developed by Mark van der Laan and colleagues, combines machine learning with a targeted updating step that achieves the same orthogonality property that DML does. It is the basis of much of the modern epidemiological literature on causal effect estimation and is particularly useful when the outcome model is more complex than a standard regression.

Inverse Probability Weighting reweights the sample by the inverse of the propensity score — the probability of receiving the treatment given the deconfounders. The method rebalances the treatment and control groups so that their deconfounder distributions match, mimicking randomization. It has a long history, refined and combined with machine learning over the past two decades.

Matching methods — propensity score matching, exact matching, coarsened exact matching — construct a pseudo-randomized experiment by pairing each treated unit with a control unit that has similar deconfounders. The method has the virtue of transparency: the analyst can examine the matched pairs and verify they look comparable. It has the drawback of discarding unmatched data, which can be inefficient when the population is large and heterogeneous.

G-methods, developed by Jamie Robbins, handle time-varying treatments and time-varying confounders — the kind of structure that arises in long-term medical treatment, sequential business decisions, and longitudinal studies. We will encounter G-methods again in Chapter 11.

The choice among these methods is partly stylistic and partly determined by the specifics of the problem: linear versus nonlinear, high-dimensional versus low-dimensional, time-varying versus cross-sectional. What is the same across all of them is the structural commitment — the causal diagram and the backdoor criterion identifying the deconfounding set. The methods differ in how they perform the adjustment, not in what they are adjusting for.

---

## When the Effect Is Not Identifiable

Sometimes the diagram does not allow the backdoor criterion to be satisfied. There is a backdoor path that cannot be blocked by any set of observable variables. In that case, the effect is not identifiable from observational data using this approach.

Three responses are available. The first is to look for other identification strategies. The front-door criterion sometimes works when the backdoor does not; instrumental variables, covered in Chapter 11, sometimes work when neither does. The second is to conduct a randomized experiment. A randomized intervention on $X$ severs the backdoor paths by design. If the experiment is feasible and ethical, it produces unbiased estimates regardless of the deconfounding structure. The third is to conduct sensitivity analysis. If neither of the above is possible, the analyst can report bounds on the effect under varying assumptions about the unmeasured confounder. If the effect is robust to plausible amounts of confounding, the conclusion is supported. If the effect could plausibly disappear under modest unmeasured confounding, the conclusion is not supported. Making the dependence on assumptions visible is what distinguishes credible causal inference from speculation.

---

## Looking Forward

Chapter 9 takes us to the third rung of the Ladder — counterfactuals. The methods in this chapter operate on Rung 2: they estimate the average or heterogeneous causal effect of an intervention on a population or subpopulation. They do not answer the question of what would have happened in a specific case if a different decision had been made. That is a Rung 3 question, and it requires the structural causal model in full force — the abduction-action-prediction procedure that lets us reason about worlds that did not happen, carrying forward the case-specific idiosyncrasies that population averages erase.

---

## Student Activities

**Problem 8.1 — Backdoor Criterion Applied.** A health insurance company is studying the causal effect of a wellness program ($W$) on healthcare costs ($C$). Their causal diagram includes the following variables: age ($A$), pre-existing conditions ($P$), wellness program enrollment ($W$), exercise frequency ($E$), and annual healthcare costs ($C$). The causal relationships are: $A \rightarrow P$, $A \rightarrow W$, $P \rightarrow W$, $A \rightarrow C$, $P \rightarrow C$, $W \rightarrow E$, $E \rightarrow C$, and $W \rightarrow C$ (direct). (a) Identify all backdoor paths from $W$ to $C$. (b) Identify the minimal sufficient deconfounding set using the backdoor criterion. (c) Explain why exercise frequency ($E$) should or should not be included in the adjustment set, and what would happen to the estimate if it were included. (d) An analyst argues that including $P$ in the regression "inflates" the effect of the wellness program by making high-risk patients look more similar to low-risk patients. Evaluate this argument using the backdoor criterion.

**Problem 8.2 — Mediator vs. Confounder Identification.** In each of the following scenarios, determine whether the proposed control variable is a confounder, mediator, or collider, and state whether it should be included in the adjustment set to estimate the total causal effect of $X$ on $Y$. (a) Estimating the effect of job training ($X$) on wages ($Y$), controlling for employment status ($Z$), where job training causes employment which causes wages. (b) Estimating the effect of a social media campaign ($X$) on brand awareness ($Y$), controlling for click-through rate ($Z$), where the campaign causes clicks which causes awareness. (c) Estimating the effect of air pollution ($X$) on hospital admissions ($Y$), controlling for whether a patient has respiratory disease ($Z$), where both pollution and an unmeasured genetic factor cause respiratory disease and hospitalization. (d) Estimating the effect of education ($X$) on income ($Y$), controlling for parental income ($Z$), where parental income affects both educational attainment and the child's eventual income.

**Problem 8.3 — Double Machine Learning Setup.** A marketing analyst wants to estimate the causal effect of email campaign exposure ($X$) on purchase probability ($Y$). The deconfounding set identified from the causal diagram is: customer tenure ($T$), past purchase frequency ($F$), and geographic region ($G$). The analyst has 200,000 customer-level observations. (a) Describe in concrete steps how to implement the DML procedure for this problem, specifying what machine learning models you would fit, what residuals you would compute, and what regression you would run. (b) Explain why the DML approach would outperform a simple OLS regression in this setting, given the deconfounders $\{T, F, G\}$. (c) The analyst discovers that customers in the email campaign were also exposed to simultaneous display advertising, and display advertising exposure is not in the dataset. Using the concept of unmeasured confounding, explain what this means for the DML estimate and whether it can be corrected without additional data. (d) The DML estimate is $\hat{\tau} = 0.08$ (an 8 percentage point lift in purchase probability) with a 95% confidence interval of $[0.04, 0.12]$. The analyst presents this as "a precisely estimated 8% lift." Critique this framing with reference to the structural assumptions it depends on.

**Problem 8.4 — Heterogeneous Effects and Decision-Making.** A retail bank has estimated the average causal effect of a proactive credit limit increase on customer spending: $\hat{\tau}_{ATE} = \$240$ per month on average. CATE analysis using causal forests reveals three segments: Segment A (40% of customers) — CATE of \$580/month; Segment B (35% of customers) — CATE of \$0/month (unresponsive); Segment C (25% of customers) — CATE of −\$150/month (spend decreases, likely due to stress response to increased credit). (a) Verify that the CATE estimates are consistent with the reported ATE. (b) The bank is deciding whether to roll out the credit limit increase to all customers. Using only the ATE, what would the bank conclude? Using the CATE, what would a better decision look like? (c) Quantify the revenue difference between the ATE-driven decision and the CATE-driven decision, assuming the bank has 500,000 eligible customers and the credit limit increase costs \$5/customer to implement. (d) What additional information about Segments A, B, and C would allow the bank to identify which customers belong to which segment before implementing the policy?

**Problem 8.5 — Sensitivity Analysis.** A researcher estimates the causal effect of a mentorship program on employee promotion rates using the backdoor criterion with a deconfounding set of $\{$seniority, department, manager rating$\}$. The estimate is that mentorship increases promotion probability by 12 percentage points. A reviewer argues that "motivation" — an unmeasured variable that causes both mentorship program participation and promotion — could explain some or all of the observed effect. (a) Draw the causal diagram including the unmeasured confounder, labeling the backdoor path that is now open. (b) Describe the direction of the bias introduced by the unmeasured confounder: does it likely inflate or deflate the estimated treatment effect? Justify your answer. (c) The researcher conducts a sensitivity analysis and finds that the estimated effect remains positive unless the unmeasured confounder has a partial $R^2$ with both treatment and outcome exceeding 0.25. Explain in plain language what this sensitivity result means and how a decision-maker should interpret it. (d) Propose one study design — using data the company might plausibly collect — that would allow the researcher to bound or eliminate the confounding due to motivation.

**Problem 8.6 — End-to-End Estimation Design.** A nonprofit organization runs a job training program and wants to estimate its causal effect on six-month employment rates. The program is voluntary and self-selected. The organization has administrative data on all participants and a comparison group of eligible non-participants, with the following variables: age, prior education level, prior employment history, geographic area, reason for applying (if applicable), program enrollment, six-month employment status. (a) Draw a plausible causal diagram for this setting. (b) Apply the backdoor criterion to identify the minimal deconfounding set. (c) Evaluate whether DML or a matching method would be more appropriate for this setting and why. (d) The program director wants to know not just whether the program works on average, but which participants benefit most. Describe how you would apply causal forests to answer this question, what the output would look like, and how the director could use the output to improve program targeting. (e) An auditor argues that the six-month employment outcome may itself be a collider if it was used to select participants into a second-stage intervention that also affected long-term employment. Evaluate this concern and propose how you would address it in the analysis.

---

## Key Terms

**Average Causal Effect (ACE).** The expected causal effect of an intervention $X$ on outcome $Y$, averaged across the full population: $E[Y(x') - Y(x)]$. The ACE is the quantity estimated by the backdoor criterion combined with standard regression or DML. It is often not the quantity that a decision requires, because individual units respond heterogeneously and the decision concerns specific subpopulations.

**Backdoor Criterion.** A graphical criterion for identifying a sufficient deconfounding set, given a causal diagram. A set $Z$ satisfies the backdoor criterion for the pair $(X, Y)$ if: (1) $Z$ blocks all backdoor paths from $X$ to $Y$, and (2) $Z$ contains no descendants of $X$. If such a $Z$ exists and its variables are observed, the causal effect of $X$ on $Y$ is identifiable from observational data by adjusting for $Z$.

**Backdoor Path.** A path from $X$ to $Y$ in a causal diagram that starts with an arrow pointing *into* $X$. Backdoor paths represent channels through which spurious correlation can flow from $X$-causes to $Y$. The backdoor criterion aims to block all such paths by conditioning on an appropriate set of variables.

**Causal Forest.** An extension of the random forest algorithm, developed by Athey and Wager, that estimates the conditional average treatment effect (CATE) rather than the average treatment effect. Causal forests produce a personalized treatment effect estimate for each unit defined by its covariates. They satisfy the same orthogonality properties as double machine learning.

**Conditional Average Treatment Effect (CATE).** The expected causal effect of treatment $X$ on outcome $Y$ for a subpopulation defined by observed characteristics $x$: $E[Y(1) - Y(0) \mid X = x]$. The CATE is the quantity estimated by causal forests. It is the decision-relevant fact when the treatment can be targeted at specific subpopulations.

**Deconfounding Set.** A set of variables $Z$ that, when conditioned on, removes all spurious association between treatment $X$ and outcome $Y$ flowing through backdoor paths. A deconfounding set allows the causal effect to be estimated from observational data. Identified by the backdoor criterion.

**Double Machine Learning (DML).** A method for estimating causal effects with high-dimensional deconfounders, developed by Chernozhukov and colleagues. The procedure uses machine learning to partial out the influence of the deconfounders on both treatment and outcome, then regresses the outcome residual on the treatment residual. The resulting estimator is orthogonal to small errors in the machine learning models.

**Heterogeneous Treatment Effect (HTE).** The variation in the causal effect of treatment across units or subpopulations. When HTEs are present, the average causal effect is an incomplete description of the intervention's impact. Estimating HTEs requires methods like causal forests that can produce unit- or subgroup-level effect estimates.

**Instrumental Variable (IV).** A variable $Z$ that (1) causes variation in treatment $X$, (2) affects outcome $Y$ only through $X$, and (3) is independent of all unmeasured confounders. IV methods can estimate causal effects even when no sufficient deconfounding set exists in the observational data. Covered in Chapter 11.

**Orthogonality (in DML).** The property that the causal effect estimate produced by double machine learning is not strongly affected by small errors in the machine learning models for the treatment and outcome conditional expectations. Orthogonality means the method is robust to the kind of imprecision that machine learning typically introduces.

**Propensity Score.** The probability of receiving treatment $X = 1$ given observed covariates $Z$: $P(X = 1 \mid Z)$. The propensity score is a one-dimensional summary of the deconfounders. Conditioning on the propensity score removes confounding by $Z$ under the same assumptions as conditioning on $Z$ directly. Used in inverse probability weighting and propensity score matching.

**Sensitivity Analysis (causal).** A formal analysis of how much an unmeasured confounder would have to affect both treatment and outcome in order to change the estimated causal effect's sign or render it insignificant. Sensitivity analysis reports the assumption under which the conclusion stands, making the dependence on unverifiable assumptions visible.

---

**What would change my mind:** A class of practical problems where standard regression, applied without causal reasoning, reliably produces causal estimates as good as those from properly identified methods. There are stylized cases — randomized data, no confounding — where this happens trivially. I do not know of practical observational cases where it does. I would update if shown one.

**Still puzzling:** The relationship between estimation precision and structural assumptions. Methods like causal forests give very precise estimates of heterogeneous effects, but the precision is contingent on the deconfounding assumption being correct. A tighter confidence interval is not necessarily a more credible estimate; it is a more confident one, conditional on assumptions that may or may not hold. Communicating this distinction to decision-makers is hard.

---

**Tags:** backdoor criterion estimation, double machine learning, causal forests Athey Wager, heterogeneous treatment effects, why average effect is wrong question

---

###  LLM Exercise — Chapter 8: Estimating Effects

**Project:** Build Your Own Living Model

**What you're building this chapter:** A working estimation pipeline — a Python script that takes your DAG, applies the backdoor criterion to identify the deconfounding set, runs double machine learning for the average treatment effect (ATE), and runs a causal forest for heterogeneous effects (CATE) by segment. If you don't have data, you'll generate plausible synthetic data from your SCM and run the pipeline against that.

**Tool:** Claude Code — this is the first chapter that produces runnable code.

---

**The Prompt:**

```
I am working through Chapter 8 of "Living Models." I have a DAG for my decision (variables, edges, structural equations) saved in this Project.

Chapter 8 teaches that to estimate the causal effect of X on Y from observational data, I need to:
1. Identify all BACKDOOR PATHS from X to Y (paths starting with an arrow into X) — these carry spurious correlation.
2. Find an ADJUSTMENT SET Z that blocks all backdoor paths, where Z must not contain colliders or descendants of X.
3. Estimate E[Y | do(X), Z] using a method that controls for Z without imposing strong functional form assumptions. Double Machine Learning (DML, Chernozhukov et al.) is the modern default. Causal Forests (Athey-Wager) extend this to estimate heterogeneous effects (CATE) by segment.

Build a Python script that does the following, using my DAG variables (replace placeholders):

PHASE 1 — IDENTIFICATION:
- Take my DAG (I'll provide it inline or as a networkx definition).
- Identify all backdoor paths from my candidate intervention X to outcome Y.
- Find a minimal adjustment set Z that satisfies the backdoor criterion.
- Print the path enumeration and the chosen Z, with justification.

PHASE 2 — ESTIMATION:
- If I have real data: load it from a CSV [GIVE PATH OR SAY "GENERATE SYNTHETIC"].
- If synthetic: generate ~10,000 rows from my SCM with realistic noise. Use the structural equations from my Project context.
- Run DML using `econml`'s LinearDML or DML estimator with random-forest nuisance models. Output the ATE point estimate and 95% confidence interval.
- Run a Causal Forest using `econml`'s CausalForestDML. Output CATE estimates segmented by 2–3 of my moderator variables. Identify the segment with the largest treatment effect (the "persuadables" — the customers, accounts, or units most responsive to the intervention).

PHASE 3 — INTERPRETATION:
- Print a one-paragraph plain-language summary of the ATE result, including the units (e.g., "raising price by $30 reduces renewal probability by an estimated 4.2 percentage points, 95% CI [2.1, 6.3]").
- Identify the persuadable segment by name and quantify how much larger their CATE is than the overall ATE.
- Flag any backdoor path that the adjustment set cannot block — i.e., the cases where the effect is NOT IDENTIFIABLE from this DAG and data. Recommend the next move (instrumental variable? front-door? RCT?).

Use econml, networkx, pandas, numpy, scikit-learn. Make the script self-contained and well-commented. Save it to my Living Model folder as estimate-effects.py.

Here is my DAG and the candidate intervention X and outcome Y: [PASTE FROM CHAPTER 6 / 7].
```

---

**What this produces:** A runnable Python script that identifies the adjustment set, estimates ATE via DML, estimates CATE via Causal Forest, names the persuadable segment, and flags any non-identifiable effects. Output: a numeric estimate of your intervention's causal effect plus the segments where it works hardest.

**How to adapt this prompt:**
- *For your own project:* If you don't have observational data, the synthetic-data path lets you stress-test the pipeline before real data exists. The ATE will be whatever you encoded in your SCM — this is a sanity check, not a real estimate.
- *For ChatGPT / Gemini:* Works as-is in code-interpreter modes. ChatGPT's Code Interpreter and Gemini's code execution can both run this if you upload data.
- *For Claude Code:* Recommended. Claude Code will execute the script in your terminal, install dependencies, and iterate on bugs.
- *For a Claude Project:* Run the script outside the Project, then paste the output back in for interpretation in the Chapter 9 exercise.

**Connection to previous chapters:** Chapter 7's CPDAG told you which edges in your DAG are defensible. This chapter operationalizes the defensible portion — turning the graph into a number you can put in a recommendation.

**Preview of next chapter:** Chapter 9 takes one historical decision in your domain that turned out badly and runs the abduction-action-prediction counterfactual on it — answering "what would have happened if we had done the other thing?"

---

## 🕰️ AI Wayback Machine

The ideas in this chapter didn't appear from nowhere. **Gertrude Cox** was co-authoring *Experimental Designs* (1950, with William Cochran) — the foundational textbook on factorial experiments and adjustment for confounding that the modern backdoor and DML estimators inherit from decades before most people had heard of the backdoor criterion and the deconfounded effect estimate. Here's a prompt to find out more — and then make it better.

![Gertrude Cox, c. 1940s. AI-generated portrait based on a public domain photograph (Wikimedia Commons).](images/gertrude-cox.jpg)
*Gertrude Cox, c. 1940s. AI-generated portrait based on a public domain photograph.*

**Run this:**

```
Who was Gertrude Cox, and how does her work on experimental design and statistical adjustment connect to the modern apparatus of the backdoor criterion and double machine learning for estimating causal effects from observational data? Keep it to three paragraphs. End with the single most surprising thing about her career or ideas.
```

→ Search **"Gertrude Cox"** on Wikipedia after you run this. See what the model got right, got wrong, or left out.

**Now make the prompt better.** Try one of these:

- Ask it to explain *adjustment for confounding* in plain language, as if you've never run a regression
- Ask it to compare Cox's hand-tabulated factorial designs to a modern double-machine-learning pipeline
- Add a constraint: "Answer as if you're writing the introduction to a chapter on identifying causal effects from observational data"

What changes? What gets better? What gets worse?

