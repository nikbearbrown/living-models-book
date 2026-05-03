# Chapter 11 — Treatments

*Randomized controlled trials, Esther Duflo's experimental design at scale, the translation from clinical "treatment" to organizational "intervention," and the social systems that violate SUTVA by design.*

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. **Explain** why randomization establishes causation where observational methods cannot, using the *do*-operator and backdoor path logic from prior chapters.
2. **Decompose** SUTVA into its two components — no interference and no hidden variations of treatment — and identify which component fails in a given study.
3. **Diagnose** SUTVA violations in organizational intervention designs: interference, spillover, network effects, and compliance heterogeneity.
4. **Select** the appropriate design alternative — cluster randomization, time-staggered rollout, encouragement design — given the structure of the violation.
5. **Evaluate** a real or hypothetical intervention for the gap between trial-level effect and deployment-level effect, and communicate what the trial result does and does not guarantee.

**Prerequisites:** Chapters 6–8 (structural causal models, backdoor criterion, identification from observational data) provide the formal apparatus this chapter builds on. Chapter 10's treatment of latent confounders and observational limits is the direct predecessor — this chapter answers the question Chapter 10 raises: if observational methods have limits, what do we do instead? The *do*-operator, introduced in Chapter 2 and developed formally in Chapters 6–7, appears throughout. No new mathematical machinery is introduced here; the chapter applies the existing apparatus to the experimental setting.

**Why this chapter matters:** The randomized controlled trial is the strongest causal inference tool we have, and understanding why it works — not just that it works — is what lets you use it correctly and recognize when it fails. The failure modes are not exotic. They show up in almost every organizational intervention you will encounter: a training program that travels through an instructor, a pricing change that competitors can see, a technology rollout that employees discuss. SUTVA is the assumption you are making every time you run an A/B test or evaluate a policy rollout. This chapter makes it visible so you can audit it.

---

## The Question That Requires an Experiment

Chapter 10 ended in a hard place. Observational methods — even the best ones, armed with backdoor adjustment, instrumental variables, and double machine learning — cannot fully escape the problem of unmeasured confounding. If there is a variable that affects both treatment and outcome, and we cannot measure it, no amount of methodological sophistication eliminates the bias it introduces. Sensitivity analysis tells us how bad the unmeasured confounder would have to be to change our conclusions. But it does not tell us the confounder does not exist.

The randomized controlled trial is the escape. It does not require measuring confounders. It does not require identifying a sufficient adjustment set. It does not require arguing about what we might have missed. It addresses confounding structurally, at the design stage, before any data is collected — by making the treatment independent of every potential confounder, measured or not.

This is the deepest reason why the randomized trial is the gold standard of causal inference, and why it occupies the chapter that closes Part Two. But the escape has limits. Understanding those limits — precisely, not vaguely — is what this chapter teaches. The limits are not reasons to abandon randomization when it is available. They are conditions under which randomization needs to be designed more carefully, or complemented with additional methods, or acknowledged as insufficient for the specific causal question being asked.

I want to start with the mechanism. Why does randomization work? Not the intuitive answer. The formal one.

---

## Concept 1 — Why Randomization Works: The Formal Account

### Confounding as a Structural Problem

Part Two has developed a precise vocabulary for confounding. A confounder is a common cause of both the treatment variable $X$ and the outcome variable $Y$. In a causal graph, confounders create backdoor paths — paths from $X$ to $Y$ that run "backward" through $X$'s parents, rather than forward through the causal mechanism we want to study. When backdoor paths exist, the observational distribution $P(Y | X)$ conflates the causal effect with the spurious association created by the shared cause.

The backdoor criterion tells us when we can block all backdoor paths by conditioning on a measured adjustment set $Z$. When such a set exists and is measured, we can identify $P(Y | do(X))$ from observational data. When no such set exists — because a common cause is unmeasured — identification fails. No statistical adjustment can recover the causal effect from the observational data alone.

Randomization solves this by surgical intervention on the causal graph. Formally, assigning treatment at random is equivalent to applying the *do*-operator: it sets $X$ to a specific value by external fiat, severing every arrow that previously pointed into $X$. After randomization, $X$ has no parents in the modified graph. The treatment is caused by nothing except the random assignment mechanism. Every backdoor path is severed — not blocked by conditioning, but eliminated at the structural level.

The result is that the distribution of outcomes in the treated group, $P(Y | X=1)$, is identical to the interventional distribution $P(Y | do(X=1))$. The two quantities, which are different in observational data, become equal under randomization. The average treatment effect — the difference in expected outcomes between the treated and control groups — estimates $E[Y | do(X=1)] - E[Y | do(X=0)]$ without any further adjustment.

<!-- → [DIAGRAM: Two causal graphs side by side. Left graph (observational): node U (unmeasured confounder, shown as dashed circle) with arrows pointing to both X (treatment) and Y (outcome); X has an arrow to Y; the path X ← U → Y is highlighted in red and labeled "backdoor path: unblockable (U unmeasured)." Right graph (after randomization): the U→X arrow is replaced by R→X (R = random assignment, shown as a box); U still points to Y but has no path to X; the former backdoor path is marked with an X and labeled "eliminated: X has no parents except R." Caption: "Randomization does not condition on U — it removes the arrow from U to X entirely."] -->

### Randomization Is Do-Calculus in Practice

There is a precise sense in which running a randomized trial is the physical implementation of the *do*-operator. The *do*-operator is a mathematical instruction: "set $X$ to this value, cutting all incoming arrows." A randomized trial implements that instruction physically: the random assignment mechanism sets the treatment, cutting any influence from the subject's background characteristics, preferences, or health status on which group they are in.

This is why the randomized trial answers the question $P(Y | do(X))$ directly, while observational studies must identify that quantity indirectly. The trial produces the interventional distribution by construction. The observational study must argue that an adjustment procedure recovers it.

The formal account has a practical implication: randomization is as strong as the physical implementation of the random assignment. If assignment is truly random — if there is no way for background characteristics to influence which group a subject ends up in — then the confounding structure is fully severed. If assignment is only approximately random, or if there is any possibility of selective compliance or dropout that correlates with background characteristics, the strength of the design is reduced.

<!-- → [DIAGRAM: Two-panel correspondence diagram. Left panel (mathematical): the expression P(Y | do(X=1)) with annotation "set X by fiat; sever all incoming arrows." Right panel (physical): a flowchart — subjects enter → random number generator assigns group → treatment applied — with annotation "assignment mechanism severs any influence of subject characteristics on group membership." A horizontal double-headed arrow between the panels is labeled "the trial is the physical realization of the do-operator." Below both panels: a note reading "Strength of equivalence = strength of the physical randomization."] -->

### What Randomization Does Not Do

Randomization severs backdoor paths. It does not solve every causal inference problem.

It does not guarantee that the estimated effect generalizes to populations other than the one studied — transportability, covered in Chapter 8, is a separate problem that randomization does not address. It does not guarantee that the estimated effect in the trial matches the deployed effect in production — a problem we will take up in the third concept of this chapter. And it does not guarantee that the assumptions specific to the trial design are valid — assumptions like SUTVA, which is the subject of the next concept.

The gold standard is gold. It is not infallible.

---

## Concept 2 — SUTVA: The Hidden Assumption in Every Trial

### The Assumption and Its Two Parts

Don Rubin formalized a pair of assumptions in the 1970s that had been implicit in experimental practice for decades. He called them together the **Stable Unit Treatment Value Assumption**, abbreviated SUTVA. Every potential outcomes analysis — every analysis that compares treated to untreated units — invokes SUTVA, explicitly or not. When SUTVA holds, the analysis is clean. When it fails, the estimated treatment effect is biased in ways that require additional modeling to address.

SUTVA has two components. I will state each precisely, then show where it fails.

**Component 1: No interference.** The potential outcome for unit $i$ depends only on whether unit $i$ is treated, not on whether any other unit $j$ is treated. In notation: $Y_i(x_1, x_2, \ldots, x_n) = Y_i(x_i)$. The outcome for person $i$ is a function of $i$'s own treatment, not of the entire treatment assignment vector.

**Component 2: No hidden variations of treatment.** There is only one version of each treatment level. If unit $i$ receives treatment $X = 1$, the treatment received is the same regardless of which unit receives it or how. "Treatment 1" means the same thing for every treated unit.

These two components address different failure modes. No-interference fails when units affect each other through spillover, social networks, markets, or shared environments. No-hidden-variations fails when the "same" treatment is actually delivered differently to different units — through variable implementation, compliance, or context-dependence.

<!-- → [TABLE: Two-column reference table for SUTVA components. Row headers: Component 1 (No interference) and Component 2 (No hidden variations). Columns: (1) Formal statement in plain language, (2) What it requires in practice, (3) Canonical failure mode, (4) Direction of bias when violated. Component 1 / Direction of bias: typically toward zero (control group benefits from spillover). Component 2 / Direction of bias: unpredictable (depends on whether high- or low-quality implementations dominate the trial). Reader should use this as a diagnostic reference when auditing any trial.] -->

### Where Component 1 Fails: Interference and Spillover

The vaccine example is the canonical illustration of interference. When some members of a community are vaccinated against an infectious disease, the unvaccinated members benefit through reduced transmission. The probability that an unvaccinated person contracts the disease depends not just on their own vaccination status but on how many of their contacts are vaccinated. This is exactly what SUTVA's no-interference component prohibits: unit $i$'s outcome (disease status) depends on unit $j$'s treatment (vaccination status).

The consequence for trial analysis is subtle and important. If we randomize 50% of a community to receive a vaccine and 50% to receive placebo, the placebo group's outcomes will be better than they would be in an unvaccinated community, because the partially vaccinated environment has already reduced transmission. The estimated treatment effect — the difference in disease incidence between the vaccinated and placebo groups — understates the true effect, because the comparison group has been contaminated with spillover benefits from the treated group.

Interference is not confined to infectious disease. It appears wherever units interact:

*Pricing experiments.* Randomize some customers to a lower price and others to the standard price. If the low-price customers tell their friends, those friends — in the standard-price group — may also demand lower prices, generating pressure that changes outcomes in the control group.

*Training programs.* Randomize some employees to receive training. The trained employees return to their teams and share what they learned. Untreated teammates benefit — or are influenced — by the spillover. The control group's outcomes improve, and the estimated treatment effect shrinks.

*Market entry.* Randomize some geographic markets to receive a new product offering. Customers in untreated markets may hear about the new offering through social media or travel, shifting their expectations and their behavior. The untreated markets are contaminated by awareness spillover.

In each case, the standard analysis — comparing treated to untreated units as if they were independent — produces an estimate that is biased toward zero. The true treatment effect is larger than estimated, because the comparison group has been partially treated by the spillover.

<!-- → [DIAGRAM: Network diagram illustrating interference in a social spillover scenario. Nodes = individuals arranged in a social graph; edges = connections. Left panel (assignment): half the nodes are filled (treated) and half are open (control) — randomly assigned. Right panel (spillover): red arrows flow from treated nodes to adjacent control nodes; control nodes with many treated neighbors are shaded light red to indicate partial contamination. Two annotations: (1) "SUTVA requires: Y_i depends only on own treatment." (2) "Actual: Y_i depends on neighbors' treatment — control group is partially treated by spillover." Bottom annotation: "Estimated effect = true effect − spillover benefit to controls." Reader should see that SUTVA violation is a structural feature of the network, not a measurement error.] -->

### Where Component 2 Fails: Hidden Variations of Treatment

The second SUTVA component fails more quietly. The training program assigned to the treatment group is not one treatment — it is many treatments, one for each combination of instructor, employee, and organizational context. The drug in a clinical trial is one treatment at the correct dose; but if patients vary in compliance, the "treatment" is a mixture of the intended dose, partial doses, and missed doses. The software rollout to treated stores is one treatment in the protocol; but if stores vary in how they configure and use the software, the effective treatment varies across stores.

Hidden variations of treatment create heterogeneity in the treatment effect. If treatment implementation is good in some units and poor in others, the estimated average effect will be a weighted average of the good-implementation effect and the poor-implementation effect. That average may not describe any unit's actual experience particularly well.

The problem is compounded when the variation in implementation is correlated with other variables. Suppose experienced managers implement a training program well, and inexperienced managers implement it poorly. The treatment effect in experienced-manager teams is large; in inexperienced-manager teams, it is small. If we estimate a single average effect without accounting for this, we will misattribute the manager-experience effect to the training program itself. The "treatment" appears to work better in some settings than others — but the reason is not the treatment; it is who is delivering it.

<!-- → [DIAGRAM: Side-by-side comparison of treatment variation in clinical vs. organizational settings. Left column (clinical trial): a single bottle labeled "Drug A, 10mg daily." Below it, a compliance bar chart — 90% full dose, 8% partial dose, 2% no dose. The variation is narrow and measurable. Right column (organizational training program): a single document labeled "Program X, 2-day workshop." Below it, an implementation quality bar chart across 12 instructors, rated 1–5 — wide variation from 1.5 to 4.8. The variation is wide and partially unmeasured. Caption: "What was assigned (top) vs. what was received (bottom). In organizational settings, the gap is structural." Both columns annotated with: "SUTVA requires: one version of treatment. Reality: many versions."] -->

### The Combined Failure: When Both Components Break

The JTPA study — the randomized evaluation of the US Job Training Partnership Act, conducted across multiple sites in the late 1980s — is the pedagogically rich case where both SUTVA components failed simultaneously. Job training program eligibility was randomized, giving the study its causal identification. But:

The training services varied dramatically across sites. Some sites offered classroom instruction; others on-the-job training; some both. The protocol differed enough across sites that "receiving job training" meant different things in different places. Hidden variations of treatment were substantial.

Compliance was imperfect. Some individuals randomized to the treatment group did not participate in the training. The intent-to-treat analysis — comparing all randomized individuals regardless of actual participation — gives a smaller estimated effect than the treatment-on-the-treated analysis. Both are valid estimates; they answer different questions. The intent-to-treat answer is "what is the effect of making training available?" The treatment-on-the-treated answer is "what is the effect of actually receiving training?" Neither is the answer to "what is the effect of the training program as designed?" — because the program as designed was not consistently received.

In dense local labor markets, training enough workers may depress wages for program graduates through increased labor supply. The individual-level analysis compares trained to untrained within each market. But if the program creates enough trained workers to shift the local wage for trained jobs, the trained graduates earn less than they would in a market without the program — even though the training itself improved their skills. This is a spillover effect operating through the market, not through direct social contact.

The JTPA study has been re-analyzed many times using methods that address one or more of these violations. Each re-analysis produces different effect estimates, depending on which assumptions it invokes. The headline number — "job training raises earnings by X percent" — is not a fact about the training. It is a fact about the training, the compliance, the site, and the labor market, under a specific set of assumptions about SUTVA violations. The analyst who reports a single number without flagging these assumptions is, at best, simplifying, and at worst, misleading.

<!-- → [TABLE: JTPA study SUTVA audit — three rows, one per violation: (1) Cross-site variation in services (Component 2 — hidden variations; mechanism: different training types across sites; effect on estimate: attenuates average, masks heterogeneity). (2) Imperfect compliance (Component 2 — hidden variations; mechanism: some treated individuals received no training; effect on estimate: ITT < TOT, both valid but answer different questions). (3) Local labor market wage spillover (Component 1 — interference; mechanism: program graduates compete with each other, depressing local wages; effect on estimate: underestimates individual skill gain, overestimates market-level wage impact). Each row also lists what design change or analytic adjustment would address it. Reader should see the JTPA case as a structured audit, not a narrative of failures.] -->

---

## Concept 3 — Design Alternatives When SUTVA Fails

Knowing that SUTVA fails is not the end of the analysis. It is the beginning of a design problem. The question is: given the structure of the SUTVA violation, what experimental design can still produce a credible causal estimate?

Three main alternatives have been developed in the literature. I will describe each in terms of the violation structure it addresses and the conditions under which it applies.

### Cluster-Randomized Trials

The cluster-randomized trial addresses interference by moving the randomization unit from individual units to groups of units — clusters — within which interference is expected to occur.

The logic: if customers in the same city are likely to talk to each other (spillover within cluster), but customers in different cities are unlikely to interact (no spillover across clusters), then randomizing at the city level ensures that the treatment and control groups are drawn from different spillover systems. SUTVA holds at the cluster level even if it fails at the individual level.

The cost is statistical efficiency. When we randomize clusters instead of individuals, the effective sample size is the number of clusters, not the number of individuals. If we have 20 cities with 1,000 customers each, we have 20 independent observations, not 20,000. The variance of the estimated treatment effect is determined by variability across clusters, which is typically much larger than variability across individuals within a cluster. We need more clusters — not more individuals per cluster — to achieve the same precision.

The cluster-randomized design also requires a choice about what to estimate. The *average treatment effect* in a cluster-randomized trial is the average of cluster-level effects, not the average of individual-level effects. These differ when cluster size is correlated with treatment effect — a complication that requires careful handling in the analysis.

Cluster randomization is the standard design for evaluation of community-level interventions: vaccines, sanitation programs, community health worker programs, and similar policies where the interference is defined by physical proximity or shared infrastructure. Duflo's work on deworming programs in Kenya used cluster randomization at the school level, because the transmission of intestinal worms happens within schools (students share the same soil) but not, primarily, across schools.

<!-- → [DIAGRAM: Two-panel diagram contrasting individual and cluster randomization for a school deworming intervention. Left panel (individual randomization): three school outlines, each containing a mix of treated (filled circles) and control (open circles) children; red transmission arrows cross between treated and control children within each school — SUTVA violation annotated. Right panel (cluster randomization): three school outlines — two fully filled (treated schools) and one fully open (control school); transmission arrows are contained within each school and therefore within treatment status — SUTVA holds at school level. Below both panels: "Unit of randomization must match unit of interference." Annotation pointing to the cluster design: "Spillover is now contained within the treatment group — it affects the estimate of the cluster-level effect, not the between-cluster comparison."] -->

### Time-Staggered Rollouts

Some interventions cannot be cluster-randomized because the organization requires that the intervention eventually reach all units. A company rolling out a new software platform to all offices cannot randomize half of them to never receive it. But it can randomize the *order* in which offices receive it — which offices go first, which go second, which go last.

The time-staggered rollout creates a natural comparison: early adopters can be compared to later adopters before the latter have been treated. The early adopters are the treatment group; the not-yet-treated units are the comparison group. The comparison is valid as long as the timing of rollout is not correlated with the outcome — that is, as long as the first offices to receive the software were not selected because they were already trending upward.

The statistical method that exploits staggered rollouts is called difference-in-differences. The estimator compares the change in outcomes at treated units (before and after treatment) to the change in outcomes at not-yet-treated units over the same period. The before-after comparison at treated units absorbs unit-level baseline differences. The comparison to the not-yet-treated units absorbs time trends. The difference of the two differences estimates the causal effect.

The validity of difference-in-differences rests on the parallel trends assumption: in the absence of treatment, treated and untreated units would have followed the same trend in outcomes. This assumption is not guaranteed by design — it has to be argued or tested. When the assumption holds, the staggered rollout produces estimates comparable to a randomized trial even though it is technically observational.

<!-- → [CHART: Time-series line chart for a difference-in-differences illustration. X-axis: time periods, divided into pre-treatment and post-treatment by a vertical dashed line. Y-axis: outcome level. Two lines: solid line = treated unit; dashed line = not-yet-treated unit. Pre-period: both lines are parallel (parallel trends). Post-period: solid line rises (treatment effect); dashed line continues at the same slope. Annotations: (1) A bracket labeled "DiD estimator" spanning the gap between the treated unit's actual post-treatment level and its projected counterfactual trend (dashed line extended). (2) "Parallel trends assumption: what treated unit would have done without treatment = dashed line's slope." (3) "Assumption not guaranteed by design — must be argued or tested using pre-period data."] -->

### Encouragement Designs

Sometimes neither cluster randomization nor staggered rollout is feasible, and the organization cannot randomize treatment directly. The treatment is a choice — participation in a training program, adoption of a new tool, enrollment in a benefit — and the organization cannot mandate who takes it. But it can randomize who is *encouraged* to take it.

The encouragement design assigns random encouragement — an invitation, a subsidy, a reminder, a recommendation — to some units but not others. Some encouraged units will take up the treatment; some will not. Some unencouraged units will find the treatment anyway and take it; most will not. The randomization of encouragement is clean; the actual treatment uptake is not. The encouragement is an instrument for treatment.

This is exactly the instrumental variables design from Chapter 8, applied in the experimental setting. The instrument is the random encouragement. The structural requirements are the same: the instrument must affect treatment (relevance), must be independent of confounders (satisfied by random assignment of encouragement), and must affect the outcome only through treatment (exclusion restriction). When these hold, the treatment effect among compliers — those who take up treatment when encouraged and would not take it without encouragement — can be identified from the data.

The encouragement design estimates the *local average treatment effect* (LATE) — the average effect for compliers specifically — not the average effect for the entire population. This is a genuine limitation: if compliers are not representative of the population of interest, the LATE may not be the number the decision-maker wants. The LATE is still a valid causal estimate, and it is often the best available estimate in settings where direct randomization is infeasible.

<!-- → [DIAGRAM: 2×2 schematic of the encouragement design. Rows: Encouraged / Not Encouraged. Columns: Takes Treatment / Does Not Take Treatment. Four cells labeled: (Encouraged, Takes Treatment) = Compliers; (Encouraged, Does Not Take) = Never-takers; (Not Encouraged, Takes Treatment) = Always-takers; (Not Encouraged, Does Not Take) = Never-takers. Highlight the Complier cell. Annotations: (1) "LATE is estimated for compliers only — those whose treatment decision changes with encouragement." (2) "First stage: encouraged group has higher uptake rate than not-encouraged group — this difference is the instrument's strength." (3) "Exclusion restriction: encouragement affects outcome only through treatment, not directly." Mapping to Chapter 8 instrumental variables: "Encouragement = instrument; treatment = endogenous variable."] -->

### Choosing Among the Alternatives

The three designs address different violation structures. The choice among them follows from the diagnosis.

When the violation is interference — spillover across units through a known network or physical proximity — the cluster-randomized trial moves the randomization to a level where SUTVA holds. The key question is: can you identify natural clusters within which interference is likely but across which it is negligible?

When the violation makes direct randomization infeasible — the organization will eventually treat all units, or the treatment is a choice — the staggered rollout exploits the timing of treatment as a source of variation. The key question is: is the timing of rollout plausibly unrelated to the outcome trend?

When the treatment is a choice and the timing is not stageable, the encouragement design uses randomized encouragement as an instrument. The key question is: can you design an encouragement that is strong enough to create meaningful variation in uptake and credibly satisfies the exclusion restriction?

These are not always clean choices. Real organizational settings often have multiple violations simultaneously. The JTPA study had compliance heterogeneity (hidden variations), market-level spillover (interference), and cross-site variation (more hidden variations). No single design addresses all three. The appropriate response is to address the most consequential violation by design and handle the others through modeling, sensitivity analysis, and explicit acknowledgment in the analysis.

<!-- → [TABLE: Design selection guide — three rows, one per design alternative (Cluster-randomized trial, Time-staggered rollout, Encouragement design). Four columns: (1) SUTVA violation addressed, (2) Key diagnostic question to determine applicability, (3) What the design estimates, (4) Primary limitation. The table should be formatted as a decision aid — a practitioner facing a new evaluation question should be able to read across the row of the recommended design and understand both what it buys and what it costs. Final row: a note reading "When multiple violations are present, address the most consequential by design; handle residual violations analytically."] -->

---

## Integration — The Trial Result and the Deployment Gap

The three concepts in this chapter build toward a single practical skill: evaluating what a trial result tells you, what it does not tell you, and what questions you must answer to go from the trial result to a credible deployment decision.

Esther Duflo's metaphor of the economist as plumber captures the gap precisely. The plumber's question — why isn't this thing working in practice? — is a question about the distance between the trial result and the deployment result. That distance has a structure, and the structure is now visible in the vocabulary this chapter has built.

The trial result tells you: under trial conditions, with this population, with this implementation, randomizing $X$ produces an expected change in $Y$ of $\delta$. Four qualifiers are attached to every trial result, and all four can drive a gap between the trial estimate and the deployed effect.

**Trial conditions vs. deployment conditions.** The trial was run in specific sites, under specific constraints, with specific resources allocated to implementation quality. Deployment may involve different sites, different constraints, different resources. If the trial conditions that produced the effect are not reproducible in deployment, the effect attenuates.

**Trial population vs. deployment population.** The trial enrolled a specific population. If the deployment population differs — different demographics, different baseline behavior, different organizational context — transportability becomes the question. Chapter 8's framework for transportability applies here.

**Trial implementation vs. deployment implementation.** Hidden variations of treatment mean that the average trial effect is an average over the range of implementation quality in the trial. Deployment implementation may be better or worse than the trial average. If the trial's most skilled instructors ran the workshops, deployment with average instructors will produce smaller effects.

**SUTVA in trial vs. SUTVA in deployment.** The trial may have been designed to minimize spillover — by cluster-randomizing, by running in isolated markets. Deployment may occur in a setting where spillover is present and unavoidable. The trial effect was estimated under suppressed spillover; deployment operates with full spillover. Whether the spillover amplifies or attenuates the effect depends on the structure of the interference.

The Living Model architecture in Part Three formalizes this. A recommendation that emerges from the model does not just report the trial effect. It reports the trial effect, the deployment conditions assumed, the expected attenuation factors, and the uncertainty around the deployed effect. The recommendation is honest about what it does not know. What the trial established with confidence, and what the deployment projection required further assumptions to produce, are separated in the output.

<!-- → [DIAGRAM: Horizontal deployment gap diagram. Left anchor: box labeled "Trial result: δ̂ (estimated effect under trial conditions)." Right anchor: box labeled "Deployed effect: δ_deploy (actual effect in production)." Between them: four labeled wedges opening downward (the gap factors), each annotated with the chapter concept that addresses it: (1) Condition gap → transportability (Ch. 8); (2) Population gap → transportability (Ch. 8); (3) Implementation gap → hidden variations of treatment (Concept 2, this chapter); (4) SUTVA gap → design alternatives (Concept 3, this chapter). Below each wedge: a one-line question the practitioner should ask (e.g., "Are trial site conditions reproducible?"; "Does the deployment population match the trial population?"; "Who is delivering the intervention in deployment?"; "Does the deployment context have more spillover than the trial?"). Caption: "The gap is not noise. It is a set of addressable questions."] -->

The worked comparison from the JTPA study is worth revisiting in this integrated frame. The study randomized job training eligibility — a clean design for its core causal question. The reported effect was positive: job training raised earnings. But four attenuation factors were present. Implementation varied across sites. Compliance was imperfect, shrinking the intent-to-treat estimate. Local labor market spillover depressed wages for graduates in dense markets. And the study population — low-income adults in the late 1980s — may not be perfectly transportable to a different population in a different decade.

A deployment decision based on the headline effect number, without accounting for these four factors, would likely overestimate the effect of a new job training deployment. A deployment decision based on the JTPA study with explicit modeling of the four gap factors would produce a more conservative but more honest projection — and would identify which implementation decisions (site selection, compliance support, market density) most affect the deployed outcome.

That is the plumber's contribution. Not skepticism about whether interventions work. Precision about what makes them work, where, and for whom.

---

## Exercises

### Warm-Up

**1. Name what randomization does.**
*(Tests Objective 1 — explain why randomization works)*
*(Difficulty: Low)*

For each statement, identify whether it correctly describes what randomization accomplishes — and if it is incorrect, explain the error using the *do*-operator and backdoor path logic.

a. "Randomization ensures that the treatment and control groups are identical on all baseline characteristics."
b. "Randomization severs every arrow pointing into the treatment variable in the causal graph."
c. "Randomization eliminates the need to worry about unmeasured confounders."
d. "Randomization guarantees that the estimated effect in the trial will match the effect in deployment."

---

**2. Decompose SUTVA.**
*(Tests Objective 2 — decompose SUTVA into its two components)*
*(Difficulty: Low)*

For each scenario, identify which SUTVA component is violated — no-interference, no-hidden-variations, both, or neither — and explain the mechanism of the violation.

a. A pharmaceutical company runs a trial of a new antibiotic. Some patients take the full course; others stop early when they feel better.
b. A school district randomizes classrooms to receive a new reading curriculum. Students in treated classrooms talk to students in control classrooms at recess.
c. A tech company randomizes users to see a new product recommendation algorithm. Users who see the new algorithm may tell friends about products they found, influencing friends' purchase behavior.
d. A clinical trial of a blood pressure medication assigns treatment by random number. Patients in treatment and control groups attend the same clinic and do not interact outside appointments. The medication dosage is fixed by protocol.

---

**3. Identify the interference structure.**
*(Tests Objective 3 — diagnose SUTVA violations)*
*(Difficulty: Low)*

For each intervention, describe the interference mechanism: how does treatment status of one unit affect outcomes of another? Name whether the interference channel is through physical contact, social network, market prices, or shared infrastructure.

a. A city randomizes some neighborhoods to receive improved street lighting.
b. An online retailer randomizes some customers to receive free shipping promotions.
c. A university randomizes some students to participate in a peer tutoring program.
d. A government randomizes some small businesses to receive a subsidy for hiring new workers.

---

### Application

**4. Design the right trial.**
*(Tests Objective 4 — select appropriate design alternative)*
*(Difficulty: Medium)*

For each scenario, a researcher wants to estimate the causal effect of an intervention but faces a design challenge. Identify the challenge (which SUTVA component fails, and why), then recommend the most appropriate design alternative — cluster randomization, time-staggered rollout, or encouragement design. Explain why that design addresses the specific violation.

a. A hospital network wants to evaluate whether a new hand-hygiene reminder protocol reduces hospital-acquired infections. The hospital has 30 wards. Nurses in different wards talk regularly and share best practices. The hospital requires that all wards eventually adopt the protocol.

b. A consumer packaged goods company wants to evaluate the effect of a price reduction on a product. The product is sold nationwide. Customer social networks span regions.

c. A company wants to evaluate whether a voluntary mental health support program improves employee productivity. The program is opt-in. Employees who hear about the program from colleagues may seek it out regardless of whether they were formally offered it.

d. An education nonprofit wants to evaluate whether teacher coaching improves student outcomes. Schools are organized into districts. Teachers within a school share students and coordinate on curriculum. Teachers across districts do not interact professionally.

---

**5. Diagnose the JTPA violations.**
*(Tests Objectives 2 and 3 — decompose and diagnose SUTVA failures)*
*(Difficulty: Medium)*

The JTPA study randomized job training program eligibility for low-income adults across multiple US sites. Review the description of the study from the chapter and answer the following:

a. The study had three SUTVA-related complications. For each one, identify whether it is a no-interference violation, a no-hidden-variations violation, or something else. Be precise about the mechanism.
b. The study reported an intent-to-treat estimate and a treatment-on-the-treated estimate. These answered different questions. State precisely what question each estimate answers, in terms of the potential outcomes framework.
c. Suppose you were designing a follow-up study to address the most consequential SUTVA violation in the original JTPA study. Which violation would you prioritize, and what design change would address it?

---

**6. Evaluate a cluster-randomized design.**
*(Tests Objective 4 — cluster randomization)*
*(Difficulty: Medium)*

A public health organization wants to evaluate a community health worker program in 40 rural counties. Each county has approximately 5,000 households. The program assigns community health workers to visit households and provide health education. The organization can randomly assign 20 counties to receive the program and 20 counties to serve as controls.

a. Explain why individual-level randomization (randomizing households rather than counties) would likely violate SUTVA in this setting. What is the interference mechanism?
b. Cluster randomization at the county level addresses the interference. What is the effective sample size of this design — 40 counties or 200,000 households? Explain.
c. Suppose the 40 counties vary substantially in baseline health outcomes. Some counties are wealthier and have better baseline health. How should the randomization be conducted to ensure that this baseline variation is balanced across treatment and control groups? (Hint: what modification to pure random assignment helps here?)
d. After the trial, the organization finds that the effect of the program is larger in poorer counties than in wealthier counties. Is this finding consistent with the trial design? What does it imply for deployment decisions?

---

### Synthesis

**7. From trial to deployment: the gap analysis.**
*(Tests Objective 5 — evaluate trial-to-deployment gap)*
*(Difficulty: High)*

A large retail chain ran a randomized controlled trial of an employee training program in 50 stores across three regions. The trial found that stores receiving the training program had 8% higher customer satisfaction scores than control stores over the six-month trial period. The chain is now deciding whether to roll out the program to all 400 stores.

You are given the following information about the trial:
- The training was delivered by a specialist team hired for the trial; in deployment, existing store managers will deliver the training.
- The 50 trial stores were in mid-sized cities; the chain's other 350 stores include rural stores and large urban stores.
- The trial was run during a period of low competitive pressure; the chain is now operating in a more competitive environment.
- Compliance in the trial was high (92% of employees completed the training); the deployment will be voluntary with no dedicated follow-through support.

a. For each of the four pieces of information, identify which gap factor it represents (condition gap, population gap, implementation gap, or SUTVA gap) and explain the direction of the likely attenuation (will the deployed effect be larger or smaller than the trial estimate, and why?).
b. The store operations director argues: "We have a randomized trial showing an 8% improvement. That's rigorous evidence — we should deploy." Evaluate this argument. What is correct about it, and what does it miss?
c. Propose a deployment design that would allow the chain to learn about the deployment effect while rolling out the program. What would you randomize, what would you measure, and what SUTVA violations would you need to monitor?

---

**8. The vaccine herd immunity case.**
*(Tests Objectives 1, 2, 3, and 4)*
*(Difficulty: High)*

A public health authority is designing a trial to evaluate the effectiveness of a new vaccine against a moderately contagious respiratory illness. The authority is choosing between two trial designs:

**Design A:** Randomize 5,000 individuals within a single city to receive the vaccine or placebo.
**Design B:** Randomize 20 cities: 10 cities receive a mass vaccination campaign (targeting 70% of residents), and 10 cities receive no intervention. Compare illness incidence across cities.

a. Identify the SUTVA violation that Design A faces but Design B is designed to address. Explain the interference mechanism in concrete terms.
b. Design A would underestimate the true effectiveness of the vaccine. Explain the direction and mechanism of the bias: why would the comparison group in Design A have better outcomes than a true unvaccinated comparison group?
c. Design B estimates a different causal quantity than Design A. State precisely what question each design answers. Which question is more relevant to the public health decision of whether to conduct a mass vaccination campaign?
d. Design B has far fewer clusters (20 cities) than Design A has individuals (5,000). A statistician argues this makes Design B less reliable. Is this correct? What determines the precision of Design B's estimate, and how would you address the precision concern?

---

### Challenge

**9. Design a SUTVA-aware trial for your organization.**
*(Tests Objectives 2, 3, 4, and 5 — full design skill)*
*(Difficulty: Open-ended)*

Identify an intervention your organization has implemented or is considering — a training program, a policy change, a technology rollout, a pricing change, or a similar organizational intervention.

a. Apply the SUTVA diagnostic. For your intervention, assess whether the no-interference component is likely to hold and whether the no-hidden-variations component is likely to hold. For each component, describe the mechanism of potential violation and rate its severity (likely negligible / moderate / severe).
b. Based on your SUTVA diagnosis, recommend an experimental design for evaluating the intervention: standard RCT, cluster-randomized trial, time-staggered rollout, or encouragement design. Justify your recommendation in terms of which violation it addresses.
c. Describe the deployment gap factors you would expect between a trial result and the actual deployed effect. For each gap factor, identify what information you would collect during deployment to monitor it.
d. Write a one-paragraph briefing for a decision-maker who wants to know whether "the evidence shows this intervention works." Your briefing should be honest about what a trial result establishes and what it does not, in plain language.

---

**10. Where experimental methods break.**
*(Tests Objectives 1–5 at the framework's edges)*
*(Difficulty: Open-ended)*

The chapter presents a hierarchy of experimental designs, with the standard RCT at the top and encouragement designs at the lower end when direct randomization is infeasible. But there are settings where no experimental design — not even a cluster-randomized trial or an encouragement design — can cleanly identify the causal effect.

a. Identify a class of policy or organizational interventions for which randomization at any level is structurally infeasible — not just expensive or logistically difficult, but impossible in principle. Explain why randomization cannot be applied.
b. For the class you identified, which observational methods from Chapters 6–10 would be most appropriate? What assumptions would those methods require, and how confident can we be that the assumptions hold?
c. Duflo argues that economists should think like plumbers — focused on making interventions work in practice, not just establishing effects in trials. For the class of interventions in part (a), what does the plumber's mindset look like? What is the decision-maker doing when a clean trial is unavailable and observational estimates carry substantial uncertainty?
d. Is there a risk that the framing of "experimental methods as gold standard, observational as second best" causes practitioners to underinvest in observational methods, or to treat observational evidence as inherently suspect even when it is the best available? Defend your answer.

---

## Chapter Summary

You can now do something you could not do before this chapter: encounter a trial result — published in a journal, presented by a vendor, reported in an internal analysis — and audit what it actually establishes. Randomization is the gold standard because it physically implements the *do*-operator, severing backdoor paths at the design stage rather than adjusting for them after the fact. But the gold standard makes assumptions, and the most consequential assumption — SUTVA — fails routinely in social and organizational settings.

The one idea from this chapter that matters most: **SUTVA is the assumption you are making every time you compare treated to untreated units, and it has two independent failure modes.** No-interference fails when units affect each other through spillover, markets, or networks. No-hidden-variations fails when the same treatment label covers different actual treatments across units. Both failures bias the estimated treatment effect, in directions that depend on the structure of the violation. Neither failure is obscure. Both are the normal condition in organizational intervention evaluation.

The common mistake to watch for: treating a randomized trial result as a deployment guarantee. The trial estimates the effect under trial conditions, in the trial population, with trial implementation, and under whatever SUTVA structure the trial design managed to control. Deployment changes all four of these. The gap between the trial estimate and the deployed effect is not noise — it is a structured set of questions that can be anticipated, monitored, and addressed.

The Feynman test for this chapter: can you explain to a colleague who just cited a randomized trial result exactly what that result establishes — and exactly what it does not — with specific reference to the SUTVA assumptions the trial required and the deployment gap factors that apply to your organization's context?

---

## Connections Forward

Part Two is now complete. The apparatus has been built: causal graphs, the do-operator and its physical implementation in randomized trials, backdoor identification and its extensions, double machine learning, counterfactual reasoning, sensitivity analysis, and the SUTVA framework for experimental design. The apparatus is not a guarantee of correctness. It is a discipline of working honestly under uncertainty — of making assumptions visible, conclusions auditable, and the gap between what we know and what we need to know explicit.

Part Three builds the architecture that operationalizes the apparatus. Chapter 13 defines the four properties of a Living Model and distinguishes it from the lesser objects it is sometimes confused with — dashboards, predictive models, digital twins. The Living Model uses the full Part Two apparatus as its analytical core: causal graphs to represent structure, the do-operator to answer intervention questions, SUTVA-aware experimental designs to produce estimates where feasible, and the deployment gap analysis to communicate what the estimates do and do not establish.

The reader who has worked through Part Two now has everything needed to evaluate the architectural choices in Part Three — not as a consumer of methodology but as a practitioner who understands why each piece of the architecture is built the way it is, and what would go wrong if it were built differently.

---

*What would change my mind:* A demonstration that a major organizational intervention has been successfully evaluated using purely observational methods, with credibility comparable to a well-run RCT, in a setting where SUTVA was clearly violated. The advances in spillover analysis suggest this should become possible. I have not yet seen a case where it has been done at scale. I would update if shown one.

*Still puzzling:* The relationship between trial-level effects and deployment-level effects in organizational interventions. The trial effect, even when cleanly estimated, attenuates in deployment for reasons that are hard to predict in advance — instructor variability, manager follow-through, market response. I do not yet have a confident framework for predicting the attenuation, and the literature does not seem to either. This may be a frontier where Living Models can make progress in the coming decade.

---

**Tags:** randomized controlled trials gold standard, Esther Duflo experimental design plumber, SUTVA stable unit treatment value, spillover effects network interference, JTPA job training causal inference, cluster randomization, encouragement design instrumental variables, deployment gap trial-to-production
