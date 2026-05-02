# Chapter 6 — Graphs That Think

*DAGs as maps of mechanism, structural causal models, and the difference between an equation and a cause.*

---

Suppose you have two variables, X and Y, and you fit a linear regression. The result is an equation:

$$Y = 0.6X + 1.2$$

The slope is 0.6, the intercept is 1.2, and the fit is good. You can read this in any number of ways. *On average, an increase of 1 in X is associated with an increase of 0.6 in Y.* That is the safest interpretation. But people — especially people who want to make a decision — almost never stop there. They want to say: *X causes Y, with a coefficient of 0.6*.

Here is the difficulty. The regression does not know, and cannot know, which variable causes which. If we had regressed X on Y instead — fit the equation the other way around — we would get a different slope and a different intercept. Both equations describe the same correlation in the same data. Neither tells us anything about cause. Statistics knows correlation. Statistics has never known cause.

This is not a recent insight. Sewall Wright, a geneticist working at the U.S. Department of Agriculture in the 1920s, was the first to put it in writing with mathematical force. He pointed out that the regression coefficient and what he called the *path coefficient* — the causal effect — are different objects, even when computed from the same data. The regression coefficient is symmetric; it does not care which variable is the cause. The path coefficient is asymmetric; it points from cause to effect and can be different in different directions.

Wright invented a formal notation for the asymmetry. He drew it as a diagram with arrows. *X → Y* means X is a cause of Y. The diagram cannot be reduced to an equation, because the equation is symmetric and the diagram is not. The arrows carry information the equation cannot.

<!-- → [DIAGRAM: Two side-by-side representations of the same X–Y relationship — left: a regression equation Y = 0.6X + 1.2 with a double-headed dashed arrow between X and Y labeled "symmetric: describes correlation only"; right: a DAG with a single-headed arrow X → Y labeled "asymmetric: encodes a causal claim"; caption reads "Same numbers, different objects"] -->

This chapter is about that diagram — the directed acyclic graph, or DAG — and about what we get when we add functional form to it: the structural causal model, or SCM. These are the two objects on which the rest of the book runs.

---

## What a DAG Encodes

A causal diagram is a set of nodes connected by arrows. Each node is a variable. Each arrow points from a cause to an effect. *Acyclic* means there are no closed loops — you cannot start at a node, follow the arrows, and return to where you started.

What does the diagram claim? Three things, and only three things.

First, a **structural commitment**. When we draw an arrow from X to Y, we commit to the claim that X is a direct cause of Y in the system we are modeling. *Direct* means "not mediated by any other variable in the diagram." If the effect of X on Y flows entirely through another variable Z, we draw two arrows — X → Z → Y — and no arrow directly from X to Y.

Second, **missing arrows are claims too**. When we don't draw an arrow from X to Y, we commit to the claim that there is no direct causal effect of X on Y. This is often the more important commitment. A diagram with arrows everywhere claims nothing; a diagram with most arrows missing claims a great deal.

Third, **no claim about magnitudes**. The DAG, on its own, does not tell us how *strong* the causal effects are. It tells us only their structure — who listens to whom. To get magnitudes, we add functions or, in the linear case, path coefficients. That is the move from DAG to SCM.

The graph is a language. Each diagram is a sentence in that language, encoding a specific theory about the structure of cause and effect in the system. Two researchers can disagree about which sentence is true — that is a substantive scientific disagreement. What the graph buys us is the ability to make the disagreement precise. Each researcher can draw their diagram, and we can examine what each implies about the data we should observe.

---

## A Worked Example: Ice Cream and Drownings

The classic example of a spurious correlation is the relationship between ice cream sales and drowning deaths. Both rise in summer; both fall in winter; the correlation is real and quite strong. No competent statistician would conclude that ice cream causes drownings — but the data, on their own, do not rule it out.

There are at least three causal stories that produce the same correlation.

*Story 1.* Ice cream sales cause drownings. Perhaps cold ice cream gives swimmers cramps. The correlation reflects a real causal effect.

*Story 2.* Drownings cause ice cream sales. Perhaps a tragic news story prompts comfort eating. The correlation reflects a real causal effect going the other direction.

*Story 3.* Some third variable causes both. Hot weather causes more people to swim — and therefore drown — and also causes more people to buy ice cream. Ice cream and drowning are joint effects of a common cause.

The first two stories are absurd; the third is obviously correct. But how do we *know*? The data are equally consistent with all three. Our knowledge that hot weather affects both swimming and ice cream consumption is causal knowledge. It comes from outside the data. It is exactly the kind of knowledge a DAG encodes:

$$\text{Heat} \rightarrow \text{Swimming} \rightarrow \text{Drowning}, \quad \text{Heat} \rightarrow \text{Ice cream sales}$$

The diagram makes the structural claim explicit, and once it is explicit, we can examine its consequences. *Heat* is a confounder of ice cream and drownings; intervening on ice cream sales — handing out free cones at the beach — would not increase drownings, because ice cream is not a cause of drowning in the diagram. The graph predicts this. The data, on their own, do not.

<!-- → [DIAGRAM: Three candidate DAGs for the ice cream–drowning correlation — (1) Ice Cream → Drowning, (2) Drowning → Ice Cream, (3) Heat → Ice Cream and Heat → Drowning with no direct edge between them; each labeled with its causal story; a caption noting that all three are consistent with the same observed correlation, and that only Story 3 survives domain knowledge] -->

---

## From DAG to Structural Causal Model

A DAG encodes the structure of cause-and-effect relationships. To get numbers — to compute interventional and counterfactual probabilities — we need to add the functional form of those relationships. The result is a structural causal model.

For each node $Y$ in the diagram, the SCM specifies a function:

$$Y = f_Y(\text{parents of } Y,\ U_Y)$$

where the parents of $Y$ are the nodes with arrows pointing into $Y$, and $U_Y$ is an exogenous noise term capturing everything else affecting $Y$ that we have not modeled explicitly. The function $f_Y$ can be linear, nonlinear, deterministic, or stochastic — anything appropriate to the system. It is a statement about the *mechanism* by which $Y$'s parents determine $Y$'s value.

A small example. Consider three variables: temperature ($T$), ice cream sales ($I$), and drowning incidents ($D$). The SCM might be:

$$T = U_T$$
$$I = \alpha T + U_I$$
$$D = \beta T + U_D$$

Here $T$ is exogenous — driven only by its noise term — and both $I$ and $D$ are linear functions of $T$ plus their own noise. This SCM is consistent with the DAG $T \rightarrow I$, $T \rightarrow D$. It encodes the claim that ice cream sales are not a cause of drownings; the only path between them runs through $T$.

Now suppose we want to ask a Rung 2 question: what is the probability of drowning if we intervene to set ice cream sales to a particular value? Symbolically, $P(D \mid do(I = i))$. The do-operator instructs us to *modify* the SCM: delete the equation for $I$ and replace it with the constraint $I = i$. The modified system becomes:

$$T = U_T$$
$$I = i \quad \text{(set by intervention)}$$
$$D = \beta T + U_D$$

Now $D$ depends only on $T$ and noise, with no dependence on $I$. The probability of drowning is the same regardless of what value we set ice cream sales to. The intervention does nothing. The SCM, modified by the do-operator, gives us this answer immediately.

That is the move from DAG to SCM, and it is the move that makes Rung 2 computable. The DAG tells us which equations to delete when we intervene; the SCM tells us how to compute outcomes once the deletion is done.

<!-- → [DIAGRAM: Side-by-side of the original SCM and the do-modified SCM for the ice cream example — left panel shows the three equations with arrows intact, right panel shows I = i with the incoming arrow from T removed; a label "do(I = i) severs this edge" pointing to the deleted arrow; caption: "Intervention is surgery on the model, not observation of the world"] -->

---

## Why the Same Data Can Fit Multiple DAGs

The same observational data can be consistent with multiple, structurally different DAGs. This is a mathematical fact about the relationship between graphs and probability distributions — not a limitation of any particular algorithm.

Consider three arrangements involving two observed variables: $X \rightarrow Y$, $Y \rightarrow X$, and $X \leftarrow Z \rightarrow Y$ with $Z$ unobserved. All three can produce the same correlation between $X$ and $Y$. If we observe only $X$ and $Y$, no statistical test on the data can distinguish them. They are *Markov equivalent* — they imply the same independence and dependence relations among the observed variables.

<!-- → [DIAGRAM: The three Markov-equivalent two-node structures — X → Y, Y → X, X ← Z → Y (with Z shown as a dashed node to indicate it is unobserved); beneath each, a note stating "same P(X,Y)" ; a label spanning all three: "No statistical test on X and Y alone can distinguish these"] -->

This is the seed of the equivalence problem, which we take up in Chapter 7. For now, the implication is the one we have already noted: the choice among Markov-equivalent DAGs is not a statistical decision. It cannot be made from the data. It must be made on substantive grounds — knowledge of mechanism, temporal precedence, or controlled experiment.

This is why the DAG is a scientific commitment, not a derivation. The researcher draws the diagram based on what she knows about the system. She is not deriving the diagram from the data; she is using the data, in combination with the diagram, to estimate quantities the diagram identifies as estimable. The same data, combined with a different diagram, would yield different estimates of different quantities. The diagram is the prior commitment that gives the data meaning.

This sounds like a vulnerability, and in a sense it is. If the diagram is wrong, the estimates will be wrong. But the alternative — pretending we can do without a diagram — is worse. Without a diagram, the estimates are still based on causal assumptions, but the assumptions are hidden. The diagram makes them visible. Visible assumptions can be debated, tested against domain knowledge, refined, and ultimately falsified. Hidden assumptions cannot.

---

## What the Graph Does That the Equation Cannot

It is worth being precise about what the graph adds beyond the equation.

The graph **distinguishes cause from effect**. A regression of $Y$ on $X$ gives the same numerical fit as a regression of $X$ on $Y$, in the sense that the data alone do not prefer one direction. The graph picks a direction.

The graph **distinguishes direct from indirect effects**. In a regression with multiple predictors, the coefficient on $X$ tells you the effect of $X$ on $Y$ holding the other predictors constant. The graph tells you which predictors *should* be held constant — based on whether they are confounders, mediators, or colliders — and the answer is different for each role. We will work through this in detail in Chapter 8.

The graph **supports interventional reasoning**. Given a graph, we can apply the do-operator and compute interventional probabilities. Without a graph, we have only conditional probabilities, which live on Rung 1. The graph is the bridge to Rung 2.

The graph **supports counterfactual reasoning**. Given a structural causal model — a graph with functions attached — we can perform the abduction–action–prediction procedure to compute counterfactual probabilities. Without the model, we have only the actual world; with the model, we have the counterfactual worlds that the actual world is consistent with.

The graph **makes assumptions explicit**. A regression equation buries its causal assumptions in the choice of predictors and the interpretation of coefficients. The graph puts them in plain view. Other researchers can challenge the diagram in a way they cannot easily challenge a regression specification.

The graph **supports a global view**. Regression analyzes one outcome variable at a time. The graph represents the entire system of variables and their relationships, and supports queries about any of them. The Living Model architecture in Part Three is built on this global view.

<!-- → [TABLE: Regression equation vs. structural causal model — rows: distinguishes cause from effect, handles direct vs. indirect effects, supports do-operator, supports counterfactuals, makes assumptions visible, global vs. single-outcome scope; columns: Regression Equation, DAG, SCM; cells: Yes/No/Partial with a one-sentence explanation each] -->

---

## The Graph as Operating System

A causal diagram is not the answer to a particular question. It is the operating system on which causal questions can be asked. Each query — *what is the effect of X on Y? what would have happened if we had done Z? which intervention has the highest expected value?* — runs on the diagram. The diagram does not answer the queries directly; it specifies what computations are needed and what data those computations require.

This is a different stance from the one most data scientists take. The default assumption is that data is the primary asset and the model is a derivative — something you fit to the data after the fact. In causal inference, the model is the primary asset. The data is what you bring to the model to estimate quantities the model identifies as estimable. Without the model, the data is mute.

This stance has implications for how organizations should organize their analytical work. The first move on any consequential question is not "what data do we have?" but "what does our causal model of this domain look like?" The second move is "what queries do we want to answer?" The third move is "what data, given our model, would let us answer those queries?" Only then does data collection or analysis begin. We will return to this workflow in Part Three.

---

## Looking Forward

Chapter 7 takes up the central limitation of causal diagrams: the same observational data can be consistent with multiple structurally different DAGs. This is the equivalence problem, and it has a sharp mathematical resolution. The class of diagrams consistent with a given dataset is called the *Markov equivalence class*, and the canonical representative of that class is called a *Completed Partially Directed Acyclic Graph* (CPDAG). The CPDAG tells us which causal relationships the data can identify and which it cannot — and where, therefore, we must bring expert knowledge to bear.

The lesson of Chapter 7 will be that human domain knowledge is not a convenience that supplements data. It is a mathematical necessity for orienting edges that data alone cannot orient. This is one of the most important results of the causal revolution, and it is the reason every Living Model architecture in Part Three centers the expert at the heart of model construction.

---

## Student Activities

**Problem 6.1 — Reading the Graph.** Consider a system with four variables: marketing spend (M), website traffic (W), product quality (Q), and sales (S). A colleague proposes the following DAG: M → W → S, Q → S, with no direct edge from M to S or from Q to W. (a) State in plain English the causal claims encoded by each arrow and each *missing* arrow in this diagram. (b) Identify one alternative DAG that would be consistent with positive correlations between all four variables. (c) Explain what data — beyond simple pairwise correlations — would help you distinguish between your two diagrams.

**Problem 6.2 — Regression vs. Causation.** A dataset of 10,000 employees shows that salary ($S$) and years of experience ($E$) have a strong positive correlation ($r = 0.71$). A regression of $S$ on $E$ gives a slope of 4,200 (dollars per year of experience). (a) Write out the regression equation and state what it can and cannot claim causally. (b) Propose two distinct causal DAGs consistent with this correlation — one in which $E$ causes $S$ and one that involves a third variable. (c) Using the $P(Y \mid X)$ vs. $P(Y \mid do(X))$ distinction from Chapter 1, explain what the slope of 4,200 does and does not entitle the HR director to conclude about the effect of a mentorship program that increases effective experience.

**Problem 6.3 — Writing the SCM.** Consider a three-variable system: hours of study ($H$), test anxiety ($A$), and exam score ($X$). Suppose the causal structure is $H \rightarrow X$, $A \rightarrow X$, and $H \rightarrow A$ (more studying reduces anxiety). (a) Draw the DAG. (b) Write a linear SCM for this system, introducing appropriate noise terms and coefficients. (c) Apply the do-operator to compute $P(X \mid do(H = h))$: write out the modified SCM and trace which edges are deleted and which remain. (d) Explain in one sentence why the modified SCM gives a different answer from the conditional distribution $P(X \mid H = h)$ estimated from observational data.

**Problem 6.4 — The Confounder, the Mediator, and the Collider.** Three structural roles in a DAG — confounder, mediator, collider — each require different handling when estimating causal effects. (a) Draw a minimal DAG illustrating each role. (b) For each, state whether you should condition on that variable when estimating the effect of $X$ on $Y$, and why. (c) Construct a realistic business example in which mistaking a mediator for a confounder would lead a decision-maker to recommend the wrong intervention. Express your answer in terms of the do-operator.

**Problem 6.5 — Markov Equivalence.** Consider the three two-node DAGs discussed in this chapter: $X \rightarrow Y$, $Y \rightarrow X$, and $X \leftarrow Z \rightarrow Y$ with $Z$ unobserved. (a) Explain in plain language why no observational test on $X$ and $Y$ alone can distinguish the first two. (b) Describe one experimental design — involving intervention, not just observation — that would distinguish $X \rightarrow Y$ from $Y \rightarrow X$. (c) Explain why the ability to intervene, rather than merely observe, is the critical tool for resolving Markov equivalence. Connect your answer to Rung 2 of Pearl's Ladder.

**Problem 6.6 — DAG Critique.** A consulting team presents the following claim: "We ran a regression of customer lifetime value (CLV) on customer satisfaction scores (CSS) and found a positive coefficient of 0.43. We recommend investing in satisfaction improvement programs." (a) Identify every causal assumption buried in this recommendation that the regression alone cannot support. (b) Draw two DAGs — one that would justify the recommendation and one that would undermine it — and explain what each predicts about the outcome of the proposed investment. (c) Design a minimal data-collection exercise that would allow you to choose between your two DAGs. (d) If the team's causal diagram turns out to be wrong, describe the class of error that results and name an analogous case from earlier in the book.

---

## Key Terms

**Acyclic.** The property of a directed graph in which no path following the arrows can return to the starting node. Acyclicity is a required property of a DAG used for standard causal inference. Systems with genuine feedback loops require extensions of the basic DAG framework.

**Collider.** A variable with two or more arrows pointing *into* it from variables on a path between cause and effect. Conditioning on a collider opens a path that is otherwise closed, inducing spurious associations between its parent variables. Colliders must not be conditioned on when estimating causal effects along paths they block.

**Confounder.** A common cause of both the treatment variable and the outcome variable, whose presence induces a correlation between them that does not reflect a direct causal effect of treatment on outcome. Confounders create the most common source of spurious correlation in observational data.

**Directed Acyclic Graph (DAG).** A graph in which each edge is an arrow pointing from cause to effect, and in which no sequence of arrows forms a closed loop. In causal inference, each arrow represents a direct causal relationship; each missing arrow represents the absence of a direct causal relationship. The DAG encodes the qualitative structure of a causal model without specifying the magnitudes of effects.

**Do-Operator ($do(\cdot)$).** The mathematical notation, introduced by Judea Pearl, for deliberate intervention. $P(Y \mid do(X = x))$ represents the probability of outcome $Y$ when $X$ is set to $x$ by action, as opposed to $P(Y \mid X = x)$, the probability of $Y$ when $X$ is merely observed to equal $x$. Applying the do-operator to an SCM means deleting the equation for the intervened variable and replacing it with a fixed value.

**Exogenous Variable.** A variable in a structural causal model whose value is determined entirely outside the model — by its own noise term, with no parent variables. Temperature in the ice cream example is treated as exogenous.

**Markov Equivalence.** Two DAGs are Markov equivalent if they imply the same set of conditional independence relations among the observed variables, and therefore cannot be distinguished by any statistical test on those variables. Resolving Markov equivalence requires either experimental intervention or domain knowledge.

**Mediator.** A variable that lies on the causal path between a treatment and an outcome, transmitting (part of) the effect of treatment on outcome. Conditioning on a mediator when estimating the total effect of treatment blocks the indirect pathway and biases the estimate. Mediators are useful when estimating *direct* effects specifically.

**Path Coefficient.** Sewall Wright's term for a coefficient in a causal model that represents the direct effect of one variable on another. Unlike a regression coefficient — which is symmetric and does not distinguish cause from effect — a path coefficient is asymmetric: it quantifies the effect of the parent variable on the child variable, not the reverse.

**Structural Causal Model (SCM).** A DAG augmented with a functional equation for each node, specifying how the node's value is determined by its parents and an exogenous noise term. The SCM is the mathematical object that makes Rung 2 (interventional) and Rung 3 (counterfactual) reasoning computable. The functional equations encode mechanisms, not just correlations.

**Structural Equation.** One equation in a structural causal model, of the form $Y = f_Y(\text{parents of } Y, U_Y)$. The equation specifies the mechanism by which $Y$ is determined. It is asymmetric: the right-hand side causes the left-hand side, not the reverse. This asymmetry distinguishes structural equations from regression equations, which are symmetric descriptions of correlation.

---

**What would change my mind:** A demonstration that some non-graphical representation of causal structure carries more information than a DAG without sacrificing tractability. There are extensions of DAGs — chain graphs, ancestral graphs, structural equation models with feedback — that handle situations DAGs do not. None of them, in my reading, supersede the DAG as the primary tool for ordinary causal inference. I would update if shown a more powerful default representation.

**Still puzzling:** Whether the SCM's functional commitment is too strong in practice. The full machinery of counterfactuals requires not just the structure but the functions. In many real applications we have only the structure and partial information about the functions. The interesting work — both theoretical and applied — is in extracting as much as possible from partial specifications. This is a frontier I do not yet feel I have mapped completely.

---

**Tags:** directed acyclic graphs, structural causal models, regression vs causation, Sewall Wright path coefficients, graphs as maps of mechanism
