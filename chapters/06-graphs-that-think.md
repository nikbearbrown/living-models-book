# Chapter 6 — Graphs That Think

*DAGs as maps of mechanism, structural causal models, and the difference between an equation and a cause.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *Graphs That Think*
- *DAGs as Maps of Mechanism*
- *The Difference Between an Equation and a Cause*

**TL;DR.** A regression equation is a description of correlation; a directed acyclic graph is a claim about mechanism. The two look similar on the page and are routinely confused. The graph encodes an asymmetry — who listens to whom — that the equation cannot. Once we add functional form to the graph, we get a structural causal model: the apparatus that makes Rungs 2 and 3 of the Ladder of Causation computable.

---

## Two Equations, One Number, No Cause

Suppose you have two variables, X and Y, and you fit a linear regression. The result is an equation:

> Y = 0.6 X + 1.2

The slope is 0.6, the intercept is 1.2, and the fit is good. You can read this in any number of ways. *On average, an increase of 1 in X is associated with an increase of 0.6 in Y.* That is the safest interpretation. But people, especially people who want to make a decision, almost never stop there. They want to say: *X causes Y, with a coefficient of 0.6*.

Here is the difficulty. The regression does not know, and cannot know, which variable causes which. If we had regressed X on Y instead — fit a different equation,

> X = 1.4 Y − 0.7

— we would get a different slope and a different intercept. Both equations describe the same correlation in the same data. Neither tells us anything about cause. Statistics knows correlation. Statistics has never known cause.

This is not a recent insight. Sewall Wright, a geneticist working at the U.S. Department of Agriculture in the 1920s, was the first person to put it in writing with mathematical force. He pointed out that the regression coefficient and the *path coefficient* — his term for what we now call the causal effect — are different objects, even when they happen to be computed from the same data. The regression coefficient is symmetric; it does not care which variable is the cause. The path coefficient is asymmetric; it points from cause to effect, and it can be different in different directions.

Wright invented a formal notation for the asymmetry. He drew it as a diagram with arrows. *X → Y* means X is a cause of Y. *Y → X* means Y is a cause of X. The diagram cannot be reduced to an equation, because the equation is symmetric and the diagram is not. The arrows carry the information that the equation cannot.

This chapter is about that diagram, which has come to be called a directed acyclic graph (DAG), and about what we get when we add functional form to it: the structural causal model (SCM). These are the two objects on which the rest of the book runs.

---

## What a DAG Encodes

A causal diagram is a set of nodes connected by arrows. Each node is a variable. Each arrow points from a cause to an effect. *Acyclic* means there are no closed loops — you cannot start at a node, follow the arrows, and return to where you started. (We will return to the question of feedback loops, which violate this assumption, in a later chapter.)

What does the diagram claim? Three things, and only three things.

**One: structural commitment.** When we draw an arrow from X to Y, we commit to the claim that X is a direct cause of Y in the system we are modeling. *Direct* here means "not mediated by any other variable in the diagram." If the effect of X on Y is mediated entirely through another variable Z, then we draw two arrows — X → Z → Y — and no arrow directly from X to Y.

**Two: missing arrows are claims too.** When we *don't* draw an arrow from X to Y, we commit to the claim that there is no direct causal effect of X on Y. This is often the more important commitment. A diagram with arrows everywhere claims nothing; a diagram with most arrows missing claims a great deal.

**Three: no claim about magnitudes.** The DAG, on its own, does not tell us how *strong* the causal effects are. It tells us only their structure — who listens to whom. To get magnitudes, we have to add functions or, in the linear case, path coefficients. That is the move from DAG to SCM.

The graph is a language. Each diagram is a sentence in that language, encoding a specific theory about the structure of cause and effect in the system. Two researchers can disagree about which sentence is true; that is a substantive scientific disagreement. What the graph buys us is the ability to make the disagreement precise. Each researcher can draw their diagram, and we can examine what each diagram implies about the data we should observe.

---

## A Worked Example: Ice Cream and Drownings

The classic example of a spurious correlation is the relationship between ice cream sales and drowning deaths. Both go up in the summer; both go down in the winter; the correlation is real and quite strong. No competent statistician would conclude that ice cream causes drownings, but the data do not, on their own, rule it out.

There are at least three causal stories that produce the same correlation:

*Story 1.* Ice cream sales cause drownings. Perhaps cold ice cream gives swimmers cramps, perhaps the sugar disorients them. The correlation reflects a real causal effect.

*Story 2.* Drownings cause ice cream sales. Perhaps a tragic news story prompts comfort eating. The correlation reflects a real causal effect, going the other direction.

*Story 3.* Some third variable causes both. Hot weather causes more people to swim (and therefore drown), and also causes more people to buy ice cream. Ice cream and drowning are joint effects of a common cause.

The first two stories are absurd; the third is obviously the right one. But how do we *know*? The data are equally consistent with all three. Our knowledge that hot weather affects both swimming and ice cream consumption is causal knowledge. It comes from outside the data. It is the kind of knowledge a DAG can encode: *Heat → Swimming → Drowning*, *Heat → Ice cream sales*. The diagram makes the structural claim explicit, and once it is explicit, we can examine its consequences.

This is the structure of every causal-vs.-correlational dispute. The data tell us *that* X and Y are correlated. The DAG tells us *why* — through which mechanisms — and what would happen if we intervened on one of them. *Heat* is a confounder of ice cream and drownings; intervening on ice cream sales — handing out free cones at the beach, say — would not increase drownings, because ice cream is not a cause of drowning in the diagram. The graph predicts this. The data, on their own, do not.

---

## From DAG to Structural Causal Model

A DAG encodes the structure of cause-and-effect relationships. To get to numbers — to compute interventional and counterfactual probabilities — we need to add the functional form of those relationships. The result is a structural causal model.

For each node Y in the diagram, the SCM specifies a function:

> Y = f<sub>Y</sub>(parents of Y, U<sub>Y</sub>)

where the parents of Y are the nodes with arrows pointing into Y, and U<sub>Y</sub> is an exogenous noise term that captures everything else affecting Y that we have not modeled explicitly. The function f<sub>Y</sub> can be linear, nonlinear, deterministic, stochastic — anything appropriate to the system. It is a statement about the *mechanism* by which Y's parents determine Y's value.

A small example. Consider a system with three variables: temperature (T), ice cream sales (I), and drowning incidents (D). The SCM might be:

> T = U<sub>T</sub>
> I = α T + U<sub>I</sub>
> D = β T + U<sub>D</sub>

Here T is exogenous (driven only by its noise term U<sub>T</sub>), and both I and D are linear functions of T plus their own noise. This SCM is consistent with the DAG *T → I*, *T → D*. It encodes the claim that ice cream sales are not a cause of drownings; the only path between them goes through T.

Now suppose we want to ask a Rung 2 question: what is the probability of drowning if we intervene to set ice cream sales to a particular value? Symbolically, P(D | do(I = i)). The do-operator instructs us to *modify* the SCM: we delete the equation for I and replace it with the constraint I = i. The modified SCM becomes:

> T = U<sub>T</sub>
> I = i (set by intervention)
> D = β T + U<sub>D</sub>

Now D depends only on T and noise, with no direct or indirect dependence on I. The probability of drowning is the same regardless of what value we set ice cream sales to. The intervention does nothing. The SCM, modified by the do-operator, gives us this answer immediately.

That is the move from DAG to SCM, and it is the move that makes Rung 2 computable. The DAG tells us which arrows to delete; the SCM tells us how to compute outcomes once the arrows are deleted.

---

## Why the Same Data Can Fit Multiple DAGs

A point we touched on in Chapter 5 deserves a closer look here. The same observational data can be consistent with multiple, structurally different DAGs. This is a mathematical fact about the relationship between graphs and probability distributions, not a limitation of any particular algorithm.

Consider three two-node diagrams: *X → Y*, *Y → X*, and *X ← Z → Y* with Z unobserved. All three can produce the same correlation between X and Y. If we only observe X and Y, no test on the data can distinguish them. They are *Markov equivalent* — they imply the same independence and dependence relations among the observed variables.

This fact is the seed of the equivalence problem, which we take up in Chapter 7. For now, the implication is the one we have already noted: the choice among Markov-equivalent DAGs is not a statistical decision. It cannot be made from the data. It must be made on substantive grounds — knowledge of mechanism, temporal precedence, or controlled experiment.

This is why the DAG is a scientific commitment, not a derivation. The researcher draws the diagram based on what she knows about the system. She is not deriving the diagram from the data; she is using the data, in combination with the diagram, to estimate quantities the diagram identifies as estimable. The same data, combined with a different diagram, would yield different estimates of different quantities. The diagram is the prior commitment that gives the data meaning.

This sounds like a vulnerability, and in a sense it is. If the diagram is wrong, the estimates will be wrong. But the alternative — pretending we can do without a diagram — is worse. Without a diagram, the estimates are still based on causal assumptions, but the assumptions are hidden. The diagram makes them visible. Visible assumptions can be debated, tested against domain knowledge, refined, and ultimately falsified. Hidden assumptions cannot.

---

## What the Graph Does That the Equation Cannot

It is worth being precise about what the graph adds beyond the equation.

**The graph distinguishes cause from effect.** A regression of Y on X gives the same numerical fit as a regression of X on Y, in the sense that the data alone do not prefer one direction. The graph picks a direction.

**The graph distinguishes direct from indirect effects.** In a regression with multiple predictors, the coefficient on X tells you the effect of X on Y *holding the other predictors constant*. The graph tells you which predictors *should* be held constant — based on whether they are confounders, mediators, or colliders — and the answer is different for each role. We will see this in detail in Chapter 8.

**The graph supports interventional reasoning.** Given a graph, we can apply the do-operator and compute interventional probabilities. Without a graph, we have only conditional probabilities, which live on Rung 1. The graph is the bridge to Rung 2.

**The graph supports counterfactual reasoning.** Given a structural causal model — a graph with functions attached — we can perform the abduction-action-prediction procedure to compute counterfactual probabilities. Without the model, we have only the actual world; with the model, we have the counterfactual worlds that the actual world is consistent with.

**The graph makes assumptions explicit.** A regression equation buries its causal assumptions in the choice of predictors and the interpretation of coefficients. The graph puts them in plain view. Other researchers can challenge the diagram in a way they cannot easily challenge a regression specification.

**The graph supports a global view.** Regression analyzes one outcome variable at a time. The graph represents the entire system of variables and their relationships, and supports queries about any of them. The Living Model architecture in Part Three is built on this global view.

---

## The Graph as Operating System

A causal diagram is not the answer to a particular question. It is the operating system on which causal questions can be asked. Each query — *what is the effect of X on Y? what would have happened if we had done Z? which intervention has the highest expected value?* — runs on the diagram. The diagram does not answer the queries directly; it specifies what computations are needed to answer them, and what data those computations require.

This is a different stance from the one most data scientists take. The default assumption is that data is the primary asset and the model is a derivative — something you fit to the data after the fact. In causal inference, the model is the primary asset. The data is what you bring to the model to estimate quantities the model identifies as estimable. Without the model, the data is mute.

This stance has implications for how organizations should organize their analytical work. The first move on any consequential question is not "what data do we have?" but "what does our causal model of this domain look like?" The second move is "what queries do we want to answer?" The third move is "what data, given our model, would let us answer those queries?" Only then does data collection or analysis begin. We will return to this workflow in Part Three.

---

## Looking Forward

Chapter 7 takes up the central limitation of causal diagrams: the same observational data can be consistent with multiple structurally different DAGs. This is the equivalence problem, and it has a sharp mathematical resolution. The class of diagrams consistent with a given dataset is called the *Markov equivalence class*, and the canonical representative of that class is called a *Completed Partially Directed Acyclic Graph* (CPDAG). The CPDAG tells us which causal relationships the data can identify and which it cannot — and where, therefore, we must bring expert knowledge to bear.

The lesson of Chapter 7 will be that human domain knowledge is not a convenience that supplements data. It is a mathematical necessity for orienting edges that data alone cannot orient. This is one of the most important results of the causal revolution, and it is the reason every Living Model architecture in Part Three centers the expert at the heart of model construction.

---

**What would change my mind:** A demonstration that some non-graphical representation of causal structure carries more information than a DAG without sacrificing tractability. There are extensions of DAGs — chain graphs, ancestral graphs, structural equation models with feedback — that handle situations DAGs do not. None of them, in my reading, supersede the DAG as the primary tool for ordinary causal inference. I would update if shown a more powerful default representation.

**Still puzzling:** Whether the SCM's functional commitment is too strong in practice. The full machinery of counterfactuals requires not just the structure but the functions. In many real applications we have only the structure and partial information about the functions. The interesting work, both theoretical and applied, is in extracting as much as we can from partial specifications. This is a frontier I do not yet feel I have mapped completely.

---

**Tags:** directed acyclic graphs, structural causal models, regression vs causation, Sewall Wright path coefficients, graphs as maps of mechanism
