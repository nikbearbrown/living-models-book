# Chapter 14 — The Expert in the Room

*Why causal graphs cannot be built from data alone, what Knowledge Engineering with Bayesian Networks knows after twenty years of refining structured elicitation, and why the discipline has not yet reached the corporate boardroom.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *The Expert in the Room*
- *The Knowledge Bottleneck*
- *What Two Decades of KEBN Knows and the Boardroom Doesn't*

**TL;DR.** Causal graphs cannot be built from data alone. This is a mathematical result — Markov equivalence — not a limitation of current algorithms. The discipline that has been working on the bridge between expert knowledge and causal structure for two decades is Knowledge Engineering with Bayesian Networks (KEBN). Its protocols have produced validated successes in medical diagnosis, military intelligence, and environmental risk assessment. The protocols are slow, require trained facilitators, and produce outputs that do not fit existing analytics workflows. That is why KEBN has not crossed over into corporate strategy work, and it is the gap the Living Model architecture in the next several chapters is built to close.

---

## Twelve Weeks in the Goulburn-Broken Catchment

In the early 2000s, a team of ecologists set out to model the impact of management interventions on native fish populations in the Goulburn-Broken Catchment, a watershed in Victoria, Australia. They were working with a problem the modern data science playbook is poorly suited for. The data were thin — fish populations are difficult and expensive to monitor, and the historical record had gaps measured in decades. The interactions among variables were poorly understood, with feedbacks among habitat quality, flow regime, predation, water temperature, and dozens of other factors. The interventions under consideration — adjusting environmental flow releases, restoring riparian vegetation, controlling invasive species — had never been tried at the scale being contemplated, so the outcome data the team would have wanted simply did not exist.

What the team had was experts. Six of them, with collective decades of fieldwork and academic study of the system. The team's question was whether the experts' implicit knowledge could be made explicit, structured into a probabilistic model, and used to support the management decisions the catchment authority was about to make.

For twelve weeks, a facilitator worked with the six experts through a structured elicitation process. They identified variables — not the dozens the experts initially wanted, but the smaller set that mattered most for the management question. They drew arrows — debating, refining, sometimes erasing and starting over. They specified probabilities — not as point estimates but as ranges, with confidence levels attached. They tested the model against scenarios the experts knew well, looking for places where the model's predictions surprised the experts and forcing the elicitation deeper into those cases.

What emerged at the end of the twelve weeks was a Bayesian network with about thirty nodes and a complete specification of the conditional probabilities at each node. The model could be run forward to predict the consequences of management interventions; it could be queried for the relative importance of different drivers of fish population dynamics; it could be updated as new monitoring data arrived. The catchment authority used the model to prioritize among the candidate interventions and to communicate the reasoning to stakeholders in a way that the underlying ecological intuitions had not previously supported.

Twelve weeks. Six experts. One model. No machine learning. No big data. The output was a working representation of how a complex ecological system actually functions, capable of supporting consequential decisions about how to manage it.

This chapter is about how that kind of model gets built, why it cannot be built any other way for problems like the catchment, and why — despite a track record of successes in environmental science, clinical medicine, and intelligence analysis — this style of work has not crossed over into the corporate boardroom. The next several chapters of Part Three describe the architectural moves that, I argue, can finally bring it across.

---

## Why the Data Cannot Do It Alone

Chapter 7 introduced the equivalence problem. The same observational data can be consistent with multiple, structurally different causal diagrams. The class of diagrams indistinguishable on the data is the Markov equivalence class. Causal discovery algorithms operating on observational data return, at best, the equivalence class — not a specific diagram within it.

When I described this in Chapter 7, I treated the equivalence class as a manageable thing — a set of candidate diagrams the analyst chooses among with the help of an expert. For small systems with strong v-structures, this characterization is fair. The class might contain a handful of diagrams, and domain knowledge resolves them quickly.

For real-world systems of any complexity, the picture is far worse. Recent theoretical results have established that for sparse random graphs — the kind that describe most real systems, in which each node connects to a small number of others — the size of the Markov equivalence class scales exponentially with the number of variables. For a graph of twenty variables, the equivalence class can contain thousands of diagrams. For fifty variables, the number is astronomical. The data do not tell you which diagram is right. They tell you only that one of them is, and that the algorithm cannot determine which.

The picture is even worse if we relax the assumptions that make Markov equivalence cleanly defined. If unmeasured confounders exist — and they always do — the relevant equivalence class is over Acyclic Directed Mixed Graphs (ADMGs), and the expected size of the class is super-exponential. If the system contains feedback loops — as ecological, biological, and most social systems do — the equivalence class for Directed Cyclic Graphs scales exponentially in a different but equally catastrophic way.

This is not a limitation we can engineer our way out of. It is not waiting for the next algorithm or the next compute jump. It is a mathematical property of the relationship between observational data and causal structure. The information needed to identify a unique diagram is not in the data; it has to come from somewhere else.

The somewhere else is one of two places. We can intervene — perform a randomized experiment, or a natural experiment, or a quasi-experiment that breaks the symmetry between candidate diagrams. Or we can bring in an expert, whose knowledge of the mechanism resolves what the data cannot. In most consequential settings — medical diagnosis without randomized trials, ecological management without experimental control of watersheds, military intelligence about adversaries who are not running A/B tests for our convenience — the only available source is the expert.

This is the precise sense in which the expert is mathematically necessary. The expert is not a convenience that supplements an otherwise complete data-driven workflow. The expert is the source of information without which the workflow cannot, in principle, produce a unique answer.

---

## The Knowledge Bottleneck

The expert is necessary. Getting what the expert knows into a usable model is hard. This is the *knowledge bottleneck*, named by Edward Feigenbaum in the early days of expert systems research, and it has been the central challenge of practical AI for forty years.

Feigenbaum's *Knowledge Principle* states the situation cleanly. The high-level performance of an intelligent system is primarily a function of the specific knowledge it can bring to bear, not the power of its inference engine. Modern machine learning has, at moments, made the principle look obsolete — large models trained on enormous corpora produce impressive results without explicit knowledge engineering. But the appearance is misleading. Where machine learning works, it works because the knowledge is implicit in the data. Where the data does not contain the knowledge — including, by mathematical necessity, the causal structure questions of this book — the knowledge has to come from somewhere, and the bottleneck reasserts itself.

What makes the bottleneck a bottleneck is the gap between *implicit* knowledge in the expert's head and *explicit* knowledge in a structured model. The expert can recognize a pattern in seconds, having seen thousands of cases, but cannot articulate the cues that triggered the recognition. The expert can predict the consequences of an action, but cannot list the causal pathways that produced the prediction. The expert can warn that a proposed model is wrong, but cannot reliably specify what the right model would look like. *Compiled knowledge* — the heuristics and intuitions of expertise — does not decompose easily into the variables, arrows, and conditional probabilities a Bayesian network requires.

Three specific obstacles make the externalization especially difficult.

The first is the *combinatorial explosion of conditional probability tables*. A node in a Bayesian network with three binary parents requires the specification of eight conditional probabilities. With four parents, sixteen. With five, thirty-two. With ten parents, more than a thousand. An expert asked to provide all of these will either give up, give random numbers, or provide answers that are internally inconsistent. The structure that the formal model demands exceeds what the expert can produce by introspection.

The second is *linguistic ambiguity*. Experts use the same word for different concepts and different words for the same concept, often without realizing it. A clinician might say "renal function" and mean creatinine clearance, or glomerular filtration rate, or one of several other measures that do not coincide. A finance executive might say "exposure" and mean different things in different sentences. When the elicitation pins the expert down on what the variable actually is, the expert often discovers that her own usage has been imprecise — and the imprecision was not visible to her until the formal process forced it into view.

The third is *the gap between recognition and explanation*. Experts often know that a relationship exists without being able to say why. The model demands not just the relationship but the structure that produced it. Asked to explain how she knows that increased water temperature is associated with reduced fish recruitment, an expert may produce a story that is itself underdetermined — citing several plausible mechanisms whose relative contributions she cannot say. The elicitation process has to navigate this without forcing the expert to invent a story that she does not actually believe.

A discipline that takes the knowledge bottleneck seriously, then, has to do three things at once. It has to extract knowledge from experts who cannot fully articulate what they know. It has to structure the knowledge into a form a probabilistic model can use. And it has to do both while controlling for the cognitive biases — anchoring, availability, overconfidence, motivated reasoning — that affect every act of human introspection. Knowledge Engineering with Bayesian Networks is the discipline that has been working on this problem for twenty years.

---

## What KEBN Knows

A KEBN engagement is structured around two outputs that have to be produced in order: the qualitative graph and the quantitative conditional probabilities. The qualitative phase comes first because the structure of the graph determines what conditional probabilities the model needs and what data, if any, can support them. The discipline has accumulated specific protocols for each phase.

**Variable identification.** The first move is to identify the variables that belong in the model. The temptation is to include everything the expert mentions; the discipline resists this. The right set of variables is the smallest set that captures the question being asked. A variable belongs if changing it would plausibly change the outcome of interest, and if it is either measurable or estimable from data the modeler has access to. Variables that are too broad ("organizational health"), too granular ("the temperature in conference room B at 2 p.m."), or too downstream of the decision to be actionable are excluded. The expert is asked, repeatedly, *what does this variable mean precisely; how would you measure it; what would a one-unit change in it look like.* Linguistic ambiguity gets pinned down at this stage.

**Edge elicitation.** Once the variables are identified, the structural questions begin. *Does X cause Y? Does Y cause X? Are both effects of some third variable?* Experts find these questions hard to answer in the abstract, and one of the most useful tricks the discipline has developed is to substitute *temporal precedence* for direct causal language. Instead of asking *which causes which*, the facilitator asks *which comes first*. Experts who cannot orient an arrow on causal grounds can usually do so on temporal grounds, and for most practical models the temporal answer is a sufficient proxy. When the temporal question itself is ambiguous (the variables operate on similar timescales, or the relationship is bidirectional within the modeling window), the facilitator notes the ambiguity and either expands the model to include lagged variables or accepts the bidirectionality and works around it.

**Consistency checking.** Once the graph is drafted, it is tested for internal consistency. Does the structure imply patterns the expert agrees with? Does it imply patterns the expert recognizes as wrong? A common move is to walk the expert through the d-separation implications of the graph — *under your model, X and Y should be independent given Z; does that match your intuition?* If it does not, something in the graph is wrong, and the facilitator probes until the discrepancy is resolved either by changing the graph or by surfacing an additional consideration the expert had not previously made explicit.

**Conditional probability assessment.** Once the graph is stable, the conditional probabilities at each node have to be specified. For nodes with few parents, this can be done directly — the expert provides the probability of each outcome conditional on each combination of parent values. For nodes with many parents, the combinatorial explosion makes direct elicitation infeasible, and the discipline has developed simplifying functional forms. *Noisy-OR* and *noisy-MAX* models assume that each parent contributes independently to the outcome, with a specified strength; the expert provides the per-parent strengths rather than the full table. The cost is a structural assumption (independence of parental effects) that may or may not hold; the benefit is an enormous reduction in elicitation burden.

**Confidence calibration.** Experts vary in how well-calibrated their confidence is. Some are overconfident, claiming high certainty about uncertain quantities; some are underconfident, hedging on quantities they are actually quite sure about. Calibration exercises — where the expert estimates quantities with known answers and the facilitator scores the calibration — help correct for systematic miscalibration before the real elicitation begins. The expert is also typically asked to provide ranges rather than point estimates, with explicit confidence levels — *the value is between 0.2 and 0.4 with 80% probability* — which allows the model to represent epistemic uncertainty rather than treating elicited probabilities as facts.

The practitioner's tool for managing the elicitation across multiple experts is one of two protocols.

The *Delphi method* uses sequential, anonymous rounds. Each expert provides estimates privately. The facilitator aggregates and shares the distribution back to the experts, who then revise their estimates having seen what their peers said. The aggregation continues until estimates converge or stabilize. The principal benefit is the elimination of "committee dynamics" — the bandwagon effect, the dominant personality, the social pressure to agree with the senior person in the room — that distort group judgment.

The *IDEA protocol* — Investigate, Discuss, Estimate, Aggregate — combines individual elicitation with structured discussion. Experts first investigate the question privately, then discuss in a moderated session that surfaces disagreements without forcing premature consensus, then provide private second-round estimates, then have those estimates mathematically aggregated. The structured discussion phase resolves linguistic ambiguity and exposes assumptions that pure Delphi rounds tend to leave hidden.

Both protocols share a common premise: structured group elicitation, properly run, produces estimates that outperform the best single expert in the group. The premise is supported by extensive empirical work across scientific disciplines. The implication for KEBN is that the right unit of analysis is not the individual expert but the elicitation panel, and the discipline has developed substantial methodology for assembling and running panels well.

---

## What KEBN Has Built

KEBN has been most visible in three application domains where the cost of being wrong is high enough to justify the slow, careful elicitation process.

**Medical diagnosis.** Bayesian networks have been built for a wide range of clinical conditions, from common diagnoses where they supplement standard clinical reasoning to rare or novel conditions where they fill a gap that standard reasoning leaves open. During the SARS outbreak in 2003 [verify specific reference], a Delphi-based BN was built to support diagnostic triage in settings where cases were appearing faster than the formal clinical literature could keep up; the model was updated as new clinical information accumulated, and supported decision-making in hospitals that did not have access to the specialized expertise centralized in the major research centers. Similar models have been built for personalized diabetes management, early Alzheimer's screening, and a variety of oncological decisions where the relevant probabilities depend on patient-specific factors that the published literature averages over.

**Environmental risk assessment.** The Goulburn-Broken Catchment work that opened this chapter is one of many such engagements. KEBN has been applied to coral reef management, to fisheries, to forestry, to invasive species control, and to climate adaptation planning. The common feature of these applications is the combination of high stakes, thin data, and complex feedbacks among biological, hydrological, and human-management variables. The discipline's success in this domain owes much to the willingness of environmental agencies to invest the time the protocols require, and to the clarity of the management questions — most of these engagements were tied to specific decisions that had to be made, with timelines that the elicitation could be paced against.

**Military intelligence and defense.** Anomaly detection in maritime traffic — identifying ships behaving in patterns that depart from a learned model of normal behavior — has been one of the longest-running KEBN applications in defense. The model combines expert templates of normal behavior with AIS tracking data from millions of vessels; the BN structure makes the model's reasoning auditable, which matters when an analyst's decision will affect interdiction or surveillance priorities. Similar models exist for cybersecurity threat detection, where the question is whether a pattern of network activity is consistent with normal operations or with adversary behavior, and for force protection planning, where the question is the probability of attack under various deployment configurations.

In all three domains, the BN serves a function that black-box machine learning cannot. It is auditable. The analyst, the clinician, the catchment manager can ask why the model is recommending what it is recommending, and the answer comes back in the form of arrows and conditional probabilities rather than as the inscrutable output of a deep network. Auditability matters in proportion to the stakes of the decision. In high-stakes settings, the cost of moving from a black-box recommendation to an auditable one is the cost of slower elicitation, and the cost is generally accepted.

---

## Why the Boardroom Has Not Embraced This

Given the track record, a reasonable question is why this discipline has not crossed over into corporate strategy work. The strategic decisions executives make have stakes comparable to those in clinical medicine and environmental management. The thin-data, novel-context conditions that make machine learning unreliable apply at least as strongly to strategy as to ecology. The boardroom should be a natural home for KEBN, and yet KEBN engagements in corporate settings remain rare.

Several structural barriers explain the gap.

**The elicitation is slow.** Twelve weeks, the duration of the catchment engagement, is the order of magnitude. Some clinical engagements have taken longer; some smaller-scale engagements have been faster. But the timescale is weeks to months of expert time, with a trained facilitator coordinating the process. Corporate strategy work operates on a timescale of weeks at most. The board needs an answer for the next meeting; the executive needs a recommendation for the operating committee on Friday. The KEBN timeline, on its current implementation, does not match the cadence of corporate decision-making.

**Trained facilitators are scarce.** KEBN facilitation is a skill that takes years to develop. The facilitator has to understand probability and graphical models well enough to translate expert statements into model commitments; she has to understand the domain well enough to recognize when an expert's claim does not parse; she has to understand group dynamics well enough to run the elicitation without falling into the cognitive-bias traps the protocols are designed to avoid. There are perhaps a few hundred people in the world with all three capabilities at the required level. The number of organizations that need them is in the tens of thousands. The supply-demand gap means that even the rare organizations that want a KEBN engagement struggle to find the personnel.

**The output does not fit existing analytics workflows.** A corporate analytics team is typically organized around regression-based, classification-based, or optimization-based models. The data flows in, the model fits, the predictions flow out, and the predictions feed dashboards or downstream applications. A KEBN model is shaped differently. It is qualitative as well as quantitative. It supports interventional and counterfactual queries that the existing dashboards do not have a place for. Its outputs are typically distributions rather than point estimates, and ranked recommendations rather than predictions. Slotting it into an existing analytics estate requires architectural work that most organizations do not know how to scope, let alone fund.

**The risk matrix is the incumbent.** In most corporate boardrooms, the existing tool for the kind of question KEBN addresses is the risk matrix or heat map — the 5×5 grid of probability and impact that we critiqued in Chapter 4. The risk matrix is fast, visually intuitive, and produces the kind of one-page summary boards are accustomed to consuming. It is also, as we showed in Chapter 4, structurally unable to handle interactions among risks and routinely produces decisions that a more rigorous analysis would not endorse. But it is incumbent. Replacing it with anything more sophisticated requires that the more sophisticated tool also be fast and visually intuitive, and KEBN in its traditional form is neither.

**KEBN engagements are typically one-off.** A medical BN is built for a specific clinical condition and remains in use, sometimes for decades, supporting diagnostic decisions in that condition. A corporate KEBN engagement is typically commissioned for a specific strategic question — a market entry, an acquisition, a major operational change — and once the question is answered, the model is shelved. The reusability problem means that the cost of the engagement is amortized over a single decision rather than spread across hundreds. The unit economics are bad.

**The discretization issue.** Many BN tools require continuous variables to be discretized into bins for computational tractability. The choice of bins is an arbitrary decision that affects the model's outputs, and stakeholders who notice this lose trust in models whose conclusions depend on bin definitions. Modern BN methods have made progress on this, but the perception persists, and it is a real barrier to adoption.

These barriers are real. They are not, however, insuperable. The Living Model architecture in the next several chapters is, in part, an answer to each of them. Chapter 16 describes an LLM-guided elicitation system that compresses the twelve-week timeline to a forty-five-minute first-pass conversation, with subsequent refinement against data. Chapter 17 describes how that first-pass model gets handed off to automated discovery algorithms that refine the structure and quantify the probabilities. Chapter 18 describes how the refined model produces the kind of ranked recommendation that fits into corporate decision workflows. The entire architectural pattern is designed to preserve the rigor of KEBN while addressing the operational mismatch that has kept it out of the boardroom for two decades.

---

## What This Chapter Has Set Up

Three things follow from the argument of this chapter, and the rest of Part Three depends on all three.

The first is that the expert is at the center of the Living Model architecture, not at its periphery. Most modern analytics architectures treat the domain expert as an occasional consultant — someone who validates a model the data scientists have built, or interprets results the model has produced. The Living Model architecture inverts this. The expert is the source of the structural commitments that make the model causal, and the model cannot be built without her. This inversion forces architectural choices that the next several chapters work through.

The second is that the elicitation is the architecturally critical step. If the elicitation is well-designed, the rest of the modeling — parameter estimation, query handling, recommendation generation — follows from established techniques. If the elicitation is poorly designed, no amount of downstream sophistication can recover. The reason most corporate efforts at causal modeling fail is that the elicitation step is treated as preliminary scoping rather than as the architecturally central step. Chapters 15 and 16 take up the elicitation directly.

The third is that the corporate context is different from the clinical, environmental, and intelligence contexts where KEBN has succeeded. Corporate decision-makers will not sit through twelve weeks of facilitated elicitation. The architecture has to compress the timeline. Chapter 16 describes how — by leveraging large language models as scaffolding for the elicitation, by compressing the multi-round Delphi protocol into a single guided conversation, and by structuring the conversation around the specific structural questions the discipline has identified as most important — the KEBN protocol can be brought into the boardroom timeframe without sacrificing the rigor that makes it work.

What has been missing for two decades is not the discipline. The discipline has been there, with its track record and its protocols. What has been missing is an architecture that operationalizes the discipline at corporate speed. That architecture is the subject of the chapters that follow.

---

## Looking Forward

Chapter 15 takes up a topic the discipline has been quieter about than it should have been: the systematic ways in which experts get causation wrong. The fact that experts are mathematically necessary does not mean experts are infallible. There are specific cognitive biases — collider blindness, feedback-loop simplification, domain-matching heuristics — that affect expert causal reasoning in predictable ways. A mature elicitation architecture has to know what these biases are and structure the elicitation to mitigate them. Chapter 15 catalogs the principal biases and their consequences. Chapter 16 then describes how the elicitation architecture engages the expert without falling into the traps Chapter 15 has cataloged.

---

**What would change my mind:** A demonstration that a fully automated causal discovery pipeline, operating on observational data alone, can recover causal structures comparable in quality to those produced by KEBN engagements in the same domain. The mathematical results on Markov equivalence suggest this should be impossible for non-trivial systems. There may be settings — large numbers of weakly connected variables, strong v-structures throughout the graph — where the automated approach is competitive. I have not seen such a setting in the corporate strategy domain. I would update if shown one.

**Still puzzling:** Whether KEBN's failure to cross over is fundamentally an operational problem (slow timelines, scarce facilitators, workflow mismatch) or whether there is also an epistemic resistance — corporate decision-makers preferring black-box recommendations because they let the decision-maker disclaim responsibility for the underlying reasoning. If the resistance is partly epistemic, then better operational architecture will not fully solve the problem; the cultural shift to comfort with explicit causal models will also have to occur. I do not yet know how much of the gap is operational and how much is epistemic.

---

**Tags:** Knowledge Engineering Bayesian Networks KEBN, Markov equivalence class scaling, Feigenbaum knowledge bottleneck, IDEA Delphi protocol structured elicitation, Goulburn-Broken Catchment ecological risk
