# Chapter 12 — The Plumber's Objection

*Duflo's "Economist as Plumber," what fills the distance between a correct causal estimate and an effective organizational change, and why a Living Model has to do more than estimate effects.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *The Plumber's Objection*
- *The Distance Between Estimate and Outcome*
- *What Fills the Gap*

**TL;DR.** The apparatus of Part Two — causal graphs, identification, estimation, counterfactual reasoning — is necessary for sound decision-making. It is not sufficient. Between a correct causal estimate and an effective organizational change lies a wide gap filled with administrative friction, default architectures, behavioral responses to incentives, and institutional inertia. Esther Duflo, in her 2017 American Economic Association presidential address, called the work of attending to this gap *plumbing*. The plumber's objection to causal modeling is not that the modeling is wrong; it is that the modeling, on its own, is incomplete. The Living Model architecture in Part Three exists in part because the plumber's objection is correct, and the architecture is the answer to it.

---

## Tangier, 2007

Researchers in Tangier, Morocco, were studying take-up of piped water connections among low-income households that did not yet have one. The Moroccan government, working with international partners, had made subsidized loans available so that any household could afford the connection fee. The expected economics were clear. Households without piped water spent significant time and money obtaining water from public taps and private vendors. A piped connection would save them both. The loan made the connection affordable. By any conventional analysis, the program should have produced near-universal take-up.

It did not. In the early phase of the program, fewer than 10% of eligible households took the loan. The model was correct on the merits. The economic case was sound. The take-up rate was a tenth of what the model predicted.

The researchers — Devoto, Duflo, Dupas, Pariente, and Pons [verify specific citation] — wondered why. They went into the field and watched what households actually did. The answer turned out to have nothing to do with the loan's terms or the value of the connection. It had to do with the application process. To get the loan, a household had to obtain a copy of an identity document, photocopy it, fill out a form, and travel to the water utility office during specific hours. Each of these steps was small. The cumulative effect was a barrier. Households intended to apply, ran into one of the steps, did not finish, and never came back.

The researchers ran an experiment. In the treatment group, the research team helped with the procedural steps — they brought the form, helped fill it out, photocopied the documents, and submitted the application on the household's behalf. The intervention added nothing to the loan's economic terms. It only removed the administrative friction.

Take-up in the treatment group was 69%. The same households, faced with the same loan, with the same economic logic, had increased take-up by a factor of seven, simply because someone else handled the paperwork.

The Tangier study is a small example of a large pattern. The model said the loan would produce piped water connections. The model was correct on the causal structure — the loan does cause connections, in households where the loan is taken. The implementation produced almost no connections, because the path between loan availability and loan acceptance was clogged with administrative steps that no model on Earth would have predicted to matter at the scale they did. The distance between *correct causal estimate* and *effective organizational change* turned out to be enormous, and what filled it was something the modeling apparatus had given the practitioners no preparation for.

This chapter is about that gap, what fills it, and what a Living Model architecture has to do beyond what we have so far described.

---

## Scientist, Engineer, Plumber

In her 2017 presidential address to the American Economic Association — the Richard T. Ely Lecture — Esther Duflo offered a metaphor that has propagated rapidly through development economics and beyond. The economist, she suggested, has historically aspired to be a *scientist*, identifying universal laws that govern human and economic behavior. Some economists, particularly those working on mechanism design, have functioned as *engineers*, taking the scientist's laws and building specific systems — kidney-exchange algorithms, school-choice procedures, auction designs — that apply the laws to real problems.

Duflo's argument was that neither role captures what is actually needed when an intervention has to be made to work. The scientist establishes causal knowledge. The engineer designs an intervention informed by that knowledge. But the intervention then has to be installed in a real organization, with its real people, its real procedures, its real bureaucracies, and its real history. The installation is the work of a *plumber*. The plumber takes the engineer's design and figures out how to fit it into the existing system of pipes, what couplings to use, where the pipes are likely to leak, what to do about the awkward bend by the meter that no one anticipated.

Plumbers, in Duflo's framing, do not have the prestige of scientists or the elegance of engineers. They get their hands dirty. They tinker. They make decisions without complete knowledge of the system they are working in, because no one has complete knowledge of the system they are working in. They monitor for leaks after the installation is done, because installations always leak somewhere, and the leaks tell them where the next intervention has to go.

The plumber metaphor has resonated because it names something practitioners had felt for decades and lacked a vocabulary for. Most policy interventions, most strategic initiatives, most organizational changes do not fail because the underlying analysis was wrong. They fail because the implementation was poor. The model said *do X*. The organization did something that looked like X from the outside and was structurally different on the inside, and the result was not what the model had predicted. The plumber's work is the work of making the implementation actually be X, in the messy organization where X has to live.

The objection the plumber raises against causal modeling is therefore not that the modeling is wrong. It is that the modeling, taken on its own, is incomplete. A correct causal estimate tells the decision-maker what should happen if the intervention is delivered as designed. It does not tell her how to deliver it as designed, what will go wrong in the delivery, or what to monitor to detect the problems early. Those are plumber's questions, and they are answerable only by attending to a layer of detail that most causal analysis treats as out of scope.

---

## What Fills the Gap

If the plumber is right that there is a gap between correct estimate and effective change, then the contents of the gap deserve specification. Across hundreds of field experiments, the development economics literature has converged on a recurring set of factors. They are not exotic. They are familiar to anyone who has ever tried to make an organizational change actually take effect. The discipline of paying attention to them, however, is harder than it sounds.

*Administrative friction* is the first factor and was the principal lesson of Tangier. Small procedural costs — a form to fill out, a document to obtain, an office to visit, a click to perform — reduce take-up of an intervention dramatically and out of proportion to their cost in monetary terms. The literature on what behavioral economists call *hassle costs* has documented this pattern across health care enrollment, retirement savings, college financial aid, food assistance, and dozens of other domains. The cost of removing hassle is typically tiny compared to the cost of the underlying intervention. The benefit, in terms of actual take-up, is often a multiple.

*Default architectures* are the second. The choice the agent faces in the absence of any active decision is, very often, the choice the agent ends up making. Switching the default from opt-in to opt-out routinely produces order-of-magnitude changes in take-up, again with no change in the economic terms. The classic case is retirement savings: countries and companies that have switched their pension defaults to opt-out have seen savings rates rise dramatically. The pension's economics did not change. The plumbing did.

*Incentive architectures* — how individual workers and bureaucrats respond to performance metrics — are the third. The model assumes that the people implementing the intervention will execute it as designed. In practice, the executors respond to whatever they are measured on. If a hospital is measured on readmission rates, doctors will manage the patients whose readmission would worsen the metric, even at the cost of better outcomes for other patients. If a program for nurse attendance installs a time-stamp machine, the nurses and their supervisors will collaborate to circumvent the machine. The implementation creates an environment in which the executors operate, and the executors' response to that environment is part of the intervention, whether the modeler accounts for it or not.

*Behavioral biases* are the fourth. People have limited attention, limited willpower, and limited ability to act on their long-term interests. They are sensitive to features of the environment — framing, social context, the specific phrasing of a notification — that the underlying economic model treats as background noise. A causal model that assumes rational agents will frequently produce predictions that fail because the agents are not rational in the way the model assumed.

*Institutional inertia* is the fifth. Organizations have their own history, their own cultures, their own informal norms that govern how things get done. A new policy lands in this preexisting context, and the context shapes how the policy is interpreted and executed. The same policy can produce dramatically different outcomes in different organizations, not because the underlying causal mechanism is different but because the organizational substrate transforms the policy as it is delivered.

The plumber's objection is that the modeling apparatus, as developed in Part Two, gives the practitioner no guidance on any of these factors. The apparatus tells her what would happen if the intervention were delivered cleanly. The plumbing factors determine whether the intervention is delivered cleanly. The bridge between *would happen* and *will happen* is the work of the plumber, and the modeling apparatus is silent on it.

---

## Ne-FMS: A Worked Example

The most consequential case study of plumbing in the development literature is the National Electronic Fund Management System (Ne-FMS) reform of India's Mahatma Gandhi National Rural Employment Guarantee Scheme. The case is worth working through in some detail, because it shows what fixing the plumbing actually looks like and why the result was, by any reasonable measure, transformational.

MGNREGA — Mahatma Gandhi National Rural Employment Guarantee Act — is one of the largest workfare programs in the world. It guarantees one hundred days of paid employment per year to any rural household whose adult members are willing to do unskilled manual labor. The program is enormous: as of the early 2010s, it disbursed funds to tens of millions of households. The economic logic of the program is not at issue. Cash transfers to working rural households, in a country with a large and persistent rural poverty problem, are causally productive of consumption smoothing, food security, and a variety of downstream development outcomes. The intervention is well-supported by causal analysis. The implementation, by 2010, was visibly broken.

Funds flowed through a four-layer hierarchy. The central government released funds to state governments. State governments released funds to district administrations. District administrations released funds to village-level councils called *panchayats*. Panchayats then paid the laborers. At each layer, money disappeared. Some of it was lost to delays — funds sat in higher-level accounts while laborers waited months for wages. Some of it was lost to corruption — local officials paid ghost workers, paid laborers less than the official wage, or simply pocketed the difference. Some of it was lost to administrative friction — laborers spent days traveling to collect payments that should have been delivered automatically. The leaks were systemic. Estimates of the share of MGNREGA funds that reached intended beneficiaries varied, but the consensus was that a substantial fraction was being lost in the pipes.

Abhijit Banerjee, Duflo, Clément Imbert, Santhosh Mathew, and Rohini Pande studied an alternative architecture in Bihar in 2012 and 2013 [verify specific evaluation citation]. The reform — Ne-FMS — replaced the four-layer hierarchy with a direct flow. The central or state government released funds directly into laborers' bank accounts, bypassing the intermediate layers entirely. The flow was electronically tracked end to end. Local officials no longer handled the funds.

Note carefully what the reform did not do. It did not change the policy. The 100-day guarantee remained. It did not change the wages. The official daily rate was unchanged. It did not change the eligibility rules or the work requirements. The intervention was, on the level of policy specification, identical before and after.

What changed was the plumbing. The pipes were rerouted to bypass the layers where the leaks were largest. The architecture was made transparent in a way the previous architecture was not. The control points where corruption had been concentrated were removed.

The results were striking. Funds reached beneficiaries faster. Leakage — measured by the gap between funds disbursed and funds received — declined substantially. The cost of the program, per beneficiary served, dropped, because less money was disappearing into the pipes. The reform's success in Bihar was sufficiently clear that the Indian Union Cabinet approved a national rollout of the Ne-FMS architecture in 2015.

The structural lesson is the one Duflo had been arguing for. The intervention's effect — the causal estimate produced by careful analysis of MGNREGA — was real. The intervention's outcome — what laborers actually received — was a fraction of the estimate, because the plumbing leaked. Fixing the plumbing did not require new policy or new analysis. It required attention to the architecture of fund flow, the points where intermediaries had control, and the design of mechanisms that bypassed the leaks. Once the plumbing was fixed, the gap between estimate and outcome closed substantially, and the intervention's potential was, for the first time, realized in the field.

I want to underline that the Ne-FMS reform was not a failure of the underlying causal model. The model was right. The reform was not a triumph of better causal modeling. The model did not change. The reform was a triumph of plumbing — of attending to the architecture of delivery, finding the points of failure, and rebuilding them. This is precisely what Duflo's metaphor names, and it is precisely the work that any deployed Living Model has to be capable of supporting.

---

## Why Models Cannot Tell the Plumber What Matters

A reasonable response to the Tangier and Ne-FMS cases is to ask whether the modeling apparatus can be extended to include the plumbing factors. If administrative friction, defaults, and institutional inertia matter, can we put them in the diagram? Can we treat them as variables, estimate their effects, and incorporate them into the causal estimate?

Sometimes we can. There is nothing in the apparatus that prevents administrative friction from being a variable in a causal diagram. If we have data on hassle costs, default settings, and institutional substrates, we can include them, estimate their effects, and produce a recommendation that accounts for them. The Living Model framework is, in principle, expansive enough to encompass plumbing factors as a class of variables.

The difficulty is at the practical level. The model can include the plumbing factors only if we know in advance which ones will matter. We rarely do. The whole point of the plumber's objection is that the factors that determine the success or failure of an implementation are typically not the ones the modeler thought of in advance. They surface during deployment. They are specific to the organization, the context, the moment. They cannot reliably be enumerated at modeling time, because the relevant ones depend on details of the delivery environment that the modeler is unlikely to have access to.

This is what Duflo means when she says the work of the plumber is *tinkering*. The plumber does not know in advance what will leak. She installs the system, watches what happens, finds the leaks, and adjusts. The adjustments accumulate into knowledge of where leaks tend to occur in this kind of system, but the knowledge is local and provisional. It does not transfer cleanly across contexts. A reform that worked in Bihar may face different leaks when applied to Tamil Nadu. A pricing rollout that worked in one customer segment may face different leaks in another. The plumbing knowledge is empirical, accretive, and context-bound.

What this means for Living Models is that the model alone cannot deliver the outcome. The model can produce a recommendation. The deployment of that recommendation has to be instrumented in a way that detects the leaks, surfaces them to the people who can fix them, and feeds the resulting knowledge back into the model. A Living Model that produces a recommendation and does not also provide the instrumentation is, at deployment time, just another inert piece of analysis — better than a non-causal recommendation, but still subject to the gap between estimate and outcome that the plumber's objection names.

The continually updated property of a Living Model, which we discussed in Chapter 13's preview and will operationalize in Chapter 20, is partly an answer to this. A model that updates itself as deployment proceeds can absorb the plumbing knowledge as it accrues. The model's current recommendation reflects not just the original causal estimate but also the leaks that have been detected and addressed since deployment began. The treatment-oriented property is partly an answer too. A model whose output is not a one-time prediction but an ongoing ranking of interventions can re-rank as new plumbing problems appear, prioritizing fixes that close the largest leaks. The orchestration concept I introduced in Chapter 13 is, in fact, the operational answer to the plumber's objection. An orchestrated outcome is one in which the recommendation is delivered, the delivery is monitored, the leaks are detected, and the model learns from the leaks. Without orchestration, the recommendation is just a number. With orchestration, the recommendation becomes part of an evolving system that can repair its own pipes.

---

## A Corporate Plumbing Story

The development economics literature gives us the cleanest examples, but the same dynamics operate in every corporate setting. I want to walk through one example to show what plumbing looks like in a context most readers of this book will recognize.

A large business-to-business software company, alarmed at rising churn among its mid-market customers, commissions a careful causal analysis. The analysis identifies onboarding quality as the principal driver of first-year retention. Customers whose first thirty days include three or more meaningful product interactions retain at a much higher rate than customers whose first thirty days do not. The estimated causal effect is substantial: improving the rate at which customers reach the three-interaction threshold from 40% to 70% would, the analysis projects, lift first-year retention by 14 percentage points.

The company invests. It builds an onboarding program designed to drive the three-interaction milestone — automated email sequences, in-app prompts, scheduled check-in calls from customer success managers. It rolls the program out to all mid-market customers. Six months later, the retention numbers are evaluated.

Retention has barely moved. The lift is two percentage points, well within the noise of the previous baseline. The CFO is unhappy. The customer success leader is unhappy. The analyst who did the original work is unhappy. Everyone is asking the same question: was the analysis wrong?

The analysis was not wrong. The causal estimate was correct, in the sense that customers who reached the three-interaction threshold did, in fact, retain at higher rates. What had failed was the delivery. A close investigation found three plumbing problems that had not appeared in the causal analysis.

The first was a sales-to-success handoff problem. New customers were assigned to a customer success manager (CSM) only after the contract was signed and the account was provisioned. The provisioning process, depending on the customer's configuration, took anywhere from three to twelve days. During that window, customers received automated onboarding emails but had no human contact. Many of them reached out to the sales team with questions, were redirected to the CSM, and had to wait for the CSM to be assigned. By the time the CSM was reachable, the customer's onboarding momentum had stalled. The three-interaction window was effectively shorter than thirty days for most customers, and meaningfully shorter for customers with complex configurations.

The second was a CSM workload problem. CSMs were assigned a portfolio of accounts averaging around 80 customers each. The new onboarding program had been added on top of their existing responsibilities without any change in portfolio sizes. CSMs were not refusing to do the new work; they were doing it with the time they could spare from existing work, which was not enough. The check-in calls were scheduled but not consistently held. The in-app prompts were sent but not personalized. The program existed; the execution did not.

The third was a metric definition problem. The analysis had defined a *meaningful product interaction* in terms of specific events the product logged. The onboarding program had not been specifically designed to drive those events; it had been designed to drive *engagement*, broadly construed. The result was that customers were engaging — in support tickets, in marketing emails, in webinar attendance — without triggering the specific events the analysis had identified as the causal levers. The lift in engagement did not translate to a lift in the variable the analysis had causally connected to retention.

None of these three problems was visible in the causal model. None of them could have been predicted by the analyst at the time the original estimate was made. They surfaced only during deployment, and they surfaced only because someone went and looked. The plumbing fixes — restructuring the sales-to-success handoff, reducing CSM portfolio sizes during the transition, redefining the program's success metrics to map onto the causally-relevant events — were not derivable from the model. They were derived from observation of what was actually going wrong in the delivery.

After the plumbing fixes, the program produced retention lifts in the range the original analysis had projected. The model had been right all along. The pipes had been leaking, and once they were repaired, the water arrived where it was supposed to.

---

## What This Demands of the Living Model

The plumber's objection, taken seriously, has three implications for what a Living Model has to do. Each of them shapes the architecture of Part Three.

The first is *deployment instrumentation*. A Living Model must be coupled to instrumentation that monitors what actually happens during deployment, not just what the recommendation predicted would happen. The instrumentation has to track the delivery of the intervention — was the recommended action actually executed, in the way it was specified, to the units it was supposed to be applied to? — separately from the outcome. Without separating these, leaks at the delivery layer get conflated with failures of the underlying causal claim, and the model gets blamed for plumbing problems that are not its fault.

The second is *continual structural updating*. The Living Model has to update its structure, not just its parameters, as plumbing problems are discovered. If the deployment reveals that an unmodeled variable — say, CSM portfolio size — is shaping the treatment effect, the model has to be capable of incorporating that variable. The continually updated property from Chapter 13 has to extend beyond parameter refresh to structural revision. Chapter 20 takes up how this works operationally.

The third is *human-in-the-loop orchestration*. The discovery of leaks is not, currently, automatable. It requires people who go and look at what is happening — sales-to-success handoffs, CSM workloads, metric definitions — and who can recognize when the gap between estimate and outcome is the result of plumbing. The Living Model architecture has to support the work of these people, not replace it. It has to flag where leaks are likely to be, surface deployment metrics that make leaks visible, and route the resulting plumbing fixes back into the model's structural commitments. Chapter 19's executive report and Chapter 20's continual update protocol both depend on this human-machine collaboration.

The plumber's objection is not that causal modeling is wrong. It is that causal modeling is a starting point, not an endpoint. The architecture in Part Three is, in part, a response to the objection. The four properties of a Living Model — causal, counterfactual, continually updated, treatment-oriented — together with the orchestration that ties them into a closed loop, are what allow a deployed model to do the plumbing work that a static model cannot. The development economics literature has, over the past two decades, given us the most rigorous examples of plumbing in action. The Living Model is the corporate inheritance of that tradition.

---

## Looking Forward

This chapter closes Part Two. The reader who has worked through the apparatus — the Ladder of Causation, the structural causal model, the equivalence problem, the methods of effect estimation, counterfactual reasoning, the limits of observational data, randomized trials and their SUTVA caveats, and now the plumber's objection — is in possession of the conceptual and technical foundation that Part Three builds on. The architecture of Part Three is designed to operationalize the apparatus while taking seriously the gap between estimate and outcome that the plumber's objection names.

Chapter 13 opens Part Three by defining precisely what a Living Model is — the four properties that distinguish it from dashboards, predictive models, digital twins, and ontological systems, and the orchestration that emerges when the four properties operate together. Chapter 14 takes up the central architectural challenge: how to elicit a causal model from a domain expert, given that the expert holds the knowledge the data alone cannot supply. Chapter 16 describes the LLM-guided elicitation system that compresses what has historically been a months-long Knowledge Engineering with Bayesian Networks engagement into a forty-five-minute first-pass conversation. Chapters 18 through 20 take up the operationalization — from graph to ranked recommendation, the executive report, and the maintenance of the model over time as the world changes and the leaks shift.

The plumber's objection is honored in each of these chapters. The architecture is not a piece of analytical kit. It is a system designed to do the work the plumber identified — to deliver the recommendation, to monitor for leaks, to repair the pipes, and to learn from each deployment what the next one needs to be. The apparatus of Part Two makes this work possible. The architecture of Part Three is what makes it real.

---

**What would change my mind:** A documented case in which a sophisticated causal analysis, deployed without explicit attention to plumbing factors, produced its predicted effect at scale and over time. The literature is heavy with the opposite — careful analyses whose deployment effects fell well short of the predictions because plumbing was not attended to. I have not seen the reverse. I would update if shown a robust case in which plumbing genuinely did not matter.

**Still puzzling:** Whether the discovery of leaks can ever be substantially automated. The current state requires people who go and look. Automated detection of deployment anomalies — the recommendation was issued but the action was not taken, the metric did not move as projected, the population that received the intervention does not match the population the intervention was designed for — is technically feasible and is part of any serious deployment monitoring. But the *interpretation* of the anomaly, the decision about which plumbing fix to apply, currently remains human work. I do not yet know how much of this can be automated and how much will remain irreducibly human.

---

**Tags:** Esther Duflo economist as plumber AEA Ely lecture, Tangier piped water hassle costs, Ne-FMS MGNREGA fund flow reform, plumber's objection to causal AI, distance between causal estimate and organizational change
