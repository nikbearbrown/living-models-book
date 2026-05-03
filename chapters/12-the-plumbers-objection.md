# Chapter 12 — The Plumber's Objection

*Duflo's "Economist as Plumber," what fills the distance between a correct causal estimate and an effective organizational change, and why a Living Model has to do more than estimate effects.*

---

In 2007, researchers in Tangier, Morocco, were studying take-up of piped water connections among low-income households that did not yet have one. The Moroccan government had made subsidized loans available so that any household could afford the connection fee. The expected economics were clear. Households without piped water spent significant time and money obtaining water from public taps and private vendors. A piped connection would save them both. The loan made the connection affordable. By any conventional analysis, the program should have produced near-universal take-up.

It did not. In the early phase, fewer than 10% of eligible households took the loan.

The model was correct on the merits. The economic case was sound. The take-up rate was a tenth of what the model predicted.

The researchers went into the field and watched what households actually did. The answer turned out to have nothing to do with the loan's terms or the value of the connection. It had to do with the application process. To get the loan, a household had to obtain a copy of an identity document, photocopy it, fill out a form, and travel to the water utility office during specific hours. Each step was small. The cumulative effect was a barrier. Households intended to apply, ran into one of the steps, did not finish, and never came back.

The researchers ran an experiment. In the treatment group, the research team helped with the procedural steps — they brought the form, helped fill it out, photocopied the documents, and submitted the application on the household's behalf. The intervention added nothing to the loan's economic terms. It only removed the administrative friction.

Take-up in the treatment group was 69%. The same households, faced with the same loan, with the same economic logic, had increased take-up by a factor of seven simply because someone else handled the paperwork.

<!-- → [CHART: Bar chart comparing take-up rates — three bars: Control group (~10%), Treatment group (69%), and a reference bar at 100% labeled "Model prediction (near-universal)"; purpose is to make visceral the scale of the gap between correct causal estimate and actual take-up before the chapter names why] -->

![Figure 12.1 — Bar chart comparing take-up rates](images/12-the-plumbers-objection-fig-01.jpg)


The Tangier study is a small example of a large pattern. The model said the loan would produce piped water connections. The model was correct on the causal structure — the loan does cause connections in households where the loan is taken. The implementation produced almost no connections, because the path between loan availability and loan acceptance was clogged with administrative steps that no model on Earth would have predicted to matter at the scale they did. The distance between *correct causal estimate* and *effective organizational change* turned out to be enormous, and what filled it was something the modeling apparatus had given the practitioners no preparation for.

This chapter is about that gap, what fills it, and what a Living Model architecture has to do beyond what we have so far described.

---

## Scientist, Engineer, Plumber

In her 2017 presidential address to the American Economic Association — the Richard T. Ely Lecture — Esther Duflo offered a metaphor that has propagated rapidly through development economics and beyond. The economist, she suggested, has historically aspired to be a *scientist*, identifying universal laws that govern human and economic behavior. Some economists, particularly those working on mechanism design, have functioned as *engineers*, taking the scientist's laws and building specific systems — kidney-exchange algorithms, school-choice procedures, auction designs — that apply the laws to real problems.

Duflo's argument was that neither role captures what is actually needed when an intervention has to be made to work. The scientist establishes causal knowledge. The engineer designs an intervention informed by that knowledge. But the intervention then has to be installed in a real organization, with its real people, its real procedures, its real bureaucracies, and its real history. The installation is the work of a *plumber*. The plumber takes the engineer's design and figures out how to fit it into the existing system of pipes — what couplings to use, where the pipes are likely to leak, what to do about the awkward bend by the meter that no one anticipated.

Plumbers, in Duflo's framing, do not have the prestige of scientists or the elegance of engineers. They get their hands dirty. They tinker. They make decisions without complete knowledge of the system they are working in, because no one has complete knowledge of the system they are working in. They monitor for leaks after the installation is done, because installations always leak somewhere, and the leaks tell them where the next intervention has to go.

<!-- → [DIAGRAM: Three-role progression — Scientist → Engineer → Plumber; each role labeled with its question ("What are the laws?", "How do we build it?", "How do we make it work here?"), its primary output (causal knowledge, system design, working installation), and its characteristic limitation (doesn't address deployment, doesn't address context, no prestige but indispensable); an arrow spanning all three labeled "gap between estimate and outcome grows as you move right"] -->

![Figure 12.2 — Three-role progression](images/12-the-plumbers-objection-fig-02.jpg)


The plumber metaphor has resonated because it names something practitioners had felt for decades and lacked a vocabulary for. Most policy interventions, most strategic initiatives, most organizational changes do not fail because the underlying analysis was wrong. They fail because the implementation was poor. The model said *do X*. The organization did something that looked like X from the outside and was structurally different on the inside, and the result was not what the model had predicted. The plumber's work is the work of making the implementation actually be X, in the messy organization where X has to live.

The objection the plumber raises against causal modeling is therefore not that the modeling is wrong. It is that the modeling, taken on its own, is *incomplete*. A correct causal estimate tells the decision-maker what should happen if the intervention is delivered as designed. It does not tell her how to deliver it as designed, what will go wrong in the delivery, or what to monitor to detect the problems early. Those are plumber's questions, and they are answerable only by attending to a layer of detail that most causal analysis treats as out of scope.

---

## What Fills the Gap

If the plumber is right that there is a gap between correct estimate and effective change, the contents of the gap deserve specification. Across hundreds of field experiments, the development economics literature has converged on a recurring set of factors. They are not exotic. They are familiar to anyone who has ever tried to make an organizational change actually take effect. The discipline of paying attention to them, however, is harder than it sounds.

**Administrative friction** is the first factor and was the principal lesson of Tangier. Small procedural costs — a form to fill out, a document to obtain, an office to visit, a click to perform — reduce take-up of an intervention dramatically and out of proportion to their cost in monetary terms. The literature on what behavioral economists call hassle costs has documented this pattern across health care enrollment, retirement savings, college financial aid, food assistance, and dozens of other domains. The cost of removing hassle is typically tiny compared to the cost of the underlying intervention. The benefit, in terms of actual take-up, is often a multiple.

**Default architectures** are the second. The choice the agent faces in the absence of any active decision is, very often, the choice the agent ends up making. Switching the default from opt-in to opt-out routinely produces order-of-magnitude changes in take-up, again with no change in the economic terms. The classic case is retirement savings: countries and companies that have switched their pension defaults to opt-out have seen savings rates rise dramatically. The pension's economics did not change. The plumbing did.

**Incentive architectures** — how individual workers and bureaucrats respond to performance metrics — are the third. The model assumes that the people implementing the intervention will execute it as designed. In practice, the executors respond to whatever they are measured on. If a hospital is measured on readmission rates, doctors will manage the patients whose readmission would worsen the metric, even at the cost of better outcomes for other patients. If a program for nurse attendance installs a time-stamp machine, the nurses and their supervisors will collaborate to circumvent the machine. The implementation creates an environment in which the executors operate, and the executors' response to that environment is part of the intervention, whether the modeler accounts for it or not.

**Behavioral biases** are the fourth. People have limited attention, limited willpower, and limited ability to act on their long-term interests. They are sensitive to features of the environment — framing, social context, the specific phrasing of a notification — that the underlying economic model treats as background noise. A causal model that assumes rational agents will frequently produce predictions that fail because the agents are not rational in the way the model assumed.

**Institutional inertia** is the fifth. Organizations have their own history, their own cultures, their own informal norms that govern how things get done. A new policy lands in this preexisting context, and the context shapes how the policy is interpreted and executed. The same policy can produce dramatically different outcomes in different organizations, not because the underlying causal mechanism is different but because the organizational substrate transforms the policy as it is delivered.

The plumber's objection is that the modeling apparatus developed in Part Two gives the practitioner no guidance on any of these factors. The apparatus tells her what would happen if the intervention were delivered cleanly. The plumbing factors determine whether the intervention is delivered cleanly. The bridge between *would happen* and *will happen* is the work of the plumber, and the modeling apparatus is silent on it.

<!-- → [TABLE: Five plumbing factors — rows: Administrative Friction, Default Architectures, Incentive Architectures, Behavioral Biases, Institutional Inertia; columns: Definition, Classic example from the literature, Why causal models miss it, Plumbing fix category; purpose is to make the gap between causal estimate and effective change concrete and scannable] -->

*Figure 12.3*

| | **Property** | **Value** |
|---|---|---|
| **Administrative Friction** | _fill in_ | _fill in_ |
| **Default Architectures** | _fill in_ | _fill in_ |
| **Incentive Architectures** | _fill in_ | _fill in_ |
| **Behavioral Biases** | _fill in_ | _fill in_ |
| **Institutional Inertia** | _fill in_ | _fill in_ |

: {.comparison-table}


---

## Ne-FMS: A Worked Example

The most consequential case study of plumbing in the development literature is the National Electronic Fund Management System reform of India's Mahatma Gandhi National Rural Employment Guarantee Scheme. The case is worth working through in detail, because it shows what fixing the plumbing actually looks like and why the result was transformational.

MGNREGA guarantees one hundred days of paid employment per year to any rural household whose adult members are willing to do unskilled manual labor. As of the early 2010s, it disbursed funds to tens of millions of households. The economic logic of the program is not at issue. Cash transfers to working rural households, in a country with a large and persistent rural poverty problem, are causally productive of consumption smoothing, food security, and downstream development outcomes. The intervention is well-supported by causal analysis. The implementation, by 2010, was visibly broken.

Funds flowed through a four-layer hierarchy. The central government released funds to state governments. State governments released funds to district administrations. District administrations released funds to village-level councils called *panchayats*. Panchayats then paid the laborers. At each layer, money disappeared. Some was lost to delays — funds sat in higher-level accounts while laborers waited months for wages. Some was lost to corruption — local officials paid ghost workers, paid laborers less than the official wage, or simply pocketed the difference. Some was lost to administrative friction — laborers spent days traveling to collect payments that should have been delivered automatically. The leaks were systemic. Estimates of the share of MGNREGA funds that reached intended beneficiaries varied, but the consensus was that a substantial fraction was being lost in the pipes.

The Ne-FMS reform replaced the four-layer hierarchy with a direct flow. The central or state government released funds directly into laborers' bank accounts, bypassing the intermediate layers entirely. The flow was electronically tracked end to end. Local officials no longer handled the funds.

Note carefully what the reform did not do. It did not change the policy. The 100-day guarantee remained. It did not change the wages. The official daily rate was unchanged. It did not change the eligibility rules or the work requirements. The intervention was, on the level of policy specification, identical before and after. What changed was the plumbing. The pipes were rerouted to bypass the layers where the leaks were largest. The architecture was made transparent in a way the previous architecture was not. The control points where corruption had been concentrated were removed.

The results were striking. Funds reached beneficiaries faster. Leakage — measured by the gap between funds disbursed and funds received — declined substantially. The cost of the program, per beneficiary served, dropped, because less money was disappearing into the pipes. The reform's success in Bihar was sufficiently clear that the Indian Union Cabinet approved a national rollout of the Ne-FMS architecture in 2015.

<!-- → [DIAGRAM: MGNREGA fund flow before and after Ne-FMS — left panel: four-layer hierarchy with labeled leak points at each layer (delay, corruption, friction); right panel: direct electronic transfer from central account to laborer's bank account, with the intermediate layers removed; amounts or percentages of leakage at each point if estimable; caption: "The plumbing fix did not change the policy — it rerouted the pipes"] -->

![Figure 12.4 — MGNREGA fund flow before and after Ne-FMS](images/12-the-plumbers-objection-fig-04.jpg)


The structural lesson is exactly what Duflo had been arguing for. The intervention's effect — the causal estimate produced by careful analysis of MGNREGA — was real. The intervention's outcome — what laborers actually received — was a fraction of the estimate, because the plumbing leaked. Fixing the plumbing did not require new policy or new analysis. It required attention to the architecture of fund flow, the points where intermediaries had control, and the design of mechanisms that bypassed the leaks. Once the plumbing was fixed, the gap between estimate and outcome closed substantially, and the intervention's potential was realized in the field.

The Ne-FMS reform was not a failure of the underlying causal model. The model was right. The reform was not a triumph of better causal modeling. The model did not change. The reform was a triumph of plumbing — of attending to the architecture of delivery, finding the points of failure, and rebuilding them. This is precisely what Duflo's metaphor names, and it is precisely the work that any deployed Living Model has to be capable of supporting.

---

## Why Models Cannot Tell the Plumber What Matters

A reasonable response to the Tangier and Ne-FMS cases is to ask whether the modeling apparatus can be extended to include the plumbing factors. If administrative friction, defaults, and institutional inertia matter, can we put them in the diagram? Can we treat them as variables, estimate their effects, and incorporate them into the causal estimate?

Sometimes we can. There is nothing in the apparatus that prevents administrative friction from being a variable in a causal diagram. If we have data on hassle costs, default settings, and institutional substrates, we can include them, estimate their effects, and produce a recommendation that accounts for them. The Living Model framework is, in principle, expansive enough to encompass plumbing factors as a class of variables.

The difficulty is practical. The model can include the plumbing factors only if we know in advance which ones will matter. We rarely do. The whole point of the plumber's objection is that the factors that determine the success or failure of an implementation are typically not the ones the modeler thought of in advance. They surface during deployment. They are specific to the organization, the context, the moment. They cannot reliably be enumerated at modeling time, because the relevant ones depend on details of the delivery environment that the modeler is unlikely to have access to.

This is what Duflo means when she says the work of the plumber is *tinkering*. The plumber does not know in advance what will leak. She installs the system, watches what happens, finds the leaks, and adjusts. The adjustments accumulate into knowledge of where leaks tend to occur in this kind of system, but the knowledge is local and provisional. It does not transfer cleanly across contexts. A reform that worked in Bihar may face different leaks when applied to Tamil Nadu. A pricing rollout that worked in one customer segment may face different leaks in another. The plumbing knowledge is empirical, accretive, and context-bound.

What this means for Living Models is that the model alone cannot deliver the outcome. The model can produce a recommendation. The deployment of that recommendation has to be instrumented in a way that detects the leaks, surfaces them to the people who can fix them, and feeds the resulting knowledge back into the model. A Living Model that produces a recommendation and does not also provide the instrumentation is, at deployment time, just another inert piece of analysis — better than a non-causal recommendation, but still subject to the gap between estimate and outcome that the plumber's objection names.

The continually updated property of a Living Model, operationalized in Chapter 20, is partly an answer to this. A model that updates itself as deployment proceeds can absorb the plumbing knowledge as it accrues. The model's current recommendation reflects not just the original causal estimate but also the leaks that have been detected and addressed since deployment began. The treatment-oriented property is partly an answer too. A model whose output is an ongoing ranking of interventions, not a one-time prediction, can re-rank as new plumbing problems appear, prioritizing fixes that close the largest leaks. Orchestration — the concept introduced in Chapter 13 — is the operational answer to the plumber's objection. An orchestrated outcome is one in which the recommendation is delivered, the delivery is monitored, the leaks are detected, and the model learns from the leaks. Without orchestration, the recommendation is just a number. With orchestration, the recommendation becomes part of an evolving system that can repair its own pipes.

---

## A Corporate Plumbing Story

The development economics literature gives us the cleanest examples, but the same dynamics operate in every corporate setting. What plumbing looks like in a context most readers of this book will recognize is worth a close look.

A large business-to-business software company, alarmed at rising churn among its mid-market customers, commissions a careful causal analysis. The analysis identifies onboarding quality as the principal driver of first-year retention. Customers whose first thirty days include three or more meaningful product interactions retain at a much higher rate than customers whose first thirty days do not. The estimated causal effect is substantial: improving the rate at which customers reach the three-interaction threshold from 40% to 70% would lift first-year retention by 14 percentage points.

The company invests. It builds an onboarding program designed to drive the three-interaction milestone — automated email sequences, in-app prompts, scheduled check-in calls from customer success managers. It rolls the program out to all mid-market customers. Six months later, the retention numbers are evaluated. Retention has barely moved. The lift is two percentage points, well within the noise of the previous baseline.

Was the analysis wrong?

The analysis was not wrong. The causal estimate was correct — customers who reached the three-interaction threshold did retain at higher rates. What had failed was the delivery. A close investigation found three plumbing problems that had not appeared in the causal analysis.

The first was a sales-to-success handoff problem. New customers were assigned to a customer success manager only after the contract was signed and the account was provisioned. The provisioning process took anywhere from three to twelve days depending on the customer's configuration. During that window, customers received automated onboarding emails but had no human contact. Many reached out to the sales team with questions, were redirected to the CSM, and had to wait for the CSM to be assigned. By the time the CSM was reachable, the customer's onboarding momentum had stalled. The three-interaction window was effectively shorter than thirty days for most customers.

The second was a CSM workload problem. CSMs were assigned a portfolio averaging around 80 customers each. The new onboarding program had been added on top of their existing responsibilities without any change in portfolio sizes. CSMs were not refusing to do the new work; they were doing it with the time they could spare from existing work, which was not enough. The check-in calls were scheduled but not consistently held. The in-app prompts were sent but not personalized. The program existed; the execution did not.

The third was a metric definition problem. The analysis had defined a *meaningful product interaction* in terms of specific events the product logged. The onboarding program had not been specifically designed to drive those events; it had been designed to drive engagement, broadly construed. Customers were engaging — in support tickets, in marketing emails, in webinar attendance — without triggering the specific events the analysis had identified as the causal levers. The lift in engagement did not translate to a lift in the variable causally connected to retention.

None of these three problems was visible in the causal model. None could have been predicted by the analyst at the time the original estimate was made. They surfaced only during deployment, and only because someone went and looked. The plumbing fixes — restructuring the sales-to-success handoff, reducing CSM portfolio sizes during the transition, redefining the program's success metrics to map onto the causally-relevant events — were not derivable from the model. They were derived from observation of what was actually going wrong in the delivery.

After the plumbing fixes, the program produced retention lifts in the range the original analysis had projected. The model had been right all along. The pipes had been leaking, and once they were repaired, the water arrived where it was supposed to.

<!-- → [DIAGRAM: Three-plumbing-problem anatomy for the B2B onboarding case — a horizontal flow from Causal Recommendation ("drive 3 interactions in 30 days") through three labeled leak points (Sales-to-CSM Handoff Delay, CSM Workload Squeeze, Metric Definition Mismatch) to Observed Outcome (2pp lift); below the leak points, a second flow showing the same path after plumbing fixes with the 14pp projected lift; purpose is to show concretely that all three gaps are delivery failures, not model failures] -->

![Figure 12.5 — Three-plumbing-problem anatomy for the B2B onboarding case](images/12-the-plumbers-objection-fig-05.jpg)


---

## What This Demands of the Living Model

The plumber's objection, taken seriously, has three implications for what a Living Model has to do. Each shapes the architecture of Part Three.

The first is **deployment instrumentation**. A Living Model must be coupled to instrumentation that monitors what actually happens during deployment, not just what the recommendation predicted would happen. The instrumentation has to track the delivery of the intervention — was the recommended action actually executed, in the way it was specified, to the units it was supposed to be applied to? — separately from the outcome. Without separating these, leaks at the delivery layer get conflated with failures of the underlying causal claim, and the model gets blamed for plumbing problems that are not its fault.

The second is **continual structural updating**. The Living Model has to update its structure, not just its parameters, as plumbing problems are discovered. If the deployment reveals that an unmodeled variable — say, CSM portfolio size — is shaping the treatment effect, the model has to be capable of incorporating that variable. The continually updated property has to extend beyond parameter refresh to structural revision. Chapter 20 takes up how this works operationally.

The third is **human-in-the-loop orchestration**. The discovery of leaks is not currently automatable. It requires people who go and look at what is happening — sales-to-success handoffs, CSM workloads, metric definitions — and who can recognize when the gap between estimate and outcome is the result of plumbing. The Living Model architecture has to support the work of these people, not replace it. It has to flag where leaks are likely, surface deployment metrics that make leaks visible, and route the resulting plumbing fixes back into the model's structural commitments. Chapter 19's executive report and Chapter 20's continual update protocol both depend on this human-machine collaboration.

<!-- → [TABLE: Three Living Model demands from the plumber's objection — rows: Deployment Instrumentation, Continual Structural Updating, Human-in-the-Loop Orchestration; columns: What it requires, What breaks without it, Which Part Three chapter operationalizes it; purpose is to connect the plumber's objection directly to the architecture of Part Three before the reader enters it] -->

*Figure 12.6*

| | **Property** | **Value** |
|---|---|---|
| **Deployment Instrumentation** | _fill in_ | _fill in_ |
| **Continual Structural Updating** | _fill in_ | _fill in_ |
| **Human-in-the-Loop Orchestration** | _fill in_ | _fill in_ |

: {.comparison-table}


The plumber's objection is not that causal modeling is wrong. It is that causal modeling is a starting point, not an endpoint. The architecture in Part Three is, in part, a response to the objection. The four properties of a Living Model — causal, counterfactual, continually updated, treatment-oriented — together with the orchestration that ties them into a closed loop, are what allow a deployed model to do the plumbing work that a static model cannot. The development economics literature has, over the past two decades, given us the most rigorous examples of plumbing in action. The Living Model is the corporate inheritance of that tradition.

---

## Looking Forward

This chapter closes Part Two. The reader who has worked through the apparatus — the Ladder of Causation, the structural causal model, the equivalence problem, the methods of effect estimation, counterfactual reasoning, the limits of observational data, randomized trials and their SUTVA caveats, and now the plumber's objection — is in possession of the conceptual and technical foundation that Part Three builds on. The architecture of Part Three is designed to operationalize the apparatus while taking seriously the gap between estimate and outcome that the plumber's objection names.

Chapter 13 opens Part Three by defining precisely what a Living Model is — the four properties that distinguish it from dashboards, predictive models, digital twins, and ontological systems, and the orchestration that emerges when the four properties operate together. Chapter 14 takes up the central architectural challenge: how to elicit a causal model from a domain expert, given that the expert holds the knowledge the data alone cannot supply. Chapter 16 describes the LLM-guided elicitation system that compresses what has historically been a months-long Knowledge Engineering with Bayesian Networks engagement into a forty-five-minute first-pass conversation. Chapters 18 through 20 take up the operationalization — from graph to ranked recommendation, the executive report, and the maintenance of the model over time as the world changes and the leaks shift.

The plumber's objection is honored in each of these chapters. The architecture is not a piece of analytical kit. It is a system designed to do the work the plumber identified — to deliver the recommendation, to monitor for leaks, to repair the pipes, and to learn from each deployment what the next one needs to be. The apparatus of Part Two makes this work possible. The architecture of Part Three is what makes it real.

---

## Student Activities

**Problem 12.1 — Classify the Leak.** A state government implements a program providing free nutritional supplements to pregnant women at public health clinics. The program is well-funded and the evidence base for the supplements' efficacy is strong. After one year, maternal health outcomes have not improved. An investigation finds that (a) clinic staff were not informed about the new supplements until six weeks after launch, (b) the supplements were being stored incorrectly, reducing their efficacy, and (c) many eligible women were not aware the program existed. For each of these three findings, identify which of the five plumbing factors it represents, explain why the original causal model would not have predicted it, and propose a specific plumbing fix.

**Problem 12.2 — Redesigning the Tangier Loan.** Using the Tangier piped water case as your reference, design an alternative loan delivery system that would have eliminated the administrative friction barrier without changing the loan's economic terms. Your design should (a) specify every step a household currently has to complete, (b) identify which steps contribute the most friction, (c) propose a redesigned delivery process that removes or automates the high-friction steps, and (d) estimate, qualitatively, how your redesign would change take-up. Identify any new leaks your redesign might introduce.

**Problem 12.3 — The Ne-FMS Counterfactual.** The Ne-FMS reform bypassed intermediate layers of the MGNREGA fund-flow hierarchy. (a) Draw a causal diagram of the pre-reform fund flow, labeling the nodes where leakage occurred and the mechanisms of leakage (delay, corruption, friction) at each node. (b) Draw the post-reform flow on the same diagram and indicate which nodes were bypassed. (c) Using the $P(Y \mid do(X))$ notation from Chapter 1, express the counterfactual the Ne-FMS reform was designed to test. (d) Explain why the counterfactual could not have been answered from the pre-reform observational data alone, and what additional information the reform's rollout provided.

**Problem 12.4 — The B2B Plumbing Audit.** The corporate onboarding case identified three plumbing problems: the sales-to-CSM handoff delay, the CSM workload squeeze, and the metric definition mismatch. For each: (a) explain which of the five plumbing factors it most closely represents, (b) describe a specific, observable signal that would have surfaced the problem during deployment monitoring before the six-month evaluation, and (c) propose a plumbing fix. Your fixes should be implementable without changing the underlying causal recommendation — that is, without revising the claim that three interactions in the first thirty days drives retention.

**Problem 12.5 — Model Extension or Plumbing Fix?** A public health agency has a causal model showing that smoking cessation counseling increases quit rates. It deploys a program where primary care physicians provide counseling at annual checkups. After two years, quit rates have not improved. An investigation finds that physicians are not having the counseling conversations because the conversations take twelve minutes and appointments are scheduled for ten. (a) Explain whether this is a failure of the causal model or a plumbing failure, and why. (b) The agency's data scientist suggests adding "appointment length" as a variable to the causal model. Evaluate this proposal — would it fix the problem, partially fix it, or fail to fix it? (c) Propose a plumbing fix that does not require changing the causal model. (d) Describe the deployment instrumentation you would add to detect this class of problem automatically in future program deployments.

**Problem 12.6 — Living Model Design.** You are designing a Living Model to support a retailer's promotional pricing decisions. The causal analysis shows that targeted promotions sent to price-sensitive customers increase basket size by 18% without eroding margin. (a) Identify three plumbing factors that are likely to create a gap between the 18% estimate and the actual lift observed after deployment. For each, describe the specific leak mechanism and how large you would expect it to be. (b) Specify the deployment instrumentation you would add to the Living Model to detect each of the three leaks you identified. (c) Describe the structural updating protocol: if one of your leaks turns out to be a previously unmodeled variable (e.g., email open rate, segment misclassification, promotion cannibalization), how would the Living Model incorporate that variable after deployment? (d) Identify one aspect of the plumbing problem you would not expect to be automatable, and explain why it requires a human in the loop.

---

## Key Terms

**Administrative Friction (Hassle Costs).** The cumulative effect of small procedural steps — forms, documents, travel, clicks — that reduce take-up of an intervention out of proportion to their monetary cost. The Tangier study showed that removing administrative friction could increase take-up by a factor of seven without changing the intervention's economic terms.

**Behavioral Biases.** Systematic deviations from rational decision-making — limited attention, limited willpower, sensitivity to framing — that cause agents to respond to features of the delivery environment that the underlying causal model treats as irrelevant. Behavioral biases are a class of plumbing factor that is particularly difficult to anticipate at modeling time.

**Default Architecture.** The choice an agent faces in the absence of any active decision. Because most agents accept the default, the design of the default is often the most consequential plumbing variable in a program. Switching pension enrollment from opt-in to opt-out is the canonical example.

**Deployment Instrumentation.** Monitoring infrastructure that tracks what actually happens during the deployment of a recommendation — specifically, whether the recommended action was executed as specified, and to the intended units — separately from the downstream outcome. Instrumentation is the tool that surfaces plumbing leaks before they get attributed to failures of the underlying causal model.

**Human-in-the-Loop Orchestration.** The collaborative process by which human practitioners and a Living Model jointly identify deployment leaks, diagnose their cause, apply plumbing fixes, and incorporate the resulting knowledge into the model's structure. Currently the discovery and diagnosis of plumbing problems is not automatable and requires human judgment.

**Incentive Architecture.** The structure of performance metrics and rewards facing the individuals who implement an intervention. Because executors respond to what they are measured on, not what the intervention was designed to do, incentive architectures frequently redirect behavior in ways that undermine the intervention's intended mechanism. Nurse attendance stamp machines and hospital readmission metrics are canonical examples.

**Institutional Inertia.** The tendency of organizational cultures, informal norms, and historical practices to transform an intervention as it is delivered, producing outcomes that differ from those observed in the original study environment. Institutional inertia is why the same policy can produce dramatically different outcomes in different organizations.

**Living Model (structural updating).** The property of a Living Model by which new variables discovered during deployment — variables that the original model did not include but that turn out to affect the treatment effect — can be incorporated into the model's causal structure. Distinguished from parameter updating, which changes coefficient values without changing the graph.

**Orchestration.** In the Living Model architecture, the closed-loop process by which a recommendation is issued, its delivery is monitored, leaks are detected, fixes are applied, and the resulting knowledge is fed back into the model. Orchestration is the architectural answer to the plumber's objection: it transforms a static recommendation into an evolving system that can repair its own pipes.

**Plumber's Objection.** The objection, named after Esther Duflo's "Economist as Plumber" metaphor, that a correct causal estimate is necessary but not sufficient for an effective intervention. The objection holds that the gap between *would happen under ideal delivery* and *will happen under actual delivery* is filled by plumbing factors — administrative friction, defaults, incentive architectures, behavioral biases, institutional inertia — that the modeling apparatus does not address.

**Tinkering.** Duflo's term for the empirical, iterative, context-bound work of finding and fixing deployment leaks. Distinguished from modeling (which produces estimates) and engineering (which designs interventions) by its dependence on observation of what is actually happening in a specific deployment context. Plumbing knowledge accumulated through tinkering does not transfer cleanly across contexts.

---

**What would change my mind:** A documented case in which a sophisticated causal analysis, deployed without explicit attention to plumbing factors, produced its predicted effect at scale and over time. The literature is heavy with the opposite — careful analyses whose deployment effects fell well short of the predictions because plumbing was not attended to. I have not seen the reverse. I would update if shown a robust case in which plumbing genuinely did not matter.

**Still puzzling:** Whether the discovery of leaks can ever be substantially automated. The current state requires people who go and look. Automated detection of deployment anomalies — the recommendation was issued but the action was not taken, the metric did not move as projected, the population that received the intervention does not match the population the intervention was designed for — is technically feasible and is part of any serious deployment monitoring. But the *interpretation* of the anomaly, the decision about which plumbing fix to apply, currently remains human work. I do not yet know how much of this can be automated and how much will remain irreducibly human.

---

**Tags:** Esther Duflo economist as plumber AEA Ely lecture, Tangier piped water hassle costs, Ne-FMS MGNREGA fund flow reform, plumber's objection to causal AI, distance between causal estimate and organizational change

---

###  LLM Exercise — Chapter 12: The Plumber's Objection

**Project:** Build Your Own Living Model

**What you're building this chapter:** A friction map — the second graph that lives alongside your DAG, naming every administrative, behavioral, default, incentive, and institutional friction that would block your intervention from producing its modeled effect even if the math is perfect.

**Tool:** Claude Project (continue).

---

**The Prompt:**

```
Continuing my Living Model project. My DAG (Chapter 6) and trial protocol for the top intervention (Chapter 11) are in the Project context.

This chapter teaches Duflo's three-role framework: scientist (causal estimate), engineer (translate estimate into intervention), plumber (handle the messy details that determine whether the intervention actually does anything in deployment). The Tangier piped-water case showed take-up at 10% with a perfect economic model and 69% with the same model + procedural help. The model wasn't wrong; the plumbing wasn't done.

Five categories typically fill the gap between correct causal estimate and effective change:

1. ADMINISTRATIVE FRICTION — paperwork, sign-offs, multi-step processes, hours-of-availability constraints.
2. DEFAULTS — what happens when no one acts; opt-in vs opt-out architectures.
3. INCENTIVE ARCHITECTURE — what the people who must execute the intervention are actually paid (or penalized) for.
4. BEHAVIORAL BIASES — present bias, status quo bias, mental accounting, loss aversion in the people on whom the intervention will land.
5. INSTITUTIONAL INERTIA — existing systems, contracts, vendor relationships, legacy processes that resist change.

For my top intervention, build the FRICTION MAP. Do four things:

1. INVENTORY — For each of the five categories, list 2–4 specific frictions that apply to my intervention in my organization. Be granular — not "administrative friction" but "the regional VP must sign off on every price exception over 8%, and her calendar is booked three weeks out."

2. QUANTIFY — For each friction, estimate (with stated assumptions) what fraction of the intended effect it would absorb. The Tangier study showed paperwork absorbed roughly 7× the effect. Most frictions absorb less, but several together can absorb most of an intervention's value.

3. RANK — Order the frictions by total expected absorption. The top three are where deployment will actually fail.

4. PLUMBING PROPOSALS — For the top three frictions, propose a specific plumbing fix. The fix should be implementable inside the next quarter and should be small enough to test (i.e., it has its own measurable proxy for whether it worked). Examples: "Pre-fill the price exception form for all eligible accounts so the VP only has to approve, not author"; "Make opt-out the default for the new pricing tier rather than opt-in"; "Add 'successful intervention deployment' to the regional VP's bonus calculation at a 10% weight."

Then construct the DEPLOYMENT FORECAST: what fraction of my modeled effect should I actually expect in deployment, before plumbing fixes? After plumbing fixes? Be honest — most well-modeled interventions deploy at 30–60% of their trial effect; with explicit plumbing they can reach 70–85%.

Output:
- Friction inventory table: Category | Friction | Expected Absorption | Proposed Fix
- Deployment forecast: Trial effect → Pre-plumbing deployment effect → Post-plumbing deployment effect
- A one-paragraph note for the executive sponsor explaining that the most expensive part of this project will not be the model — it will be the plumbing, and that a Living Model that doesn't budget for plumbing will not produce the projected return.
```

---

**What this produces:** A friction inventory table, a quantified deployment forecast (trial effect → deployment effect), and a memo for the executive sponsor explaining why the plumbing budget matters more than the modeling budget.

**How to adapt this prompt:**
- *For your own project:* If you don't know the absorption fractions, give Claude your best guess and treat the numbers as scenario analysis, not estimates.
- *For ChatGPT / Gemini:* Works as-is.
- *For Claude Code:* Optional — could automate friction-detection by parsing process docs, but typically not worth it.
- *For a Claude Project:* Recommended.

**Connection to previous chapters:** Chapter 11 designed the trial. This chapter says the trial result is an upper bound — the deployment effect will be less, and the friction map tells you by how much and where. Together they give you a defensible deployment forecast rather than the over-optimistic projection that produces most disappointed executives.

**Preview of next chapter:** Chapter 13 closes Part Two and opens Part Three. You'll audit everything you've built so far against the four properties of a Living Model — causal, counterfactual, continually updated, treatment-oriented — and identify which properties are present and which still need to be built in Part Three.
