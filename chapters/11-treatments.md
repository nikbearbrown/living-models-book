# Chapter 11 — Treatments

*Randomized controlled trials, Esther Duflo's experimental design at scale, the translation from clinical "treatment" to organizational "intervention," and the social systems that violate SUTVA by design.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *Treatments*
- *From Clinical Trial to Organizational Intervention*
- *SUTVA and the Limits of the Gold Standard*

**TL;DR.** The randomized controlled trial (RCT) remains the strongest single tool for establishing causation, because randomization severs the backdoor paths that observational studies have to address indirectly. Esther Duflo's work shows that RCTs are feasible at scale even in policy and development settings. But the translation from clinical "treatment" to organizational "intervention" is harder than it looks. The Stable Unit Treatment Value Assumption (SUTVA) — that one unit's outcome does not depend on another unit's treatment — fails routinely in social systems. Spillover effects, network interference, and herd dynamics violate SUTVA by design, and a Living Model architecture that ignores these violations will produce misleading results.

---

## The Economist as Plumber

In 2017, Esther Duflo, who would receive the Nobel Prize in Economics two years later, gave a lecture titled *The Economist as Plumber*. She argued that economists, particularly those who study development and public policy, should think of themselves not just as theorists but as plumbers — practitioners who deal with the real-world complications of getting interventions to work.

The metaphor was deliberate. Duflo had spent two decades running randomized controlled trials of policy interventions in India, Kenya, Indonesia, and elsewhere. She had run trials of school deworming, microfinance, fertilizer subsidies, mosquito nets, and dozens of other interventions. The trials had produced substantial knowledge about which interventions work and which don't. But the trials had also produced a more uncomfortable lesson: even when an intervention works in a randomized trial, getting it to work in deployment is a different problem. The plumber's question — *why isn't this thing working in practice?* — turned out to matter as much as the economist's question.

This chapter is about the relationship between the randomized controlled trial — the gold standard of causal inference — and the actual practice of intervening in social and organizational systems. The trial answers a precise question. The deployment of the intervention requires answers to further questions that the trial does not ask. The Living Model architecture in Part Three has to handle both.

---

## Why Randomization Works

The randomized controlled trial works for a reason that is mathematically simple. By assigning treatment at random, we sever any causal arrow from confounders to treatment. The treatment becomes, by construction, exogenous — determined by the random number generator and nothing else. Backdoor paths do not exist because the variable on the causal pathway has no upstream causes other than randomness.

In the language of structural causal models, randomization is equivalent to applying the *do*-operator. The random assignment forces a specific value on the treatment variable, regardless of the variables that would otherwise have determined it. The resulting comparison of treated and control groups gives us, directly, an estimate of P(Y | do(X)).

This is why RCTs are the gold standard. They do not require the analyst to identify a deconfounding set, to argue about which variables to control for, to perform sensitivity analysis to unmeasured confounding. The randomization does all of this implicitly. The RCT is, in a sense, a Rung 2 experiment that produces Rung 2 evidence directly.

But the RCT has limits that are sometimes glossed over. We have already mentioned several.

**Ethical limits.** We cannot randomize people to smoke, to take harmful drugs, to be exposed to environmental toxins. For studies of harm, observational methods are forced.

**Practical limits.** We cannot randomize geography, history, weather, the timing of recessions. Macroeconomic policies and interventions at the level of nations or large institutions are typically infeasible to randomize.

**Resource limits.** RCTs are expensive and slow. The pharmaceutical RCT process takes years and costs hundreds of millions of dollars. The smaller-scale RCTs that organizations sometimes attempt can still strain budgets and timelines.

**Generalizability limits.** RCTs are run on specific populations under specific conditions. Generalizing the results to different populations or different conditions requires further assumptions, and those assumptions are often invisible in the trial's reporting. We saw transportability in Chapter 8 of Pearl's framework; the same problem arises here.

For all these reasons, the RCT is not always available. When it is, it should usually be used. When it isn't, the methods we have developed — backdoor adjustment, double machine learning, instrumental variables, sensitivity analysis — are how we proceed honestly.

---

## SUTVA and What It Hides

Most introductory accounts of randomized trials skip past an assumption that, in social and organizational contexts, is the most consequential one. The Stable Unit Treatment Value Assumption (SUTVA), articulated by Don Rubin in the 1970s, says two things:

1. *No interference.* One unit's outcome does not depend on another unit's treatment.

2. *No hidden variations of treatment.* The treatment received by every unit in the treatment group is the same.

Both parts of SUTVA are routinely violated in social and organizational settings, and the violations matter.

**Interference.** A vaccine trial in a community has interference: vaccinating some people protects others through reduced transmission. A pricing trial that exposes some customers to a higher price has interference if those customers tell their friends. A training program that trains some employees has interference if those employees teach their colleagues what they learned. In each case, the outcome of an untreated unit depends on what was done to other units. The standard RCT analysis, which treats each unit as independent, will produce biased estimates.

**Hidden variations.** A drug trial assumes the drug is the same for every patient. In practice, patients vary in compliance — some take the drug as prescribed, some don't. The "treatment" is not the same across patients. Likewise, a training program varies in how it is delivered across instructors and locations; a software rollout varies in how it is configured across deployments. The "treatment" is heterogeneous.

When SUTVA fails, the estimates from a randomized trial are biased in ways that are not always easy to characterize. The trial may underestimate the effect (if untreated units benefit from spillover) or overestimate it (if treated units' effect is amplified by social diffusion). The direction of bias depends on the structure of interference, which is itself hard to estimate.

Modern causal inference takes SUTVA violations seriously. The literature on *spillover effects* and *network interference* — including work by Edoardo Airoldi, Dean Eckles, Susan Athey, and others — develops methods for estimating treatment effects when SUTVA fails. The methods generally require additional assumptions about the structure of interference (the units form a network with known connections, or interference happens only within geographic clusters) and additional data (knowledge of who knows whom, or which units are in which clusters).

---

## The Translation Problem

Even when SUTVA holds and randomization is feasible, the translation from clinical "treatment" to organizational "intervention" introduces complications.

In a clinical trial, the treatment is a specific drug, dosage, and protocol. The protocol is well-defined. The patient takes the drug, and the drug enters the bloodstream regardless of the patient's beliefs or preferences. The treatment is, in a strict sense, a physical intervention.

In an organizational intervention, the "treatment" is typically a policy, a program, or a managerial decision. The policy is announced; managers implement it; employees respond. The implementation is mediated by humans whose beliefs, preferences, and skills affect what actually happens. The "treatment" is heterogeneous in ways that no protocol can fully specify.

Consider a clinical-style trial of a corporate training program. The "treatment" is a 2-day workshop. We randomize employees to attend or not attend. The workshop is run by an instructor; the instructor's quality varies. Some employees attend with full attention; others are distracted by deadlines. After the workshop, the employees return to their managers; some managers reinforce the training, others ignore it. The "treatment" is now a mix of the workshop content, the instructor's effectiveness, the employee's engagement, and the manager's follow-through. The randomized assignment captures the workshop attendance; it does not capture the rest.

This is what Duflo means by the plumber's problem. Even after the trial has identified which interventions work in principle, deploying them in practice requires attention to the implementation details — the things that vary across instructors, managers, and contexts. The trial gives us the average effect under the conditions of the trial. Deployment requires us to ensure conditions similar to the trial, or to adapt the intervention to new conditions, or to accept that the deployed effect will be smaller than the trial effect.

The Living Model architecture in Part Three has to handle this. The recommendation that comes out of the model is not just "do this intervention." It is "do this intervention, given these conditions, expecting this distribution of outcomes." The conditions and the distribution are part of the recommendation, not addenda to it.

---

## Spillover and the Vaccine Example

Vaccines are the classical example of an intervention with spillover effects. When some members of a community are vaccinated, the unvaccinated members benefit through reduced transmission — herd immunity. The benefit to the unvaccinated is not a side effect; it is part of how the intervention works in the population.

The spillover effect makes the standard randomized trial analysis misleading. If we randomize 50% of a community to receive a vaccine and 50% to receive placebo, the placebo group's outcomes will be better than they would have been in a community with no vaccination, because the placebo group benefits from the partial herd immunity created by the vaccinated 50%. The estimated treatment effect — the difference in outcomes between the two groups — will *understate* the true effect of the vaccine, because the comparison group is contaminated with spillover benefits.

The methods for handling this fall into two categories. *Cluster-randomized trials* randomize at the level of clusters (whole communities, schools, towns) rather than individuals. The treatment effect is then estimated by comparing outcomes across clusters. The within-cluster spillover is captured in the comparison; SUTVA holds at the cluster level even if it fails at the individual level. *Two-level randomization* designs randomize the *intensity* of treatment within clusters as well as the assignment of clusters to treatment. The result is a richer dataset that allows estimation of both the direct effect and the spillover effect separately.

These methods are technical; the principle is simple. When SUTVA fails, the unit of analysis has to be adjusted to a level at which it holds. When that adjustment is infeasible, additional structural assumptions about the spillover are required, and the credibility of the conclusion depends on those assumptions.

---

## Herd Immunity and the Innovator's Dilemma

The same dynamics that produce herd immunity in epidemiology produce more uncomfortable phenomena in business and policy. A pricing change rolled out to half a market affects the half that did not receive it, because the customers talk to each other, the competitors respond to the change, and the market structure adjusts. A regulatory change affects industries that are not directly regulated, because firms anticipate broader changes and adjust preemptively. A new technology adopted by some firms changes the competitive dynamics for all firms.

The Innovator's Dilemma, which we discussed in Chapter 2 as a problem of incentives and time horizons, has a SUTVA-violation aspect as well. The disruption that an entrant produces in a market does not stay confined to the customers that the entrant serves directly. It changes the competitive dynamics, the customer expectations, the price points, the bundles that competitors must offer. The incumbent's response to the entrant is shaped by these market-level changes, not just by the direct customer-level dynamics.

A randomized trial of a market entry strategy — exposing some customers to the new offering and not others — would underestimate the effect, because the customers who were not exposed to the new offering would still experience changes in their existing vendor relationships as the incumbent adjusts. The trial captures the direct effect on the treated; it does not capture the indirect market-level effects.

This is one of the reasons strategic-level interventions are hard to evaluate even in principle. The intervention does not stay within the units to which it was applied. The market is the unit, and the market is one. Randomization within the market does not give us a comparison group that is unaffected by the intervention.

---

## When RCTs Are Possible at the Organizational Level

Despite all these caveats, there are organizational settings where RCTs work well. The principle is that the units must be sufficiently independent that SUTVA approximately holds.

**Customer-level randomization at scale.** Tech platforms run millions of A/B tests every year, randomizing customers to different versions of features, prices, layouts, and recommendations. SUTVA approximately holds because customers are largely independent — most don't talk to each other about the specific features they're seeing. The exceptions (when customers do talk, or when network effects matter) are studied separately, often using cluster-randomized designs.

**Site-level randomization.** Retail chains randomize stores to different policies (pricing, layout, staffing) and compare outcomes across stores. SUTVA approximately holds because stores serve different customer populations with limited overlap. The exceptions (cross-store cannibalization for stores in the same city) are managed by avoiding intra-cluster randomization.

**Time-staggered rollouts.** When a policy must eventually be rolled out to all units, randomizing the *order* of rollout creates a natural experiment. Units receiving the policy early are compared to units receiving it later, before the latter have been treated. This is not a clean RCT but is closer to one than alternative observational designs.

**Encouragement designs.** When direct randomization of the treatment is infeasible, randomization of *encouragement* to take the treatment can support an instrumental-variables analysis. We will see this approach in more detail in the architecture chapters of Part Three.

The lesson is that RCTs, where feasible, should be used; where infeasible, the next-best alternatives include cluster designs, time-staggered rollouts, encouragement designs, and the observational methods of Chapter 8 with explicit sensitivity analysis. The Living Model architecture supports all of these and chooses among them based on the structure of the question and the constraints of the setting.

---

## A Worked Comparison: The JTPA Study

The Job Training Partnership Act (JTPA) Study, which we will encounter again in Part Three, illustrates many of these issues at once. The study, conducted in the late 1980s, randomized job training program eligibility for low-income adults across multiple US sites. The "treatment" was access to job training services; the outcome was earnings over the subsequent 18 months.

The study was a randomized trial — a strong design for establishing causal effects. But the trial faced several SUTVA-related complications:

- *Hidden variations.* The job training services varied across sites. Some sites offered classroom instruction; others offered on-the-job training; some offered both. The "treatment" was different in each site, even though all sites were classified as treated in the analysis.

- *Compliance.* Some individuals randomized to the treatment group did not actually participate in the training. The "intent-to-treat" analysis (comparing all randomized individuals regardless of participation) gave a smaller estimated effect than the "treatment-on-the-treated" analysis (comparing only those who actually participated). Both estimates are valid; they answer different questions.

- *Local labor market effects.* If the program trains many people in a local labor market, the wages of program graduates may be depressed by the increased supply of trained labor. The trained graduates earn less than they would in a market without the program — even though the program improved their skills. The standard analysis, which compares treated to untreated within each site, may underestimate the true effect.

The JTPA study has been re-analyzed many times in the decades since, with each re-analysis using different causal inference methods to address one or more of these complications. The estimated treatment effects vary depending on the method used and the assumptions made. The headline number — "job training raises earnings by X percent" — depends on what we are willing to assume about hidden variations, compliance, and spillover.

This is the messy reality of causal inference in organizational and policy settings. The randomized trial gives us a clean estimate under clean assumptions. The clean assumptions sometimes hold; often they don't. The discipline of causal inference is to be honest about which assumptions we are making and to choose methods appropriate to the violations of the assumptions we expect.

---

## What the Living Model Inherits

The Living Model architecture in Part Three uses RCTs where feasible and observational methods where not. It uses cluster-randomized designs where SUTVA fails at the individual level. It uses encouragement designs where direct randomization is infeasible. It uses sensitivity analysis to communicate the dependence of conclusions on assumptions.

The architecture also acknowledges the plumber's problem. A causal effect estimated from a randomized trial is the effect under trial conditions. Deploying the intervention in production requires further reasoning about which trial conditions are reproducible, which are not, and what the deployed effect will plausibly be. This is not a weakness of the trial; it is a feature of how interventions work in the real world. The Living Model communicates the trial result, the deployment conditions, and the expected attenuation in production. The recommendation to the executive includes all three.

We end this chapter, and Part Two, with the recognition that causal inference is not a guarantee of correctness. It is a discipline of working honestly under uncertainty, of making the assumptions visible and the conclusions auditable. The randomized trial is the strongest single tool, but it is not the only tool, and it is not always available. The methods of Part Two — graphs, identification, estimation, counterfactuals, sensitivity analysis — are how we work when the gold standard is unavailable, and how we make the most of it when it is.

---

## Looking Forward to Part Three

Part Two has built the mathematical apparatus. Part Three builds the architecture — the systems and processes that operationalize the apparatus in production. Chapter 13 defines the four properties of a Living Model and contrasts it with the lesser objects (dashboards, predictive models, digital twins) that it is sometimes confused with. Chapters 14 through 17 take up the central architectural challenge: how to elicit a causal model from a domain expert, given that the model must be specified in terms the apparatus of Part Two can use. Chapters 18 through 20 build out the operational infrastructure — from graph to decision, the executive report, and the maintenance of the model over time.

The reader who has worked through Part Two now possesses the conceptual and technical apparatus to evaluate the architectural choices in Part Three. The architecture is not arbitrary. Each piece of it is justified by something we have learned in Part Two, and the integrity of the system depends on the apparatus underneath being applied correctly.

---

**What would change my mind:** A demonstration that a major organizational intervention has been successfully evaluated using purely observational methods, with credibility comparable to a well-run RCT, in a setting where SUTVA was clearly violated. The advances in spillover analysis suggest this should become possible. I have not yet seen a case where it has been done at scale. I would update if shown one.

**Still puzzling:** The relationship between trial-level effects and deployment-level effects in organizational interventions. The trial effect, even when cleanly estimated, attenuates in deployment for reasons that are hard to predict in advance — instructor variability, manager follow-through, market response. I do not yet have a confident framework for predicting the attenuation, and the literature does not seem to either. This may be a frontier where Living Models can make progress in the coming decade.

---

**Tags:** randomized controlled trials gold standard, Esther Duflo experimental design plumber, SUTVA stable unit treatment value, spillover effects network interference, JTPA job training causal inference
