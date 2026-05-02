# Chapter 17 — Resolving the Graph

*PC, GES, NOTEARS, FCI; the CPDAG handoff between expert elicitation and data-driven discovery; when to gather more data and when to run more expert sessions; and the validated graph as a living artifact under governance.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *Resolving the Graph*
- *The CPDAG Handoff*
- *The Validated Graph as a Living Artifact*

**TL;DR.** The forty-five-minute interview from Chapter 16 produces a Completed Partially Directed Acyclic Graph (CPDAG) — a graph with directed edges where the expert committed on direction and undirected edges where she could not. Resolving the undirected edges, refining the directed ones against data, and confirming the overall structure is the work of this chapter. Four algorithm families do most of the work — PC, GES, NOTEARS, FCI — each with strengths and limits the practitioner has to understand. When the data and the expert disagree, the diagnostic is whether the disagreement is *aleatory* (more data will resolve it) or *epistemic* (only another expert session will). Once resolved, the graph is not finished; it is a versioned artifact that has to be governed, monitored for drift, and re-elicited when the world changes enough.

---

## After the Interview

A product team has just finished a forty-five-minute elicitation with the chief product officer. The CPDAG is on the screen. Eighteen variables, twenty-four edges. Six of the edges are undirected — the CPO knew the variables were causally related but could not commit on which direction the arrow goes. Three of those six undirected edges are on the critical path for the question the team is trying to answer. *If we redesign the activation flow for new users, what is the effect on twelve-month retention?*

The team has options. They have observational data — twenty-four months of user behavior, hundreds of thousands of accounts, every event the product logs. They could run causal discovery algorithms against the data and see whether the algorithms can orient the undirected edges. They could schedule another expert session — perhaps with the head of growth, who has different intuitions than the CPO. They could conduct a small randomized experiment on a subset of new users. They could combine all three.

The choice depends on what is producing the ambiguity, and the chapter is largely about how to make that diagnosis. Some of the undirected edges are undirected because the data have not yet been examined; an algorithm running on the data may resolve them in seconds. Others are undirected because the data, no matter how much of it is available, cannot resolve them — Markov equivalence again, the structural impossibility from Chapter 7. For these, more data will not help, and only further expert input or experimental intervention will. A third category sits between, where the data may resolve the ambiguity but not with the statistical power available; more data of the right kind would help, but not the same kind the team currently has.

The diagnostic moves the team from the elicitation phase to the operational phase. The CPDAG is the handoff artifact. From here forward, the team is doing engineering — running algorithms, evaluating their outputs, deciding which structural commitments to trust, governing the resulting graph as it evolves. The forty-five-minute interview gave them a draft; this chapter is about how the draft becomes an instrument that can support a Living Model in production.

---

## Four Algorithms, Four Paradigms

The literature on causal discovery contains dozens of algorithms. Four of them have become enough of a lingua franca that any practitioner working in this space should understand what each does and when to reach for it. They divide into three paradigms — constraint-based, score-based, and continuous-optimization-based — and the choice among them depends on the data, the assumptions the practitioner is willing to make, and the question the graph has to support.

**The PC algorithm** is the workhorse of constraint-based discovery. The name comes from its inventors, Peter Spirtes and Clark Glymour. The algorithm starts from a fully connected graph and prunes edges by running conditional independence tests on the data. *Are X and Y independent? Are they independent given Z? Given Z and W?* For each pair of variables, the algorithm searches for a conditioning set that renders them independent. If one is found, the edge between them is removed; if none can be found, the edge stays. After the skeleton — the set of remaining edges, ignoring direction — is identified, the algorithm orients edges by detecting *v-structures*: configurations where two variables are not adjacent but share a common neighbor whose conditional independence pattern reveals it as a collider. These v-structures fix some edge directions; further orientation rules propagate the directions where they can.

The PC algorithm is fast, well-understood, and produces a CPDAG. Its principal weakness is its assumption of *causal sufficiency* — that there are no unmeasured common causes of the variables in the system. This is a strong assumption that is rarely strictly true, and the PC algorithm's output is unreliable when latent confounding is present. The algorithm is also sensitive to errors in the conditional independence tests, particularly in small samples or with high-dimensional data; a single false test result can cascade into a structurally wrong graph.

**Greedy Equivalence Search (GES)** approaches the same problem from a different direction. Rather than testing independencies, it searches over the space of CPDAGs for the one that best fits the data, where *best fits* is defined by a penalized likelihood score (the Bayesian Information Criterion is the most common choice). The search proceeds in two phases: a *forward* phase that adds edges to improve the score, and a *backward* phase that prunes edges to remove unnecessary complexity. GES has a desirable theoretical property — it is *consistent* in the large-sample limit, meaning it converges to the true CPDAG when the assumptions hold and enough data is available. In practice, GES often produces more reliable orientations than PC when the data are well-behaved, although it shares PC's vulnerability to causal-sufficiency violations.

**NOTEARS** — *Non-combinatorial Optimization via Trace Exponential and Augmented Lagrangian for Structure learning* [verify exact expansion] — represents a more recent paradigm. Its insight is to recast the discovery problem as a continuous optimization rather than a combinatorial search. The number of possible DAGs over n variables grows super-exponentially, which makes exhaustive search infeasible for any non-trivial problem. NOTEARS replaces this by parameterizing the graph as a real-valued adjacency matrix and expressing the acyclicity constraint as a smooth differentiable function. The result is a problem that can be solved with standard continuous-optimization methods, and the methods scale to systems with thousands of variables that constraint-based and score-based approaches cannot handle.

NOTEARS is, however, sensitive to specific properties of the data — particularly the distribution of the noise terms — and several published critiques have shown that under some conditions it can return spurious structures. The algorithm and its successors (DYNOTEARS for time-varying systems, several variants for non-Gaussian noise) are an active research area, and the practitioner should treat NOTEARS results with the same scrutiny they would give any other algorithm: as input to a process that includes verification, not as output that can be used directly.

**Fast Causal Inference (FCI)** is the algorithm to use when causal sufficiency is implausible — when the practitioner suspects latent confounders that are not in the variable set. FCI extends the PC framework to handle latent variables and selection bias, and its output is not a CPDAG but a *Partial Ancestral Graph* (PAG). A PAG distinguishes more types of edges than a CPDAG: directed edges with arrowheads on both sides represent definite causal influence; circles represent uncertainty about whether a latent confounder mediates the relationship. The PAG is a richer representation than the CPDAG, and it is honest about the kinds of structural ambiguity that latent variables introduce.

FCI is computationally more expensive than PC and produces graphs that are harder to read and harder to use directly for downstream analysis. The trade-off is that FCI's output does not silently assume the data is unconfounded. In domains where latent confounders are likely — clinical medicine, social science, much of organizational behavior — FCI's PAG is the more honest object to start from, even if the team eventually has to commit to a directed graph for operational use.

The four algorithms are not interchangeable. PC is the default for clean, well-measured systems with no obvious latent confounders. GES is the default when one wants the score-based discipline of likelihood maximization. NOTEARS is the default for high-dimensional systems where combinatorial methods are infeasible, with the caveat that its results require careful verification. FCI is the default when latent confounders are likely and the practitioner wants the algorithm to admit it. A mature deployment of the architecture from this chapter runs more than one algorithm and compares the outputs; consistent results across algorithms are stronger evidence than results from any one of them.

---

## The CPDAG Handoff

Whichever algorithms the team runs, the output is a graph — typically a CPDAG, sometimes a PAG, sometimes a fully directed graph if the algorithms have made strong assumptions. This output is then compared against the CPDAG that came out of the Chapter 16 elicitation. Three patterns are possible.

**Pattern one: agreement.** The directed edges in the elicitation CPDAG are confirmed by the algorithm. The undirected edges in the elicitation are oriented by the algorithm in a way the expert finds plausible when shown the result. This is the desirable pattern, and it is more common than skeptics expect. When the expert and the data agree, the team can move forward to parameterization with high confidence in the structure.

**Pattern two: refinement.** The algorithm produces a graph that is consistent with the elicitation but adds or sharpens specific commitments. Perhaps the expert was uncertain whether two variables had a direct edge or a mediated relationship; the algorithm finds support in the data for one or the other. Perhaps the expert had committed on direction at the boundary of her domain knowledge; the algorithm confirms or contradicts the commitment based on conditional independence patterns. Refinement is the usual case, and it is the pattern that most justifies the data-driven step. The data refines what the elicitation could not specify.

**Pattern three: disagreement.** The algorithm produces a graph that contradicts a specific commitment from the elicitation. The expert said *A causes B*; the algorithm finds the conditional independence pattern more consistent with *B causes A* or with both being effects of an unmeasured third variable. This pattern is the most diagnostically valuable, because it surfaces a question that has to be resolved before the graph can be trusted.

The protocol for resolving disagreements is what separates a defensible Living Model from a fragile one. The protocol I recommend has three steps.

First, examine the algorithm's output for sensitivity. Run the algorithm with different conditional independence tests, different penalty parameters, different bootstrap subsamples of the data. If the contested edge orientation is stable across these variations, the algorithm's claim is well-supported. If it flips depending on the choices, the algorithm's claim is not strong enough to override the expert.

Second, examine the expert's commitment for vulnerability to the biases of Chapter 15. Was the expert relying on collider-prone reasoning when she made the commitment? Was she simplifying a feedback loop? Was she importing a mechanism from another domain? If the expert's commitment has a recognizable bias signature, the algorithm's contradiction is more credible.

Third, if the disagreement persists after both checks, schedule another expert session. This time, present the disagreement directly: *the data are most consistent with this orientation; you committed to the other; what would explain the discrepancy?* Often the second elicitation surfaces a consideration that did not come up in the first — a variable the model is missing, a temporal nuance, a subpopulation effect. The second session is short (typically ten to twenty minutes) because the question is targeted, and its output is either a revised commitment that resolves the disagreement or a documented note that the disagreement remains and the team has chosen one orientation while flagging the alternative for sensitivity analysis.

The handoff is the point in the architecture where the data-driven and expert-driven phases meet. It is not a single moment; it is a structured exchange between two sources of knowledge, each of which has things the other does not. The exchange is what produces the validated graph.

---

## More Data or More Experts?

When the graph is not yet validated — when undirected edges remain, or when disagreements persist — the team faces a strategic question. *Should we collect more data, or should we run more expert sessions?* The two are not interchangeable. Either may be the right move; making the right choice depends on diagnosing the source of the remaining uncertainty.

The diagnostic categories are the philosophically familiar ones: *aleatory* and *epistemic*. Aleatory uncertainty is uncertainty about a quantity whose value is determined by data. With enough of the right data, the quantity becomes known. Epistemic uncertainty is uncertainty about structure or mechanism — about which causal model is correct in the first place. No amount of data of the kind we have can resolve it, because the alternatives are observationally indistinguishable.

**More data is the right move when:**

The skeleton of the graph — which edges exist, regardless of direction — is unstable across resamplings. Bootstrap the data, rerun the algorithm, see whether the same edges appear each time. If they do not, the issue is statistical power: the conditional independence tests are noisy because the sample is too small or the dimensionality is too high relative to the sample. More data of the same kind will stabilize the skeleton.

The structure is settled but specific quantities are not. Once the structure is fixed, more data refines the parameters — the strengths of effects, the conditional probability tables — without affecting which way the arrows point. This kind of refinement is most productively pursued with targeted data collection: more observations on the variables whose conditional distributions are least precisely estimated.

A specific subpopulation is underrepresented. If the question of interest concerns customers in a particular segment, region, or use case, and the existing data is thin on that segment, more data from the segment can resolve uncertainties that were specific to it without affecting the rest of the model.

**More expert sessions are the right move when:**

Edges remain undirected after the algorithm has run. If the algorithm's CPDAG output still has undirected edges in the parts of the graph that matter for the question being asked, the data alone has done what it can. Markov equivalence is an absolute limit on what observational data can identify, and resolving the residual ambiguity requires either expert input or experimental intervention.

The data and the expert disagree, and the disagreement is structural rather than parametric. If the algorithm's output contradicts the elicitation in a way that does not depend on parameter choices, the disagreement is about which causal structure is correct, and only further expert input or controlled experiment can adjudicate it.

The model is in a domain where the expert's prior knowledge is much stronger than what the available data can reveal. In novel domains, in domains with very thin observational data, in domains where the expert has experimental experience the data cannot capture, the expert's input is the higher-information source. Investing in expert sessions before investing in data collection is the right priority.

The data may exist but capturing it is more expensive than running another expert session. This is often the case in corporate settings, where the data of interest may be locked behind compliance reviews, legacy system migrations, or sampling difficulties. A two-hour expert session is cheap compared to a six-month data-collection effort.

**Both at once is the right move when:**

The stakes of the decision are high enough that defensibility requires triangulation. For decisions that will be reviewed by a board, audited by regulators, or contested in litigation, having both data and expert support for the same conclusion is more defensible than having either alone. The cost of the additional review is justified by the reduced risk of being wrong in a consequential way.

The system is changing fast enough that data and expertise are individually outdated. In rapidly evolving environments — emerging product markets, post-pandemic shifts, technological transitions — the data may reflect a world that no longer exists, while the expert's intuition may have been formed in conditions that no longer hold. Combining both, with appropriate weighting, is the only honest approach.

The diagnostic itself is uncertain. If the team cannot tell whether the remaining ambiguity is aleatory or epistemic, the right move is to do a small amount of both — a targeted data analysis to test a specific independence, a short expert session focused on the most consequential undirected edge — and see which produces more clarity. The cost of the small parallel investment is small; the value of resolving the diagnostic is high.

The decision framework is not a formula. It requires judgment, and it requires the team to be honest about what kind of uncertainty they are facing. The temptation is always to default to the kind of investment the team is already comfortable with. Data teams default to more data; consulting teams default to more expert sessions. Mature deployments resist these defaults and choose based on the diagnostic.

---

## The Validated Graph as a Living Artifact

The graph that emerges from the handoff and the resolution of disagreements is not the end of the work. It is the start of a different kind of work: the governance of a living artifact.

The metaphor matters. A static document — a strategy memo, an architecture diagram, a methodology specification — is delivered, reviewed, and filed. The validated causal graph is not delivered and filed. It is checked into version control, where it accumulates a revision history. Each change to the graph — an added node, a re-oriented edge, a refined parameter — is associated with the evidence that justified the change, the person or process that made it, and the date the change went live. The graph has provenance.

This is closer to how software is governed than to how analysis is governed. The discipline of software engineering has, over the past several decades, developed practices for managing artifacts that change — version control, code review, continuous integration, deployment monitoring. The same practices, adapted to the specifics of causal graphs, are what governance of a Living Model looks like.

Specifically, four practices are worth describing.

**Version control.** The graph is checked into a repository alongside the metadata that gives each edge its provenance — which expert oriented it, which algorithm confirmed it, which data set was the basis for the parameters. Each commit to the repository is associated with a description of what changed and why. The repository is the source of truth: when there is a question about what the model says, the answer is whatever the latest commit says.

**Drift monitoring.** As new data accumulate, the data start to imply structures that may differ from the validated graph. The monitoring system periodically reruns the discovery algorithms against the latest data and compares the result to the validated graph. The comparison can be quantified — the *Jaccard distance* between the edge sets is a common metric, with values close to 1 indicating high similarity and values dropping toward 0 indicating that the data is implying a different structure. When the drift exceeds a threshold, the system alerts the team that a re-elicitation may be needed. The threshold is tuned to balance the cost of re-elicitation against the cost of operating on a stale graph.

**Re-elicitation triggers.** Drift monitoring triggers re-elicitation when the data drifts; other triggers exist as well. A material change in the business — a new product line, a market entry, an organizational restructuring — should trigger a review of which variables are still relevant and whether new ones need to be added. A regulatory change or a major external event may invalidate parts of the graph. A periodic schedule (annually, semiannually) ensures that the graph receives review even when no specific trigger has fired. The re-elicitation is shorter than the original elicitation in most cases — it focuses on the parts of the graph that have changed or are suspect — but it is structured around the same protocol from Chapter 16.

**Auditability.** The graph and its history are queryable artifacts. When a recommendation produced by the Living Model is challenged — by a board member who is skeptical, by a regulator reviewing a decision, by a lawyer evaluating liability — the team can answer the challenge by walking through the provenance. *This edge was committed on the basis of expert input from this person in this session; it was confirmed by the data with this confidence; it has been stable through these monitoring cycles.* Auditability is what makes the graph a defensible artifact. A model whose conclusions cannot be traced to specific evidence is, in any high-stakes setting, indefensible regardless of how good the model is.

The four practices together constitute what I would call *causal governance maturity*. Organizations vary widely in how much of this they actually do. The lowest-maturity case treats the graph as a one-time deliverable — built for a specific decision, used, and forgotten. The highest-maturity case treats the graph as core organizational infrastructure, on the same governance footing as the data warehouse, the customer database, or the financial reporting system. The transition from low to high maturity is partly a technical investment (the version-control system, the drift-monitoring infrastructure, the audit logs) and partly a cultural investment (the discipline of treating the graph as a serious artifact rather than a one-off analytical product).

I want to emphasize one practical implication of the living-artifact framing. A Living Model that is not under governance will degrade. The world changes, the data shifts, the experts move on, the original commitments fade from institutional memory. Without explicit governance, what was a high-quality model at the time of construction becomes a low-quality model within a year or two. Organizations that invest in causal modeling without investing in governance are setting themselves up for the slow accumulation of legacy models that nobody trusts and nobody can update. The governance investment is not optional. It is what makes the modeling investment durable.

---

## Looking Forward

Chapter 18 takes up the next operational step. With a validated graph in hand, the question becomes: how do we use it to produce ranked recommendations? The graph specifies the structure of the system. To produce recommendations, we need to parameterize the structure (estimate the conditional distributions at each node), run the counterfactual machinery from Chapter 9, rank candidate interventions by their expected causal effect, and translate the ranking into a portfolio of decisions under the constraints the organization actually faces — budget, capacity, risk tolerance, regulatory limits. Chapter 18 walks through each of these steps and produces the input that Chapter 19's executive report will consume.

---

**What would change my mind:** A demonstration that one of the four algorithm families — most likely a successor to NOTEARS in the continuous-optimization paradigm — reliably produces directed acyclic graphs of comparable quality to expert-validated CPDAGs across a wide range of domains, without requiring expert input. The mathematical results on Markov equivalence make this hard, but additional assumptions (non-Gaussian noise, specific structural priors) can sometimes resolve equivalence in ways that look promising. I have not seen the resolution work robustly across domains. I would update if shown a method that did.

**Still puzzling:** How to handle the case where drift monitoring triggers re-elicitation but the experts who built the original model are no longer available. The provenance of the graph is documented, but the *reasoning* behind individual commitments is often tacit knowledge that did not get captured. A new expert, looking at the existing graph, may not have the context to evaluate whether a drift signal warrants a structural change or is an artifact of changes in the data-generation process. I do not yet have a clean architectural answer to the *graph stewardship* problem, and I suspect it will become more visible as more organizations operate Living Models for long enough to lose their original modelers.

---

**Tags:** PC GES NOTEARS FCI causal discovery algorithms, CPDAG handoff expert data integration, aleatory versus epistemic uncertainty, validated graph as living artifact, causal governance Jaccard drift monitoring
