# Chapter 7 — The Equivalence Problem

*Markov equivalence, the CPDAG, and why human domain knowledge is mathematically necessary, not merely useful.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *The Equivalence Problem*
- *CPDAGs and the Necessity of Knowledge*
- *Why Data Cannot Orient Every Edge*

**TL;DR.** The same observational data can be consistent with many structurally different causal diagrams. The set of diagrams that are statistically indistinguishable from each other is called a Markov equivalence class. Automated causal discovery algorithms, no matter how sophisticated, can return at best the equivalence class, not a specific diagram within it. Choosing among the diagrams in the class requires information from outside the data — domain knowledge, temporal precedence, or controlled experiment. This is a mathematical fact, not a limitation of current methods.

---

## The Algorithm That Ran Out of Data

A startup data team is running a causal discovery analysis on customer behavior. They have rich observational data — months of click streams, session durations, purchase histories, support tickets — and they have implemented one of the standard algorithms (PC, GES, NOTEARS, take your pick). They want to know what causes high-value customers to churn.

The algorithm runs. After a few minutes of computation, it produces a result. But the result is not a single causal diagram. It is something stranger: a graph in which some edges have arrows on them and some are just lines, undirected. The algorithm has identified relationships among the variables — these things are causally related to those things — but it has refused to commit to direction on a substantial fraction of the edges.

The data team's first reaction is that the algorithm is broken. The second reaction is that they need more data. The third reaction, after reading some of the literature, is the correct one: the algorithm is working as designed, and no amount of additional observational data will fix the ambiguity.

What they are looking at is a *Completed Partially Directed Acyclic Graph*, or CPDAG. It is the honest output of automated causal discovery from observational data: it tells you which causal relationships the data can identify, and which it cannot. The undirected edges are not a bug. They are a confession. The data alone cannot determine which way these arrows should point.

This chapter is about why that confession is unavoidable, what the CPDAG actually represents, and what the practical consequences are for anyone trying to build a causal model from data.

---

## Markov Equivalence, Made Specific

In Chapter 6, we noted that the diagrams *X → Y*, *Y → X*, and *X ← Z → Y* (with Z unobserved) all produce the same correlation between X and Y. They are observationally indistinguishable. This is the simplest case of Markov equivalence.

The general definition: two causal diagrams are *Markov equivalent* if they imply the same conditional independence relations among the variables. Two diagrams that are Markov equivalent will produce the same observational distribution, no matter how the data are generated. They are observationally indistinguishable in the strongest possible sense.

A theorem due to Verma and Pearl (1990) gives a sharp characterization. Two DAGs are Markov equivalent if and only if they have the same *skeleton* (the same edges, ignoring direction) and the same *v-structures* (the same set of unshielded colliders). A v-structure is a configuration *X → Y ← Z* where X and Z are not directly connected. The data can detect v-structures because of the special independence pattern they produce: X and Z are independent unconditionally, but become dependent when we condition on Y.

Outside of v-structures, edge direction is not detectable from observational data. If a chain *X → Y → Z* and a fork *X ← Y → Z* and a reverse chain *X ← Y ← Z* all share the same skeleton (X-Y-Z) and have no v-structures, then they are Markov equivalent. The data cannot tell them apart.

This has a practical consequence. The output of any causal discovery algorithm operating on observational data is, at best, the Markov equivalence class — the set of all DAGs that share the same skeleton and v-structures. The class can have many members. For a graph with ten variables, the equivalence class can contain thousands of DAGs. The algorithm cannot tell you which one is correct, because the data cannot tell you.

---

## The CPDAG as Honest Output

The Completed Partially Directed Acyclic Graph is the canonical representative of the Markov equivalence class. It has the same skeleton as every member of the class. Its directed edges are the edges that *every* member of the class agrees on — the edges whose direction is determined by the data. Its undirected edges are the edges that members of the class disagree on — the edges whose direction the data cannot determine.

Reading a CPDAG, you can see exactly what the data identify and what the data leave undetermined. The directed edges are conclusions you can draw without further input. The undirected edges are decisions you have to make on grounds external to the data.

The honesty of the CPDAG is its principal virtue. It forces the analyst to confront the limits of what the data can tell her. The temptation in causal discovery is to treat the algorithm's output as a fully specified causal model, ready to plug into a downstream effect estimator. The CPDAG resists that temptation. It hands the analyst a partial specification and says: here is what the data tell you. The rest is up to you.

In practice, working with a CPDAG involves three moves.

**One: orient the undirected edges using domain knowledge.** For each undirected edge X-Y, the analyst asks: based on what I know about the system, which direction does this arrow go? The temporal order of the variables often resolves it (causes precede effects). Sometimes the mechanism is obvious; sometimes it is not. The analyst commits, on each undirected edge, to a direction or to "I do not know."

**Two: check that the resulting orientations do not introduce new v-structures.** This is a constraint of the CPDAG: the orientations the analyst adds must be consistent with the directed edges already present. If she tries to orient an edge in a way that would introduce a new v-structure — and therefore a new conditional independence pattern that the data did not show — the orientation is forbidden. The CPDAG enforces a kind of consistency check on the analyst's domain knowledge.

**Three: report the dependencies.** If the analyst is unable to orient certain edges, she reports this honestly. The downstream causal effect estimates depend on the diagram, and if the diagram has ambiguities, the estimates will too. Sensitivity analysis — varying the orientation and reporting how the estimates change — is the appropriate response.

---

## Why Knowledge Is Mathematically Necessary

The argument in the previous section can be sharpened. It is not just that domain knowledge is *useful* in causal discovery; it is mathematically *necessary* whenever the data cannot identify a unique diagram. If the equivalence class contains more than one DAG, then no procedure operating on the data alone can pick one. The pick must come from outside the data.

This has profound implications for the relationship between data scientists and domain experts. In the dominant paradigm of machine learning, the data scientist is the primary actor; the domain expert is consulted, perhaps, to validate features or interpret results. In causal inference, this relationship inverts. The domain expert is the primary actor at the modeling stage. The data scientist supports the analysis by gathering data, running estimation procedures, and reporting results — but the structural commitments come from the expert.

This inversion is not a stylistic preference. It is forced by the mathematics. The data scientist who insists on doing causal modeling without consulting the domain expert is, whether she realizes it or not, *substituting her own implicit assumptions* for the expert's explicit ones. The implicit assumptions are typically worse than the expert's, because they are not informed by the substantive knowledge of the domain — and because, being implicit, they are not available for inspection or critique.

The pattern is so universal that it deserves a name: *the Expert in the Room*. Every causal model needs one, and every architecture for causal modeling has to make space for one. Chapter 14 in Part Three takes up this principle in detail and describes a discipline, *Knowledge Engineering with Bayesian Networks*, that has been developing methods for structured expert elicitation for several decades.

---

## How Interventional Reasoning Resolves Equivalence

If observational data cannot distinguish among Markov-equivalent DAGs, what can? The answer is intervention.

Consider the simplest case: the chain *X → Y* and the reverse chain *Y → X*. They are Markov equivalent in the observational data — both produce the same correlation between X and Y. But they make different predictions about interventions.

If we *intervene* on X, setting it to a specific value by external manipulation, then in the chain *X → Y* we should see Y respond, while in the reverse chain *Y → X* we should see no effect on Y (because X has no causal arrow into Y in that diagram). A single intervention experiment, even on a single variable, distinguishes the two diagrams unambiguously.

The general principle is: while observational data cannot orient every edge, interventional data can. An intervention on X, combined with measurement of all other variables, identifies all the edges out of X. A complete set of interventions — one on each variable — would identify the entire diagram.

This is, in essence, what randomized controlled trials provide. The reason RCTs are the gold standard for causal inference is not magical; it is that they provide interventional data, which observational data cannot replace. Chapter 11 takes up RCTs in detail.

But interventional data is expensive, slow, and often impossible to obtain. The role of expert knowledge is to substitute for interventional data in cases where intervention is infeasible. The expert's claim that *Y depends on X, not the other way around* is, in effect, a claim about what an intervention would show. The expert is reporting the result of an intervention she has not performed but has good reason to predict.

This is why the domain expert is the central figure in causal modeling. She is not bringing arbitrary opinions to the data; she is bringing causal knowledge derived from past observation, theory, and accumulated experience. Her role is to identify the orientation of edges that the current data cannot orient. When she gets it right, the resulting model will support correct interventional and counterfactual reasoning. When she gets it wrong, it will not — and the responsibility for the error is shared between her and the architecture that elicited her input.

---

## A Worked Example

Consider a small system with four variables: a marketing campaign (M), customer engagement (E), product usage (U), and retention (R). We have observational data on all four. A causal discovery algorithm returns a CPDAG with the following structure:

- *M → E* (directed; the algorithm has identified that M precedes E and the data show a v-structure that orients this edge)
- *E — U* (undirected; the data cannot orient this)
- *U — R* (undirected; the data cannot orient this)
- No edge between *M* and *U*, *M* and *R*, or *E* and *R*

The algorithm tells us: the marketing campaign affects engagement; engagement and usage are causally related, but we cannot tell from the data which direction; usage and retention are causally related, but again we cannot tell the direction.

The analyst now consults the domain expert. The expert says: usage causes engagement (more product usage leads to higher engagement scores) and usage causes retention (customers who use the product more are more likely to renew). She supports this with reasoning about the mechanism: the usage event is logged by the system; the engagement score is computed downstream from usage events; retention is a downstream consequence of cumulative usage.

The analyst orients the edges accordingly: *U → E*, *U → R*. The resulting fully directed graph is *M → E*, *U → E*, *U → R*. This is the model on which downstream causal effect estimation will run.

Note what has happened. The data identified a structure (the skeleton and one v-structure). The expert added directional commitments based on her knowledge of the system. The combination produced a fully specified model. Neither the data alone nor the expert alone would have produced it. This is the standard pattern of causal modeling, and it is the pattern the rest of this book builds on.

---

## When the Equivalence Class Is Large

The example above had only four variables and a manageable equivalence class. In practice, equivalence classes can be enormous. A CPDAG with twenty variables can have thousands of members. The analyst cannot examine each one individually; she must rely on the expert's domain knowledge to orient the undirected edges, accepting that some orientations may be wrong and conducting sensitivity analysis to gauge the impact.

The size of the equivalence class is, in some sense, a measure of how much the observational data identifies versus how much remains for the expert. A small equivalence class means the data is doing most of the work. A large equivalence class means the expert is. This is a useful diagnostic. If the equivalence class is too large to be confidently resolved by domain knowledge, the appropriate response is to seek interventional data — to run a controlled experiment on at least one variable, which will further constrain the class.

There is also a useful negative result. If the equivalence class contains DAGs that produce dramatically different causal effect estimates, then the model is sensitive to the expert's orientation choices. This is information the analyst should report. If the equivalence class is large but the resulting estimates are similar across its members, then the model is robust, and the orientations the expert has not yet made do not affect the answer to the question we are asking.

---

## Practical Implications

Three practical implications follow from the equivalence problem.

**First: causal discovery from observational data is partial.** Any algorithm that promises to recover a complete causal model from data alone is, at minimum, oversimplifying. The honest output is the CPDAG. Tools that report a single fully directed graph are usually making implicit assumptions — often that the variables are arranged in temporal order, or that the relationships are linear with non-Gaussian noise, or some other assumption that resolves equivalence in a particular way. These assumptions should be made explicit.

**Second: the expert's role is structural, not decorative.** The expert is not validating the model; she is constructing it. Her judgments about edge direction are first-class inputs to the causal inference, and the architecture of the modeling exercise should be built around eliciting them carefully. Chapter 16 in Part Three takes up the architecture of expert elicitation in detail.

**Third: the model is testable.** Even though the orientation of some edges depends on expert input, the diagram as a whole has empirical content. It implies specific conditional independence relations among the variables. If the data violate those relations, the diagram is wrong, and we have to revise it. This is the empirical discipline of causal modeling: the model is not derived from data, but it is constrained by data, and the constraints are sharp enough to falsify it.

---

## Looking Forward

Chapter 8 takes the next step. Given a fully specified causal diagram, how do we estimate the causal effect of one variable on another from the data? The methods include the backdoor criterion (which we previewed in Chapter 1), double machine learning (a modern method for handling high-dimensional confounders), and causal forests (which estimate heterogeneous treatment effects — the way an effect varies across subpopulations). The unifying theme is that the diagram tells us which adjustments to make; the data, properly adjusted, gives us the effect.

---

**What would change my mind:** A demonstration that observational data, in some specific class of problems, can identify causal structure beyond the Markov equivalence class. There are extensions of the standard framework — for example, exploiting non-Gaussianity in the noise terms — that resolve some equivalences. None of them are universal, and all of them require additional assumptions. I would update if shown a method that resolves equivalence under assumptions weaker than the ones currently on the table.

**Still puzzling:** How best to communicate the necessity of expert input to organizations that have invested heavily in data-driven decision-making and are uncomfortable with the idea that data alone is insufficient. The mathematics is clear; the cultural shift is harder. I do not have a confident answer to the change-management problem, and I suspect that solving it is more important than solving any technical problem in causal inference.

---

**Tags:** Markov equivalence class, CPDAG completed partially directed acyclic graph, causal discovery algorithms PC GES NOTEARS, expert knowledge mathematically necessary, observational vs interventional identification
