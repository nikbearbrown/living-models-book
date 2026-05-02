# Chapter 16 — The Machine That Interviews the Expert

*LLM-guided causal elicitation, the Nina-framework architectural parallel, the forty-five-minute first-pass DAG, and the multi-agent design that makes consistency and equivalence-checking tractable.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *The Machine That Interviews the Expert*
- *Forty-Five Minutes to a First-Pass Causal Graph*
- *Why the Architecture Already Exists in Brand Strategy*

**TL;DR.** The architecture the corporate world needs for causal elicitation already exists, in a different field. LLM-guided interview systems in brand strategy — the Nina framework is one example — have demonstrated that a well-designed system prompt can do what a skilled human interviewer does: ask one question at a time, refuse to proceed until the answer is sufficient, hold prior answers in context, surface contradictions, and progressively build a structured output from unstructured expert knowledge. Applied to causal elicitation, the same architecture compresses the months-long Knowledge Engineering with Bayesian Networks engagement of Chapter 14 into a forty-five-minute first-pass conversation. The design is multi-agent: one agent elicits, one checks consistency, one flags equivalence ambiguities, one watches for the cognitive biases of Chapter 15. The output is a first-pass DAG suitable for handoff to the automated discovery algorithms of Chapter 17.

---

## The Founder's Interview

A founder sits down for a Nina interview at her brand strategist's recommendation. She has been told the interview will take about an hour. She has no idea what to expect.

The interview opens on a screen with a chat-like interface. The first question is direct: *Tell me, in two or three sentences, what your company does.* She types an answer. The system reads it, reflects part of it back to confirm understanding, and asks the next question. *Who is the person who buys what you sell, and what is going on in their life when they do?* She types. The system probes, gently. It does not move on. It asks for specifics where she has been general. It asks for an example where she has spoken in abstractions.

Forty minutes later, she has answered around eighty questions, ranging across her customers, her competitors, the moment of purchase, the moment of regret, the language her customers use when they recommend her, the language they use when they complain. The system has not let her skip questions. It has not let her speak in marketing jargon. When she said *we are an authentic brand*, it asked her what authentic meant in her specific context, and refused to accept the answer until she had said something substantive.

At the end of the interview, the system produces a creative brief. The brief contains a brand archetype, a set of positioning statements, a tone-of-voice guide, and three or four suggested directions for the next phase of work. The founder reads it. She recognizes it. The brief contains things she has known for years and never articulated, things her marketing agency has been chasing without quite landing, and a couple of things that surprise her — connections the interview surfaced that she had not consciously seen.

A skilled human brand strategist could have done the same work, with similar quality, in three to five hours of meetings spread across two weeks. The Nina framework — and several systems like it — does it in forty-five minutes, with no human strategist required for the elicitation step. The strategist's role shifts from interviewer to reviewer. The interview itself, the disciplined extraction of unstructured expert knowledge into a structured output, has been done by software.

This chapter is about how to do for causal elicitation what Nina has done for brand strategy. The architectural pattern transfers cleanly. The vocabulary is different — variables instead of brand attributes, edges instead of positioning statements, conditional independence instead of voice — but the structural moves are the same. The expert's tacit knowledge is the input. A disciplined sequence of questions is the apparatus. A structured causal model is the output. The whole thing fits in one focused conversation.

---

## The Insight: Interview as Software

The deepest insight from the Nina framework is that a structured interview is a piece of software, and that software can be written.

This sounds banal until you consider how the interview process has historically worked in causal modeling. A trained facilitator — typically a person with years of expertise in both probability theory and the substantive domain — sits with one or more experts. The facilitator asks structured questions, drawn from a protocol like IDEA or Delphi (Chapter 14). The facilitator interprets the answers, translates them into structural commitments on the diagram, and steers the conversation forward. The whole process takes weeks to months because the facilitator is a scarce resource and the experts have other obligations.

The Nina framework's contribution is to recognize that most of what the facilitator is doing can be specified. The questions to ask, the conditions for moving on, the responses that should trigger probing, the cases where the interview should slow down — all of these can be encoded as a system prompt or a structured workflow. The cases where the encoding fails, where genuine human judgment is required, are far rarer than the facilitator's day-to-day work suggests. Most of the interview is following a protocol that someone has written down. If the protocol can be written down, it can be executed by software.

This is not the claim that LLMs can replace expertise. They cannot. The expertise resides in the protocol — in the structure of questions and the response logic — and the protocol still has to be written by someone who understands the domain. What changes is that the expertise is encoded once and then executed many times, rather than being applied person-to-person. The economics shift dramatically. An interview that costs forty hours of expert facilitator time becomes an interview that costs forty-five minutes of expert time and almost nothing of facilitator time. The corporate adoption barrier described in Chapter 14 — that trained facilitators are scarce — dissolves once the facilitation is encoded.

The remainder of this chapter describes the protocol for causal elicitation specifically. The chapter assumes the protocol will be executed by an LLM-guided system, but the protocol itself is independent of the implementation. A team that wanted to run the same protocol with a human facilitator could do so; the system simply makes it cheaper and more consistent.

---

## Four Phases of Causal Elicitation

The protocol I describe here has four phases. Each phase produces a specific output that the next phase requires. The phases are not interchangeable; they have to happen in order.

**Phase one: variable identification.** The interview opens by asking the expert to enumerate the variables that matter for the question at hand. The questioning is structured around three filters. *What are you trying to decide or affect?* — this fixes the outcome variable. *What levers do you have available?* — this fixes the candidate treatment variables. *What other factors do you believe shape this outcome?* — this fixes the candidate confounders, mediators, and effect modifiers.

The interview then enforces the discipline that variables have to be measurable, or estimable from data the team has access to. Each candidate variable is probed: *how would you measure this? what would a one-unit change in it look like? what data already exists on it?* Variables that cannot be answered cleanly are either reframed in measurable terms or explicitly tagged as latent — present in the model but not directly observed.

The interview also enforces linguistic discipline. When the expert uses terms that admit multiple interpretations — *engagement, quality, performance, trust* — the interview probes for the specific operationalization. The expert is forced to commit. Linguistic ambiguity, which Chapter 14 identified as one of the principal obstacles to elicitation, is resolved at variable-identification time, before it can propagate into the structural commitments downstream.

The output of phase one is a list of variables, each with a precise operationalization, a measurement strategy, and an indicator of whether data exists. Phase one typically takes ten to fifteen minutes.

**Phase two: edge elicitation.** With the variables in hand, the interview moves to structural questions: which variables cause which? The Chapter 14 trick — *substitute temporal precedence for direct causal language* — is used here. Instead of asking *which causes which*, the interview asks *which comes first*. For most variable pairs, the temporal answer is unambiguous, and it serves as a sufficient proxy for causal direction. For pairs where the temporal answer is itself ambiguous, the interview either expands the model to include lagged versions of the variables or explicitly marks the pair as bidirectional and revisits it at the equivalence-resolution stage.

This phase is where the bias mitigations from Chapter 15 do their work. After each proposed edge, the interview asks two questions that probe the cognitive failures most likely to have produced an incorrect commitment.

The first is the collider check. *If we knew the value of this child variable and we knew the value of this parent, would knowing the other parent's value change our belief about the relationship?* This is a behavioral probe of whether the expert is implicitly conditioning on a variable that should not be conditioned on. If the expert says *yes, knowing the third variable would change things*, the interview surfaces the third variable, examines whether it is a collider, and adjusts the diagram accordingly.

The second is the feedback-loop probe. *When the outcome of this process happens, does it change the inputs to the next iteration of the process?* If the answer is yes, the interview asks for the time scale and either explicitly represents the feedback as time-indexed nodes or flags the cycle for special handling at the equivalence-resolution stage.

The output of phase two is a draft diagram with directed edges where the temporal answer was clean, undirected edges where it was not, and explicit annotations on cycles and time scales. Phase two typically takes fifteen to twenty minutes — the longest of the four phases.

**Phase three: equivalence resolution.** The draft diagram from phase two will, almost always, contain edges whose direction the expert has not been able to commit on. Some of these are genuine ambiguities that the expert cannot resolve; some are ambiguities that the expert could resolve with a different prompt. Phase three is the targeted resolution of the most consequential of these.

The interview computes which undirected edges, if oriented one way versus the other, would most affect the downstream causal effect estimates. (The computation is mechanical given the diagram and the question being asked; it can be done by the system without requiring expert input.) For each high-impact undirected edge, the interview poses a structural question to the expert. *Suppose we held variable X constant, by some external means, while letting Y vary as it would. Would changes in Y propagate to Z?* This is a behavioral probe that tests for interventional reasoning rather than associational reasoning. If the answer is *yes*, the edge orients one way; if *no*, the other way. The expert does not need to understand why this question resolves the structural ambiguity; she only needs to be able to answer it.

For the small number of edges that the expert genuinely cannot orient even with the structural prompt, the interview leaves them undirected and flags them for resolution by data — the handoff to Chapter 17. The output of phase three is a Completed Partially Directed Acyclic Graph (CPDAG) — the formal object Chapter 7 introduced — in which the directed edges are the ones the expert has committed on and the undirected edges are the ones the data will have to resolve. Phase three typically takes ten minutes.

**Phase four: confidence calibration.** The fourth and final phase asks the expert to specify, for each edge in the diagram, how confident she is in the orientation, and to provide rough magnitudes for the strength of each relationship. The interview uses the language of probability ranges rather than point estimates: *with how much certainty are you committing to this edge? what is the strongest you would say this effect could be? the weakest?* The interview also includes a small number of calibration questions on quantities the expert should know — *what fraction of cases like this have outcome Y?* — to detect systematic over- or underconfidence. Where the calibration questions reveal miscalibration, the system applies a correction to the elicited confidences before passing them downstream.

The output of phase four is a CPDAG annotated with confidence ranges and rough magnitude estimates on each edge. Phase four typically takes five to ten minutes.

The total: forty to fifty minutes, depending on how many variables the system is working with and how many ambiguities the equivalence-resolution phase has to address. The forty-five-minute benchmark is the median, not a hard ceiling.

---

## The Forty-Five-Minute Benchmark

I want to be specific about what the forty-five-minute benchmark is and is not. It is not the time to a fully validated, production-ready causal model. That would take longer — Chapter 17 takes up the data-driven refinement step, Chapter 18 the parameterization, Chapter 19 the operationalization. The forty-five minutes produces a *first-pass* DAG: a structurally complete causal diagram with directed edges where expert input has resolved direction, undirected edges where it has not, and confidence annotations throughout. It is the input to the rest of the pipeline, not the output.

The benchmark matters for two reasons.

The first is that it fits in a single expert session. The expert does not have to come back. The interview happens once, at the convenience of one person, with no need to schedule recurring meetings or coordinate with other experts. This unlocks the elicitation step in organizations where expert time is the binding constraint, which is most of them.

The second is that it produces an output the expert can verify in real time. The forty-five-minute interview ends with a diagram. The expert sees the diagram, recognizes it as her own thinking, and either approves it or points out specific things she would change. The verification happens before the expert leaves the session. Compared to the alternative — a multi-week elicitation in which the diagram is constructed offline and shown to the expert weeks later — the in-session verification dramatically reduces the rate at which the diagram fails to match the expert's actual model.

The benchmark depends, of course, on the protocol being well-designed. A poorly designed forty-five-minute interview produces a poorly specified diagram. The architecture I am describing is not a substitute for thoughtful protocol design; it is the means by which good protocol design becomes scalable. The intellectual work is in the protocol. The system executes what the protocol specifies.

I should also acknowledge a second-order consideration. Some elicitations should not aim for forty-five minutes. Highly novel domains, where the expert herself is uncertain about the basic structure, may require longer engagements with iterative reflection. Multi-expert elicitations, where the structure is contested, will require additional time for the system to manage disagreements. The forty-five minutes is the case for which the protocol is optimized; it is the modal case in corporate strategic decision-making, but it is not every case. A mature deployment of the architecture will distinguish between the modal case and the harder cases and allocate time accordingly.

---

## Multi-Agent Design

The single-prompt architecture — one LLM, one long system prompt, one conversation — works for simple elicitations. For the kind of structured, four-phase, bias-aware interview I have described, it is insufficient. The required logic is too varied and too specific for a single prompt to handle reliably. The architecture has to be multi-agent, with specialized roles dividing the work.

I will describe four roles. The roles can be implemented as separate LLM agents, or as a single LLM with explicit role-switching, or as a hybrid; the architectural pattern is what matters, not the implementation.

**The Interviewer agent.** This is the role the expert directly interacts with. It runs the four-phase protocol, asks the questions, listens to the responses, and updates the working state as the interview proceeds. Its primary job is conversational fluency and protocol adherence. It does not do structural reasoning or consistency-checking; it focuses on getting the expert through the interview.

**The Consistency agent.** This agent monitors the working state of the diagram in real time. After each commitment from the expert — each new variable, each new edge, each orientation — it checks whether the commitment is consistent with prior commitments. *Did the expert just orient an edge that creates a cycle with edges already in the diagram? Did the expert just claim that two variables are independent in a way that contradicts a previously implied dependence?* When the consistency agent flags a problem, it raises it to the Interviewer agent, which surfaces the issue to the expert with a probe. The expert then resolves the inconsistency, either by revising the new commitment or by revising a prior one.

**The Equivalence agent.** This agent runs in the background, computing the implications of the current diagram for what can and cannot be identified from observational data. As the diagram develops, it identifies which edges have been resolved and which remain in the equivalence class. It flags the ambiguities that the equivalence-resolution phase will need to address, ranks them by how much they affect the downstream causal effects of interest, and supplies the Interviewer agent with the targeted prompts that will resolve the highest-priority ones.

**The Bias-Watch agent.** This agent watches for the cognitive failures Chapter 15 cataloged. When the expert proposes an edge that involves a variable that might be a collider, the bias-watch agent flags the proposal to the Interviewer agent and supplies the appropriate diagnostic question. When the expert simplifies a known feedback loop, the bias-watch agent surfaces the cycle. When the expert imports a mechanism from another domain, the bias-watch agent prompts for examination of the cross-domain assumptions. The bias-watch agent does not directly intervene with the expert; it intervenes through the Interviewer, who incorporates the bias prompts into the conversation in ways that do not feel adversarial.

The four agents share a common working state — the developing diagram and its annotations — and communicate through structured messages rather than through the natural-language interview. This separation is important because it makes the system's reasoning about the diagram auditable. After the interview, the team can examine which agents flagged what, which prompts were inserted by the bias-watch agent, which equivalences the equivalence agent identified. The audit trail supports the kind of accountability Chapter 19's executive report will require.

A more elaborate architecture would add a fifth agent — a *Calibration* agent that handles the confidence-elicitation phase and applies miscalibration corrections — and a sixth that interfaces with available data to provide background statistics during the interview. These extensions are useful in production deployments. The four-agent core is sufficient for the first-pass DAG.

---

## What the System Can and Cannot Do

A production deployment of this architecture is, as of this writing, a partial reality. The components exist. The conversational fluency required of the Interviewer agent is well within the capabilities of current LLMs. The structural reasoning required of the Consistency and Equivalence agents has been demonstrated in research prototypes. The bias-watch logic can be encoded in prompts that produce reliable behavior. What does not yet exist, as far as I know, is a unified product that integrates these components for general causal elicitation. The architectural pattern I have described is, at the time of writing, a prescription rather than an off-the-shelf system.

This matters for what the chapter is honestly claiming. I am not saying that an organization can buy this system today and run forty-five-minute causal elicitations next week. I am saying that the architecture is feasible — every component has been demonstrated separately — and that the integrated system is a matter of engineering rather than research. Organizations that want to build this system can do so; organizations that want to wait for someone to build it for them will, in my estimate, have access to such products within two to three years from publication of this book.

I also want to be honest about what the system does not solve. It does not solve the problem of *the expert who is wrong*. If the expert's model of the domain is fundamentally mistaken, the interview will faithfully capture the mistaken model. The bias-watch agent reduces the rate at which the expert makes the specific cognitive errors of Chapter 15, but it does not protect against the expert simply having the wrong substantive theory. The protection against substantive error comes from the data-driven refinement step in Chapter 17, where the diagram is tested against observational data and revised where the data and the diagram conflict. The interview is the first pass; the data is the second.

The system also does not solve the problem of *the contested model*. When two experts disagree about the structure, the interview produces two diagrams, not one. Reconciling them requires a mechanism — possibly another structured interview that surfaces the disagreement, possibly a data-driven adjudication where the data disambiguates, possibly a human strategist who decides. The architecture supports the reconciliation but does not perform it.

Finally, the system does not solve the problem of *the bored or hostile expert*. Some experts will engage with the interview and produce excellent output; some will not. The protocol is calibrated for cooperative experts who understand why they are being interviewed and are willing to invest forty-five minutes of focused attention. For hostile or distracted experts, no protocol will work well, and human facilitation may be necessary even when the system is available.

These limits are not, in my view, reasons to delay deployment. They are reasons to be careful about what the system is being asked to do. The forty-five-minute interview is the first-pass elicitation, performed on a cooperative expert, in a domain the expert genuinely understands. Within those boundaries, the system can do work that historically required weeks of trained facilitator time. Outside those boundaries, the system's role is more circumscribed, and the architecture has to be augmented with human judgment in the places where the system is weakest.

---

## Looking Forward

Chapter 17 takes up the next step in the pipeline. The forty-five-minute interview produces a CPDAG with confidence annotations, undirected edges where the expert could not commit on direction, and structural commitments that may or may not match what the data implies. Chapter 17 describes how the data-driven discovery algorithms — PC, GES, NOTEARS, FCI — refine the expert-provided structure. The handoff is bidirectional: the data refines the diagram, and discrepancies between data and diagram are routed back to the expert for resolution. The combined diagram, validated against data and committed to by the expert, is the input to the parameterization and operationalization steps of Chapters 18 through 20.

---

**What would change my mind:** A demonstration that an unstructured LLM conversation, without the protocol described here, produces causal diagrams of comparable quality to the structured interview. There is some recent work on LLM-based causal reasoning that does not use this kind of structured architecture, and the results are mixed. If the unstructured approach matched the structured one in head-to-head comparison, my preference for structure would be weakened. The current evidence, which I have been following with interest, suggests structure helps substantially, but the gap is not as large as I would expect, and the question is genuinely open.

**Still puzzling:** How to handle the cases where the expert's confidence is well-calibrated for some parts of the diagram and poorly calibrated for others. The kind/wicked distinction from Chapter 15 implies that we should differentiate, but the operationalization within a single interview is hard. The expert does not announce *I know this part well and that part badly*; she speaks with similar confidence about both, and the calibration questions catch the average miscalibration but may miss the variance. I do not yet have a clean way to detect within-expert calibration variance during the interview, and I suspect the resolution will require either longer interviews with explicit calibration sub-rounds or post-interview review against data that surfaces the high-variance regions.

---

**Tags:** LLM-guided causal elicitation, Nina framework brand strategy interview, forty-five-minute first-pass DAG, multi-agent architecture interviewer consistency equivalence bias-watch, CPDAG handoff to data-driven discovery
