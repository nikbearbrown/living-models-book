# Chapter 7 — The Equivalence Problem

*Markov equivalence, the CPDAG, and why human domain knowledge is mathematically necessary, not merely useful.*

**Nik Bear Brown**

---

## Learning Objectives

By the end of this chapter, you should be able to:

1. **Define** Markov equivalence: explain what it means for two causal diagrams to be observationally indistinguishable, and state the Verma-Pearl condition that characterizes when two DAGs are equivalent.
2. **Read** a CPDAG: distinguish between directed and undirected edges, explain what each represents, and describe the three moves required to work with one.
3. **Explain** why domain knowledge is mathematically necessary — not merely useful — whenever the equivalence class contains more than one DAG.
4. **Describe** how interventional data resolves equivalence ambiguities that observational data cannot, and connect this to why randomized controlled trials are the gold standard for causal inference.
5. **Apply** the equivalence framework to a practical causal discovery scenario: given a CPDAG, identify which edges the data has determined and which require expert input, and describe the consequences if the expert input is wrong.
6. **Recognize** the practical implications of large equivalence classes: describe when sensitivity analysis is needed, when interventional data should be sought, and when a model is robust to unresolved orientations.

**Prerequisites:** Chapter 6 (directed acyclic graphs; the do-operator; structural causal models; what it means for a variable to cause another; how causal structure encodes independence). Chapter 5 (Pearl's Ladder of Causation; why observational data cannot answer Rung 2 questions). This chapter closes the conceptual arc from Chapter 5 (the limit of observation) through Chapter 6 (what we need structurally) to the specific mathematical form that limit takes.

**Why this chapter matters:** Chapter 6 described the structural causal model and the causal diagram as the mathematical apparatus for Rung 2 reasoning. This chapter explains why building that diagram is harder than it looks — and harder in a mathematically precise way. The equivalence problem is not a deficiency of current algorithms; it is a theorem about what observational data can and cannot determine. Understanding it sets the stage for Part Three's architecture for structured expert elicitation, which the mathematics shows to be unavoidable.

---

## The Algorithm That Ran Out of Data

A startup data team is running a causal discovery analysis on customer behavior. They have rich observational data — months of click streams, session durations, purchase histories, support tickets — and they have implemented one of the standard algorithms (PC, GES, NOTEARS, take your pick). They want to know what causes high-value customers to churn.

The algorithm runs. After a few minutes, it produces a result. But the result is not a single causal diagram. It is a graph in which some edges have arrows on them and some are just lines — undirected. The algorithm has identified relationships among the variables, but it has refused to commit to direction on a substantial fraction of the edges.

The data team's first reaction is that the algorithm is broken. The second reaction is that they need more data. The third reaction, after reading some of the literature, is the correct one: the algorithm is working as designed, and no amount of additional observational data will fix the ambiguity.

What they are looking at is a *Completed Partially Directed Acyclic Graph*, or CPDAG. It is the honest output of automated causal discovery from observational data: it tells you which causal relationships the data can identify, and which it cannot. The undirected edges are not a bug. They are a confession. The data alone cannot determine which way these arrows should point.

This chapter is about why that confession is unavoidable, what the CPDAG actually represents, and what the practical consequences are for anyone trying to build a causal model from data.

---

## Concept 1: Markov Equivalence and What It Means

### When Two Diagrams Look the Same to the Data

The most intuitive entry point into Markov equivalence is a case we already know. Consider three variables, X, Y, and Z, with the chain *X → Y → Z*. In this structure, X causes Y and Y causes Z. Now consider two other structures with the same three variables: the fork *X ← Y → Z* (Y causes both X and Z) and the reverse chain *X ← Y ← Z* (Z causes Y causes X). All three have the same set of edges — X-Y and Y-Z — but with different arrow directions.

Do these three diagrams make different predictions about what we will observe in the data?

Surprisingly, no. They all predict the same pattern of conditional independences. In all three, X and Z are marginally correlated (both connected to Y). In all three, X and Z are conditionally independent given Y (once we know Y, knowing X gives no additional information about Z, and vice versa). A statistician who examined any amount of observational data on X, Y, and Z could never tell these three diagrams apart — because they produce identical statistical signatures in the data.

This is Markov equivalence: the three diagrams are *equivalent* in the sense that they imply the same joint distribution over the variables, regardless of the specific data-generating process.

<!-- → INFOGRAPHIC: three causal graph panels — left panel: chain "X → Y → Z"; middle panel: fork "X ← Y → Z"; right panel: reverse chain "X ← Y ← Z" — all three labeled with the same conditional independence statement: "X ⊥ Z | Y (X and Z are independent given Y)" — annotation below all three: "Same observational data. Three incompatible causal stories." — student should see that identical data is consistent with multiple causal structures and understand why this creates a problem for purely data-driven causal discovery -->

### The Verma-Pearl Characterization

The systematic theory of Markov equivalence gives a precise answer to the question: when are two DAGs observationally indistinguishable? The Verma-Pearl theorem (1990) provides the condition.

Two DAGs are Markov equivalent if and only if they have:
1. The same **skeleton** — the same set of edges, ignoring direction.
2. The same **v-structures** — the same set of *unshielded colliders*.

A v-structure (also called an unshielded collider) is the configuration $X \rightarrow Z \leftarrow Y$ where X and Y are not directly connected. This is the distinguishing configuration because it produces a unique statistical signature: X and Y are independent unconditionally, but become *dependent* when we condition on Z. This pattern — independence that becomes dependence when a third variable is conditioned on — does not appear in chains or forks, and the data can therefore detect v-structures.

Every other aspect of edge direction is undetectable from observational data. Two DAGs with the same skeleton and the same v-structures are Markov equivalent, period. No statistical test, no matter how powerful, applied to any amount of observational data, will distinguish them.

<!-- → INFOGRAPHIC: v-structure explanation — three nodes X, Y, Z — X and Y have arrows pointing into Z (the collider) — X and Y are not connected to each other — two panels: left panel "Without conditioning on Z": X and Y shown as independent (no correlation line), labeled "X ⊥ Y marginally"; right panel "Conditioning on Z": X and Y shown as correlated (dashed line), labeled "X and Y become dependent — this is the v-structure signature" — annotation: "Only v-structures produce this pattern. The data can detect them. Everything else about edge direction is invisible to the data." — student should understand why v-structures are special and why they are the only edge-direction information observational data contains -->

### The Markov Equivalence Class

Given any DAG, the *Markov equivalence class* is the set of all DAGs that are Markov equivalent to it — all the DAGs with the same skeleton and same v-structures. The class can be small (if the original DAG has many v-structures that constrain the skeleton's orientations) or large (if the DAG has few v-structures and many edges whose direction is undetermined by the data).

For the chain *X → Y → Z*, the equivalence class contains exactly three members: the chain, the fork, and the reverse chain we saw above. For larger graphs, the equivalence class can be much larger. A DAG with ten variables and few v-structures can have an equivalence class with thousands of members.

The implication for causal discovery is sharp. An algorithm that takes observational data and returns a causal model cannot do better than returning the equivalence class. Any algorithm that claims to return a unique DAG — a fully specified arrow-by-arrow causal structure — is making assumptions not present in the data, whether it says so or not.

### Worked Example: The Four-Variable System

Consider a company's data team studying four variables: a marketing campaign (M), customer engagement (E), product usage (U), and retention (R). They run a causal discovery algorithm on one year of observational data. The algorithm returns a CPDAG with:

- M → E: a directed edge from M to E.
- E — U: an undirected edge between E and U.
- U — R: an undirected edge between U and R.
- No edges between M and U, M and R, or E and R.

The directed edge M → E tells them something the data could identify: M and E are causally related and the data contains a v-structure (or temporal ordering information) that determines M precedes E. The undirected edges E — U and U — R tell them something the data could not identify: E and U are causally related, and U and R are causally related, but the direction of each relationship is undetermined.

How many DAGs are in the equivalence class? The two undirected edges each have two possible orientations, giving four candidate diagrams: (E → U, U → R), (E → U, R → U), (U → E, U → R), (U → E, R → U). Each of these is consistent with the observational data. Each would produce a different causal effect estimate if used for downstream analysis.

<!-- → INFOGRAPHIC: four-panel diagram showing the four candidate DAGs in the equivalence class for the marketing company CPDAG — each panel has the same four nodes (M, E, U, R) and the same directed edge M → E — they differ in the E-U and U-R orientations: panel 1 (E→U, U→R), panel 2 (E→U, R→U), panel 3 (U→E, U→R), panel 4 (U→E, R→U) — all four panels labeled "consistent with the observational data" — annotation: "Four incompatible causal models. Four incompatible strategic recommendations." — student should see that the same data supports multiple models with different implications -->

**The lesson:** A four-variable problem with two undirected edges already has four candidate causal models. The data cannot choose among them. Something from outside the data must.

---

## Concept 2: The CPDAG as Honest Output and Working with It

### Reading the CPDAG

The Completed Partially Directed Acyclic Graph is the canonical representative of the Markov equivalence class. It has three structural features that make it the right object to work with.

First, it has the same skeleton as every member of the equivalence class. The edges present in the CPDAG (directed or undirected) are the edges present in every member of the class. The edges absent from the CPDAG are absent from every member.

Second, its directed edges are the edges that *every* member of the equivalence class agrees on. A directed edge in the CPDAG means: every DAG consistent with the observational data has this arrow in this direction. These are the orientations the data has determined.

Third, its undirected edges are the edges that members of the equivalence class *disagree* on. An undirected edge in the CPDAG means: some DAGs in the class have the arrow one way, some have it the other. These are the orientations the data cannot determine.

Reading a CPDAG, you can see at a glance what the data identify and what the data leave open. The directed edges are conclusions you can draw without additional input. The undirected edges are decisions you must make on grounds external to the data.

The honesty of the CPDAG is its principal virtue. It forces the analyst to confront the limits of what the data can tell her. Tools that automatically resolve undirected edges by applying additional assumptions (linearity, non-Gaussianity, temporal ordering) are not dishonest, but they are making commitments the CPDAG does not make — and those commitments should be stated explicitly.

### Three Moves for Working with a CPDAG

Working with a CPDAG in practice involves three moves, applied in order.

**Move 1: Orient undirected edges using domain knowledge.** For each undirected edge X-Y, the analyst asks: based on what I know about the system, which direction does this arrow go? The temporal order of the variables often resolves it — causes precede effects, and if we know that X is measured before Y, then X → Y is the only causal interpretation. Sometimes the mechanism is obvious from theory or operational knowledge. Sometimes it is not.

**Move 2: Check consistency with existing directed edges.** The orientations the analyst adds must not introduce new v-structures that would be inconsistent with the conditional independence patterns the data showed. This is a constraint that the CPDAG enforces. If the analyst tries to orient an edge in a way that would create a new v-structure — and therefore a new conditional independence that the data did not exhibit — the orientation is forbidden. This consistency check is the CPDAG's way of preventing the analyst from committing to a structure the data actively contradicts.

**Move 3: Acknowledge and report unresolved edges.** If the analyst is unable to orient certain edges with confidence, she reports this honestly. The downstream causal effect estimates depend on the diagram's edge directions. If some directions are uncertain, the estimates will carry that uncertainty. Sensitivity analysis — varying the orientation and reporting how the estimates change — is the appropriate response to unresolved ambiguity.

<!-- → INFOGRAPHIC: three-panel "working with the CPDAG" flow — panel 1 "Start: CPDAG from algorithm" — CPDAG with mix of directed and undirected edges, annotated "Data identified these: directed edges (arrows)"; panel 2 "Move 1: Domain knowledge" — same CPDAG, undirected edges being resolved by expert annotations (temporal precedence examples, mechanism examples), annotated "Expert resolves these: undirected edges become directed"; panel 3 "Result: Oriented DAG (or partially oriented)" — fully directed DAG with any unresolvable edges flagged for sensitivity analysis, annotated "Use this for effect estimation; conduct sensitivity analysis on flagged edges" — student should see the transition from CPDAG to usable model and understand where each type of input enters -->

### Common Mistakes When Working with CPDAGs

Two mistakes are common enough to name explicitly.

The first is treating the CPDAG as a finished causal model. The CPDAG is not a finished model — it is an input to model construction. Acting on the directed edges as if they were the complete causal story, while ignoring the undirected edges, is equivalent to assuming the equivalence class has one member when it has many. The undirected edges represent real uncertainty about the causal structure, and that uncertainty propagates into the effect estimates.

The second mistake is resolving undirected edges by default rather than by deliberate commitment. If an analyst resolves an undirected edge by assuming X causes Y simply because X appears earlier in the dataset or has a higher variable number, she has not done causal modeling — she has done causal modeling with undisclosed default assumptions. The expert's deliberate commitment to an orientation, with stated reasoning, is a first-class input. The algorithm's default resolution of an orientation is not.

### Worked Example: Resolving the Four-Variable CPDAG

Returning to the marketing company's CPDAG: the data team consults the product manager. The conversation is structured around each undirected edge.

*E — U (engagement and usage):* The product manager explains that product usage is logged by the system as a behavioral event; engagement scores are computed as a downstream metric that aggregates usage events over a rolling window. Usage happens first; the engagement score is calculated from it. Temporal precedence plus mechanistic reasoning both support U → E. The edge is oriented: U → E.

*U — R (usage and retention):* The product manager explains that retention decisions — whether to renew a contract — are influenced by the customer's cumulative product usage, not the other way around. Retention is a once-a-year decision; usage is a daily behavior. Usage precedes and causes retention. The edge is oriented: U → R.

The analyst checks consistency: orienting U → E and U → R does not create any new v-structures that would conflict with the CPDAG's existing directed edges. The constraint check passes.

The resulting DAG: M → E ← U → R. This is the model the analyst will use for effect estimation. The model says: the marketing campaign affects engagement; product usage also affects engagement; product usage determines retention. The marketing campaign, notably, has no direct effect on retention — it only affects engagement, and engagement, in this model, does not affect retention.

The analyst notes the key implication: if the goal is to increase retention, improving the marketing campaign is not the right lever. Improving product usage is.

<!-- → INFOGRAPHIC: completed four-variable DAG — nodes M (Marketing campaign), E (Engagement), U (Usage), R (Retention) — directed arrows: M → E, U → E, U → R — annotation bubbles explaining each orientation: M → E "Determined by data (temporal v-structure)"; U → E "Expert: usage logged first, engagement computed from it"; U → R "Expert: usage behavior precedes annual renewal decision" — boxed conclusion: "Key finding: marketing does not directly affect retention; usage does. Improving engagement without improving usage does not improve retention." — student should see how the expert's orientations changed the causal interpretation and therefore the actionable recommendation -->

**The lesson:** Two expert orientations — each justified by specific reasoning about mechanism and temporal order — changed the causal model from an ambiguous CPDAG to a fully directed DAG with a specific strategic implication. Without the expert, the model could not produce a recommendation. With the expert, it can.

---

## Concept 3: Why Knowledge Is Mathematically Necessary, Not Merely Useful

### The Sharpened Argument

The previous concept showed that working with CPDAGs requires expert input. This concept sharpens the argument: expert input is not just practically necessary, it is mathematically necessary whenever the equivalence class has more than one member.

The argument is direct. The Markov equivalence class contains all DAGs consistent with the observational data. Every member of the class is, by definition, equally supported by the data. If the class has more than one member, no procedure operating on the data alone can choose one. The choice must come from outside the data.

This is not a claim about the limits of current statistical methods, which might someday be improved. It is a claim about what observational data is and what it is not. Observational data is a record of the joint distribution of variables — what values tended to co-occur. The joint distribution is fully characterized by the Markov equivalence class. Within the class, the data provides no information. To move from the class to a specific DAG, you need information the data does not contain.

### The Role of the Domain Expert

In the dominant paradigm of machine learning, the data scientist is the primary actor. The domain expert is consulted, perhaps, to validate features or interpret results. In causal inference, this relationship inverts. The domain expert is the primary actor at the structural modeling stage. The data scientist supports the analysis by gathering data, running estimation procedures, and reporting results — but the structural commitments come from the expert.

This inversion is not a stylistic preference. It is forced by the mathematics. The data scientist who insists on doing causal modeling without consulting the domain expert is, whether she realizes it or not, substituting her own implicit assumptions for the expert's explicit ones. The implicit assumptions are typically worse than the expert's, because they are not informed by the substantive knowledge of the domain — and because, being implicit, they are not available for inspection or critique.

<!-- → TABLE: two-column comparison — left column: "Machine learning paradigm," right column: "Causal inference paradigm" — rows: Primary actor (data scientist / domain expert); Domain expert's role (validates features, interprets outputs / makes structural commitments about edge directions); Where key assumptions live (implicit in algorithm and feature selection / explicit in the causal diagram); What happens when expert is excluded (model still runs; produces predictions / model produces the wrong causal model; estimates are biased in unknown direction); Auditability (hard to audit implicit assumptions / structural commitments are visible in the diagram) — student should see the inversion of roles and understand why it is forced by the mathematics, not by preference -->

The pattern is universal enough to deserve a name. I call it *the Expert in the Room principle*: every causal model needs an expert, and every architecture for causal modeling has to make space for one. Chapter 14 in Part Three takes up this principle and describes a discipline — *Knowledge Engineering with Bayesian Networks* — that has been developing methods for structured expert elicitation for several decades. Chapter 16 describes the LLM-guided architecture that makes the elicitation tractable at scale.

### How Intervention Resolves What Observation Cannot

If observational data cannot distinguish among Markov-equivalent DAGs, what can? The answer is intervention.

Consider the simplest case: the chain *X → Y* and the reverse chain *Y → X*. They produce the same observational correlation. But they make different predictions about an intervention.

If we intervene on X — setting it to a specific value by external manipulation, severing whatever normally causes X — then:
- In *X → Y*: Y responds to our manipulation of X. We have set X, so Y will change.
- In *Y → X*: Y does not respond. We have set X by force, but since X does not cause Y in this diagram, Y is unaffected.

A single intervention experiment, even one of modest size, distinguishes the two diagrams unambiguously. Interventional data resolves equivalences that unlimited observational data cannot.

This is, in essence, what randomized controlled trials provide. The reason RCTs are the gold standard for causal inference is not that they have larger samples or better measurement. It is that they provide interventional data, and interventional data contains information that observational data does not. Chapter 11 takes up RCTs in detail.

But interventional data is expensive, slow, and often impossible to obtain. Expert knowledge substitutes for interventional data in cases where intervention is infeasible. The expert's claim that *Y depends on X, not the other way around* is, in effect, a claim about what an intervention would show. She is reporting the result of an intervention she has not performed but has good reason to predict — based on theory, prior experiments elsewhere, or accumulated professional experience.

### When the Equivalence Class Is Large

The four-variable example had two undirected edges and four members in the equivalence class. In practice, equivalence classes can be much larger. A CPDAG with twenty variables can have thousands of members. The analyst cannot examine each individually.

The practical response to a large equivalence class involves two assessments.

*Does the unresolved ambiguity matter for the question we are asking?* If the edges whose direction is unresolved are not on the paths between the treatment variable and the outcome variable of interest, their direction may not affect the causal effect estimate. The analyst should examine which paths in the CPDAG connect treatment to outcome, and check whether the undirected edges are on those paths. If they are not, the estimates may be robust to the unresolved orientations.

*How much do the causal effect estimates vary across the equivalence class?* Sensitivity analysis — computing the causal effect estimate under each possible orientation of the undirected edges — quantifies the impact of the ambiguity. If the estimates are similar across all members of the class, the analyst can report a confident conclusion. If the estimates vary substantially, the analyst must either resolve the orientations (via expert input or intervention) or report the range of estimates as the honest answer.

### Worked Example: Sensitivity Analysis on an Unresolved Edge

In the marketing company's analysis, suppose the product manager is uncertain about one orientation: E — R (the relationship between engagement and retention). She is not sure whether engagement causes retention, retention causes engagement, or there is no direct edge (the original CPDAG had no edge between E and R, so this is not a live question, but let us construct a variant).

The analyst reframes this as a sensitivity question: if we cannot resolve the E-R direction, how much does it affect our estimate of the effect of U (usage) on R (retention)?

She runs the effect estimation under both orientations: E → R (engagement causes retention) and R → E (retention causes engagement, perhaps through a feedback loop where retained customers become more engaged). Under E → R, the effect of U on R operates through two paths: U → R directly, and U → E → R. Under R → E, U's effect on R operates only through U → R directly, and the engagement-retention relationship is reversed.

The estimates differ. Under E → R, usage has a larger estimated effect on retention (because it also works through engagement). Under R → E, the direct usage-retention effect is estimated with higher precision (because the engagement path is now reversed).

<!-- → CHART: two-panel bar or path diagram — left panel "Model A: E → R" — causal paths from U to R shown: direct path U → R and indirect path U → E → R; total effect of U on R is the sum of both paths; right panel "Model B: R → E" — only direct path U → R remains; indirect path through engagement is severed; total effect of U on R is smaller — student should see the sensitivity of the U-on-R estimate to the E-R edge orientation, and understand why reporting a range of estimates is the honest output when this edge cannot be resolved -->

The analyst reports: "The effect of usage on retention ranges from [lower bound] to [upper bound] depending on the orientation of the engagement-retention edge. To narrow this range, we recommend either a targeted expert consultation on the mechanism linking engagement to retention, or a controlled experiment in which engagement is varied by intervention while holding usage constant."

**The lesson:** Sensitivity analysis is not an admission of failure. It is the honest quantification of what the model does not know, paired with a prescription for what would reduce the uncertainty. The unresolved orientation is a question waiting for an answer, not a defect in the analysis.

---

## Integration: Implications for the Living Model Architecture

### The Three Practical Implications

Three implications follow from the equivalence problem, and all three are load-bearing for Part Three's architecture.

**Implication 1: Causal discovery from observational data is partial.** Any algorithm that promises to recover a complete causal model from data alone is oversimplifying. The honest output is the CPDAG. Tools that report a single fully directed graph are making additional assumptions — temporal ordering, linearity with non-Gaussian noise, or something else — that resolve equivalence in a particular way. Those assumptions should be stated explicitly and treated as inputs to the model, not as outputs of the data.

**Implication 2: The expert's role is structural, not decorative.** The expert is not validating the model; she is constructing it. Her judgments about edge direction are first-class inputs to the causal inference. The architecture of the modeling exercise must be built around eliciting those judgments carefully and recording them explicitly. A causal model whose edge orientations are undocumented — where the analyst cannot say why an edge was oriented one way rather than another — is a model whose structural commitments cannot be audited or challenged.

**Implication 3: The model is testable.** Even though orientation of some edges depends on expert input, the diagram as a whole has empirical content. It implies specific conditional independence relations among the variables. If the data violate those relations, the diagram is wrong, and revision is required. The model is not derived from data, but it is constrained by data. The constraints are sharp enough to falsify it, and falsification is the mechanism by which causal modeling progresses.

### Connecting to the Living Model Pipeline

In the Living Model architecture, the CPDAG is the handoff between the elicitation step (Chapter 16) and the data-driven refinement step (Chapter 17). The expert produces a set of structural commitments — oriented edges based on domain knowledge. The causal discovery algorithm produces a CPDAG from the observational data. The two are combined: the expert's orientations are applied to the CPDAG's undirected edges, the data's directed edges and skeleton constrain what the expert can commit to, and the result is a fully or mostly directed DAG suitable for downstream effect estimation.

The equivalence problem explains why both inputs are necessary. The data alone cannot produce a fully directed DAG. The expert alone cannot produce the skeleton and v-structures — she may not know which variables are related to which without the data. The combination, structured by the Markov equivalence framework, produces a model that neither the data nor the expert could produce independently.

This is the architecture the Living Model is built on. The chapters that follow — on effect estimation (Chapter 8), randomized trials (Chapter 11), expert elicitation (Chapter 14 and 16), and parameterization (Chapter 18) — all assume this joint data-and-expert construction. The equivalence problem is why the construction is joint.

### Putting It All Together

The startup data team from the opening of this chapter now has a framework for what they are looking at. Their CPDAG is not a broken output. It is the algorithm doing exactly what it can do: identifying the skeleton, identifying the v-structures, and declining to pretend it knows more than it does.

The next steps are determined. For each undirected edge, they need someone who knows the system well enough to say which variable comes first, which mechanism operates. That someone is the domain expert — the product manager, the growth analyst, the person who has been working with these customers long enough to know whether usage causes engagement or engagement causes usage.

The data built the scaffold. The expert fills in the directions. The combination builds the model. The model, once built, is testable against the data's conditional independence implications. If it passes the test, it is used for effect estimation. If it fails, it is revised.

This is causal modeling, done honestly. The equivalence problem is not an obstacle; it is a specification of exactly what needs to be done.

---

## Chapter Summary

The Markov equivalence class is the set of all DAGs that are observationally indistinguishable from each other — all DAGs with the same skeleton and the same v-structures. The CPDAG is the canonical representative of the class: its directed edges are orientations every class member agrees on; its undirected edges are orientations class members disagree on.

Automated causal discovery algorithms can return at best the CPDAG. No algorithm, operating on observational data alone, can return a unique DAG — not because algorithms are weak, but because the data do not contain the information that would distinguish among class members. This is the Verma-Pearl theorem, a mathematical result, not a practical limitation.

Moving from the CPDAG to a fully directed DAG requires information from outside the data. Domain knowledge — the expert's understanding of which variables precede which, which mechanisms operate — is the standard source. Interventional data — controlled experiments — is another. The two sources provide different kinds of information and are often used together.

**The one idea that matters most from this chapter:** Causal structure is not in the data. The data shows patterns of association. The causal structure that produced those patterns is not uniquely determined by them. This is not a temporary limitation; it is the nature of observational data. The Living Model architecture is built around this fact, not against it.

**The common mistake to watch for:** Treating a fully directed DAG produced by a causal discovery algorithm as a data-derived fact. Every fully directed DAG produced by a standard algorithm on observational data is either (a) the algorithm applying additional assumptions to resolve the CPDAG's undirected edges, or (b) a DAG that happens to be the unique member of a small equivalence class (rare in practice). The structural commitments that resolve equivalence ambiguities are always inputs, not outputs.

**The Feynman test:** Close the book and explain to a colleague why two causal diagrams can produce identical observational data. Use the chain-fork-reverse-chain example. Then explain what information would allow you to choose among them, and why a domain expert providing that information is doing causal modeling, not just validating a data analysis. If you can do both, you have understood this chapter.

---

## Exercises

### Warm-Up

**W1.** *(Objective 1 — Define Markov equivalence; Difficulty: Low)*

The chain *A → B → C*, the fork *A ← B → C*, and the reverse chain *A ← B ← C* are all Markov equivalent. Explain in two to three sentences why they produce the same conditional independence pattern — specifically, why A and C are conditionally independent given B in all three structures.

**W2.** *(Objective 2 — Read a CPDAG; Difficulty: Low)*

A causal discovery algorithm returns a CPDAG with the following structure: X → Y, Y — Z, W → Z, and no edge between X and Z or X and W.

(a) Which edges have the data determined the direction of?
(b) Which edge has the data left undirected?
(c) What are the two possible fully directed DAGs consistent with this CPDAG?
(d) What information would allow you to choose between them?

**W3.** *(Objective 4 — Explain why intervention resolves equivalence; Difficulty: Low)*

Two DAGs are Markov equivalent: *Price → Demand* and *Demand → Price*. A retailer wants to know which is correct so she can estimate the effect of changing price on demand.

Describe in plain language: (a) why observational data on price and demand cannot distinguish the two, and (b) what experiment would distinguish them and why it works where observational data does not.

---

### Application

**A1.** *(Objective 3 — Apply the necessity argument; Difficulty: Medium)*

A data science team builds a churn prediction model without consulting any domain experts. They apply a causal discovery algorithm and report the resulting CPDAG as their causal model, including all undirected edges as-is.

Identify two specific ways this practice violates the mathematical necessity argument from the chapter. For each violation, describe what error could result in downstream causal effect estimation — specifically, what kind of incorrect recommendation the model could produce.

**A2.** *(Objective 5 — Work with a CPDAG; Difficulty: Medium)*

A healthcare analytics team studying hospital readmissions runs a causal discovery algorithm on their data. The CPDAG they receive has these edges: Age → Severity, Severity → Readmission, Treatment — Severity, Treatment — Readmission, and no edge between Age and Treatment or Age and Readmission.

(a) List all the undirected edges and the alternative orientations for each.
(b) How many fully directed DAGs are consistent with this CPDAG?
(c) A clinical expert says: "Treatment decisions are made after diagnosis severity is assessed; treatment does not determine severity." Apply this domain knowledge to orient one edge. Does this orientation pass the consistency check? (Does it introduce a new v-structure?)
(d) The expert is uncertain about the Treatment-Readmission edge. Describe what sensitivity analysis would look like for this edge — what two causal models would be compared and what the key question would be.

**A3.** *(Objective 6 — Assess robustness and seek intervention; Difficulty: Medium)*

A supply chain team has a CPDAG with twelve variables and seven undirected edges. The team's primary question is: what is the causal effect of supplier lead time on inventory shortfall?

(a) Describe the first diagnostic they should run: how can they determine whether the seven undirected edges are on paths between supplier lead time and inventory shortfall?
(b) Suppose they find that three of the seven undirected edges are on those paths. Describe the sensitivity analysis approach — how many models would they need to run, and what would they be looking for?
(c) Suppose the sensitivity analysis shows large variation in the effect estimate depending on the orientations of two of the three on-path edges. Describe two options for resolving the ambiguity, and for each option, state what it requires and what it produces.

**A4.** *(Objectives 3, 5 — Apply the framework to a real case; Difficulty: Medium)*

Choose a causal question from your own industry or professional experience — one where you have some organizational data available and domain knowledge. Describe: (a) two variables in that domain that are correlated in the data, and at least two plausible causal stories (different DAGs) that would produce the same correlation; (b) what domain knowledge you would use to orient the edge between them; (c) what experiment, if feasible, would definitively resolve the orientation; and (d) what the consequence would be for a downstream recommendation if you assumed the wrong orientation.

---

### Synthesis

**S1.** *(Objectives 1, 3, 4 — Integrate observation and intervention; Difficulty: High)*

The chapter argues that domain knowledge and interventional data are the two sources of information that can resolve Markov equivalence. They are not substitutes — they provide different kinds of information about different aspects of the causal structure.

Write a two-to-three paragraph analysis that: (a) explains specifically what kind of information domain knowledge provides and what kind interventional data provides — why they are different in kind, not just in quality, (b) describes a scenario in which domain knowledge is the better source for resolving a specific ambiguity, and a scenario in which interventional data is the better source, and (c) describes what a combined strategy — using both domain knowledge and a targeted small experiment — would look like for a specific causal question from business or healthcare.

**S2.** *(Objectives 2, 5, 6 — Connect the CPDAG to the Living Model pipeline; Difficulty: High)*

In the Living Model architecture, the CPDAG is the handoff between expert elicitation and data-driven refinement. The expert produces structural commitments; the algorithm produces the CPDAG; the two are combined.

Describe what can go wrong at this handoff: (a) what happens when the expert's structural commitments conflict with the algorithm's directed edges (the expert has oriented an edge one way, but the data-derived v-structure implies the other direction), (b) what happens when the expert is overconfident — she has oriented edges that should have been left undirected because the equivalence class is still large even after her input, and (c) propose a protocol for managing the handoff that would detect and resolve both failure modes before the combined model enters downstream effect estimation.

---

### Challenge

**C1.** *(Objectives 1, 3, 4, 6 — Stress-test the necessity argument; Difficulty: High)*

The chapter claims that domain knowledge is mathematically necessary when the equivalence class has more than one member. A skeptical data scientist argues: "Modern causal discovery methods exploit non-Gaussianity of the noise terms (LiNGAM), or non-linearity of the mechanisms (ANM), to identify causal direction purely from data. These methods don't need domain knowledge. Your necessity argument is out of date."

Construct a response that: (a) accurately characterizes what LiNGAM and ANM-type methods actually assume and what they identify — be specific about the assumptions and whether they are testable, (b) explains whether these methods eliminate the equivalence problem or circumvent it by making additional assumptions, (c) identifies the conditions under which these methods succeed and the conditions under which they fail (when do business and organizational datasets satisfy their assumptions?), and (d) states whether, given your analysis, the necessity argument still holds and why.

**C2.** *(Objectives 3, 6 — The change management challenge; Difficulty: High)*

The chapter's "Still puzzling" footer identifies a problem the mathematics does not solve: how to communicate the necessity of expert input to organizations that have invested heavily in data-driven decision-making and are uncomfortable with the idea that data alone is insufficient.

Write a two-to-three paragraph analysis that: (a) characterizes the specific organizational resistance you would expect — not vague "change resistance" but the specific beliefs and incentives that make executives and data teams uncomfortable with the expert-as-structural-modeler role, (b) proposes a communication approach for one type of organizational context (choose: a company with a strong ML team but no causal inference background, a company with a management consulting tradition, or a regulated industry like healthcare or finance), and (c) identifies what evidence — not theoretical argument, but empirical evidence from a real deployment — would most effectively shift organizational beliefs about the necessity of expert input.

---

## Connections Forward

This chapter has established the mathematical foundation for why the Living Model architecture requires both data and expert input. The CPDAG is the handoff object between the two.

Chapter 8 assumes the handoff has been made — that we have a fully directed DAG — and takes up the next question: how do we estimate the causal effect of one variable on another from observational data, given that diagram? The backdoor criterion, double machine learning, and causal forests are the tools. The diagram tells us which adjustments to make; the data, properly adjusted, gives us the estimate.

Chapter 11 takes up randomized controlled trials — the interventional data source that can resolve equivalence when domain knowledge is uncertain, and the gold standard against which all observational causal inference is ultimately measured.

Chapter 14 and Chapter 16 in Part Three address the expert elicitation architecture directly. Chapter 14 describes the Knowledge Engineering with Bayesian Networks discipline that has been developing structured elicitation protocols for twenty years. Chapter 16 describes the LLM-guided elicitation system that makes the forty-five-minute first-pass DAG feasible at organizational scale. Both chapters take the mathematics of this chapter as given: the expert's structural commitments are mathematically necessary inputs, and the architecture must be designed to elicit them reliably.

---

**What would change my mind:** A demonstration that observational data, in some specific class of problems, can identify causal structure beyond the Markov equivalence class. There are extensions of the standard framework — exploiting non-Gaussianity in the noise terms — that resolve some equivalences. None of them are universal, and all of them require additional assumptions. I would update if shown a method that resolves equivalence under assumptions weaker than the ones currently on the table.

**Still puzzling:** How best to communicate the necessity of expert input to organizations that have invested heavily in data-driven decision-making and are uncomfortable with the idea that data alone is insufficient. The mathematics is clear; the cultural shift is harder. I do not have a confident answer to the change-management problem, and I suspect that solving it is more important than solving any technical problem in causal inference.

---

*Tags: Markov equivalence class, CPDAG completed partially directed acyclic graph, causal discovery algorithms PC GES NOTEARS, expert knowledge mathematically necessary, observational vs interventional identification, Verma-Pearl theorem, v-structure*
