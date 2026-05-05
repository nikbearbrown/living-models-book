# Chapter 16 — The Machine That Interviews the Expert

*LLM-guided causal elicitation, the Nina-framework architectural parallel, the forty-five-minute first-pass DAG, and the multi-agent design that makes consistency and equivalence-checking tractable.*

**Nik Bear Brown**

---

## Learning Objectives

By the end of this chapter, you should be able to:

1. **Explain** why a structured expert interview is a piece of software that can be written — and what the Nina framework in brand strategy demonstrates about this claim.
2. **Describe** the four phases of causal elicitation — variable identification, edge elicitation, equivalence resolution, and confidence calibration — and specify what each phase produces and requires.
3. **Apply** the temporal-precedence substitution to orient edges during elicitation, and identify the two post-edge probes that surface the most common cognitive failures.
4. **Identify** the four roles in the multi-agent elicitation architecture — Interviewer, Consistency, Equivalence, and Bias-Watch — and explain what each does and why the work cannot be consolidated into a single agent.
5. **Characterize** what the forty-five-minute benchmark represents, what it does not represent, and under what conditions it extends beyond that time.
6. **Specify** the limits of what the LLM-guided elicitation system can and cannot solve — and what additional mechanisms are needed to address each limit.

**Prerequisites:** Chapter 14 (the Knowledge Engineering with Bayesian Networks process; the IDEA and Delphi protocols; why the traditional elicitation engagement takes weeks to months). Chapter 15 (cognitive biases in causal elicitation: availability, representativeness, anchoring, the collider blindspot, the feedback-loop simplification). Chapter 7 (the CPDAG as a formal object — what directed and undirected edges represent; Markov equivalence). This chapter is the engineering specification for a system that addresses the bottleneck Chapters 14 and 15 diagnosed.

**Why this chapter matters:** The bottleneck in deploying Living Models at scale is not the algorithms. The algorithms of Chapter 17 are fast and automated. The bottleneck is the elicitation — the extraction of expert knowledge into the causal structure that the algorithms need as a starting point. Chapter 14 showed that traditional elicitation takes weeks to months per engagement. Chapter 15 showed that even expert-led elicitation is susceptible to systematic cognitive errors. This chapter shows how to compress the elicitation to forty-five minutes and systematize the bias mitigation. Until this bottleneck is solved, the Living Model architecture remains a research project. This chapter is the specification for the solution.

---

## The Founder's Interview

A founder sits down for a Nina interview at her brand strategist's recommendation. She has been told the interview will take about an hour. She has no idea what to expect.

The interview opens on a screen with a chat-like interface. The first question is direct: *Tell me, in two or three sentences, what your company does.* She types an answer. The system reads it, reflects part of it back to confirm understanding, and asks the next question: *Who is the person who buys what you sell, and what is going on in their life when they do?* She types. The system probes, gently. It does not move on. It asks for specifics where she has been general. It asks for an example where she has spoken in abstractions.

Forty minutes later, she has answered around eighty questions, ranging across her customers, her competitors, the moment of purchase, the moment of regret, the language her customers use when they recommend her, the language they use when they complain. The system has not let her skip questions. It has not let her speak in marketing jargon. When she said *we are an authentic brand*, it asked her what authentic meant in her specific context, and refused to accept the answer until she had said something substantive.

At the end of the interview, the system produces a creative brief. The brief contains a brand archetype, a set of positioning statements, a tone-of-voice guide, and three or four suggested directions for the next phase of work. The founder reads it. She recognizes it — the brief contains things she has known for years and never articulated, things her marketing agency has been chasing without quite landing, and a couple of things that surprise her. Connections the interview surfaced that she had not consciously seen.

A skilled human brand strategist could have done the same work, with similar quality, in three to five hours of meetings spread across two weeks. The Nina framework does it in forty-five minutes, with no human strategist required for the elicitation step.

This chapter is about how to do for causal elicitation what Nina has done for brand strategy. The architectural pattern transfers. The vocabulary is different — variables instead of brand attributes, edges instead of positioning statements, conditional independence instead of voice — but the structural moves are the same. The expert's tacit knowledge is the input. A disciplined sequence of questions is the apparatus. A structured causal model is the output.

---

## Concept 1: Interview as Software

### The Core Insight

The deepest insight from the Nina framework is that a structured interview is a piece of software that can be written. This sounds banal until you consider how causal elicitation has historically worked.

A trained facilitator — typically a person with years of expertise in both probability theory and the substantive domain — sits with one or more experts. The facilitator asks structured questions drawn from a protocol like IDEA or Delphi. The facilitator interprets the answers, translates them into structural commitments on the diagram, and steers the conversation forward. The whole process takes weeks to months because the facilitator is a scarce resource and the experts have other obligations.

The Nina framework's contribution is to recognize that most of what the facilitator does can be specified. The questions to ask, the conditions for moving on, the responses that should trigger probing, the cases where the interview should slow down — all of these can be encoded as a system prompt or a structured workflow. The cases where the encoding fails, where genuine human judgment is required, are far rarer than the facilitator's day-to-day work suggests. Most of the interview is following a protocol that someone has written down. If the protocol can be written down, it can be executed by software.

<!-- → INFOGRAPHIC: two-column comparison — left column: "Traditional elicitation," right column: "LLM-guided elicitation" — rows: Time required (weeks to months / 45 minutes), Expert availability (multiple scheduled sessions / single focused session), Facilitator cost (trained specialist per engagement / protocol written once, executed many times), Consistency (varies by facilitator / consistent protocol every run), Verification (diagram shown weeks later / expert sees diagram in session) — student should see the economic difference and understand that the bottleneck is the protocol, not the execution -->

This is not a claim that LLMs replace expertise. The expertise resides in the protocol — in the structure of questions and the response logic — and the protocol still has to be written by someone who understands causal inference and the domain. What changes is that the expertise is encoded once and executed many times, rather than being applied person-to-person. The economics shift dramatically. An interview that costs forty hours of expert facilitator time becomes an interview that costs forty-five minutes of expert time and almost nothing of facilitator time. The adoption barrier described in Chapter 14 — that trained facilitators are scarce — dissolves once the facilitation is encoded.

### What Makes an Interview Encodable

Not every expert interview can be automated. The features that make causal elicitation encodable are worth naming explicitly, because they explain why the approach works here and would not work everywhere.

First, the output is formally specified. A causal elicitation interview is trying to produce a CPDAG with confidence annotations. The target is a mathematical object with known properties. This allows the interview to evaluate whether each response moves the output toward or away from completion, and to continue asking until the output is sufficiently specified. Interviews whose output is inherently subjective or whose success criteria are undefined cannot be automated on the same basis.

Second, the failure modes are cataloged. Chapter 15 identified the specific cognitive errors that produce incorrect structural commitments during causal elicitation. Because the failure modes are known, the interview can systematically probe for them. This is unlike interviews where the interviewer's job is to detect failure modes that vary unpredictably by domain and individual.

Third, the protocol's adequacy can be tested. A causal diagram produced by the protocol can be compared against the diagram that a trained human facilitator would have produced, against the diagram that observational data implies, and against the diagram's performance at predicting intervention outcomes in held-out cases. All three provide feedback that allows the protocol to be refined. Interviews whose outputs cannot be externally validated cannot be improved on the same basis.

### Worked Example: The Failure That Motivated the Architecture

Chapter 14's knowledge engineering case study — the six-week engagement that produced the first-pass diagram — illustrates the economics the architecture is designed to change. The engagement required a trained facilitator, multiple expert sessions, a synthesis step in which the facilitator translated interview transcripts into structural commitments, a review session in which the draft diagram was shown to the experts, and a revision cycle. Total elapsed time: six weeks. Total expert hours: approximately twelve. Total facilitator hours: approximately thirty.

With the LLM-guided architecture, the twelve hours of expert time compress to forty-five minutes — one expert, one session, one diagram. The facilitator hours compress to near zero for the execution step, with residual hours for protocol maintenance and review of flagged anomalies. The elicitation bottleneck, which consumed six weeks and thirty facilitator hours, becomes a one-session activity that any member of the analytics team can schedule.

<!-- → CHART: side-by-side bar chart — two grouped bars for "Traditional elicitation" and "LLM-guided elicitation" — bars for: expert hours (12 hours / 0.75 hours), facilitator hours (30 hours / ~2 hours for protocol maintenance), elapsed calendar time (6 weeks / 1 day), sessions required (4-6 sessions / 1 session) — student should see the order-of-magnitude compression and understand which component (facilitator hours) drives most of the time savings -->

The diagram produced is a first-pass CPDAG, not a finished model. The data-driven refinement of Chapter 17 and the parameterization of Chapter 18 still follow. But the first pass — historically the bottleneck — is no longer the bottleneck. The remaining steps are faster and more automatable; what was slow was the beginning, and the architecture addresses the beginning.

**The lesson:** The bottleneck in the Living Model pipeline is not the algorithms — those are automated. The bottleneck is structured knowledge extraction from a human expert. Once that bottleneck is resolved, the rest of the pipeline runs quickly.

---

## Concept 2: The Four-Phase Protocol

### Phase Architecture

The elicitation protocol has four phases. Each phase produces a specific output that the next phase requires. The phases are not interchangeable and must happen in order. Skipping a phase does not save time; it pushes the work into later stages where it is harder to do well.

<!-- → INFOGRAPHIC: horizontal pipeline showing four phases in sequence — Phase 1: Variable Identification (output: variable list with operationalizations, ~10-15 min); Phase 2: Edge Elicitation (output: draft diagram with directed and undirected edges, ~15-20 min); Phase 3: Equivalence Resolution (output: CPDAG with high-impact ambiguities resolved, ~10 min); Phase 4: Confidence Calibration (output: CPDAG with confidence ranges and magnitude estimates, ~5-10 min) — arrows between phases showing what each phase receives as input; total time annotation: 40-50 minutes — student should use this as a reference map for the protocol -->

### Phase One — Variable Identification

The interview opens by asking the expert to enumerate the variables that matter for the question at hand. The questioning is structured around three filters.

*What are you trying to decide or affect?* This fixes the outcome variable. The expert identifies what she wants to influence or understand. The interview probes until the outcome is specific, measurable, and scoped to a time horizon.

*What levers do you have available?* This fixes the candidate treatment variables. The expert identifies what she can actually control. The interview probes for practical constraints — not all levers the expert can imagine, but levers the organization can realistically pull in the relevant decision cycle.

*What other factors do you believe shape this outcome?* This fixes the candidate confounders, mediators, and effect modifiers. The expert identifies background conditions, market dynamics, competitive actions, and organizational factors that she believes influence the outcome even when the treatments are held fixed.

After each candidate variable is named, the interview enforces measurability discipline. *How would you measure this? What would a one-unit change in it look like? What data already exists on it?* Variables that cannot be answered cleanly are either reframed in measurable terms or explicitly tagged as latent — present in the model but not directly observed. A latent variable that the expert cannot measure is not discarded; it is carried forward with a flag that will inform how the data-driven step in Chapter 17 handles it.

The interview also enforces linguistic discipline. When the expert uses terms that admit multiple interpretations — *engagement, quality, performance, trust* — the interview probes for the specific operationalization. Terms that slide between meanings across different parts of the diagram will produce structural incoherence. The expert is forced to commit to a definition at variable-identification time.

**Output:** A list of variables, each with a precise operationalization, a measurement strategy, and an indicator of whether data exists. Typically eight to fifteen variables for a corporate strategic question, fewer for a narrow operational question.

**Typical duration:** Ten to fifteen minutes.

### Phase Two — Edge Elicitation

With the variables established, the interview moves to structural questions: which variables cause which? The guiding principle is the temporal-precedence substitution from Chapter 14. Instead of asking *which causes which* — a question that triggers the philosophical resistance Chapter 14 documented — the interview asks *which comes first in time when this relationship plays out.*

For most variable pairs, the temporal answer is unambiguous. Sales training precedes this quarter's sales performance. Customer acquisition precedes customer lifetime value. Market entry timing precedes competitive response. Temporal precedence is a reliable proxy for causal direction in most organizational contexts, and experts find it much easier to answer than direct causal questions.

For pairs where the temporal answer is itself ambiguous — because both variables update simultaneously, or because the relationship runs in both directions at different time scales — the interview either expands the model to include lagged versions of the variables (this quarter's customer satisfaction causing next quarter's retention, rather than treating both as contemporaneous) or explicitly marks the pair as bidirectional and defers it to the equivalence-resolution stage.

After each proposed edge, the interview applies two probes. These probes are the primary mechanism by which the Bias-Watch agent (described in Concept 3) flags the cognitive failures most likely to have produced an incorrect commitment.

**The collider probe.** *If we knew the value of [child variable] and we knew the value of [parent A], would knowing [parent B]'s value change our belief about the relationship between [parent A] and [child]?* This is a behavioral probe of whether the expert is implicitly conditioning on a variable that should be modeled as a collider. If the expert says yes, the interview surfaces the third variable, examines whether it is a collider, and adjusts the diagram accordingly. If the expert says no, the edge is confirmed and the interview moves on.

**The feedback-loop probe.** *When [child variable] changes, does it eventually feed back to change [any of its parent variables] over the relevant time horizon?* If the answer is yes, the interview asks for the time scale. If the feedback operates within the decision cycle, the cycle is explicitly represented as time-indexed nodes. If it operates outside the decision cycle, it is flagged but not represented in the diagram.

<!-- → INFOGRAPHIC: phase two interview excerpt — shows a three-turn dialogue between the system and an expert: system asks "Which variable typically changes first in time — customer satisfaction or repeat purchase rate?"; expert answers "satisfaction comes first, it drives the repeat purchases"; system asks collider probe "If we knew the repeat purchase rate and satisfaction, would knowing a third variable — say, product quality — change how we interpret the satisfaction-to-purchase relationship?"; expert responds "yes, product quality affects both independently"; system responds "Good — let me add product quality as a separate variable that influences both. I'll mark those two edges. Now, when repeat purchase rate changes, does it eventually feed back to affect customer satisfaction over a 12-month horizon?" — student should see the specific conversational moves and understand how the temporal probe and the two post-edge probes work in sequence -->

**Output:** A draft diagram with directed edges where temporal precedence was clean, undirected edges where it was not, and explicit annotations on feedback cycles and time scales.

**Typical duration:** Fifteen to twenty minutes. This is the longest phase because most variables require at least one probe after the initial edge proposal.

### Phase Three — Equivalence Resolution

The draft diagram from phase two will, almost always, contain undirected edges — variable pairs whose direction the expert could not commit on. Phase three resolves the most consequential of these.

The interview begins by computing — via the Equivalence agent — which undirected edges, if oriented one way versus the other, would most affect the downstream causal effect estimates relevant to the original question. This computation is mechanical: given the diagram and the outcome variable of interest, the effect on the identifiable causal effects of each possible edge orientation can be calculated. The interview prioritizes the ambiguities that matter most for what the model will ultimately be used to decide.

For each high-priority undirected edge, the interview applies a structural intervention probe. *Suppose we held [variable X] constant — by some external means, like a policy or a rule — while letting [variable Y] vary as it would naturally. Would changes in [variable Y] propagate to [variable Z]?* This probe tests for interventional reasoning rather than associational reasoning. Most experts can answer this question more reliably than they can answer direct causal questions, because the counterfactual is specific and concrete. If the answer is yes, the edge orients one way; if no, the other.

For the small number of edges that the expert genuinely cannot orient even with the structural prompt, the interview leaves them undirected and flags them for resolution by data in Chapter 17. These are the edges in the Markov equivalence class that observational data can sometimes distinguish. The system notes which algorithm-based orientation procedures are most likely to resolve them, given the data sources available.

**Output:** A Completed Partially Directed Acyclic Graph (CPDAG) in which directed edges reflect expert commitment and undirected edges are flagged for data-driven resolution. The CPDAG is the formal object Chapter 7 introduced; it is now grounded in specific expert commitments.

**Typical duration:** Ten minutes for a typical strategic question with three to five undirected edges after phase two.

### Phase Four — Confidence Calibration

The fourth phase asks the expert to quantify her confidence in each edge and provide rough magnitude estimates for each relationship.

The interview uses probability ranges rather than point estimates. *With how much certainty are you committing to this edge — would you say you're highly confident, moderately confident, or uncertain? What's the strongest you would say this effect could be? The weakest?* Point estimates invite false precision; ranges communicate the uncertainty the model needs to carry forward into the parameterization step.

The phase also includes a small set of calibration questions on quantities the expert should know from domain experience: *In your experience, what fraction of customers in this segment who experience the trigger condition eventually convert? What's the typical lag between the lever you identified and the outcome?* These questions are not needed for the structural model. They detect systematic miscalibration — experts who consistently compress their uncertainty ranges, or who have systematically inflated or deflated intuitions about base rates. Where the calibration questions reveal miscalibration, the system applies a correction to the elicited confidences before passing them downstream.

**Output:** The CPDAG from phase three, annotated with confidence ranges and rough magnitude estimates on each edge, plus a calibration flag where miscalibration was detected.

**Typical duration:** Five to ten minutes.

### Worked Example: Building the Customer Retention Diagram

A customer success director at a B2B software company is guiding a causal elicitation for a question about the drivers of net revenue retention (NRR). Here is what the four phases produce.

*Phase one output:* Six variables. NRR (outcome, measured quarterly as percentage of prior-year revenue retained plus expansions). Product adoption depth (treatment lever, measured by number of active features used per account). Customer health score (treatment lever, measured by a composite of usage and support interactions). Executive sponsorship (background factor, estimated from account records — binary). Competitive intensity (background factor, estimated from win-loss data). Onboarding quality (background factor, estimated from post-implementation survey score). All six are measurable or estimable from available data. No latent variables required.

*Phase two output:* Five directed edges with clean temporal precedence: onboarding quality → product adoption depth; executive sponsorship → product adoption depth; product adoption depth → customer health score; customer health score → NRR; competitive intensity → NRR. One undirected edge: executive sponsorship ↔ NRR (the expert could not determine whether executive sponsorship directly affects NRR beyond its effect through adoption and health score, or whether high NRR accounts selectively maintain executive sponsorship). One feedback cycle flagged: NRR → onboarding quality investment (high-NRR accounts receive more investment in future onboarding cycles), with the time scale noted as a twelve-month lag — outside the quarterly decision cycle and therefore not modeled in the current diagram.

*Phase three output:* The equivalence agent identifies the executive sponsorship ↔ NRR edge as high-priority — its orientation significantly affects whether executive sponsorship can be used as a lever independent of product adoption. The structural probe resolves it: if executive sponsorship were held constant by policy, NRR still varies with product adoption and health score. The unresolved variation in NRR is not driven by executive sponsorship. Edge oriented: executive sponsorship → NRR is removed; executive sponsorship → product adoption depth (already committed) is the operative path. The CPDAG now has six directed edges, zero undirected edges.

*Phase four output:* The expert rates all six edges as moderately to highly confident. Magnitude estimates: executive sponsorship lifts product adoption depth by 20–40 percent (wide range, the expert acknowledges uncertainty); product adoption depth → customer health score is a strong relationship, 60–80 percent of the NRR variance in her experience. Calibration check: the expert's estimate of the fraction of accounts that achieve high health scores is consistent with the company's actual data. No miscalibration flag triggered.

<!-- → INFOGRAPHIC: the completed NRR causal diagram — six nodes: Onboarding quality, Executive sponsorship, Product adoption depth, Customer health score, Competitive intensity, NRR — directed edges as described; feedback cycle shown as a dashed arrow from NRR back to Onboarding quality labeled "12-month lag — outside decision cycle, not modeled"; each edge annotated with the confidence level (moderate/high) and magnitude range from phase four — student should see how the four-phase output maps to a single coherent annotated CPDAG -->

**Total elapsed time:** Thirty-eight minutes.

**The lesson:** The four phases produce a fully specified causal diagram from a single expert session. The output — six directed edges, confidence ranges, magnitude estimates — is the input to the Chapter 17 data-driven refinement. The expert leaves the session having seen her own model articulated in a form she can review and confirm.

---

## Concept 3: The Multi-Agent Architecture

### Why Single-Agent Is Insufficient

The four-phase protocol I have described requires the system to do several things simultaneously that are in tension with each other. The Interviewer has to maintain conversational fluency and move the expert forward — it has to keep talking in a way that feels natural and productive. At the same time, the system has to maintain a formal representation of the developing diagram, check it for structural consistency, monitor the equivalence class, and detect the cognitive failure patterns of Chapter 15. These tasks require different kinds of attention and operate on different representations. A single-prompt LLM handling all of them simultaneously will sacrifice quality on some dimensions to maintain performance on others. The architecture has to be multi-agent, with specialized roles dividing the work.

The four agents share a common working state — the developing diagram and its annotations — and communicate through structured messages rather than through the natural-language interview. This separation makes the system's reasoning about the diagram auditable: after the interview, the team can examine which agents flagged what, which prompts were inserted by the Bias-Watch agent, which equivalences the Equivalence agent identified. The audit trail is the foundation for the accountability Chapter 19's executive report requires.

### The Four Roles

**The Interviewer agent** is the role the expert directly interacts with. It runs the four-phase protocol, asks the questions, listens to the responses, and updates the working state as the interview proceeds. Its primary job is conversational fluency and protocol adherence. It does not do structural reasoning or consistency-checking; it focuses on maintaining the expert's engagement and getting her through the protocol without frustration or confusion. When the other agents flag issues, the Interviewer receives structured signals and incorporates them into the natural-language conversation without exposing the multi-agent machinery to the expert.

**The Consistency agent** monitors the working state of the diagram in real time. After each commitment — each new variable, each new edge, each orientation — it checks whether the commitment is consistent with prior commitments. Did the expert just orient an edge that creates a cycle with edges already in the diagram? Did the expert just claim that two variables are independent in a way that contradicts a previously implied dependence? Did a new variable, when added, create a path that the expert has previously ruled out? When the Consistency agent flags a problem, it raises the issue to the Interviewer, which surfaces it to the expert with a targeted probe. The expert resolves the inconsistency, either by revising the new commitment or by revising the prior one.

**The Equivalence agent** runs in the background, computing the implications of the current diagram for what can and cannot be identified from observational data. As the diagram develops, it identifies which edges have been resolved into a specific direction and which remain ambiguous within the Markov equivalence class. It ranks the remaining ambiguities by how much they affect the downstream causal effects of interest — the effects the Living Model will be asked to estimate — and supplies the Interviewer agent with targeted prompts for the equivalence-resolution phase. The Equivalence agent does not interact with the expert directly; its output is a ranked list of structural probes for the Interviewer to pose during phase three.

**The Bias-Watch agent** watches for the cognitive failures Chapter 15 cataloged. When the expert proposes an edge that involves a variable that might be playing a collider role — a variable that might be jointly caused by two other variables the expert has already committed to — the Bias-Watch agent flags the proposal and supplies the collider probe to the Interviewer. When the expert describes a mechanism that simplifies a known feedback loop, the Bias-Watch agent surfaces the cycle. When the expert imports a mechanism from another domain, the Bias-Watch agent prompts for examination of whether the cross-domain import is warranted. The Bias-Watch agent does not directly intervene with the expert; all of its interventions go through the Interviewer, who incorporates them in ways that do not feel adversarial or accusatory.

<!-- → INFOGRAPHIC: multi-agent architecture diagram — four agent boxes arranged around a central "Working Diagram State" — Interviewer (top, labeled "Conversational interface; runs four-phase protocol; incorporates signals from other agents"); Consistency (left, labeled "Monitors for structural inconsistencies after each expert commitment; flags to Interviewer"); Equivalence (right, labeled "Computes Markov equivalence class; ranks ambiguities by impact on target effects; supplies probes for Phase 3"); Bias-Watch (bottom, labeled "Monitors for Chapter 15 cognitive failures; supplies collider probes and feedback-loop prompts to Interviewer") — arrows between each agent and the central Working Diagram State showing read/write access; arrows from Consistency, Equivalence, Bias-Watch to Interviewer showing structured signal flow — student should see the separation of concerns and understand why the work cannot be consolidated -->

### What the Multi-Agent Design Makes Possible

The multi-agent design makes two things possible that are not possible with a single-agent architecture.

The first is consistent quality across runs. Because the structural reasoning is separated from the conversational layer, the structural reasoning can be tested and validated independently. The Equivalence agent's ranking algorithm can be validated against known causal models. The Consistency agent's checks can be unit-tested against known inconsistency patterns. The Bias-Watch agent's trigger conditions can be calibrated against the Chapter 15 taxonomy. Each can be improved independently of the others. A single-agent system whose structural reasoning is entangled with its conversational behavior cannot be improved on the same component-by-component basis.

The second is auditability. The multi-agent design produces a structured log of every agent action: every Consistency flag, every Equivalence ranking, every Bias-Watch trigger, every Interviewer probe that resulted. The log answers questions that the expert cannot answer after the fact: *Why was this edge oriented this way? What structural probe established it? Was there a consistency conflict that required revision?* The log is the evidence base for the Chapter 19 executive report's assumptions section, and it is the basis for the domain expert's review of the elicitation output before the diagram enters the Chapter 17 refinement pipeline.

### Worked Example: A Bias-Watch Intervention

During the phase-two elicitation of the customer retention diagram, the expert proposes the edge: competitive intensity → customer health score. She reasons that when competition increases, customers pay closer attention to their existing tools and therefore their health scores improve.

The Bias-Watch agent detects a potential collider pattern. The expert has already committed to: product adoption depth → customer health score, and competitive intensity → NRR. If the expert is now proposing competitive intensity → customer health score, the Bias-Watch agent flags that customer health score is becoming a variable with two proposed parents (product adoption depth and competitive intensity), while also being a parent of NRR. This creates a potential collider structure — customer health score as a collider on the path between competitive intensity and NRR through product adoption.

The Bias-Watch agent supplies the collider probe to the Interviewer: *If we knew a customer's health score, and we knew the competitive intensity in that customer's market, would knowing the customer's product adoption depth still give us new information about competitive intensity?*

The expert pauses. *Actually, I think competitive intensity affects NRR more directly — through pricing pressure and contract renegotiation — than through health score. Health score is mostly about how they're using the product, not about the competitive landscape.* She revises: competitive intensity → NRR is retained; competitive intensity → customer health score is removed.

The Bias-Watch intervention has prevented the introduction of a spurious collider path that would have distorted the estimated causal effect of product adoption depth on NRR in the downstream analysis.

<!-- → INFOGRAPHIC: two-panel before/after causal graph for the Bias-Watch intervention — left panel "Before Bias-Watch probe": customer health score has two incoming arrows (product adoption depth → customer health score; competitive intensity → customer health score) and one outgoing (customer health score → NRR); competitive intensity also has arrow to NRR; collider structure highlighted in red with label "potential collider: conditioning on health score would induce spurious correlation between adoption depth and competitive intensity"; right panel "After expert revision": competitive intensity → customer health score arrow removed; remaining structure is clean — student should see exactly what the Bias-Watch agent detected and what the revision fixed -->

**The lesson:** The Bias-Watch agent does not tell the expert she is wrong. It surfaces a structural implication of her commitments and asks her to reason about it. The expert corrects her own model. The agent's role is to make the implicit implication explicit, not to override expert judgment.

---

## Integration: The Forty-Five-Minute Benchmark and Its Limits

### What the Benchmark Represents

The forty-five-minute benchmark is the median elapsed time for a four-phase elicitation on a corporate strategic question with eight to fifteen variables, one cooperative expert, and a domain the expert genuinely understands. It is not a hard ceiling; some elicitations run thirty minutes, some run an hour and a half. The benchmark matters for two specific reasons.

First, it fits in a single expert session. The expert does not have to come back. The interview happens once, at the convenience of one person, with no coordination overhead. This unlocks the elicitation step in organizations where expert time is the binding constraint.

Second, it produces an output the expert can verify in real time. The session ends with a diagram on screen. The expert sees her own model in structured form, recognizes it, and either approves it or flags specific changes. The verification happens before she leaves. This dramatically reduces the rate at which the diagram fails to match the expert's actual model, compared to the alternative of showing a diagram weeks later in a review session.

### When the Benchmark Extends

Three conditions push the elicitation beyond forty-five minutes.

*Highly novel domains.* When the expert herself is uncertain about the basic causal structure — when she is working at the edge of her own knowledge — the interview slows down because the probes trigger genuine reflection rather than recall. Novel domains may require an hour and a half to two hours, and may benefit from multiple shorter sessions rather than one long one.

*Multi-expert elicitations.* When two or more experts need to contribute to the same diagram, the protocol runs separate sessions for each expert and then brings the outputs together for a reconciliation phase. The reconciliation, which surfaces structural disagreements and resolves them through targeted probes or data adjudication, adds time proportional to the degree of disagreement. Chapter 14's multi-expert cases show that structured disagreement is often more productive than premature consensus; the architecture supports disagreement rather than suppressing it.

*Feedback-heavy domains.* When the domain is characterized by many short-cycle feedback loops — financial markets, competitive markets, biological systems — the phase-two edge elicitation slows because almost every edge requires a feedback-loop probe and a temporal disambiguation. These domains may require an explicit pre-session conversation to establish the time-indexing conventions before the interview begins.

### What the System Cannot Solve

Three limits are important to name explicitly, because they determine where the LLM-guided system must be augmented with human judgment.

*The system cannot correct a fundamentally wrong expert model.* If the expert's understanding of the domain is incorrect — not just imprecise, but structurally wrong — the interview faithfully captures the incorrect model. The Bias-Watch agent reduces the rate of specific cognitive errors from Chapter 15, but it does not protect against expert error that doesn't take the form of a known bias. The protection against substantive error comes from the Chapter 17 data-driven refinement, where the diagram is tested against observational data and revised where the data and the diagram conflict.

*The system cannot reconcile structurally contested models.* When two experts disagree about the causal structure, the interview produces two diagrams, not one. Reconciling them requires a mechanism — possibly a structured facilitated session that surfaces the disagreement and identifies what evidence would resolve it, possibly a data adjudication where the data disambiguates competing structural hypotheses. The architecture supports this reconciliation but does not perform it automatically.

*The system cannot elicit from an uncooperative expert.* The protocol is calibrated for cooperative experts who understand why they are being interviewed and are willing to invest focused attention. For hostile, distracted, or skeptical experts, no protocol will work well, and human facilitation may be necessary. An expert who answers at low effort — short answers, no elaboration, no engagement with probes — will produce a diagram that reflects her low effort, not her genuine model.

<!-- → TABLE: three-row limits table — columns: limit, what the system cannot do, what provides the protection, when to augment with human judgment — rows: (1) Expert model correctness: cannot detect substantive error not caused by known bias / Chapter 17 data-driven refinement tests diagram against observational data / when the diagram conflicts with data in multiple places; (2) Multi-expert contestation: cannot automatically reconcile structural disagreement / structured reconciliation session or data adjudication / when two expert diagrams share less than 60% of edges; (3) Expert cooperation: cannot elicit from distracted or hostile experts / human facilitation / when expert engagement drops below minimum protocol requirements — student should use this as a checklist for deciding when to rely on the automated system versus supplementing with human judgment -->

**The lesson:** The LLM-guided architecture compresses the first-pass elicitation dramatically and systematizes bias mitigation. It does not replace expert judgment, domain knowledge, or the data-driven validation that follows. It is the correct tool for the modal case — one cooperative expert, a domain she understands, a question that can be specified in advance. Outside this modal case, the architecture must be augmented.

---

## Chapter Summary

This chapter is the engineering specification for the elicitation bottleneck that Chapters 14 and 15 identified. The bottleneck is the extraction of expert causal knowledge into a structured diagram — historically a weeks-long process requiring a scarce trained facilitator. The specification reduces this to a forty-five-minute first-pass session by encoding the facilitation protocol into software.

The core insight is that a structured interview is a piece of software. The Nina framework in brand strategy demonstrated this at commercial scale. The same architecture applies to causal elicitation, with the vocabulary changed from brand attributes to variables and edges.

The four-phase protocol — variable identification, edge elicitation, equivalence resolution, confidence calibration — produces a CPDAG annotated with confidence ranges in a single session. Each phase has a specific output and a specific duration. The protocol enforces measurability discipline and linguistic precision at variable-identification time, applies temporal precedence as a proxy for causal direction during edge elicitation, resolves high-impact structural ambiguities during equivalence resolution, and calibrates expert confidence before handing off to the downstream pipeline.

The multi-agent architecture separates the conversational layer from the structural reasoning layer. The Interviewer maintains conversational fluency; the Consistency agent monitors structural coherence; the Equivalence agent tracks the Markov equivalence class; the Bias-Watch agent flags the cognitive failures of Chapter 15 and supplies targeted probes. The separation produces consistent quality across runs and an auditable log of every structural decision.

**The one idea that matters most from this chapter:** The bottleneck in Living Model deployment is not the algorithm — it is the elicitation. The forty-five-minute first-pass is the specification for eliminating that bottleneck. Every other part of the pipeline moves faster once the first-pass diagram exists.

**The common mistake to watch for:** Treating the forty-five-minute first-pass as a finished diagram. It is an input to the Chapter 17 refinement, not an output of the Living Model pipeline. An organization that acts on the first-pass diagram without data-driven validation is acting on the expert's prior before any evidence has tested it. The prior is essential; it is not sufficient.

**The Feynman test:** Close the book and explain to a colleague why causal elicitation can be encoded as software, what the four phases of the protocol produce, and what the multi-agent architecture does that a single-agent architecture cannot. Then identify one of the three limits of the system — and describe what human judgment is required to address it. If you can do all four, you have understood this chapter.

---

## Exercises

### Warm-Up

**W1.** *(Objective 1 — Explain why interview is software; Difficulty: Low)*

The chapter claims that most of what a trained facilitator does during a causal elicitation can be encoded as software. A skeptical colleague argues: "Human facilitators adapt in real time to things they didn't anticipate. Software can't do that." Write two to three sentences explaining what is right about this objection and what is right about the chapter's claim — and how they can both be true simultaneously.

**W2.** *(Objective 2 — Describe the four phases; Difficulty: Low)*

For each phase of the protocol, name the output it produces and one thing the interview enforces during that phase that the expert would not naturally enforce herself (without a structured protocol).

**W3.** *(Objective 4 — Identify the four agent roles; Difficulty: Low)*

A structural inconsistency has emerged during phase two: the expert has just proposed an edge that would create a cycle in the diagram. Which agent detects this? Which agent communicates the problem to the expert? What does the communication look like — which agent does it come from and in what form?

---

### Application

**A1.** *(Objective 3 — Apply temporal precedence and post-edge probes; Difficulty: Medium)*

A supply chain analytics team is conducting a causal elicitation for a model of supplier reliability. The expert has proposed the edge: *supplier financial stability → on-time delivery rate.* Apply the two post-edge probes to this proposed edge.

For the collider probe: write the specific question the Interviewer should ask, and describe what happens if the expert answers yes versus no.

For the feedback-loop probe: write the specific question the Interviewer should ask, and describe what happens if the expert identifies a feedback loop with a three-month lag versus a three-year lag.

**A2.** *(Objective 5 — Characterize the benchmark and its extensions; Difficulty: Medium)*

A hospital system is planning a causal elicitation to support a clinical pathway redesign. The question involves eight variables, two clinical experts who are known to have different views on the relative importance of pre-admission care versus in-hospital protocol, and a domain with several short-cycle feedback loops (patient condition affecting treatment decisions affecting patient condition).

Estimate the likely elapsed time for the elicitation and explain which of the three benchmark-extending conditions applies. Describe how the protocol should be adapted for this case — what pre-session work is needed, and what the post-session reconciliation looks like.

**A3.** *(Objective 6 — Specify the limits and their mitigations; Difficulty: Medium)*

A financial services firm runs the LLM-guided elicitation with its chief risk officer. The resulting diagram has twelve directed edges. When the Chapter 17 data-driven refinement runs, it finds that four of the twelve edges conflict with the observational data — in each case, the data implies the opposite orientation from what the expert committed to.

Identify which of the three limits of the system this case illustrates. Describe what the next step should be — specifically, what the team should do with the four conflicting edges and how the protocol handles the conflict between expert commitment and data implication.

**A4.** *(Objectives 2, 3 — Conduct a partial elicitation; Difficulty: Medium)*

Choose a causal question from your own industry or professional experience — ideally one you have seen an organization try to answer with data. Conduct phases one and two of the protocol on yourself, without a facilitator.

For phase one: identify the outcome variable, the treatment levers, and the background factors. Operationalize each one.

For phase two: for each variable pair where you propose an edge, apply the temporal-precedence substitution and both post-edge probes, and record your answers. Note any edges you cannot orient and the reason.

---

### Synthesis

**S1.** *(Objectives 1, 2, 4 — Integrate the architecture with the pipeline; Difficulty: High)*

A management consulting firm wants to embed the LLM-guided elicitation system into its standard strategic advisory engagement. Currently, the firm spends two to three weeks on the elicitation step at the beginning of each engagement. They have trained facilitators but are constrained by their availability across multiple simultaneous engagements.

Design the integrated workflow: (a) which parts of the elicitation the LLM system handles, (b) which parts the human facilitator retains, (c) what the facilitator's role becomes after the first-pass diagram is produced, and (d) what quality control steps are needed before the firm acts on the elicitation output. Specify how the multi-agent audit log is used in the quality control step.

**S2.** *(Objectives 3, 4, 6 — Diagnose and repair an elicitation failure; Difficulty: High)*

The LLM-guided system has produced a first-pass CPDAG for a retail pricing model. The post-session review reveals three problems: (1) a variable called "brand perception" appears in the diagram but has no operationalization in the phase-one output — the expert used the term without defining it; (2) the equivalence agent's audit log shows that the highest-impact undirected edge (between price sensitivity and promotional response) was never posed to the expert during phase three; (3) the Bias-Watch agent flagged a potential collider during phase two, the Interviewer posed the probe, the expert gave an ambiguous answer, and the interviewer moved on without resolving it.

For each problem, identify which agent failed or was bypassed, what protocol step should have prevented the problem, and what corrective action is needed before the diagram can enter the Chapter 17 pipeline.

---

### Challenge

**C1.** *(Objectives 1, 5, 6 — Stress-test the architecture; Difficulty: High)*

The chapter claims that the LLM-guided elicitation compresses weeks of traditional facilitation to forty-five minutes by encoding the facilitation protocol as software. A skeptical researcher argues: "The diagram produced by an LLM-guided forty-five-minute interview will be systematically shallower than the diagram produced by a six-week Knowledge Engineering engagement. Shallower means more missing edges, more missing variables, more unresolved equivalences — which means worse causal estimates downstream. The time saving is not worth the quality loss."

Construct a response to this argument that: (a) identifies what evidence would be needed to evaluate the claim empirically, (b) specifies the dimensions on which the two approaches are most likely to differ and why, (c) proposes conditions under which the forty-five-minute approach is likely to dominate (produce better decision outcomes despite the shallower elicitation), and (d) proposes conditions under which the six-week approach is likely to dominate — and what signals the analytics team should watch for to detect when they are in the latter case.

**C2.** *(Objectives 4, 6 — Extend the architecture; Difficulty: High)*

The chapter describes four agents: Interviewer, Consistency, Equivalence, and Bias-Watch. The chapter also mentions two possible extensions: a Calibration agent and a Data-Integration agent that surfaces background statistics during the interview.

Design a fifth agent — one not described in the chapter — for a specific problem the four-agent system cannot adequately handle. Your design should: (a) name the agent and specify the problem it addresses, (b) describe what input it reads from the working diagram state and what structured signal it sends to the Interviewer, (c) give a concrete example of an elicitation scenario where the agent would trigger and what it would do, and (d) explain why the problem cannot be handled by augmenting one of the existing four agents rather than adding a new one.

---

## Connections Forward

This chapter has described the protocol and architecture for the first-pass elicitation — the expert-facing component of the Living Model build pipeline. The output is a CPDAG with confidence annotations, ready for handoff to the data-driven refinement step.

Chapter 17 takes up the refinement. The algorithms — PC, GES, NOTEARS, FCI — test the elicited structure against observational data. Edges the expert committed to are tested; edges the data implies but the expert did not propose are surfaced; edges in the equivalence class that the expert left undirected are oriented where the data permits. The handoff is bidirectional: discrepancies between the expert-committed diagram and the data-implied diagram are routed back to the expert for resolution before the model is finalized.

The four-agent architecture described here is designed with this handoff in mind. The audit log documents every structural decision made during elicitation, so that when a Chapter 17 discrepancy surfaces, the team can trace exactly which probe established the conflicting commitment and whether the expert's reasoning, at the time, supports the committed orientation or the data-implied one.

Chapter 18 covers parameterization — the translation from the structural diagram to a model with quantified edge weights and probability distributions. The confidence ranges elicited in phase four of this chapter become the priors for the Bayesian parameterization in Chapter 18.

---

**What would change my mind:** A demonstration that an unstructured LLM conversation, without the protocol described here, produces causal diagrams of comparable quality to the structured interview. There is recent work on LLM-based causal reasoning that does not use this kind of structured architecture, and the results are mixed. If the unstructured approach matched the structured one in head-to-head comparison, my preference for structure would be weakened. The current evidence suggests structure helps substantially, but the gap is not as large as I would expect, and the question is genuinely open.

**Still puzzling:** How to handle the cases where the expert's confidence is well-calibrated for some parts of the diagram and poorly calibrated for others. The kind/wicked distinction from Chapter 15 implies that we should differentiate, but the operationalization within a single interview is hard. The expert does not announce *I know this part well and that part badly*; she speaks with similar confidence about both, and the calibration questions catch the average miscalibration but may miss the variance. I do not yet have a clean way to detect within-expert calibration variance during the interview, and I suspect the resolution will require either longer interviews with explicit calibration sub-rounds or post-interview review against data that surfaces the high-variance regions.

---

*Tags: LLM-guided causal elicitation, Nina framework brand strategy, forty-five-minute first-pass DAG, multi-agent architecture, CPDAG handoff, causal interview protocol, Living Model elicitation pipeline*

---

###  LLM Exercise — Chapter 16: The Machine That Interviews the Expert

**Project:** Build Your Own Living Model

**What you're building this chapter:** A 45-minute multi-agent causal interview on yourself or your domain expert, producing a v3 CPDAG that integrates Chapter 14's KEBN orientations and Chapter 15's bias audit, with all four agent roles (Interviewer, Consistency, Equivalence, Bias-Watch) actively running.

**Tool:** Claude Project (continue). The single Project plays all four agent roles by switching personas in the conversation — this is the prototype. (A production version would be a real multi-agent system; for the textbook exercise the Project does it sequentially.)

---

**The Prompt:**

```
Continuing my Living Model project. My v2.5 CPDAG with bias annotations (Chapters 14 + 15) is in the Project context.

This chapter teaches the four-phase causal elicitation protocol with a four-agent architecture:

PHASES (45 minutes total):
- Phase 1 — Variable Identification (8 min)
- Phase 2 — Edge Elicitation (15 min) — using temporal-precedence substitution and intervention-counterfactual probes
- Phase 3 — Equivalence Resolution (12 min) — proposing orientations for undirected edges, asking the expert to commit or refuse
- Phase 4 — Confidence Calibration (10 min) — IDEA-style: investigate, discuss, estimate, aggregate; eliciting numeric confidence in each commitment

AGENT ROLES (all four active throughout):
- INTERVIEWER — runs the protocol, asks the questions, paces the conversation
- CONSISTENCY — silently tracks every commitment; when a new claim contradicts a prior one, surfaces the contradiction for resolution
- EQUIVALENCE — silently tracks the CPDAG state; when an edge orientation is proposed, checks whether the resulting graph is in the same Markov equivalence class or a new one; surfaces the implication
- BIAS-WATCH — silently monitors for the three Chapter 15 errors (collider blindness, feedback simplification, domain-matching); interrupts when one is suspected

Run the interview now. I will play [PICK ONE: "the expert" / "the facilitator while you play the expert" / "myself"]. Use my v2.5 CPDAG as the starting state. Focus the edge-elicitation and equivalence-resolution phases on the prioritized still-undirected edges from Chapter 7 and the bias-flagged edges from Chapter 15.

For each agent intervention, label which agent is speaking ("[CONSISTENCY]: Earlier you said X drives Y. Now you're claiming Y drives X. Which holds?" or "[BIAS-WATCH]: You're orienting this edge by analogy to retail when your domain is healthcare. Is the mechanism actually parallel?"). Keep the Interviewer in the foreground; let the other agents punctuate.

Time-box each phase. If a phase runs over, note the overrun and what you cut.

Output:
- The interview transcript (or a summary if too long), with agent labels
- The v3 CPDAG (Mermaid), with each newly oriented edge tagged by which agent committed it
- A confidence score per edge (low / medium / high) from Phase 4
- A "still undirected" list — what the interview could not resolve
- An honest section: which agent was most useful in this interview? Which was least? What does that say about the architecture for production?
```

---

**What this produces:** A 45-minute interview transcript with four-agent annotations, a v3 CPDAG with confidence scores and orientation provenance, and a "still undirected" list to be resolved either by Chapter 17's algorithms or by additional data collection.

**How to adapt this prompt:**
- *For your own project:* If you find the multi-agent simulation hard to follow, do the four agent roles in sequential passes — run the Interviewer alone first, then have Consistency review the transcript, etc. Same outcome, easier to track.
- *For ChatGPT / Gemini:* Works as-is. ChatGPT's Custom GPTs can encode each agent as a separate persona.
- *For Claude Code:* If you want the production version, ask Claude Code to write a Python multi-agent system using the Anthropic API where each agent is a separate API call with its own system prompt. This is the architectural prototype Chapter 16 specifies.
- *For a Claude Project:* Recommended for the textbook exercise. The Project keeps all your prior CPDAG state in context.

**Connection to previous chapters:** Chapter 14 ran a single-expert KEBN session. Chapter 15 audited the orientation choices. This chapter wraps both into a structured architecture that works in 45 minutes instead of weeks — the key bottleneck-removal that makes the rest of the Living Model architecture deployable at organizational scale.

**Preview of next chapter:** Chapter 17 brings algorithmic causal discovery into the loop. You'll run PC, GES, and (on synthetic data from your SCM) NOTEARS over your variable list and resolve disagreements between the algorithmic CPDAG and your expert v3 CPDAG using the three-step protocol.

---

## 🕰️ AI Wayback Machine

The ideas in this chapter didn't appear from nowhere. **Margaret Mead** was developing structured ethnographic interviewing in *Coming of Age in Samoa* (1928) and the methodological work that followed — the disciplined elicitation of tacit knowledge from a domain expert who is, in this case, a culture decades before most people had heard of an LLM-guided multi-agent interview that elicits expert causal knowledge in 45 minutes. Here's a prompt to find out more — and then make it better.

![Margaret Mead, c. 1930s. AI-generated portrait based on a public domain photograph (Wikimedia Commons).](images/margaret-mead.jpg)
*Margaret Mead, c. 1930s. AI-generated portrait based on a public domain photograph.*

**Run this:**

```
Who was Margaret Mead, and how does her structured ethnographic interviewing — the disciplined elicitation of cultural knowledge from informants who couldn't articulate it directly — connect to the multi-agent LLM interview architecture (Interviewer, Consistency, Equivalence, Bias-Watch) the chapter specifies? Keep it to three paragraphs. End with the single most surprising thing about her career or ideas.
```

→ Search **"Margaret Mead"** on Wikipedia after you run this. See what the model got right, got wrong, or left out.

**Now make the prompt better.** Try one of these:

- Ask it to explain *participant observation* in plain language, as if you've never read anthropology
- Ask it to compare Mead's structured field-interview protocol to the four-agent role split in the chapter
- Add a constraint: "Answer as if you're writing the system prompt for the Interviewer agent"

What changes? What gets better? What gets worse?

