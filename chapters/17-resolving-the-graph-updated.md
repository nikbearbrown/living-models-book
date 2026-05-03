# Chapter 17 — Resolving the Graph

*PC, GES, NOTEARS, FCI; the CPDAG handoff between expert elicitation and data-driven discovery; when to gather more data and when to run more expert sessions; and the validated graph as a living artifact under governance.*

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. **Describe** how each of the four algorithm families — PC, GES, NOTEARS, FCI — works: what it searches for, what assumptions it requires, and what kind of graph it returns.
2. **Select** the appropriate algorithm given a domain, data situation, and assumption profile — and articulate the trade-offs of the choice.
3. **Diagnose** disagreements between expert elicitation output and algorithmic output, distinguishing disagreements that more data can resolve from those that require additional expert input or controlled experiment.
4. **Apply** the three-step resolution protocol when expert and algorithm conflict, and determine whether remaining uncertainty is aleatory or epistemic.
5. **Govern** the validated graph as a living artifact: version control, drift monitoring, re-elicitation triggers, and auditability.

**Prerequisites:** Chapter 7 (Markov equivalence and the limits of observational identification) is essential — this chapter's discussion of what algorithms can and cannot resolve from data is grounded in the equivalence-class concept. Chapter 16's expert elicitation protocol produces the CPDAG that this chapter takes as its starting point. Familiarity with basic probability and conditional independence (introduced in Chapters 6–7) is assumed. No new mathematical machinery beyond those chapters is introduced here.

**Why this chapter matters:** The expert elicitation in Chapter 16 gives you a CPDAG — a partially directed graph with structural commitments where the expert was confident and undirected edges where she was not. But a CPDAG is not a model. To run counterfactuals, rank interventions, or produce recommendations, you need a fully directed acyclic graph with parameters. Getting from the CPDAG to that graph is the work of this chapter. The algorithms do some of it; the resolution protocol does the rest; and governance ensures the result stays trustworthy over time. Practitioners who skip this chapter often end up with graphs that were built carefully and maintained carelessly — which is the same, in production, as having built them carelessly.

---

## What the Interview Left Behind

The forty-five-minute elicitation produced a CPDAG. Some edges are directed — the expert committed on which way the arrow runs. Some edges are undirected — the expert acknowledged a causal relationship but could not or would not specify direction. Some pairs of variables have no edge at all, because the expert said there was no direct causal relationship between them. The CPDAG is the distillation of everything the expert knows, structured into a form that the causal apparatus from Part Two can operate on.

But a CPDAG is not quite usable yet. A counterfactual query — *what would retention be if we changed the activation flow?* — requires a directed graph, not a partially directed one. The undirected edges in the critical path of the analysis are not a minor inconvenience; they are the specific places where the model cannot yet produce an answer. Orienting them is the first task of this chapter.

I want to start by being precise about what kind of problem this is. Undirected edges in a CPDAG come from two distinct sources, and the source determines what can be done about them.

The first source is **Markov equivalence**. Two directed graphs are Markov-equivalent if they imply exactly the same conditional independence relationships among the variables. Observational data can distinguish graphs that are not equivalent — different structures imply different independencies, and independence tests on the data can tell them apart — but cannot distinguish graphs within the same equivalence class. If two directed graphs are Markov-equivalent, no amount of observational data will ever orient the edge that separates them. The only resolutions are expert input (someone who knows the mechanism) or experimental intervention (randomizing the variable in question, which breaks the equivalence by design).

The second source is **insufficient data**. The independence tests that underlie most causal discovery algorithms require statistical power. With a small sample or a high-dimensional variable space, the tests may not have enough power to detect a dependence that exists, or to distinguish two edges whose conditional distributions are nearly identical. Here the edge is undirected not because orientation is structurally unknowable, but because the current data does not have the resolution to determine it. More data — more observations, or observations targeted at the relevant conditional distributions — can resolve this ambiguity.

<!-- → [DIAGRAM: Two-column illustration of the two sources of undirected edges in a CPDAG. Left column: "Source 1 — Markov Equivalence." Shows two Markov-equivalent DAGs (e.g., A → B → C and A ← B → C) with identical conditional independence tables beneath them. Annotation: "Data is equally consistent with both. No amount of observational data distinguishes them. Resolution: expert input or experiment." Right column: "Source 2 — Insufficient Data." Shows a single DAG with a correctly directed edge A → B, then a bar chart of test statistics across sample sizes (n=100, 500, 2000) — the test statistic grows with n. Annotation: "The edge exists but the test lacks power. Resolution: more data." Reader should see that the two sources call for completely different investments.] -->

![Figure 17.1 — Two-column illustration of the two sources of undirected edges in a CPDAG. Left column: "Source 1](images/17-resolving-the-graph-fig-01.jpg)


The diagnostic question — which source is producing a given undirected edge? — is what determines whether you invest in more data or more expert sessions. I will return to this decision in detail. But first we need to understand what the algorithms do.

---

## Concept 1 — The Four Algorithm Families

### Shared Foundation: What Causal Discovery Algorithms Are Doing

Every causal discovery algorithm is, at its core, doing one of two things: testing for conditional independence relationships in the data and inferring structure from them, or searching over possible graph structures for the one that best fits the data according to some criterion. The first approach is constraint-based; the second is score-based. A third approach — continuous optimization — reformulates the problem algebraically. The fourth handles a complication — latent confounders — that the first three cannot.

Each algorithm makes assumptions. Understanding those assumptions is not optional: it is the prerequisite for knowing whether the algorithm's output is trustworthy on a given problem. An algorithm applied in a setting where its assumptions are violated will return a wrong graph without announcing that it has done so.

### PC: Constraint-Based Discovery

The PC algorithm takes its name from its inventors, Peter Spirtes and Clark Glymour, who introduced it in the landmark 1993 book *Causation, Prediction, and Search* together with Richard Scheines. It is the most widely used causal discovery algorithm and the most important to understand, both for its own value and because the other constraint-based algorithms are variations on its core logic.

**What it does.** PC begins with a fully connected undirected graph — every variable connected to every other variable — and systematically removes edges by testing for conditional independence. The fundamental question the algorithm asks, repeatedly, is: *are variables X and Y independent, given some subset S of the other variables?* If a conditioning set S is found that renders X and Y independent, the edge between them is removed: there is no direct causal relationship between them, given the variables in S. If no such S can be found, the edge remains.

After establishing the *skeleton* — which edges exist, regardless of direction — the algorithm orients edges by identifying *v-structures*. A v-structure is a configuration A → C ← B where A and B are not adjacent. It can be detected from the data because in a v-structure, conditioning on C makes A and B *more* dependent (the collider activates when conditioned on), whereas if C were not a collider, conditioning on it would make them less dependent or independent. The orientation of v-structures is entailed by the data; once v-structures are oriented, additional orientation rules propagate directions to further edges where the logic forces them.

The result is a CPDAG — a graph with directed edges where the data forces a direction and undirected edges where the data does not. This is precisely the object the expert elicitation also produces, which is what makes PC a natural complement to the Chapter 16 workflow.

**What it assumes.** PC requires *causal sufficiency* — the assumption that there are no unmeasured common causes of any two variables in the system. This is the Achilles heel. Latent confounders are common in practice: a variable that was not measured, a construct that cannot be measured, a confound that wasn't thought of. When causal sufficiency is violated, PC will misinterpret the correlation produced by a latent confounder as evidence of a direct edge between the confounded variables. The skeleton will be wrong, and the orientation will propagate errors from the wrong skeleton.

PC also requires that the conditional independence tests it runs are reliable. In low-dimensional data with large samples, standard tests (partial correlations for continuous variables, chi-squared tests for discrete variables) work well. In high-dimensional settings, or when samples are small, the tests lose power and the algorithm's output becomes unreliable. A practical rule of thumb: when the number of variables is more than roughly one-tenth of the sample size, PC's output should be treated with skepticism until confirmed by other methods.

**When to use it.** PC is the default for systems with a modest number of variables (under roughly a hundred), large samples relative to the number of variables, and plausible causal sufficiency. Administrative data systems, well-monitored product behavior data, and organizational systems with comprehensive measurement are good candidates. Clinical medicine and social science, where unmeasured confounders are the norm, are poor candidates.

<!-- → [DIAGRAM: Four-panel step-by-step illustration of PC applied to a five-variable system (A, B, C, D, E). Panel 1: fully connected graph — 10 undirected edges, all present. Panel 2: skeleton after independence tests — 3 edges removed; each removed edge labeled with the conditioning set S that rendered the pair independent (e.g., "A ⊥ E | {B,C}"). Panel 3: v-structure detection — one v-structure (A → C ← D) identified and oriented; two additional edges oriented by propagation rules; undirected edges that remain shown as thin lines. Panel 4: final CPDAG — directed edges in bold, undirected edges in dashed lines. Each panel annotated with the rule applied. Caption: "PC is a sequence of decisions grounded in specific tests, not a black-box transformation. Every edge can be traced to the test that produced it."] -->

![Figure 17.2 — Four-panel step-by-step illustration of PC applied to a five-variable system (A, B, C, D, E). Panel 1: fully connected graph](images/17-resolving-the-graph-fig-02.jpg)


### GES: Score-Based Discovery

Greedy Equivalence Search approaches the structure-learning problem differently. Rather than testing for independence relationships, it searches over the space of possible CPDAGs for the one that maximizes a penalized likelihood score. The most common scoring criterion is the Bayesian Information Criterion (BIC), which balances fit (how well does the model reproduce the observed conditional distributions?) against complexity (how many edges does the model require?).

**What it does.** GES operates in two phases. The *forward phase* starts from an empty graph and greedily adds edges — one at a time, always choosing the addition that most improves the score — until no further addition improves it. The *backward phase* then greedily removes edges — one at a time, always choosing the removal that least harms the score — until no removal improves it. The result is the highest-scoring CPDAG reachable by this greedy hill-climbing procedure.

GES has a strong theoretical property that PC does not: under its assumptions and in the large-sample limit, it is *consistent* — it converges to the true CPDAG as data accumulates. The score criterion penalizes unnecessary complexity, so GES naturally resists overfitting in a way that PC's independence tests do not. GES also tends to produce more confident orientations than PC when the data are well-behaved, because the scoring approach aggregates evidence across the full dataset rather than relying on marginal independence tests.

**What it assumes.** GES shares PC's requirement for causal sufficiency. It additionally requires that the data-generating process falls within a family for which a consistent score criterion can be defined. The BIC works correctly for Gaussian data and for multinomial discrete data. For mixed continuous-discrete data, or for data with heavy tails or other distributional irregularities, the standard BIC can be misleading, and specialized score criteria are required.

GES is also computationally more expensive than PC for the same number of variables, because the score evaluation at each greedy step requires fitting a model to the full dataset rather than running a single independence test. For very large datasets with many variables, this can be a bottleneck.

**When to use it.** GES is the default when one wants the discipline of likelihood maximization — when defensibility under scrutiny requires showing that the model was selected by a principled criterion rather than by the specific choices made in setting up independence tests. GES is also preferred when the data are large and the variables few enough that the score computation is feasible. Regulatory settings, where the model-selection procedure may be audited, often benefit from GES's explicit scoring framework.

<!-- → [DIAGRAM: Two-phase illustration of GES on the same five-variable system. Phase 1 (forward search): starts from empty graph; three steps shown — each step adds one edge with a score improvement bar (+ΔBIC) labeled beside it; graph after forward phase shown with all edges added. Phase 2 (backward search): two steps shown — one removes an edge (score improves, edge unnecessary); one attempts removal but does not improve score (edge kept). Final CPDAG shown. Below the two-phase diagram: side-by-side comparison of PC output and GES output on the same graph, with annotations marking edges where they agree (most) and where they differ (one or two). Caption: "GES and PC approach the same problem from different angles. Agreement across both is stronger evidence than either alone."] -->

![Figure 17.3 — Two-phase illustration of GES on the same five-variable system. Phase 1 (forward search): starts from empty graph; three steps shown](images/17-resolving-the-graph-fig-03.jpg)


### NOTEARS: Continuous-Optimization Discovery

The constraint-based and score-based approaches share a fundamental challenge: the number of possible DAGs over $n$ variables grows super-exponentially in $n$. Over ten variables there are already 4.2 trillion DAGs. Over twenty, the number is astronomical. Both PC and GES manage this by exploiting the equivalence-class structure and searching over CPDAGs rather than DAGs, but even this space is combinatorially large. For systems with hundreds or thousands of variables, combinatorial search is computationally infeasible.

NOTEARS — whose name stands for *Non-combinatorial Optimization via Trace Exponential and Augmented lagRangian for Structure learning* — solves this by replacing the combinatorial search with a continuous optimization.

**What it does.** NOTEARS parameterizes the graph as a real-valued weighted adjacency matrix $W$, where $W_{ij}$ represents the strength of a directed edge from variable $j$ to variable $i$. Edges absent from the graph correspond to $W_{ij} = 0$; edges present correspond to nonzero $W_{ij}$. The algorithm searches for the $W$ that minimizes a loss function (how well does the model fit the data?) subject to a constraint that $W$ represents a DAG — that the graph has no cycles.

The key insight of NOTEARS is an algebraic characterization of acyclicity. The constraint *$W$ is acyclic* can be expressed as: $\text{tr}(e^{W \circ W}) - n = 0$, where $\text{tr}$ is the matrix trace and $\circ$ denotes elementwise multiplication. This characterization is differentiable with respect to $W$, which means the acyclicity constraint can be incorporated into a continuous optimization using standard methods (augmented Lagrangian in the original formulation). The result is a structure-learning algorithm that scales to hundreds of variables on standard hardware.

**What it assumes.** NOTEARS was originally derived under the assumption that the data-generating process is linear with Gaussian noise — a restrictive assumption that limits its direct applicability to many real-world systems. Extensions have relaxed this: variants handle non-Gaussian noise, nonlinear relationships, and time-varying systems (DYNOTEARS). However, each extension introduces its own assumptions, and the practitioner needs to verify that the relevant extension matches the data-generating process.

More broadly, NOTEARS and its successors are sensitive to distributional assumptions in ways that PC and GES are not. Published critiques have shown that under specific conditions — correlated noise, certain non-Gaussian distributions, data-generating processes that violate the linearity assumption without using the appropriate extension — NOTEARS can return spurious edges or incorrect orientations with high confidence. The algorithm's continuous-optimization framing makes it harder to inspect the reasoning behind any specific edge than with PC (where each edge can be traced to a specific independence test) or GES (where each edge can be traced to a score improvement).

**When to use it.** NOTEARS and its variants are the default when the number of variables exceeds the range where combinatorial methods are feasible — roughly above a hundred variables, depending on sample size and computational budget. High-dimensional genomics data, large-scale organizational behavior data, and systems with comprehensive sensor coverage are typical use cases. In these settings, the choice is not between NOTEARS and a more reliable algorithm; it is between NOTEARS and no structure learning at all. The appropriate response to NOTEARS's sensitivity to distributional assumptions is careful verification: run the algorithm, then check the output against expert knowledge, against PC or GES on a reduced variable set, and against held-out data.

<!-- → [DIAGRAM: Two-panel conceptual illustration of the NOTEARS approach. Left panel: "The combinatorial explosion." A vertical bar chart showing number of possible DAGs vs. number of variables: n=5 (1,024), n=10 (4.2 trillion), n=20 (astronomical — bar extends off the chart). Annotation: "Combinatorial search infeasible above ~20 variables for exhaustive methods." Right panel: "The continuous-optimization reframing." A schematic showing (1) a real-valued adjacency matrix W with some cells shaded (nonzero) and some empty (zero = no edge); (2) arrows pointing to two boxes: "Loss function: how well does W fit the data?" and "Acyclicity constraint: tr(e^{W∘W}) − n = 0"; (3) a gradient descent curve converging to a minimum. Caption: "NOTEARS turns a combinatorial search into a calculus problem — at the cost of distributional assumptions the other algorithms do not make."] -->

![Figure 17.4 — Two-panel conceptual illustration of the NOTEARS approach. Left panel: "The combinatorial explosion." A vertical bar chart showing number of possible DAGs vs. number of variables: n=5 (1,024), n=10 (4.2 trillion), n=20 (astronomical](images/17-resolving-the-graph-fig-04.jpg)


### FCI: Discovery Under Latent Confounding

All three algorithms discussed so far assume causal sufficiency — no unmeasured common causes. This assumption is convenient but often wrong. In clinical settings, patient characteristics that are not measured (genetic factors, lifestyle habits, unmeasured comorbidities) commonly affect multiple observed variables. In organizational settings, culture, leadership quality, and historical context are structural influences that rarely appear in the data. In social science, background characteristics of individuals and communities shape outcomes in ways that no survey fully captures.

FCI — Fast Causal Inference, developed by Spirtes and colleagues as an extension of PC — abandons the causal sufficiency assumption and handles latent confounders explicitly.

**What it does.** FCI extends the PC skeleton-discovery phase to account for the possibility of latent variables. The key observation is that a latent confounder affecting two observed variables will produce a specific pattern of conditional independence failures that differs from the pattern produced by a direct causal relationship. FCI uses augmented independence tests that can distinguish these patterns, producing a skeleton that is correct even when latent confounders are present.

FCI's output is not a CPDAG but a *Partial Ancestral Graph* (PAG). A PAG uses a richer set of edge marks than a DAG. In a DAG, an edge either exists or does not, and if it exists it has a direction. In a PAG:

- A directed edge $X \rightarrow Y$ means X causes Y, with no latent confounder on this relationship.
- A bidirected edge $X \leftrightarrow Y$ means X and Y share a latent common cause — a confounder that is not in the variable set.
- A circle-mark on an edge endpoint means the algorithm cannot determine, from the data, whether the endpoint represents a direct cause, a latent common cause, or a selection bias.

The PAG is honest about what the data can and cannot resolve. It does not force orientation decisions that are not supported; it represents uncertainty explicitly.

**What it assumes.** FCI drops causal sufficiency but retains a different assumption: *acyclicity* (no feedback loops) and a weaker version of the faithfulness assumption (the independence relationships in the data accurately reflect the causal structure). FCI also assumes that any latent confounders, if present, have the independence structure that its augmented tests are designed to detect. When these assumptions hold, FCI's PAG is a correct representation of everything that can be determined from the observational data about a system with latent confounders.

FCI is computationally more expensive than PC — the augmented tests it requires are more complex — and its PAG output is harder to use directly for downstream analysis than a CPDAG. The PAG must typically be committed to a specific directed graph (with explicit assumptions about the latent structure) before it can support counterfactual queries.

**When to use it.** FCI is the default in any domain where unmeasured common causes are likely and the practitioner wants the algorithm to be honest about them rather than confidently wrong. Clinical medicine is the paradigmatic case. Organizational behavior, social science, and any domain involving human subjects where there are relevant personal characteristics not captured in the data are also strong candidates. The cost of FCI's complexity and its harder-to-use output is the price of honesty in the presence of latent confounding.

<!-- → [DIAGRAM: Side-by-side comparison of PC and FCI output on the same four-variable dataset containing a known latent confounder L (unmeasured) that affects both X1 and X3. Left panel (PC output): CPDAG showing a direct edge X1 → X3, oriented incorrectly because PC interprets the correlation induced by L as a direct relationship. The latent L is shown in a dashed box above the graph with a note "L not in variable set — PC does not know it exists." Right panel (FCI output): PAG showing a bidirected edge X1 ↔ X3 with a label "latent common cause," and circle-marks on two other endpoints where direction is uncertain. Annotation: "PC is confidently wrong. FCI is honestly uncertain." Caption: "FCI's more complex output — bidirected edges, circle marks — is the price of operating correctly when causal sufficiency fails."] -->

![Figure 17.5 — Side-by-side comparison of PC and FCI output on the same four-variable dataset containing a known latent confounder L (unmeasured) that affects both X1 and X3. Left panel (PC output): CPDAG showing a direct edge X1 → X3, oriented incorrectly because PC interprets the correlation induced by L as a direct relationship. The latent L is shown in a dashed box above the graph with a note "L not in variable set](images/17-resolving-the-graph-fig-05.jpg)


### Choosing Among the Four

The four algorithms are not ranked — there is no universally best algorithm. The choice is always a function of the domain, the data, and the assumptions the practitioner is willing to make and defend.

The decision logic follows from the assumption profiles:

- **Start with FCI** when latent confounders are likely and the data volume is moderate. The PAG output will be richer and more honest than a CPDAG, even if harder to use.
- **Use PC** when causal sufficiency is defensible, the variable count is under roughly a hundred, and you want the transparency of edge-by-edge independence tests.
- **Use GES** when causal sufficiency is defensible and you want the defensibility of a principled scoring criterion — particularly in regulated settings where the model-selection procedure will be audited.
- **Use NOTEARS** (or a variant) when the variable count is too large for combinatorial methods, accepting that the output requires careful verification against expert knowledge.
- **Run multiple algorithms and compare** when the stakes are high enough to justify the computational cost. Consistent structure across algorithms is much stronger evidence than structure from any single one.

<!-- → [DIAGRAM: Decision tree for algorithm selection. Root node: "Are latent confounders likely in this domain?" Left branch (Yes) → FCI. Right branch (No) → "How many variables?" → Sub-branch: Under ~100 → "Is a defensible scoring criterion required (e.g., regulatory audit)?" → Yes → GES; No → PC. Sub-branch: Over ~100 → NOTEARS (with verification note). Vertical annotation on the right side of the tree, spanning all branches: "In high-stakes settings: run multiple algorithms and compare. Consistent structure = stronger evidence." Each leaf annotated with its primary limitation. Reader should trace any domain description through the tree to a recommended starting point.] -->

![Figure 17.6 — Decision tree for algorithm selection. Root node: "Are latent confounders likely in this domain?" Left branch (Yes) → FCI. Right branch (No) → "How many variables?" → Sub-branch: Under ~100 → "Is a defensible scoring criterion required (e.g., regulatory audit)?" → Yes → GES](images/17-resolving-the-graph-fig-06.jpg)


---

## Concept 2 — The CPDAG Handoff and Disagreement Resolution

### Three Patterns at the Handoff

The elicitation from Chapter 16 produces a CPDAG; the algorithms from Concept 1 produce one too. These two CPDAGs are the inputs to the handoff. The practitioner's job is to compare them and determine what to do with the comparison. Three patterns are possible.

**Agreement.** The directed edges the expert committed on are confirmed by the algorithm — the algorithm's independence tests or scoring procedure found the same structure. The undirected edges in the expert's CPDAG are oriented by the algorithm in a way the expert finds plausible when shown the result. Agreement is the best case, and it is more common than skeptics expect. A domain expert who has thought carefully about causal mechanisms will often produce a CPDAG that the data confirms. When agreement is reached, the team can proceed to parameterization with high confidence in the structure.

**Refinement.** The algorithm produces a graph consistent with the expert's commitments but adds specificity where the expert was uncertain. The expert was unsure whether the relationship between A and B was direct or mediated through C; the algorithm finds that C is not a collider on the A–B path and that the edge is direct. Refinement is the usual case, and it is what justifies running the algorithms at all. The algorithms do the work that the expert couldn't do because it requires statistical computation rather than domain knowledge.

**Disagreement.** The algorithm contradicts a specific commitment from the expert. She said A causes B; the algorithm finds the conditional independence pattern more consistent with B causes A, or with both being effects of an unmeasured third variable. Disagreement is the most diagnostically valuable pattern, because it surfaces a conflict that has to be resolved before the graph can be trusted.

<!-- → [DIAGRAM: Three-panel illustration of the CPDAG handoff patterns. Each panel shows two CPDAGs side by side — "Expert CPDAG" (left) and "Algorithm CPDAG" (right) — connected by a comparison arrow. Panel 1 (Agreement): edges match; directed edges aligned; undirected edges in expert CPDAG are now directed in the algorithm output in the same direction. Annotation: "Proceed to parameterization." Panel 2 (Refinement): expert has one undirected edge A—B; algorithm orients it as A→B; expert's directed edges are confirmed. Annotation: "Accept refinement; confirm with expert; document." Panel 3 (Disagreement): expert has A→B; algorithm shows B→A. The conflicting edge is highlighted in red. Annotation: "Apply three-step resolution protocol." Reader should see that the three patterns call for different next actions.] -->

![Figure 17.7 — Three-panel illustration of the CPDAG handoff patterns. Each panel shows two CPDAGs side by side](images/17-resolving-the-graph-fig-07.jpg)


### The Three-Step Resolution Protocol

Not all disagreements are equal, and not all should be resolved the same way. The three-step protocol distinguishes the cases.

**Step 1: Test the algorithm's claim for stability.** Disagreements that evaporate under mild perturbation are not strong evidence. Run the algorithm with different hyperparameters — different significance thresholds for independence tests, different penalty parameters for the BIC, different random seeds for bootstrap subsamples of the data. If the contested edge orientation is stable across these variations, the algorithm's claim is well-supported by the data. If it flips depending on the choices, the algorithm is operating near the edge of its power: the data is not strong enough to determine orientation, and the specific output is an artifact of the specific choices made.

An unstable disagreement is resolved in favor of the expert by default. The algorithm's claim is that the edge should point one way; the expert's claim is that it should point the other; the data is not strong enough to adjudicate. In this case, the expert's domain knowledge is the more reliable source, and the edge should be oriented as the expert committed.

A stable disagreement — one that persists across algorithm choices — requires the next step.

**Step 2: Examine the expert's commitment for bias.** Chapter 15 documented the most common reasoning biases in expert elicitation. Was the expert reasoning from correlation rather than mechanism? Did she simplify a feedback loop into a directed edge because the feedback loop is harder to articulate? Did she import a causal story from a different domain where the mechanism runs in the opposite direction? Did she commit on direction primarily to avoid admitting uncertainty — the "forced commitment" problem that a careful facilitator should have caught?

If the expert's commitment has a recognizable bias signature, the algorithm's stable contradiction is credible. The expert may be wrong for a specific reason that can be surfaced and corrected.

If the expert's commitment shows no obvious bias — if she had genuine domain-specific reasons for the direction she specified, reasons that are not captured in the data — then the conflict is more substantive.

**Step 3: Run a targeted second elicitation.** If the disagreement persists after both checks — stable across algorithm choices, no recognizable bias in the expert's reasoning — schedule a focused session, typically ten to twenty minutes, addressing the disagreement directly. Present the conflict without prejudging its resolution: *the algorithm finds this conditional independence pattern in the data; you committed to this edge direction; can you help us understand what would explain the discrepancy?*

This targeted session often surfaces something that the first elicitation did not. A variable the model is missing. A temporal nuance — the mechanism runs one direction in the short run and the opposite in the long run. A subpopulation effect — in the data, which is dominated by one group, the relationship looks different than the expert's knowledge, which is informed by across-group reasoning. The second session is explicitly about the disagreement, so it draws out the considerations most relevant to resolving it.

The output of the second session is either a revised commitment — the expert, seeing the data pattern, updates her view — or a documented note that the disagreement remains and the team has chosen one orientation as the working assumption while flagging the alternative for sensitivity analysis. Either outcome is acceptable. What is not acceptable is leaving the disagreement unresolved and the graph in an ambiguous state.

<!-- → [DIAGRAM: Flowchart of the three-step resolution protocol. Entry point: "Algorithm contradicts expert commitment on edge X→Y." Step 1 box: "Test stability — rerun algorithm with varied hyperparameters and bootstrap subsamples." Two exits: (1) "Unstable: orientation flips across runs" → resolution box "Resolve in favor of expert. Data too weak to adjudicate. Document." (2) "Stable: orientation consistent" → Step 2. Step 2 box: "Examine expert commitment for bias (Chapter 15 taxonomy: correlation reasoning, feedback loop simplification, cross-domain import, forced commitment)." Two exits: (1) "Bias detected" → "Algorithm's stable contradiction is credible. Facilitate expert revision." (2) "No recognizable bias" → Step 3. Step 3 box: "Run targeted second elicitation (~15 min). Present data pattern and expert commitment. Ask: 'What would explain the discrepancy?'" Two exits: (1) "Expert updates view" → "Revise edge. Document revised commitment." (2) "Expert holds" → "Document disagreement. Choose working assumption. Flag for sensitivity analysis." Reader should trace any disagreement to a specific resolution action.] -->

![Figure 17.8 — Flowchart of the three-step resolution protocol. Entry point: "Algorithm contradicts expert commitment on edge X→Y." Step 1 box: "Test stability](images/17-resolving-the-graph-fig-08.jpg)


---

## Concept 3 — More Data or More Experts?

When the graph is not yet validated — undirected edges remain in the critical path, or unresolved disagreements persist — the team faces a resource allocation question. More data collection or more expert sessions? The answer depends on a diagnosis that most teams skip, defaulting instead to whatever they are already good at.

### The Aleatory / Epistemic Distinction

The distinction is philosophically old but practically essential. *Aleatory uncertainty* is uncertainty about the value of a quantity that is, in principle, determined by data. The outcome of a coin flip is aleatory — we don't know what it will be before the flip, but after the flip, there is a fact of the matter. With more observations, we can estimate the coin's bias with arbitrary precision. More data resolves aleatory uncertainty.

*Epistemic uncertainty* is uncertainty about what the right model is — about the structure of the situation, not just the parameters. If two causal models are Markov-equivalent, they imply the same conditional independence structure and therefore the same data distribution. No amount of observational data will ever distinguish between them; there is no observation we could make that is more consistent with one than the other. The uncertainty is not about the value of a quantity; it is about which of two structurally different worlds we are in. Observational data cannot resolve it. Experimental intervention can (by breaking the equivalence); expert knowledge about mechanism can (by specifying which direction the arrow runs on mechanistic grounds rather than probabilistic ones).

**Undirected edges in a CPDAG can arise from both sources.** Some undirected edges are undirected because of Markov equivalence — flipping their direction would produce an equivalent graph, and the data is equally consistent with both. This is epistemic. Some undirected edges are undirected because the data lacks power — a dependence exists, but the sample is too small to detect it reliably. This is aleatory.

The diagnostic for which is which: bootstrap the data (resample with replacement, run the algorithm on each resample) and measure how stable the edge's orientation is. If the edge flips with roughly equal frequency in both directions across bootstrap samples, the data has no systematic signal about orientation — likely epistemic. If the edge is consistently oriented one way but with low confidence (low test statistics, marginal score improvements), the signal exists but is weak — likely aleatory, with more data the remedy.

<!-- → [CHART: Bootstrap orientation frequency chart for three example edges. X-axis: orientation frequency (0% to 100%, where 50% = equal split). Y-axis: three edge labels (A—B, C—D, E—F). For each edge, a horizontal bar spans from the left orientation frequency to the right orientation frequency, with a center marker at 50%. Edge A—B: bar spans 48%–52% — nearly perfectly split; labeled "Epistemic — Markov equivalent alternatives." Edge C—D: bar spans 85%–15%; labeled "Aleatory — systematic signal, low power." Edge E—F: bar spans 95%–5%; labeled "Strong signal — likely orientable with current data." Callout: "The position of the bar relative to 50% is the diagnostic." Reader should use this chart type to read any bootstrap orientation result.] -->

![Figure 17.9 — Bootstrap orientation frequency chart for three example edges. X-axis: orientation frequency (0% to 100%, where 50% = equal split). Y-axis: three edge labels (A—B, C—D, E—F). For each edge, a horizontal bar spans from the left orientation frequency to the right orientation frequency, with a center marker at 50%. Edge A—B: bar spans 48%–52%](images/17-resolving-the-graph-fig-09.jpg)


### When More Data Is the Right Move

More data resolves aleatory uncertainty. The specific situations where it is the right investment:

The skeleton of the graph is unstable. If different bootstrap subsamples of the current data produce different sets of edges — not just different orientations, but different adjacencies — the issue is that the independence tests have insufficient power to determine which edges exist. This instability is aleatory: with enough data, the skeleton will converge. More data is the right investment.

Parameters are imprecise. Once the structure is fixed, additional data refines the conditional distributions at each node. If the model's counterfactual predictions are sensitive to parameter uncertainty — if small changes in the estimated effect of A on B substantially change the ranking of interventions — targeted data collection on the conditionally underestimated relationships is worth the investment.

A specific subgroup is underrepresented. If the question of interest concerns a particular segment and the existing data is thin on that segment, more targeted data collection — acquiring more observations from the relevant segment — can resolve uncertainties specific to it without affecting the rest of the model.

### When More Expert Sessions Are the Right Move

More expert input resolves epistemic uncertainty. The specific situations:

Edges remain undirected after the algorithm has run and the bootstrapping suggests epistemic uncertainty. The algorithm has done what data can do. Markov equivalence has been reached. An expert who knows the mechanism — who has seen, in some other context, that this relationship runs one way rather than the other — can commit on direction in a way that the data never will.

The data and the expert disagree stably and the disagreement is structural. The three-step protocol says to run another expert session. This is the situation it was designed for.

The domain has thin data relative to the strength of expert knowledge. Emerging markets, novel product categories, organizational transitions — these are domains where observational data is scarce and the available data reflects early-phase conditions that may not persist. An expert with cross-domain knowledge or experimental intuition carries more information than a sparse dataset can.

Expert sessions are cheaper than the relevant data collection. In many corporate settings, data of the kind needed to resolve a specific ambiguity requires compliance review, system migrations, or expensive sampling. A two-hour expert session followed by twenty minutes of targeted follow-up may resolve the ambiguity more efficiently.

### The Framework in Practice

The diagnostic that determines which investment to make:

1. For each unresolved ambiguity (undirected edge or persistent disagreement), bootstrap the data and examine the stability of the algorithm's output on that specific edge.
2. Classify each ambiguity: stable orientation (epistemic — expert sessions), unstable with systematic signal (aleatory — more data), unstable without systematic signal (epistemic — expert sessions).
3. For aleatory ambiguities, estimate whether the required additional data is feasible to collect given the project's timeline and budget.
4. For epistemic ambiguities, schedule targeted expert sessions focused on the specific unresolved edges.
5. If the diagnosis is uncertain, run a small parallel investment — a targeted data analysis and a short expert session on the same ambiguity — and see which produces more clarity.

The framework is not a formula. Data teams default to more data; consulting teams default to more expert sessions. Mature deployments resist these defaults and choose based on the diagnostic.

<!-- → [DIAGRAM: Decision tree for the more-data / more-experts diagnostic. Entry: "Unresolved ambiguity in the graph (undirected edge or persistent disagreement)." Step 1 box: "Bootstrap the data. Examine orientation stability for the specific edge." Three branches: (1) "~50/50 split — no systematic signal" → "Epistemic uncertainty (Markov equivalence). More data will not resolve. → Expert session or controlled experiment." (2) "Systematic lean (e.g., 70–90%) — signal present but weak" → "Aleatory uncertainty (power). More data targeted at this conditional distribution." (3) "Strong lean (>90%)" → "Likely resolvable — consider accepting current orientation with documentation." Bottom note: "Uncertain which? Run both in parallel: targeted data analysis + short expert session. See which produces clarity first." Reader should use this tree as a field diagnostic.] -->

![Figure 17.10 — Decision tree for the more-data / more-experts diagnostic. Entry: "Unresolved ambiguity in the graph (undirected edge or persistent disagreement)." Step 1 box: "Bootstrap the data. Examine orientation stability for the specific edge." Three branches: (1) "~50/50 split](images/17-resolving-the-graph-fig-10.jpg)


---

## Concept 4 — The Validated Graph as a Living Artifact

### The Static Document Trap

The work of Concepts 1 through 3 produces a validated, fully directed graph — the CPDAG resolved, the disagreements adjudicated, the structure supported by both data and expert knowledge. This graph is, at the moment of validation, the best available structural model of the system. It should not be filed.

The trap that many organizations fall into is treating the validated graph as a deliverable. A project produced it; the project ended; the graph is archived in a shared drive. A year later, the business has changed, the data has shifted, a new product line has been added, a market entrant has restructured competitive dynamics. The graph no longer accurately represents the system. But because it is archived rather than governed, nobody is monitoring for drift, nobody has been assigned responsibility for updating it, and the Living Model built on it is silently degrading.

The analogy to software governance is deliberate. Software code is not filed when it ships; it is checked into version control, monitored in production, updated when the environment changes, and retired when it is no longer fit for purpose. The validated causal graph deserves the same treatment — not because the graph is code, but because it has the same properties: it was carefully built, its value depends on its accuracy, its accuracy degrades as the world changes, and the degradation is invisible unless actively monitored.

### Version Control

The graph is checked into a repository. Not just the current version: the full history of changes, each associated with the evidence that justified it, the person or process that made it, and the date it went live.

The commit message for a graph change is not "updated graph." It is "re-elicited the relationship between activation_flow and retention based on Q3 2024 expert session with VP Growth; previous direction (activation → retention) reversed to (retention_intent → activation) based on new evidence from feature instrumentation experiment; see session transcript reference 2024-Q3-17." The specificity is not pedantic; it is what makes the graph auditable.

Version control serves two functions. Practically, it allows rollback: if a graph change produces unexpected outputs from the Living Model, the team can revert to the prior version while the change is investigated. Analytically, it provides the provenance trail: when a model recommendation is challenged, the team can trace every structural commitment to specific evidence, showing that the graph was not produced by arbitrary choices but by documented reasoning.

### Drift Monitoring

As new data accumulate after the graph is validated, the data starts to imply structures that may differ from the validated graph. Drift monitoring periodically reruns the discovery algorithms against the latest data and compares the result to the validated graph.

The comparison can be quantified. The *structural Hamming distance* counts the number of edges that differ between two graphs — edges present in one but not the other, or edges present in both but oriented differently. The *Jaccard distance* measures the fraction of the union of edge sets that is in the symmetric difference (the edges that differ). Both metrics range from 0 (identical graphs) to 1 (completely different graphs). When the distance between the validated graph and the algorithm's output on the latest data exceeds a threshold, the monitoring system alerts the team that a re-elicitation may be warranted.

The threshold is tunable. A low threshold is sensitive: it will catch small drifts early but will also trigger false alarms when the drift is noise rather than signal. A high threshold is specific: it will only fire when the drift is large, but by then the model may have been silently degrading for some time. The right calibration depends on the cost of re-elicitation (cheap → low threshold) and the cost of operating on a stale graph (high → low threshold).

Drift monitoring is not the same as model performance monitoring. Model performance monitoring asks: *are the model's predictions accurate?* Drift monitoring asks: *is the structure the model assumes still the right structure?* Both are necessary. Performance degradation often signals structural drift, but not always — performance can degrade for parametric reasons (the conditional distributions have shifted within the same structure) even when the structure is still correct. Conversely, structure can drift in ways that do not immediately surface as performance degradation if the new and old structures make similar predictions on the current data distribution.

<!-- → [CHART: Time-series line chart showing structural drift monitoring over 24 months. X-axis: time (months since validation). Y-axis: structural Hamming distance between validated graph and algorithm output on latest data (0 = identical, higher = more different). The line starts near 0 and rises gradually. Two horizontal reference lines: a low threshold (labeled "Review threshold — triggers re-elicitation review") and a high threshold (labeled "Critical threshold — model likely stale"). The line crosses the review threshold at month 14 (annotated: "re-elicitation scheduled") and is shown returning to near 0 after the re-elicitation at month 16. Annotation: "Drift monitoring is structural, not performance-based. A model can drift structurally before performance degrades."] -->

![Figure 17.11 — Time-series line chart showing structural drift monitoring over 24 months. X-axis: time (months since validation). Y-axis: structural Hamming distance between validated graph and algorithm output on latest data (0 = identical, higher = more different). The line starts near 0 and rises gradually. Two horizontal reference lines: a low threshold (labeled "Review threshold](images/17-resolving-the-graph-fig-11.jpg)


### Re-Elicitation Triggers

Drift monitoring is one trigger for re-elicitation. Others:

*Material business change.* A new product line, a market entry, a major organizational restructuring, the departure of a key market player — any of these may invalidate parts of the graph by changing the mechanisms it represents. These triggers should be connected to the governance process explicitly: the business team responsible for the Living Model should have a protocol for notifying the modeling team when material changes occur.

*Scheduled periodic review.* Even in the absence of specific triggers, a graph should receive a review on a regular schedule — annually at minimum, semiannually for fast-moving domains. The review is shorter than the original elicitation (focused on the parts of the graph most likely to have changed) but follows the same protocol.

*Anomalous model outputs.* If the Living Model begins producing recommendations that surprise the business team — suggestions that seem wrong, rankings that contradict recent experience — this may indicate structural drift that drift monitoring has not yet caught. Anomalous outputs should trigger a structural review before the parametric investigation that would be the first response to performance degradation.

### Auditability

The graph, its history, and its provenance are queryable. When a recommendation produced by the Living Model is challenged — by a board member, a regulator, a legal proceeding — the team can trace the recommendation to the structural assumptions that produced it, and trace each structural assumption to the specific evidence that established it.

The audit trail answers: *why does the model recommend this?* The answer, fully traced, is: *because the model assumes this structure; the structure was established by these experts and this data; here is the transcript of the expert session; here is the independence test output; here are the bootstrap stability results; here is the drift monitoring log showing the structure has been stable since it was validated.* This level of traceability is what makes the model defensible in high-stakes settings, and it is what distinguishes a Living Model from a black box that produces recommendations nobody can explain.

<!-- → [DIAGRAM: Governance lifecycle — circular flow diagram with five labeled nodes connected by clockwise arrows. Node 1: "Expert Elicitation (Ch. 16)" — artifact produced: initial CPDAG. Node 2: "Algorithm Validation + Resolution Protocol (Ch. 17)" — artifact produced: validated DAG. Node 3: "Version Control" — artifact produced: versioned graph with provenance log. Node 4: "Drift Monitoring" — artifact produced: Hamming/Jaccard distance time-series; alert when threshold exceeded. Node 5: "Re-Elicitation Triggers" — triggers: drift threshold / material business change / scheduled review / anomalous outputs → loops back to Node 1 (targeted re-elicitation). Each arc annotated with what happens at that transition and how long it typically takes. Large caption centered below: "The graph is not delivered. It is governed." Reader should see governance as a continuous cycle, not a one-time project.] -->

![Figure 17.12 — Governance lifecycle](images/17-resolving-the-graph-fig-12.jpg)


---

## Integration — From Interview to Instrument

The chapter has moved through four concepts: the four algorithm families and when to use them; the CPDAG handoff and the three-step protocol for resolving disagreements; the aleatory/epistemic diagnostic for deciding whether to gather more data or run more expert sessions; and the governance practices that keep the validated graph trustworthy over time.

Let me now show how these pieces connect in the workflow that produces an instrument the Living Model can actually use.

The starting point is the CPDAG from the Chapter 16 elicitation. The first decision is which algorithm to run on the data, and the answer follows from the domain assessment: is causal sufficiency plausible? How many variables? What are the sample sizes? The algorithm produces a second CPDAG. The two are compared.

Where they agree, the team can proceed with confidence. Where the algorithm refines the expert's undirected edges, the team accepts the refinement, shows it to the expert for confirmation, and documents it. Where they disagree, the three-step protocol is applied: test stability, examine expert commitment for bias, run a targeted second session if needed.

At the end of this process, the team has a graph in one of three states. Best case: fully directed, agreed on by expert and data, ready for parameterization. Middle case: fully directed but with some edges flagged for sensitivity analysis — the team chose a working assumption on a disagreement and has documented the alternative. Acceptable case: still contains undirected edges, but only in parts of the graph that are not on the critical path for the current question — the team has documented the residual ambiguity and can proceed to parameterization while noting the limitation.

The graph is then checked into version control with full provenance. Drift monitoring is configured at a threshold calibrated to the domain. Re-elicitation triggers are connected to the business team. The instrument is live.

From this point, the workflow is maintenance rather than construction. The data accumulates; the drift monitoring runs; the business team reports material changes; the periodic review schedule fires. Each event is a potential input to a re-elicitation cycle. Each re-elicitation produces an update to the graph, committed with provenance. The instrument stays current.

This is what distinguishes a Living Model from a static analysis. The static analysis produces a graph for a specific question at a specific moment. The Living Model produces a governed graph that evolves with the system it represents — as reliable, at any given moment, as the care that has been invested in keeping it current.

<!-- → [DIAGRAM: End-to-end workflow diagram — horizontal flow from left (input) to right (output). Left: "Expert CPDAG from Ch. 16 elicitation." Arrow → "Algorithm selection (decision tree from Concept 1)." Arrow → "Run algorithm; produce algorithm CPDAG." Arrow → "Handoff comparison: agreement / refinement / disagreement?" → For disagreement: "Three-step resolution protocol (Concept 2)." → For remaining ambiguity: "Aleatory/epistemic diagnostic (Concept 3) → more data or more experts?" → "Validated DAG." Arrow → "Version control + provenance." Arrow → "Drift monitoring + re-elicitation triggers (Concept 4)." Final node at right: "Living Model instrument — continuously governed." Beneath the linear flow: a return arrow from "re-elicitation triggers" back to "Expert CPDAG," representing the governance cycle. Caption: "From interview to instrument: a structured workflow with no shortcuts."] -->

![Figure 17.13 — End-to-end workflow diagram](images/17-resolving-the-graph-fig-13.jpg)


---

## Exercises

### Warm-Up

**1. Match the algorithm to its paradigm.**
*(Tests Objective 1 — describe the four algorithm families)*
*(Difficulty: Low)*

For each description, name the algorithm it describes (PC, GES, NOTEARS, or FCI) and identify its paradigm (constraint-based, score-based, continuous-optimization, or latent-variable-aware). Explain in one sentence what the algorithm returns.

a. An algorithm that begins from a fully connected graph and removes edges by testing for conditional independence, then orients the remaining edges by detecting collider structures.
b. An algorithm that searches over graph structures by greedily adding and then removing edges, selecting the structure that maximizes a penalized likelihood score.
c. An algorithm that parameterizes the graph as a real-valued adjacency matrix and solves for structure using continuous optimization with a differentiable acyclicity constraint.
d. An algorithm that extends constraint-based discovery to handle latent confounders, returning a Partial Ancestral Graph with bidirected edges representing unmeasured common causes.

---

**2. Identify the assumption violation.**
*(Tests Objective 2 — select appropriate algorithm given domain and assumptions)*
*(Difficulty: Low)*

For each scenario, identify which of the four algorithms' core assumptions is most likely violated, and explain the consequence for the algorithm's output.

a. A team runs PC on electronic health record data from a hospital, where patients' genetic backgrounds, socioeconomic status, and lifestyle factors are not recorded but influence many measured outcomes.
b. A team runs GES on customer behavior data from a product with 800 tracked event types and 50,000 user observations.
c. A team runs NOTEARS on survey data where all variables are binary (yes/no responses) and the relationships between them are not well-described by a linear model.
d. A team runs FCI on a clean manufacturing dataset with 12 sensor readings, comprehensive process monitoring, and no plausible unmeasured common causes.

---

**3. Read the output.**
*(Tests Objective 1 — describe what each algorithm returns)*
*(Difficulty: Low)*

A team runs PC on a dataset with five variables (A, B, C, D, E) and receives the following CPDAG: A → B, B — C (undirected), B → D, D → E, C — E (undirected), A → D.

a. Which edges has the algorithm oriented? What does an oriented edge mean in the context of the PC algorithm?
b. Which edges are undirected? Name two distinct reasons why an edge might remain undirected in a PC output.
c. The team needs to answer the question: "What is the effect of changing B on E?" Is the graph in a state that supports this counterfactual query? What additional information is needed, and what would provide it?

---

### Application

**4. Apply the resolution protocol.**
*(Tests Objectives 3 and 4 — diagnose disagreements and apply the three-step protocol)*
*(Difficulty: Medium)*

A team elicits a CPDAG from a domain expert (VP of Marketing) for a model of customer conversion. The expert commits: *Ad_Exposure → Purchase_Intent → Conversion*. After running GES on twelve months of behavioral data, the algorithm's output shows: *Purchase_Intent → Ad_Exposure → Conversion* — the direction of the first edge is reversed.

a. Describe the disagreement precisely: what exactly are the expert and the algorithm claiming about the relationship between Ad_Exposure and Purchase_Intent?
b. Apply Step 1 of the resolution protocol. What would you do, and what outcome would lead you to resolve in favor of the expert versus proceed to Step 2?
c. Apply Step 2. What specific bias from the Chapter 15 taxonomy (feedback loop simplification, cross-domain import, forced commitment) might explain the expert's direction? What evidence would confirm or disconfirm each?
d. Suppose Steps 1 and 2 leave the disagreement unresolved. Design the targeted second elicitation for Step 3: what would you present to the expert, what question would you ask, and what two outcomes are possible?

---

**5. Diagnose data vs. expert.**
*(Tests Objective 4 — determine aleatory vs. epistemic uncertainty)*
*(Difficulty: Medium)*

A causal discovery team has run PC on a dataset and received a CPDAG with three undirected edges: A — B, C — D, and E — F. They bootstrap the data (1,000 resamplings) and observe the following orientation frequencies:

- A — B: A → B appears in 51% of bootstrap samples; B → A appears in 49%.
- C — D: C → D appears in 88% of bootstrap samples; D → C appears in 12%.
- E — F: E → F appears in 95% of bootstrap samples; F → E appears in 5%.

a. For each edge, classify the uncertainty as primarily aleatory or epistemic based on the bootstrap results. Explain your reasoning.
b. What investment is recommended for each edge — more data, more expert sessions, or a controlled experiment?
c. For the edge you classified as aleatory, describe specifically what kind of additional data would most efficiently resolve the uncertainty.
d. For the edge you classified as epistemic, describe the targeted expert session you would run — what you would present, what question you would ask, and why an expert can provide information the data cannot.

---

**6. Select the algorithm.**
*(Tests Objective 2 — select appropriate algorithm)*
*(Difficulty: Medium)*

For each domain description, select the most appropriate algorithm (PC, GES, NOTEARS, FCI, or a combination) and justify the selection in terms of the domain's assumption profile, variable count, and sample characteristics.

a. A pharmaceutical company wants to discover causal structure in a clinical dataset with 30 biomarkers measured on 2,000 patients. The team knows that patient genetics, which are not measured, influence several of the biomarkers.
b. A tech company wants to discover causal structure in a product telemetry dataset with 600 logged event types and 2 million user sessions. The team believes the system is causally sufficient (all relevant variables are measured) and needs the algorithm to scale to the variable count.
c. A management consulting team wants to discover causal structure in an organizational effectiveness dataset with 15 survey variables and 400 organizational units. The dataset will be presented to a client's board, who will scrutinize the model-selection procedure. The team believes causal sufficiency is approximately valid.
d. A research team is studying social mobility and has data on 25 socioeconomic variables for 5,000 households. They believe that neighborhood culture, family background, and social capital — none of which are measured — are important common causes of many of the observed variables.

---

### Synthesis

**7. Run the full validation workflow.**
*(Tests Objectives 1–4 — integrated application)*
*(Difficulty: High)*

A team is building a Living Model for a retail bank's credit risk system. The domain expert (Chief Risk Officer) has provided a CPDAG with the following committed edges: *Income → Credit_Score*, *Credit_Score → Default_Risk*, *Loan_Size → Default_Risk*, *Income — Employment_Status* (undirected). The bank has five years of loan origination data with 150,000 records and 20 variables. The team suspects that a customer's broader financial habits — not captured in the dataset — may confound several relationships.

a. Given the domain description, which algorithm is most appropriate for the data phase of the validation? Justify your choice in terms of the specific assumption that your reasoning turns on.
b. After running the algorithm, the team finds: the algorithm confirms *Income → Credit_Score* and *Credit_Score → Default_Risk*; it reverses *Loan_Size → Default_Risk* to *Default_Risk → Loan_Size*; it orients the undirected edge as *Employment_Status → Income*. Apply the resolution protocol to the disagreement on Loan_Size and Default_Risk: what is the most plausible bias explanation, what stability test would you run, and what question would you ask in the targeted second session?
c. The undirected edge between Income and Employment_Status has been oriented by the algorithm as *Employment_Status → Income*. The expert, when shown this, says she is not sure — it could go either way. Bootstrap results show 72% support for *Employment_Status → Income*. Is this aleatory or epistemic? What is your recommendation?
d. After resolution, the team finds that the edge between Loan_Size and Default_Risk remains ambiguous — the expert and the algorithm disagree, and the disagreement is stable. The team must proceed. Describe the working assumption you would document and the sensitivity analysis you would run to characterize the risk of being wrong.

---

**8. Design a governance system.**
*(Tests Objective 5 — govern the validated graph as a living artifact)*
*(Difficulty: High)*

A fintech startup has just completed its first validated causal graph for a customer churn model. The graph has 18 variables, 22 edges, and was built through a combination of expert elicitation (three sessions with the Head of Customer Success and Head of Product) and PC algorithm validation on 18 months of behavioral data. The company is growing quickly: user base is doubling annually, the product is releasing new features monthly, and the team that built the model is small (two data scientists, one domain expert who is part-time available).

a. Design the version control protocol for this graph. What information should be captured in each commit? What would the commit message look like for a re-elicitation triggered by the launch of a new onboarding feature?
b. Design the drift monitoring system. What metric would you use to measure structural drift? What threshold would you set, and how would you calibrate it given the company's growth rate and the cost of re-elicitation?
c. Identify the re-elicitation triggers beyond drift monitoring that are appropriate for this company given its growth rate and release cadence. For each trigger, describe the scope of the re-elicitation it should initiate.
d. The original expert (Head of Customer Success) leaves the company after 14 months. The company needs to update the graph for a new product feature, but the new Head of Customer Success has no experience with causal modeling and limited knowledge of the historical reasoning behind the graph's structural commitments. Describe the graph stewardship challenge this creates and propose a practical mitigation.

---

### Challenge

**9. Audit a graph in production.**
*(Tests Objectives 1–5 — full validation and governance skill)*
*(Difficulty: Open-ended)*

You have been asked to audit a Living Model that has been in production for two years at a logistics company. The model supports routing decisions. You are given the following information:

- The graph was built using GES on 18 months of historical data at the time of construction.
- No expert elicitation was performed; the graph was treated as a purely data-driven output.
- The model has not been retrained or re-elicited since initial construction.
- The logistics network has expanded significantly: three new regional hubs have been added, two major routes have been restructured, and the company entered a new market segment.
- The model's recommendation quality has been declining over the past six months according to the operations team, but no formal performance monitoring was in place.

a. Identify every governance failure in the description above, using the vocabulary of Concept 4. For each failure, explain the specific risk it creates.
b. Describe the steps you would take to assess whether the graph is still structurally valid, given that you have access to the current data but no original expert.
c. The operations team wants to know which of the model's recommendations can still be trusted. Propose a practical triage: which parts of the model (which edges, which variable relationships) are most likely to have drifted given the business changes described, and which are most likely still valid?
d. Propose a remediation plan that addresses the governance failures without requiring a full rebuild of the model from scratch. What is the minimum viable governance investment for a model of this kind?

---

**10. Where causal discovery breaks.**
*(Tests Objectives 1–4 at the framework's edges)*
*(Difficulty: Open-ended)*

The chapter presents causal discovery algorithms as powerful tools for resolving ambiguity in expert-elicited graphs. But there are settings where no discovery algorithm — regardless of which one is chosen or how carefully it is applied — can produce a reliable structural result.

a. Markov equivalence places an absolute limit on what observational discovery can achieve. Describe a specific realistic scenario where two Markov-equivalent graphs would lead to completely different intervention recommendations, and explain why no observational algorithm can distinguish between them.
b. The faithfulness assumption — that every conditional independence relationship in the data accurately reflects the causal structure — underlies all four algorithms. Describe a realistic scenario where faithfulness is violated, and explain what this does to the algorithm's output.
c. Causal discovery algorithms are typically evaluated on benchmark datasets where the ground-truth graph is known. Most benchmark datasets are synthetic or from domains with simple, well-understood structure. What concerns does this raise about relying on benchmark performance to evaluate algorithm choice in a complex organizational domain?
d. Given the limits identified in parts (a), (b), and (c), how should a practitioner communicate the confidence level of a data-driven graph to a decision-maker who expects the algorithm to have "found the right answer"?

---

## Chapter Summary

You can now do something you could not do before this chapter: take the CPDAG from the expert elicitation and produce, through a structured process, a validated, governed graph that the Living Model can actually use. The four algorithms give you the computational tools; the resolution protocol gives you the procedure for adjudicating conflicts between the algorithms and the expert; the aleatory/epistemic diagnostic gives you the framework for deciding where to invest further; and the governance practices give you the discipline to keep the result trustworthy over time.

The one idea from this chapter that matters most: **the validated graph is not a deliverable — it is a governed artifact.** Every structural commitment in it has provenance. Every edge was committed on specific evidence, by a specific person or process, on a specific date. The graph is monitored for drift, connected to re-elicitation triggers, and auditable when its conclusions are challenged. Without governance, the modeling investment degrades silently. With governance, it compounds.

The common mistake to watch for: treating algorithm output as ground truth. The algorithms do not find the true causal structure; they find the structure most consistent with the data under their specific assumptions. When those assumptions are violated — when causal sufficiency fails, when the data is non-Gaussian, when the sample is small relative to the variable count — the output is wrong with no announcement. The algorithm's job is to provide evidence; the practitioner's job is to evaluate that evidence against expert knowledge, bootstrap stability, and cross-algorithm consistency.

The Feynman test: can you explain to a colleague who has never seen a causal discovery algorithm exactly what PC does — step by step, in plain language — and then explain why it would fail on their specific dataset, using the specific assumption it requires that their domain violates?

---

## Connections Forward

Chapter 18 takes the validated graph and produces ranked recommendations. With structure in hand, the next task is parameterization: estimating the conditional distributions at each node from the data. Parameterization turns the structural skeleton into a quantitative model capable of answering specific intervention questions. Chapter 18 then runs the counterfactual machinery from Chapter 9 against the parameterized model, ranks candidate interventions by their expected causal effect, and translates the ranking into a portfolio of decisions under the constraints the organization faces — budget, capacity, risk tolerance, regulatory limits. The validated graph from this chapter is the input to that process; the quality of the recommendations it produces depends directly on the quality of the structure the graph encodes.

---

*What would change my mind:* A demonstration that one of the four algorithm families — most likely a successor to NOTEARS in the continuous-optimization paradigm — reliably produces directed acyclic graphs of comparable quality to expert-validated CPDAGs across a wide range of domains, without requiring expert input. The mathematical results on Markov equivalence make this hard, but additional assumptions (non-Gaussian noise, specific structural priors) can sometimes resolve equivalence in ways that look promising. I have not seen the resolution work robustly across domains. I would update if shown a method that did.

*Still puzzling:* How to handle the case where drift monitoring triggers re-elicitation but the experts who built the original model are no longer available. The provenance of the graph is documented, but the *reasoning* behind individual commitments is often tacit knowledge that did not get captured. A new expert, looking at the existing graph, may not have the context to evaluate whether a drift signal warrants a structural change or is an artifact of changes in the data-generation process. I do not yet have a clean architectural answer to the *graph stewardship* problem, and I suspect it will become more visible as more organizations operate Living Models for long enough to lose their original modelers.

---

**Tags:** PC GES NOTEARS FCI causal discovery algorithms, CPDAG handoff expert data integration, aleatory versus epistemic uncertainty, validated graph as living artifact, causal governance Jaccard drift monitoring, three-step resolution protocol, graph stewardship version control

---

###  LLM Exercise — Chapter 17: Resolving the Graph

**Project:** Build Your Own Living Model

**What you're building this chapter:** An algorithmic CPDAG from causal discovery (PC + GES + NOTEARS) on your data (real or synthetic), the three-step disagreement-resolution between the algorithmic CPDAG and your v3 expert CPDAG, and a governance setup committing the validated graph to version control with drift monitoring and re-elicitation triggers.

**Tool:** Claude Code — this chapter writes runnable causal-discovery code.

---

**The Prompt:**

```
I am working through Chapter 17 of "Living Models." My v3 expert CPDAG and structural equations are saved in this Project. My estimate-effects.py script from Chapter 8 is in the working directory.

This chapter teaches that algorithmic causal discovery and expert elicitation are complementary — neither alone is enough.

Four algorithm families:
- PC (constraint-based) — uses conditional independence tests; requires faithfulness assumption.
- GES (score-based) — searches DAG space by score (BIC); handles continuous data well.
- NOTEARS (continuous-optimization) — frames structure learning as a smooth optimization with an acyclicity constraint.
- FCI — handles latent confounders explicitly; outputs a Maximal Ancestral Graph (MAG) instead of a CPDAG.

The CPDAG handoff produces three patterns:
- AGREEMENT — algorithm and expert agree; commit and move on.
- REFINEMENT — algorithm fills in what expert was uncertain about; commit, label as algorithmically-determined.
- DISAGREEMENT — algorithm and expert oriented an edge differently. Triggers the three-step resolution protocol:
  1. Re-examine the data evidence (test power, sample composition, hidden colliders).
  2. Re-examine the expert reasoning (was it analogy? was it from a kind or wicked sub-area?).
  3. Decide: more data, more experts, or accept the more conservative orientation.

Aleatory vs epistemic uncertainty — aleatory is fundamental noise (more data won't help); epistemic is missing knowledge (more data or more experts will). The right next move depends on which one is binding.

Build a Python script `resolve-graph.py` that does the following:

PHASE 1 — DATA SETUP:
- If I have real data: load from CSV.
- If synthetic: generate ~5,000 rows from my SCM (use the structural equations from my Project context). Use realistic noise scales.

PHASE 2 — RUN ALL THREE DISCOVERY ALGORITHMS:
- PC using `causal-learn` (or pgmpy). Output the CPDAG.
- GES using `causal-learn`. Output the CPDAG.
- NOTEARS using `notears` package or hand-rolled. Output the directed graph.
- For each, print the graph in adjacency-list form and as Mermaid.

PHASE 3 — COMPARE TO EXPERT v3 CPDAG:
- For each of the three algorithm outputs, compare edge-by-edge to my v3 expert CPDAG (loaded as a separate file).
- Compute Structural Hamming Distance (SHD) — number of edge additions, deletions, and re-orientations needed to match.
- Tabulate the three patterns:
  - AGREEMENT edges
  - REFINEMENT edges (expert undirected, algorithm oriented)
  - DISAGREEMENT edges (oriented opposite directions)

PHASE 4 — RESOLUTION:
- For each DISAGREEMENT, propose the three-step resolution. The script doesn't decide — it surfaces. For each: print the data-side reasons (sample issue? hidden collider?) and prompt the human to choose: accept-data, accept-expert, more-data, or more-experts.

PHASE 5 — COMMIT THE VALIDATED GRAPH:
- Once disagreements are resolved (either by your script's recommendation or by my override), output the FINAL VALIDATED DAG.
- Commit to git: `git add validated-dag-v1.json && git commit -m "Validated DAG v1 — chapter 17"`.
- Generate a `governance.md` file with: the variable definitions, the orientation provenance for each edge (expert / algorithm / both), the disagreement log with how each was resolved, and the re-elicitation trigger conditions.

PHASE 6 — DRIFT MONITORING SCAFFOLD:
- Generate a `monitor-drift.py` stub that, given new data each week or month, runs PC over the new data and computes Jaccard similarity against the validated CPDAG. Flag any similarity drop > 15% as a re-elicitation trigger.

Save everything to the Living Model folder.
```

---

**What this produces:** A validated DAG committed to git, three discovery-algorithm outputs with comparison to the expert CPDAG, a documented disagreement-resolution log, a governance file, and a drift-monitoring scaffold. From this point forward your causal model is a living artifact under version control, not a one-shot deliverable.

**How to adapt this prompt:**
- *For your own project:* If `causal-learn` or `notears` won't install in your environment, ask Claude Code to use only one algorithm (PC is the most reliable to install) plus a hand-rolled comparison.
- *For ChatGPT / Gemini:* Code Interpreter / code execution can run this; iteration loops are slower than Claude Code.
- *For Claude Code:* Recommended. Claude Code installs dependencies, runs each algorithm, debugs version issues, and commits to git.
- *For a Claude Project:* Run the script outside the Project, then paste the disagreement log into the Project for the resolution discussion.

**Connection to previous chapters:** Chapter 16 produced the expert v3 CPDAG. Chapter 8 produced the estimation pipeline. This chapter brings the algorithmic perspective and resolves it against the expert one — closing the structure-learning loop. The output (validated DAG under version control) is what Chapter 18's parameterization pipeline operates on.

**Preview of next chapter:** Chapter 18 takes the validated DAG and turns it into actual recommendations. You'll parameterize it, run the counterfactual engine, rank candidate interventions by Expected Value, and solve the constrained-knapsack portfolio problem to produce the package that the executive report will consume.
