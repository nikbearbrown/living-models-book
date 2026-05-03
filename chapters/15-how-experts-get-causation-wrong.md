# Chapter 15 — How Experts Get Causation Wrong

*Collider blindness, feedback-loop simplification, domain-matching heuristics — and when to trust an expert's causal judgment versus when to interrogate it.*

---

In 1946, Joseph Berkson — a biostatistician at the Mayo Clinic — published a short paper that was, on its surface, a piece of methodological housekeeping. He had noticed something odd in the hospital's records. Among Mayo's inpatients, those with diabetes appeared to have a *lower* rate of cholecystitis — gallbladder disease — than those without diabetes. The pattern was real in the data. It was statistically clean. Several clinicians had taken note, and the discussion in the wards had begun to drift toward causal explanations. Perhaps something about diabetes — a metabolic effect, a change in bile chemistry, an alteration in tissue response — was protective against gallbladder disease. The next step, the clinicians thought, was to investigate the mechanism.

Berkson stopped the discussion before it left the building. The pattern, he showed, was an artifact of looking at hospital inpatients rather than at the population at large. Patients are admitted because they have at least one serious illness. If a patient does not have diabetes, the reason for the hospitalization must be something else, and that something else is somewhat more likely to involve a gallbladder issue. If a patient does have diabetes, the diabetes itself can be a sufficient reason for hospitalization — a gallbladder problem is not required to explain the patient's presence in the dataset.

The data did not contain evidence of a protective mechanism. It contained evidence of a collider — *being hospitalized* — that had been conditioned on by the simple act of looking at hospital records. Conditioning on the collider induced a spurious negative correlation between two conditions that, in the general population, were either uncorrelated or weakly positively correlated. The protective effect existed only inside the hospital, where it was an arithmetic consequence of the selection mechanism. Outside the hospital, in the population at large, it would not be found.

<!-- → [DIAGRAM: Berkson's collider structure — three nodes: Diabetes (D), Gallbladder Disease (G), Hospitalized (H); arrows from D to H and from G to H (both cause hospitalization); a dashed bidirectional arrow between D and G labeled "spurious correlation induced by conditioning on H"; a second panel showing the population-level graph with no arrow between D and G; caption: "Conditioning on the common effect (H) creates a correlation between its causes (D and G) that does not exist in the population"] -->

What is striking about Berkson's case is not just the bias itself. We covered collider bias in Chapter 4 and revisited it in Chapter 6. What is striking is who got it wrong. The clinicians at the Mayo Clinic in 1946 were among the best-trained physicians in the world. They were experienced. They had thoughtful colleagues to discuss findings with. They had access to a high-quality dataset. They were not careless. They saw a real correlation, formed a plausible causal hypothesis, and were preparing to investigate the underlying mechanism. Berkson's intervention saved them from a research program that would have produced confident but false conclusions — but the program would have proceeded if Berkson had not been there. The bias was invisible to the experts most equipped to detect it.

Eighty years later, the bias is still invisible to most experts most of the time. The mechanism that catches careful clinicians at the Mayo Clinic also catches careful product managers, careful policy analysts, careful military strategists. It is not a failure of effort or competence. It is a structural feature of specialized cognition under conditions where the structure of the data does not match the structure of the question. This chapter is about three such structural features — collider blindness, feedback-loop simplification, and domain-matching heuristics — and about how a Living Model architecture has to be built to extract expert knowledge while controlling for them.

---

## Three Errors, Not Random Ones

Chapter 14 argued that the expert is mathematically necessary in the causal modeling room. The argument was that observational data alone cannot identify causal structure, and the resolution requires knowledge from outside the data, which only the expert can supply. That argument stands. But it raises an immediate question the previous chapter set aside and this chapter has to take up. *If the expert is necessary, but the expert is also human, what specific errors does the expert make, and what do we do about them?*

The literature on expert judgment is vast, and a great deal of it consists of cataloging cognitive biases — anchoring, availability, confirmation, hindsight, overconfidence, framing, motivated reasoning, and several dozen more. I will not catalog all of them here. What I want to focus on is a smaller set of errors specific to *causal* reasoning — errors that arise when an expert is asked to specify the structure of cause-and-effect relationships in a domain. These errors cluster into three categories, each with a recognizable pattern, a documented track record of producing wrong conclusions in expert hands, and a corresponding architectural response in the way an elicitation should be conducted.

The three are collider blindness, feedback-loop simplification, and domain-matching heuristics. They are not the whole story of expert fallibility. They are the part that matters most when an expert is asked to draw a causal diagram or specify a structural mechanism, and they are the part the rest of Part Three's architecture has to address.

---

## Collider Blindness

Collider blindness is the failure to recognize that a variable in the diagram is a common effect of two or more other variables, and that conditioning on it — by stratifying the data, restricting the sample, or adding it to a regression — induces a spurious association between its causes.

The mechanism is precise and we have seen it before. A diagram with two independent variables $X$ and $Y$ that both cause $Z$ is silent on the $X$-$Y$ relationship in the unconditional distribution. Once we condition on $Z$, however, $X$ and $Y$ become correlated. The correlation is not a feature of the underlying process; it is an artifact of the conditioning. If we know $Z$ and we know $X$, we have information about $Y$ — because $X$ and $Y$ are partially substitutable in producing $Z$, even though neither causes the other.

Why are experts blind to this? Three reasons combine.

The first is that the standard advice in applied statistics — *control for everything you can measure* — actively promotes the error. Experts have been trained, often quite carefully, to think that controlling for more variables produces a more conservative estimate. The intuition is that by ruling out alternative explanations, the residual association is closer to the true causal effect. The intuition is half right. Controlling for confounders moves the estimate toward truth. Controlling for colliders moves it away. Without a causal diagram to distinguish between the two roles, the expert has no principled way to know which variables are which, and the default of control-everything imposes a bias that is invisible until someone draws the diagram.

The second is that many variables that turn out to be colliders look, from a domain-knowledge perspective, like reasonable controls. The expert reaches for them because they correlate with both the treatment and the outcome. We saw this in the smoking and birth-weight paradox of Chapter 5: birth weight correlated with both maternal smoking and infant mortality, and the natural research instinct was to stratify by birth weight to compare smoking effects within the low-weight category. The stratification turned out to be conditioning on a collider, and the resulting paradox — that smoking appeared to be protective for low-birth-weight infants — was an artifact of that conditioning. The expert who stratified was not careless; she was applying the standard approach, and the standard approach was wrong.

The third is that colliders are often invisible to the data-collection process itself. The Berkson example is the canonical case. The hospital dataset did not announce that *being hospitalized* was a collider. Hospitalization was simply a property of the sample, like age or sex. The experts who analyzed the data did not see hospitalization as a variable to reason about; they saw it as a defining condition of the dataset. But it was a variable, it was a collider, and conditioning on it — by virtue of the data only existing for hospitalized patients — produced the spurious result.

The architectural response to collider blindness is twofold. The elicitation has to surface the variables the expert is implicitly conditioning on, including the conditions that define the data sample. *Among whom does this relationship hold?* is a question that should be asked of every relationship the expert proposes. And the elicitation has to walk the resulting diagram and check, for each potential conditioning variable, whether it is a collider on a path between the treatment and outcome. The walk is mechanical once the diagram is drawn; what is hard is making sure the diagram contains the variables that need to be in it.

The cost of collider blindness is high enough to be worth pricing carefully. Every time an expert recommends "controlling for" a variable that is a collider, the resulting analysis is biased in a direction the analyst cannot easily characterize. The bias may be small or large; it may be in either direction. The literature is full of cases where a published estimate was, in retrospect, contaminated by collider conditioning, and where the corrected estimate gave a different — sometimes opposite — answer. Mature elicitation architecture takes this seriously, and the rest of Part Three is structured around taking it seriously.

---

## Feedback-Loop Simplification

The second structural error is feedback-loop simplification: the expert's tendency to represent dynamic systems as static, linear ones, ignoring the cycles and delayed responses that drive the system's actual behavior.

The error has a sociological explanation. Most analytical training — in statistics, economics, machine learning, even in much of operations research — privileges models in which inputs cause outputs in a forward direction without the outputs feeding back. The diagrams we draw are acyclic. The equations we fit are linear. The systems we describe are static at equilibrium. This is a consequence of the tools we have, more than of the world we are trying to model. Real systems are cyclic, nonlinear, dynamic, and almost never at equilibrium. The expert trained on the tools tends to think in terms of the tools, and to compress the world into shapes the tools can handle.

The compression has two specific consequences worth naming.

*Reinforcing-loop blindness* is the first. A reinforcing loop is a structure in which a variable's effects come back to amplify the variable itself. Bank-run dynamics are reinforcing — depositors who see other depositors withdrawing become more likely to withdraw themselves, which causes more withdrawals. Polar-ice melt is reinforcing — less ice means lower albedo means more warming means less ice. Word-of-mouth product growth is reinforcing — more users means more recommendations means more users. Reinforcing loops produce exponential dynamics, characterized by long quiet periods followed by sudden, dramatic acceleration. Experts who reason about such systems with linear models systematically underestimate the speed of the dynamics and are then surprised when the system tips into a regime they had not anticipated.

*Balancing-loop blindness* is the second. A balancing loop is a structure in which the system's responses tend to oppose changes, returning the system toward a set point. A thermostat is a balancing loop. Many homeostatic biological mechanisms are balancing loops. Markets, in many regimes, exhibit balancing dynamics — a price that rises too quickly induces supply responses that bring it back down. Experts who reason about such systems with linear models systematically overestimate the persistence of changes and are then surprised when an intervention that "should" have produced a sustained shift produces only a temporary one before the system snaps back.

<!-- → [DIAGRAM: Reinforcing vs. balancing loop comparison — left panel: a reinforcing loop with two nodes (Bank Withdrawals, Fear of Insolvency) connected by two arrows each labeled "+", creating a cycle; right panel: a balancing loop with two nodes (Price, Supply) connected by a positive and a negative arrow; beneath each, the characteristic dynamic (exponential runaway vs. oscillation/return to setpoint); caption: "The same arrow in a different cycle position produces opposite system dynamics"] -->

A worked example from the supply-chain literature illustrates both kinds of error at once. The bullwhip effect is the phenomenon in which small fluctuations in retail demand produce progressively larger fluctuations as you move upstream in the supply chain — to the wholesaler, to the manufacturer, to the parts supplier. The bullwhip arises because each layer responds to its observed demand with a buffer, and the buffer responses compound. By the time you reach the parts supplier, retail-level demand variation that was a few percent has become twenty or thirty percent variation in parts orders. Expert managers at each layer experience this as a peculiar fact of life — *we always seem to over-order or under-order* — without recognizing that the variation is being generated by the dynamics of the chain itself. A linear analysis cannot represent it. An analysis that captures the dynamics not only explains the bullwhip but suggests interventions — information sharing across layers, smoothing of order policies, reduction of lead-time delays — that can reduce its magnitude. The expert trained on linear models cannot see the structural fix; the expert trained on systems dynamics can.

The architectural response to feedback-loop simplification has two parts. First, the elicitation has to ask explicitly about cycles. *When the outcome of this process happens, does it change the inputs to the next iteration of the process?* If the answer is yes, the diagram needs to represent the cycle, even if the formal apparatus of acyclic graphs has to be augmented to handle it. Second, the elicitation has to ask about the *temporal scale* of the relationships, because cycles only matter when their time scales are short relative to the decision horizon. A cycle that takes ten years to close can be safely ignored for a next-quarter decision. A cycle that closes in days cannot be ignored for any decision with a horizon longer than that.

I want to be careful not to imply that every causal model needs to be a full systems-dynamics simulation. The cost of representing every cycle in every system is high, and most decisions can be made adequately with simpler models that approximate the dynamics over the relevant horizon. The point is not that simpler models are wrong; the point is that the simplification is a choice, and the expert who simplifies should know she is doing so. Feedback-loop blindness is the case where the expert simplifies without knowing — drawing an acyclic linear diagram of a system that is cyclic and nonlinear, then trusting the diagram as if it were accurate. The architecture has to surface the simplification, not eliminate it.

---

## Domain-Matching Heuristics

The third structural error is the domain-matching heuristic: the expert's tendency to import causal frameworks from one domain into another based on superficial similarities, even when the frameworks do not apply.

A domain, in the sense I am using here, is a coherent set of mechanisms and the language used to describe them. The mechanical domain — physical force, impact, motion — has its own vocabulary and its own implicit mechanics. The chemical domain — molecular interaction, transformation, kinetics — has its own. The electromagnetic, biological, psychological, and economic domains all have their own vocabularies and implicit mechanics. Within a domain, the expert has internalized the relevant mechanisms and can reason about them quickly and reliably. Across domains, the internalized mechanisms can mislead.

Cognitive psychologists have documented this pattern in experiments where experts are asked to identify the cause of an outcome described in domain-specific language. When the proposed cause is in the same domain as the outcome, experts are more likely to accept it, even when the actual mechanism crosses domains. The heuristic is useful when domains are genuinely separate; it becomes a source of error when the actual mechanism crosses domains — which happens often in social, organizational, and biological systems.

A particularly instructive case is what economists Andrew Lo and Mark Mueller have called *physics envy* in finance. The deterministic mathematical models that underlie physics — the harmonic oscillator, the wave equation, the laws of motion — are precise and reliable because the systems they describe are stable, repeatable, and unaffected by the act of being modeled. Physical particles do not have feelings about the equations that describe them. They do not adjust their behavior in response to being measured. The mathematics works because the domain it is modeling is genuinely amenable to the mathematics.

Financial markets are not such a domain. The participants read the same models the analysts are using and adjust their behavior in response. A pricing model that is widely adopted changes the prices it is modeling, often in ways that invalidate the model. Risk distributions that have stable shapes in physics have heavy tails and regime shifts in finance. Levels of uncertainty that are fully reducible in physical systems — uncertainty that can be eliminated with enough data — become partially reducible or irreducible in financial systems, where the system itself adapts faster than the data can be collected.

The 2007–2009 financial crisis is, in part, a story of domain-matching gone wrong. Quantitative analysts had imported risk-management techniques from physics-style modeling — value-at-risk, copulas, mean-variance optimization — into a domain where the underlying assumptions did not hold. The models worked beautifully for years because the markets happened to be in a regime where the assumptions were approximately true. When the regime shifted, the models broke catastrophically, and the experts who relied on them were caught unprepared. The error was not in the mathematics; it was in the importation of mathematics from a domain where it was reliable into a domain where its assumptions did not hold.

Domain-matching is not always a mistake. Genuine cross-domain transfer of mechanism is one of the most productive moves in science — Darwin imported population dynamics from Malthus, Shannon imported entropy from thermodynamics, much of modern epidemiology imports modeling techniques from systems engineering. The question is whether the importation is accompanied by careful examination of which assumptions transfer and which do not. When the importation is deliberate and the assumptions are checked, the result is often a productive new framework. When the importation is implicit and the assumptions are not checked, the result is the kind of failure the financial crisis illustrated.

The architectural response is to make the domain transfer explicit. When an expert proposes a causal mechanism, the elicitation should ask: *what is the source of this mechanism — direct observation in this domain, or analogy from another domain?* If the answer is analogy, the elicitation should probe whether the assumptions that make the analogy work in the source domain also hold in the target domain. Often they do not, and the expert recognizes this when forced to articulate them. Surfacing the heuristic is half the work of correcting it.

<!-- → [TABLE: Domain-matching audit framework — rows: Physics → Finance (physics envy), Engineering → Medicine (device reliability → patient outcomes), Economics → Ecology, Military → Business strategy; columns: Source domain, Target domain, Assumptions that hold in source but not target, Canonical failure case, Elicitation probe to surface the mismatch; purpose is to give elicitors a concrete reference for the most common cross-domain imports in organizational decision-making] -->

---

## Kind and Wicked Environments

A useful distinction for assessing when expert intuition can be trusted comes from the work of Robin Hogarth, recently popularized by David Epstein in *Range*. Hogarth distinguishes between *kind* learning environments — in which patterns recur, feedback is fast and accurate, and the rules are stable — and *wicked* environments — in which patterns shift, feedback is delayed or misleading, and the rules change over time.

Expert intuition is reliable in kind environments because the conditions for reliability are present. A chess grandmaster has played thousands of games, received immediate feedback on each move via consequences in subsequent moves, and operated under stable rules. Her pattern recognition is calibrated by the feedback. When she sees a configuration on the board, she can usually identify the right move within seconds, and her intuition is generally correct. The kind environment has produced an intuition that the environment has trained.

Wicked environments produce intuitions that are less reliable, sometimes positively misleading. A senior policy analyst working on macroeconomic forecasting has been in her field for decades, but the feedback she receives on her forecasts is delayed — often years — partial because the counterfactual is not observable, and contaminated because other forecasters' positions affect what gets recorded as consensus. Her intuitions feel as confident as the chess grandmaster's, but the confidence is not backed by the same calibration. She can be confidently wrong in ways the grandmaster cannot.

Most domains lie somewhere between kind and wicked, and the same domain can be kind in some respects and wicked in others. Clinical medicine is kind for diagnostic pattern recognition — the symptoms are observable, the test results provide quick feedback — and wicked for prognostic reasoning, where the long-term consequences of decisions are confounded by everything else that happens. Strategic management is mostly wicked for the central decisions, where the counterfactual is unobservable and feedback is years delayed, but kind for some operational decisions where small experiments can run quickly.

The architectural implication is that expert intuition is *context-dependent reliable*. The elicitation should pay attention to which intuitions were formed in which environments. For intuitions formed in kind environments, the expert's confidence is generally a good guide. For intuitions formed in wicked environments, the expert's confidence is not a good guide, and the elicitation should rely more heavily on formal analysis, falsifiable predictions, and explicit articulation of assumptions.

This is one of the deepest practical lessons of the entire elicitation literature. The question is not whether to trust the expert; it is *which intuitions* to trust and *which to interrogate*. A blanket policy of trusting expert judgment is wrong; a blanket policy of distrusting it is also wrong. The right policy is differentiated — calibrated to the environment in which the intuition was formed and the type of judgment being asked for. The elicitation architecture in Chapter 16 implements this differentiation.

<!-- → [TABLE: Kind vs. wicked environments applied to expert causal judgment — rows: Diagnostic clinical reasoning, Prognostic clinical reasoning, Chess, Macroeconomic forecasting, Supply chain operations, Strategic management; columns: Environment type (Kind/Wicked/Mixed), Feedback characteristics, Reliability of expert intuition, Elicitation posture (lean on intuition vs. interrogate formally); purpose is to make the kind/wicked distinction concrete across domains the book's audience will recognize] -->

---

## What This Means for Elicitation Architecture

Three implications follow from the errors this chapter has cataloged, and each shapes the elicitation architecture in Chapter 16.

The first is that the architecture cannot rely on the expert to spontaneously surface the structural features that matter. Colliders, feedback loops, and cross-domain analogies are exactly the features experts tend not to surface. The architecture has to ask about them explicitly. *What variables are you implicitly conditioning on by virtue of where this data comes from? Are there feedback loops in this system, and what are their time scales? Is the mechanism you are proposing one you have observed directly in this domain, or is it an analogy from another domain?* These prompts are the working content of the structured elicitation, and the architecture's job is to deliver them at the right moments in the conversation.

The second is that the architecture needs to support *adversarial review* of the expert's proposed model, not just elicitation. After the model is drawn, the architecture has to test it. The test is largely mechanical — d-separation walks of the diagram, sensitivity analysis on the conditioning choices, comparison of the diagram's implications against any data the team has access to. But the test has to happen, and it has to feed back into a revision of the model. Single-pass elicitations, where the expert speaks and the model is committed without being interrogated, are systematically vulnerable to the errors in this chapter.

The third is that the architecture has to differentiate by environment. Where the expert's intuitions were formed in kind environments — in domains with fast feedback and stable patterns — the architecture can lean on those intuitions and use them as primary evidence. Where the intuitions were formed in wicked environments, the architecture has to lean less on intuition and more on formal articulation, sensitivity testing, and explicit acknowledgment of the irreducible uncertainty. Most domains require both modes, applied to different parts of the same model.

These three implications are not abstract. They translate directly into design choices in the LLM-guided elicitation system Chapter 16 describes. The system is structured around the prompts that surface the errors this chapter has cataloged. It includes adversarial review steps after every major commitment. It calibrates its reliance on expert confidence by the kind/wicked classification of the relevant intuition. The architecture is the operational answer to the failure modes this chapter has documented, and the answer is specific because the failure modes are specific.

---

## Looking Forward

Chapter 16 takes up the architecture itself: an LLM-guided elicitation system that compresses the months-long Knowledge Engineering with Bayesian Networks process into a forty-five-minute first-pass conversation, while preserving the rigor that makes KEBN work. The system inherits its structural commitments from Chapter 14 (the necessity of expert input) and Chapter 15 (the specific biases that input is subject to). The result is, I argue, a practical instrument for the elicitation step of a Living Model engagement — the step that has historically been the bottleneck for corporate adoption, and the step that has to be solved before the rest of the architecture can run.

---

## Student Activities

**Problem 15.1 — Berkson's Bias in a New Setting.** A major streaming platform analyzes engagement data among its paying subscribers and finds that users who engage with the platform's recommendation algorithm have *lower* rates of subscription cancellation than users who engage primarily through search. The content team concludes that the algorithm builds loyalty. (a) Identify the potential collider in this analysis and explain the selection mechanism that would produce a spurious negative correlation. (b) Draw the causal diagram that would generate Berkson's bias in this setting and the diagram that would reflect a genuine protective causal effect. (c) Propose one data collection strategy — not just statistical adjustment — that would allow you to distinguish between the two diagrams empirically. (d) Explain why controlling for additional user characteristics would not resolve the bias.

**Problem 15.2 — Collider Detection.** A retail analytics team is studying the effect of a promotional discount on customer lifetime value. They propose to control for "whether the customer converted on the discounted purchase" on the grounds that this narrows the comparison to customers who actually used the discount. (a) Draw the causal diagram implied by this proposal. (b) Identify whether "converted on discounted purchase" is a confounder, mediator, or collider in the path from Discount to Lifetime Value, and explain your reasoning. (c) What spurious association would conditioning on this variable introduce? (d) Describe the correct analysis if the team's research question is the total effect of the discount on lifetime value, versus the direct effect of the discount that does not operate through conversion.

**Problem 15.3 — Feedback Loop Identification.** A hospital system has implemented a new patient discharge protocol designed to reduce readmission rates. Preliminary data shows that readmission rates have dropped by 8% in the first three months. The chief medical officer attributes the improvement directly to the protocol and recommends expanding it to all wards. (a) Identify at least two feedback loops that could be driving the observed reduction beyond the direct mechanism of the protocol itself. For each, indicate whether it is reinforcing or balancing, and describe the time scale on which it operates. (b) Explain how each loop could produce an initial 8% reduction that would not persist or would reverse at a longer time horizon. (c) Propose a monitoring design — specifying variables to track and at what frequency — that would distinguish the direct protocol effect from the feedback-loop dynamics. (d) If the CMO proceeds with expansion without accounting for the feedback loops, describe the most likely trajectory of the readmission rate over the following twelve months.

**Problem 15.4 — Domain-Matching Audit.** A technology company's data science team is building a causal model for predicting customer churn. A senior analyst proposes modeling customer tenure as a "decay process" analogous to radioactive decay — customers have a constant hazard rate, and survival probability declines exponentially with time. (a) Identify the domain from which this mechanism is being imported, and list three assumptions the decay model requires in its source domain. (b) Evaluate each assumption against what you know about customer churn behavior in typical software products. (c) Propose an alternative mechanism — drawn from direct observation in the customer relationship domain rather than by analogy — that would better capture the dynamics of churn. (d) Describe what data you would collect to test whether the decay analogy holds in this specific company's customer base.

**Problem 15.5 — Kind vs. Wicked Classification.** For each of the following domains, classify the expert's primary judgment task as Kind, Wicked, or Mixed. Justify your classification using Hogarth's criteria (pattern recurrence, feedback speed and accuracy, rule stability), and specify the elicitation posture — lean on intuition vs. require formal articulation — you would adopt for each. (a) A meteorologist forecasting tomorrow's weather. (b) A venture capitalist evaluating a seed-stage startup's market potential. (c) A flight surgeon assessing pilot fitness for duty. (d) A central bank economist forecasting the effect of a 50 basis point rate increase on inflation over the next 18 months. (e) A fraud analyst classifying individual transactions in real time. (f) A strategy consultant predicting the market share impact of a competitor's new product launch.

**Problem 15.6 — Elicitation Design.** You are facilitating a causal elicitation session with a chief supply chain officer at a consumer electronics company. The domain involves: component lead times, production scheduling, inventory management, demand forecasting, and supplier relationships. The CSO has 25 years of experience and speaks confidently about all five areas. (a) For each of the three structural errors in this chapter — collider blindness, feedback-loop simplification, domain-matching heuristics — design one specific elicitation prompt you would use with this expert. The prompt should be specific to the supply chain domain and should surface the error without implying the expert has already made it. (b) Identify which of the five domain areas you would classify as Kind vs. Wicked for this expert's intuitions, and explain your reasoning. (c) Design the adversarial review step you would run after the expert has proposed the initial causal diagram. Specify what the review checks for, how the checks are run, and what you would do with the results.

---

## Key Terms

**Balancing Loop.** A feedback structure in which a system's responses oppose changes to a key variable, tending to return the system to a set point or equilibrium. A thermostat, a market supply-and-demand cycle, and many biological homeostatic mechanisms are balancing loops. Experts trained on linear models systematically overestimate the persistence of interventions in balancing-loop systems.

**Berkson's Paradox.** The bias introduced when a study samples only from individuals who have been selected into a system — a hospital, a platform, a program — for reasons related to the variables under study. The selection event acts as a collider; conditioning on it (by virtue of the sample only including selected individuals) induces a spurious correlation between causes that are independent in the population. Named for Joseph Berkson (1946).

**Collider Blindness.** The expert error of failing to recognize a common effect as a collider and conditioning on it — through sample restriction, stratification, or adding it to a regression — thereby inducing a spurious association between its independent causes. The most systematic form of the error occurs when the conditioning variable is a defining property of the dataset rather than an explicit analytical choice.

**Domain-Matching Heuristic.** The tendency of experts to import causal frameworks from a domain in which they have deep experience into a superficially similar domain where the frameworks' assumptions do not hold. Physics envy in finance is the canonical case: the importation of deterministic, stationary physical models into adaptive, non-stationary financial systems.

**Feedback-Loop Simplification.** The expert error of representing a dynamic, cyclic system as a static, linear one, failing to represent the reinforcing and balancing loops that drive the system's actual behavior. Typically a consequence of training on tools — regression, optimization, acyclic DAGs — that cannot represent cycles.

**Kind Learning Environment.** An environment in which patterns recur, feedback is fast and accurate, and the rules are stable. Expert intuitions calibrated in kind environments are generally reliable guides. Chess, real-time fraud detection, and short-horizon weather forecasting are examples.

**Physics Envy.** Andrew Lo and Mark Mueller's term for the systematic importation of deterministic, high-certainty quantitative methods from physics into domains — particularly finance — where the underlying systems are adaptive, the uncertainties are partially or fully irreducible, and the mathematics' assumptions do not hold.

**Reinforcing Loop.** A feedback structure in which a variable's effects return to amplify the variable itself, producing exponential dynamics — long quiet periods followed by sudden, dramatic acceleration or collapse. Bank runs, viral growth, and ice-albedo feedback are reinforcing loops. Experts trained on linear models systematically underestimate the speed of reinforcing-loop dynamics.

**Wicked Learning Environment.** An environment in which patterns shift, feedback is delayed or misleading, and the rules change over time. Expert intuitions formed in wicked environments may carry high subjective confidence but are not reliably calibrated to the actual phenomenon. Macroeconomic forecasting, long-horizon strategic planning, and clinical prognosis are examples.

---

**What would change my mind:** A documented case in which a structured elicitation process designed around the three errors in this chapter — collider blindness, feedback-loop simplification, domain-matching heuristics — failed to detect a major instance of one of them, and produced a model that was wrong in a way the standard error catalog should have caught. The current architecture is built on the assumption that surfacing the errors is sufficient to mitigate them. If surfacing is necessary but not sufficient, that would change how I think about the elicitation design. I would update if shown such a case.

**Still puzzling:** The relationship between expert confidence and expert reliability across mixed-environment domains. Most consequential domains are partly kind and partly wicked, and most experts work across the parts without explicitly distinguishing them. An expert can be highly reliable on the kind parts and unreliable on the wicked parts, with the same level of subjective confidence in both. I do not yet have a clean way to elicit the distinction without making the expert defensive, and I suspect the resolution will require careful attention to elicitation language that I have not fully worked out.

---

**Tags:** collider blindness expert error, Berkson 1946 gallbladder paradox, feedback loop simplification linear thinking, domain matching heuristic physics envy, kind versus wicked learning environments

---

###  LLM Exercise — Chapter 15: How Experts Get Causation Wrong

**Project:** Build Your Own Living Model

**What you're building this chapter:** A stress-test of your v2 CPDAG against the three expert-reasoning errors — collider blindness, feedback-loop simplification, and domain-matching heuristics — plus a kind/wicked classification of the learning environment your expert (or your own knowledge) was forged in.

**Tool:** Claude Project (continue).

---

**The Prompt:**

```
Continuing my Living Model project. My v2 CPDAG (Chapter 14) and the expert-orientation justifications are in the Project context.

This chapter teaches three errors specific to causal reasoning that even careful experts make:

1. COLLIDER BLINDNESS — confusing the spurious correlation induced by conditioning on a common effect (X → Z ← Y; conditioning on Z) with a real causal relationship between the causes. The Mayo Clinic 1946 gallbladder example: diabetic patients appeared to have lower gallbladder disease rates, but only because both conditions cause hospitalization, and the analysis was conditioned on being hospitalized.

2. FEEDBACK-LOOP SIMPLIFICATION — collapsing reinforcing loops (R) and balancing loops (B) into linear chains. The bullwhip effect: each step in the supply chain treats the demand signal as causal when it's a feedback amplification.

3. DOMAIN-MATCHING HEURISTICS — applying the causal pattern from a familiar adjacent domain to a domain where the pattern doesn't hold. "Physics envy" in finance: treating asset returns as physical processes with stationary distributions, leading to risk models like LTCM's that handled normal volatility well and tail events catastrophically.

Plus Hogarth's distinction:
- KIND learning environment — fast, accurate, repeated feedback (chess, poker, weather forecasting in stable climates).
- WICKED learning environment — slow, noisy, ambiguous feedback (strategy, brand, long-horizon investing, organizational change). Experts in wicked environments can have high confidence and low reliability on the same question.

For my v2 CPDAG, do four things:

1. COLLIDER AUDIT — For every node in my graph, check: is this node a common effect of two or more causes (a collider)? If yes, was it conditioned on (used in any analysis, included in the adjustment set, used as a filter)? If yes-and-yes, the inferences downstream of that conditioning are suspect. List every collider that was conditioned on and what spurious correlation it might have introduced.

2. FEEDBACK CHECK — Identify every place in my domain where a feedback loop almost certainly exists (sales→capacity→fulfillment→sales; price→demand→competitor price→demand; recommendation→engagement→training data→recommendation). For each: is it represented in my DAG, or did I collapse it into a directed chain? If collapsed, name what that simplification will hide.

3. DOMAIN-MATCH AUDIT — For each major causal claim my CPDAG makes, ask: am I (or my expert) reasoning by analogy from another domain where the claim is well-established? If yes, name the source domain and check whether the causal mechanism actually transfers — or whether I'm doing physics-envy. Flag any imported assumptions.

4. KIND/WICKED CLASSIFICATION — Classify the learning environment my domain expertise (or the real expert's) was forged in. KIND, WICKED, or MIXED. If MIXED, identify which sub-areas of my graph are in the kind region (high confidence justified) versus wicked (high confidence unjustified). For wicked regions: recommend either widening the confidence intervals on those edges or scheduling explicit empirical tests before acting.

Output:
- Three audit lists (collider, feedback, domain-match) with specific findings
- Kind/wicked classification with sub-area breakdown
- A v2.5 CPDAG with annotations: each suspect edge or node tagged with the error type and the implication for downstream analysis
- A "do this before Chapter 16" list — specific edges to revisit when the multi-agent interview runs in the next chapter

End with: which of the three errors is the biggest risk for MY domain specifically, and what one architectural choice in the Chapter 16 interview design would most reduce it?
```

---

**What this produces:** Three named audit findings (collider, feedback, domain-match), a kind/wicked classification, an annotated v2.5 CPDAG, and a list of edges to revisit in the Chapter 16 interview.

**How to adapt this prompt:**
- *For your own project:* If your domain is heavily wicked (most strategy/brand/policy work), expect this audit to flag many edges. That's the honest result.
- *For ChatGPT / Gemini:* Works as-is.
- *For Claude Code:* Optional — could add an automated collider-check on a networkx graph.
- *For a Claude Project:* Recommended.

**Connection to previous chapters:** Chapter 14 produced the expert-refined graph. This chapter says the expert is human and audits accordingly. Together they give you a model that has been built deliberately AND stress-tested against the predictable failure modes of expert judgment.

**Preview of next chapter:** Chapter 16 turns expert elicitation into software. You'll set up the four-agent architecture (Interviewer, Consistency, Equivalence, Bias-Watch) and run a 45-minute multi-agent interview that produces a refined CPDAG with all three of this chapter's errors actively monitored.
