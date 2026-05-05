# Chapter 2 — The Map That Doesn't Move

*Why predictive models fail under intervention.*

**Nik Bear Brown**

---

## Learning Objectives

By the end of this chapter, you should be able to:

1. **Distinguish** between the observational distribution and the interventional distribution, and explain why the difference matters for organizational decisions.
2. **Diagnose** a given business analytics case to determine whether the model in use is observational or interventional — and what failure modes each creates.
3. **Apply** the *do*-operator logic to a real decision scenario: identify whether intervening on a variable will sever relationships that exist in the historical data.
4. **Identify** the three mechanisms — concept drift, covariate shift, and intervention — by which a deployment environment can diverge from a training environment, and distinguish them in practice.
5. **Critique** a predictive model recommendation using three diagnostic questions that surface intervention-specific failure risks.
6. **Explain** why the Innovator's Dilemma is primarily an incentive and time-horizon problem, not a perception problem — and what that implies for how a Living Model must be built.

**Prerequisites:** Chapter 1 (the silent failure mode in organizational analytics; the Living Model architecture overview). Comfort with conditional probability notation — P(Y | X) means "the probability of Y given that X is observed" — is assumed. You do not need to know calculus or linear algebra. You do need to be willing to think carefully about what "given that" actually means.

**Why this chapter matters:** Every recommendation a predictive model makes is, at bottom, a claim about what will happen when you act. This chapter is about why that claim is often wrong in ways the model will never announce — and how to catch the failure before it costs $381 million.

---

## The Algorithm That Bought Too Many Houses

On November 2, 2021, Zillow announced it was winding down Zillow Offers, the iBuying business it had built around an algorithmic pricing model. The Q3 2021 loss for the segment was $381 million. The company had bought roughly 27,000 homes through the program; thousands of those were now flagged for sale at prices below what Zillow had paid. About a quarter of Zillow's workforce — around 2,000 people — would be laid off.

<!-- → CHART: timeline of Zillow Offers program — horizontal axis 2018–2022; annotated events: program launch, peak buying volume (Q2–Q3 2021), wind-down announcement (Nov 2, 2021), Q3 loss reported ($381M), layoff announcement — student should see the gap between when buying scaled and when the loss materialized, illustrating the lag between intervention and observable consequence -->

The failure that produced this outcome is worth understanding precisely, because it is not the kind of failure people usually imagine when they imagine algorithms going wrong. The model was not buggy in the way a broken spreadsheet formula is buggy. It had been built carefully, validated against historical data, and deployed at scale. For several years it had worked: it estimated the price at which Zillow could buy a home today and resell it three to six months later, accounting for renovation costs and anticipated market movement. The estimates were, by machine learning standards, accurate.

Then they stopped being accurate.

The model had been trained on a world in which Zillow was a small participant — observing prices, occasionally buying a home, mostly watching. The training data captured the relationship between home features, neighborhood signals, market conditions, and three-to-six-month resale prices *given that participants were behaving the way they had always been behaving*. When Zillow began buying tens of thousands of homes, it stopped being a participant and started being an actor. Its purchases moved local prices. Its inventory backed up. The renovations took longer than the model had assumed, in part because Zillow's own demand for contractors was driving up labor costs. The model had no language for any of this. It kept generating estimates with the same confidence intervals that had served it well when its predictions did not have to survive the consequences of its own decisions.

What failed was not the math. What failed was an assumption buried so deep inside the system that almost no one had thought to name it: the assumption that the world the model would operate in was statistically similar to the world the model had been trained on. When Zillow scaled its buying, it changed the world the model was trying to predict.

This is not a Zillow story. It is the structure of nearly every consequential failure of predictive analytics in organizational decision-making. And it is precisely the failure mode that the Living Model architecture is designed to surface.

The rest of this chapter explains why — carefully, from first principles.

---

## Concept 1: Two Distributions That Look Identical and Behave Differently

### The Setup: What a Predictive Model Actually Does

Every predictive model — every one, regardless of how sophisticated — does the same fundamental thing. It looks at a historical record of situations and outcomes. It fits a function that captures the relationship between the features of each situation (the inputs) and the outcome that followed (the output). Then it uses that function to predict outcomes in new situations it hasn't seen before.

The implicit promise is: *the new situations are like the old situations, in the ways that matter*.

When that promise holds, the model is useful. When it breaks, the model becomes confidently wrong — which is worse than being uncertain, because confidence suppresses the doubt that might prompt you to check.

<!-- → TABLE: simple two-column table — left column: "What the model sees (training data)," right column: "What the decision requires (deployment scenario)" — rows for: data source, who controls X, relationship between X and Y, model's confidence — student should see that the model's confidence is constant even as the underlying validity shifts -->

The technical language for what the model fits is the **observational distribution**: $P(Y \mid X)$. Read this as: "the probability of outcome $Y$, given that we observe input $X$ in the data." The vertical bar "|" means "given that." Everything to the right of it is a condition that was *observed* in the historical record — not chosen, not set by fiat, just present.

Now here is the question that the model cannot answer for you: what is the probability of $Y$ when you *decide* to make $X$ happen?

That is a different question. Radically different. And the answer can be radically different too.

### The Interventional Distribution

The statistician and computer scientist Judea Pearl introduced a notational tool to mark this distinction precisely. He called it the **do-operator**.

$P(Y \mid X = x)$ — the observational distribution — reads: "the probability of $Y$ given that we happen to observe $X = x$ in the data."

$P(Y \mid do(X = x))$ — the interventional distribution — reads: "the probability of $Y$ given that we intervene to *set* $X$ to the value $x$."

The *do* is doing real work here. When you intervene to set $X$, you sever the connections between $X$ and whatever was causing $X$ in the historical data. Before the intervention, $X$ was being determined by some mix of upstream factors — the state of the market, the behavior of customers, the decisions of prior managers. Those factors influenced both $X$ and $Y$, which is why the historical data shows a correlation between them. When you intervene to set $X$ by fiat, you take control of $X$ away from those upstream factors. They can no longer influence $X$ through their usual channel. They can still influence $Y$ through other paths. But the $X$-mediated path is now yours.

<!-- → INFOGRAPHIC: causal graph with two panels — left panel: "Before intervention" — upstream variable U has arrows pointing to both X and Y; X has arrow pointing to Y — label as "observational regime: U influences X influences Y, and U also influences Y directly." Right panel: "After do(X)" — arrow from U to X is severed; X still has arrow to Y; U still has direct arrow to Y — label as "interventional regime: you control X; U can no longer reach Y through X" — student should see that the correlation between X and Y in the left panel does not equal the causal effect of X on Y in the right panel -->

This is the formal version of what happened to Zillow. In the historical data, the relationship between Zillow's offered price and subsequent resale value was real. It reflected the genuine structure of the housing market under the conditions that had prevailed. But when Zillow intervened at scale — setting its purchase volume by fiat, at a level the market had never seen from a single buyer — it severed the connection between the market's usual pricing mechanism and what Zillow was doing. The model's prediction was about $P(\text{resale price} \mid \text{purchase price})$. The decision required $P(\text{resale price} \mid do(\text{purchase price at this volume}))$. Those are not the same distribution.

### Why This Looks Like a Small Distinction and Isn't

The challenge for organizations is that the observational and interventional distributions are numerically identical under one specific condition: when the thing you are intervening on has no upstream causes that also affect the outcome. If $X$ is not caused by anything that also causes $Y$ — if there are no common causes, no confounders, no selection effects — then observing $X$ and setting $X$ are equivalent, and the predictive model's estimate is also a causal estimate.

This condition is rarely met in organizational data. Organizational data is the record of humans making decisions in response to circumstances. The circumstances are not random. The decisions are not random. The people involved have preferences, incentives, and histories that shape both what they do and what happens as a result. That is confounding, everywhere, always. And confounding means that the observational distribution and the interventional distribution diverge.

When they diverge, the model does not alert you. From the model's perspective, the deployment data is just new data. If it resembles the training data, the predictions look calibrated. If it doesn't, they degrade silently. This is the silent failure mode from Chapter 1, appearing now in its most common organizational form.

### Worked Example: The Pricing Team

Here is a stripped-down example with numbers you can verify.

A pricing team is analyzing customer retention. They observe that customers who pay higher prices for the company's software also have higher one-year retention rates. The correlation in the historical data is positive and statistically significant: correlation coefficient 0.4, $p < 0.001$, across 50,000 customer accounts. The team recommends raising prices, reasoning that higher prices are associated with better retention.

**Step 1: Identify what the model is fitting.**

The model is fitting $P(\text{retention} \mid \text{price})$. This is an observational relationship — it describes what happened in the historical data when customers happened to pay various prices.

**Step 2: Ask what the decision actually requires.**

The decision is to raise prices across all customers by 15%. This is an intervention. The decision requires $P(\text{retention} \mid do(\text{price} + 15\%))$. Is that the same distribution as the observational one?

**Step 3: Look for common causes.**

Why did some customers pay higher prices historically? The company sells to two segments: small businesses (price-sensitive, buy fewer features, churn more readily) and enterprise customers (price-insensitive, buy more features, have higher switching costs because they've integrated the product deeply into their workflows). Enterprise customers paid more because they bought more. They retained better because leaving is expensive for them. **Segment** is a common cause — it drives both the price paid and the retention observed. In the observational distribution, segment is doing the work that the correlation is crediting to price.

<!-- → CHART: two-panel scatter plot — left panel: pooled data showing positive correlation between price and retention; right panel: the same data separated by segment — enterprise cluster (high price, high retention) and small business cluster (low price, low retention) — no slope within either cluster — student should see that the pooled correlation is entirely explained by segment composition, not by price itself -->

**Step 4: Apply the do-operator.**

When you intervene to raise prices across all customers, you set $X$ by fiat — you sever the connection between segment and price. Both small businesses and enterprises now face the new, higher price. Segment still influences retention (switching costs haven't changed), but it no longer determines who pays what. The price increase now operates directly on customers who are price-sensitive — the small business segment — and triggers churn in exactly the customers the model had been treating as "low-price, moderate-retention" accounts.

**Step 5: What actually happens.**

The intervention drives a substitution within the small business segment: some customers who would have stayed at the old price cancel at the new price. Retention in the enterprise segment is roughly unchanged (switching costs still dominate). The aggregate retention rate falls. The model predicted it would rise.

The model was not wrong about the historical data. The historical correlation was real. The model was wrong about the intervention because the historical correlation was produced by segment composition, not by a causal mechanism that price controls.

**The lesson the example demonstrates:** A positive observational correlation between an action and an outcome does not imply that taking the action will produce more of the outcome. It implies only that situations in which the action was high were also situations in which the outcome was high. Whether those situations will persist after you intervene to make the action high everywhere is a separate question — one the observational model cannot answer.

---

## Concept 2: The Three Ways Deployment Diverges from Training

### Not All Failures Are the Same

The Zillow failure and the pricing-team failure are examples of a single class of problem: the deployment world diverges from the training world, and the model doesn't know it. But there are three distinct mechanisms by which this divergence can occur, and they require different responses. Conflating them — which organizations routinely do — leads to misdiagnosis and therefore to mistreatment.

The three mechanisms are **concept drift**, **covariate shift**, and **intervention**. I'll define each, show what it looks like in practice, and then explain why the third is the one that the Living Model architecture is most specifically designed to handle.

### Concept Drift

Concept drift occurs when the underlying data-generating process changes — when the relationship between inputs and outputs in the real world shifts, independent of anything you do.

A fraud detection model trained on transaction patterns from 2019 meets 2020, when the pandemic restructured purchasing behavior globally. The model learned to associate certain patterns with fraud — unusual merchants, unusual geographies, unusual times of day. Those associations reflected how legitimate customers had been behaving. When legitimate customer behavior changed structurally, many of the fraud signals became ambiguous, and the model's precision collapsed. The model didn't change. The world did.

Concept drift is the failure mode that most data science teams are trained to watch for. It is real, it is common, and it has a relatively straightforward (if expensive) response: retrain the model on more recent data, or build in continual updating so the model tracks the evolving process.

### Covariate Shift

Covariate shift occurs when the distribution of inputs changes, but the relationship between inputs and outputs is stable.

A credit risk model was trained on a borrower population that was roughly 80% traditionally employed and 20% self-employed. The lending business grows into a new market segment, and the new applicant pool is 50% self-employed. The model's mapping from applicant features to default probability is still valid — the underlying causal structure of credit risk hasn't changed — but the model is now seeing many more applicants in a region of input space where it had relatively little training data. Its predictions in that region may be poorly calibrated, not because the world changed but because the sample it learned from was unrepresentative.

Covariate shift is also manageable, though the management requires more sophistication than concept drift: importance weighting, targeted data collection in underrepresented input regions, or explicit uncertainty flags on inputs that are far from the training distribution.

### Intervention

Intervention is the mechanism this chapter has been about, and it is the one most organizationally consequential precisely because it is the least recognized.

Intervention occurs when someone — typically the model's deployer — deliberately sets a variable that the model had been treating as an exogenous input. The model had learned a relationship between that variable and the outcome. But the relationship it learned was the observational relationship — the one that held when the variable was being determined by its usual upstream causes. When you set the variable by fiat, you change who controls it and at what level, and the observational relationship no longer applies.

The key distinction from concept drift is *agency*. Concept drift happens to you. Intervention is something you do. The failure mode in concept drift is that you didn't update fast enough. The failure mode in intervention is that you didn't ask the right question before you acted.

<!-- → TABLE: three-row comparison table — columns: mechanism, trigger, symptom, appropriate response — rows: concept drift (world changes on its own / model accuracy degrades uniformly / retrain), covariate shift (input distribution shifts / model uncertain in new input regions / reweight or collect data), intervention (deployer sets a variable by fiat / model confident but systematically wrong / causal re-analysis before acting) — student should be able to classify a given failure into one of these three rows -->

The Living Model architecture's handling of intervention is the subject of Part Three. For now, the essential point is that a model can be current, well-calibrated on recent data, not suffering from concept drift or covariate shift — and still be deeply wrong about the consequence of an intervention, because the intervention severs relationships the model took for granted.

### Worked Example: The Email Campaign

A marketing team is deciding whether to increase email send frequency to improve conversion rates. They have a model trained on twelve months of customer email interaction data. The model shows that customers who receive more emails convert at higher rates. Correlation: 0.35 across 200,000 customer accounts.

**Diagnose the failure mechanism before the team acts.**

*Is this concept drift?* No — the team hasn't changed anything yet. The model is being applied to roughly the current customer population with roughly current email response patterns. The world hasn't shifted.

*Is this covariate shift?* Possibly at the margins, if the team is expanding into new customer segments. But the primary question is about frequency, not about a new input region.

*Is this an intervention risk?* Yes. The observational correlation between frequency and conversion is likely to be confounded by engagement level. Customers who receive more emails may convert more not because the emails caused conversion but because engaged customers both open emails (so they receive more, because automated systems increase frequency to engaged users) and convert more (because they're engaged). When the team intervenes to raise frequency for *all* customers — including the unengaged ones — the mechanism that produced the observational correlation no longer applies to the newly targeted group. The intervention may increase unsubscribe rates in the low-engagement segment while producing marginal conversion lift in the high-engagement segment that was already converting.

**The diagnostic move:** Before recommending the intervention, ask whether the correlation would hold if frequency were set at the new level for customers who had historically been at the low end of the frequency distribution. The observational data does not answer this directly. It requires either a causal re-analysis (identifying and controlling for engagement as a confounder) or a controlled experiment (randomly assigning frequency increases to a subset of low-engagement customers and measuring the effect). The model, consulted alone, will not flag this. You have to know to ask.

---

## Concept 3: The Innovator's Dilemma as a Time-Horizon Problem

### The Popular Reading and What It Misses

Clayton Christensen's *The Innovator's Dilemma* is one of the most influential business books of the last thirty years. The thesis, in brief: incumbent firms with strong market positions systematically fail to respond to disruptive entrants because their management processes — sustaining innovation, customer focus, financial discipline — are optimized for their current customers and current profit margins, not for the low-margin, lower-performance entrants that eventually displace them from below.

The popular reading of this argument is that incumbents are *blind* — they cannot see the disruption coming. The management lesson drawn is: watch the low end, listen to non-consumers, and don't let your best customers' voices drown out the signals from the fringe.

This reading underestimates the incumbent executives. Most of them see it. The Kodak engineers built a functioning digital camera in 1975 — twenty years before digital disrupted film. Blockbuster's executives were offered Netflix for $50 million in 2000 and declined. Intel's leadership watched ARM processors eat their market from below, segment by segment, for more than a decade before the process became irreversible. The seeing was not the problem.

### The Reframe: Incentives and Time Horizons

The Innovator's Dilemma is primarily a problem of organizational incentives and time horizons, not of perception. This reframe matters because it changes what a Living Model has to do.

Consider the position of an executive who genuinely sees the disruptive entrant coming. She is running a profitable, large business. Her compensation is structured around the next four to eight quarters of revenue, margin, and earnings per share in that business. The incumbent product line is performing well by every metric her board tracks. The disruptive entrant is small, growing, and currently unprofitable; it is serving customers at a price point that would destroy her margins if she matched it; it will not represent a material threat to her core market for — by most estimates — three to five years.

To respond — to begin cannibalizing her own product, to invest resources in a thin-margin offering for non-consumers, to redirect engineering toward a market that will not produce meaningful revenue within her current compensation horizon — she would have to take actions that hurt her measured performance now, in service of an outcome that may not arrive during her tenure.

Her choice is not "see the disruption and act" versus "miss the disruption and fail." Her choice is "act on the disruption and absorb a personal cost measured in quarterly numbers" versus "manage the disruption optically — acquire something, announce an initiative, issue a press release — and let the next CEO deal with the structural response." The rational, incentive-aligned choice is the second one. Most executives make it. The system is working as designed.

<!-- → CHART: timeline diagram — horizontal axis is time; two curves — incumbent's performance (high, declining slowly) and entrant's performance (low, growing); vertical line at "disruption horizon" where curves cross; second vertical line earlier at "executive incentive horizon" — student should see that when the incentive horizon is shorter than the disruption horizon, the rational move is to defer; when the disruption horizon compresses to match the incentive horizon, the rational move flips -->

### What This Means for the Living Model

This reframe is not just strategically interesting. It is structurally important for how the Living Model has to be built.

A model that captures only the technological substitution dynamic — the entrant's performance trajectory improving relative to the incumbent's threshold requirements — can predict displacement but offers no traction on whether the incumbent will respond. The executive's behavior, analyzed through the lens of the observational data, looks like strategic blindness. Analyzed through the lens of the incentive structure, it is rational. A model that does not capture the incentive structure, the executive's effective time horizon, the board's penalty function for quarterly margin compression, and the analyst consensus expectations — that model cannot distinguish "didn't see it coming" from "saw it coming and rationally deferred."

This is, precisely, the difference between the observational distribution and the interventional distribution applied to strategy. The observational data says: incumbents who face disruptive entrants tend not to respond aggressively. The interventional question is: if we *change* the incentive structure — redesign the compensation, restructure the board's evaluation criteria, create a separately funded innovation unit with its own P&L — what happens then? The observational correlation between "facing disruption" and "not responding" is not the causal story. The causal story runs through incentives, and the Living Model has to model that.

### The Compression Is the News

Christensen's classic cases — disk drives, integrated steel mills, department stores — operated on disruption horizons of five to seven years. The executive who chose to defer in 1990 was making a bet that she had at least four or five years before the deferral became indefensible. In most cases, she was right.

The compute, retail, and SaaS disruptions of the 2010s ran on horizons of eighteen to thirty months. The AI-native disruption arriving as I write this appears to be running on disruption horizons of six to twelve months in the industries closest to the frontier. The compression is the news. An executive whose incentive horizon is four quarters and whose disruption horizon used to be five years had time to defer. An executive whose incentive horizon is four quarters and whose disruption horizon is now twelve months does not.

<!-- → CHART: small multiples or single annotated chart showing disruption horizon length across eras — disk drives / steel mills (5–7 years), SaaS / e-commerce (18–30 months), AI-native (estimated 6–12 months) — student should see the compression trend and its implication for the gap between incentive horizon and disruption horizon -->

The Living Model, built right, includes the time-horizon dimension as a first-class input — not a footnote. It does not simply rank interventions by expected causal effect. It ranks them by expected causal effect *given the horizon over which that effect must arrive to be actionable within the current incentive structure*. And it makes visible — explicitly, to whoever is asking — what the trajectory looks like under the current incentive structure versus a hypothetical structure aligned with the actual disruption timeline.

This is an uncomfortable output to generate. It tells a board, sometimes, that the reason their company is not responding to the disruption is the way they have designed their CEO's compensation. The Living Model's job is not to make organizational politics comfortable. It is to make the actual causal story legible.

### Worked Example: The Incumbent's Decision Tree

Suppose you are advising the leadership team of a mid-sized professional services firm. The firm has been highly profitable for fifteen years, building revenue on a model of senior-expert-delivered consulting work. A new category of AI-native competitors has emerged. They deliver comparable analysis at 20–30% of the cost, with turnaround times measured in hours rather than weeks. The AI-native firms are currently serving the low end: smaller clients, simpler analyses, lower-stakes engagements.

Your task is to help the leadership team decide whether and how to respond. You have a predictive model of market share dynamics trained on historical disruption cases. The model says: firms in this position that do not restructure their service model lose 35–55% of revenue over five years.

**Diagnose the intervention risk in the model's recommendation.**

*What is the observational distribution?* Historical disruptions where firms did or did not restructure, and the resulting revenue trajectories.

*What is the intervention being proposed?* Restructuring the service model — reducing headcount of senior consultants, investing in AI tooling, redesigning the delivery process.

*What relationships does the historical data capture that may not hold after the intervention?* The historical data includes disruptions in which the restructuring decision was made by leadership teams with varying degrees of board support, capital availability, and organizational coherence. The model treats "did restructure" as a binary input. But in the causal structure, the firm's ability to restructure successfully is determined by factors — board alignment, cash position, cultural willingness to cannibalize — that also determine the revenue trajectory. The model's estimate of "restructuring firms retain revenue" is conflated with "firms that were organizationally capable of restructuring retain revenue."

*What does the do-operator require you to ask?* If you intervene to set "restructure = yes" for this firm, does the causal mechanism that produced favorable outcomes in the historical restructuring cases also apply here? Only if this firm has the organizational capability that successful historical restructurers had. The model does not measure that. You have to.

**The lesson:** The model can tell you what restructuring has historically been associated with. It cannot tell you what will happen if you force this specific firm to restructure without addressing the organizational capability gap. That is an interventional question the observational data cannot answer — and the Living Model's architecture makes this gap explicit rather than papering over it with a regression coefficient.

---

## Integration: The Three Questions Before You Trust a Model

The three concepts in this chapter fit together as a diagnostic sequence. Before acting on any model recommendation with significant organizational consequences, three questions surface the failure mode this chapter has described:

**Question 1: Was your training data generated under a regime in which actors like you were intervening at the scale you are now considering?**

If Zillow had asked this in 2019, the answer would have been no. The training data captured a world where iBuying represented a small fraction of local transaction volume. Scaling to meaningful market share was a regime the data did not contain. The *do*-operator at that scale had not been exercised. The model had no information about it.

**Question 2: What is the conditional distribution of your action, given the other variables in the data? Is your action confounded?**

If the variable you are about to set was, in the historical data, substantially determined by some other variable that also affects the outcome, then the observational correlation between your action and the outcome is at least partially that other variable's doing. When you set your action by fiat, you sever the connection between that other variable and your action — and the observational correlation does not survive the severance.

**Question 3: If you ran the model's recommendation as an intervention, what would the world look like two, three, four periods later — and are those subsequent states inputs to the next decision your model will make?**

This is the feedback loop question. A model recommendation that is correct in period one can set the system into a trajectory that makes the model wrong in period two and dangerous in period three. The Zillow model was not just wrong about resale prices. It was wrong in a way that created inventory pressure, contractor shortages, and local market distortion — all of which fed back into the inputs the model would use for its *next* round of purchase recommendations. The model had no feedback path. The Living Model's continual updating is, in part, a structural response to this question: the only honest answer to "what does the world look like after my intervention" is "let's see the updated data and re-examine."

<!-- → INFOGRAPHIC: three-panel decision flowchart — titled "The Intervention Diagnostic" — Panel 1: "Is your scale represented in the training data?" → yes/no branching → "no" leads to caution flag; Panel 2: "Is your action confounded by a common cause?" → yes/no → "yes" leads to caution flag; Panel 3: "Does the recommendation create feedback into future model inputs?" → yes/no → "yes" leads to caution flag — student should walk through this with a real case and identify which panels light up -->

A model that cannot be interrogated with these three questions is not necessarily wrong. It is operating in a regime where it cannot tell you whether it is wrong. That is the Zillow position. The three questions don't certify the model. They surface where the certification cannot go — and that is where the human decision node has to take over.

### Putting It All Together: A Full Case Walk-Through

A telecommunications company is considering a major pricing restructuring. The analytics team has built a model predicting churn as a function of price, contract length, and service tier. The model recommends a bundle-and-discount strategy: offer customers a three-year lock-in at a 20% discount from current rates. Historical data shows that customers on longer contracts have lower churn rates (observational correlation: −0.45).

Walk the diagnostic:

*Training scale question:* Has this company ever offered this bundle to all customers simultaneously? No. Historical long-contract customers self-selected into long contracts — they were predisposed to stay. The model's estimate of "long contract → low churn" reflects the behavior of customers who wanted long contracts, not the behavior of all customers if forced into them.

*Confounding question:* What was causing customers to choose long contracts historically? Satisfaction, price sensitivity, switching cost tolerance — all of which also predict churn independent of contract length. This is confounding. When you intervene to *offer* long contracts to all customers, you don't change those underlying characteristics. The customers who sign the three-year contract under coercion from the discount may have lower satisfaction to begin with, and the contract won't change that.

*Feedback question:* What happens in year three when the lock-in expires for a cohort of customers who were satisfied with neither the price nor the service? They have now been suppressing their churn signal for thirty-six months. The model, trained on continuous churn data, will have no record of their dissatisfaction accumulating. When the contracts expire, the churn event will look sudden and unexplained. The model's next recommendation will have been trained on three years of artificially suppressed churn — making its churn predictions in year four systematically optimistic.

The intervention is not necessarily wrong. The discount may produce real value for customers who genuinely want a long-term relationship. But the model's numerical confidence about the churn reduction is not the confidence you should have. The Living Model has to make the distinction between the historical observational estimate and the interventional estimate explicit — and then track the actual churn outcomes as the intervention unfolds, updating the causal structure with what it learns.

<!-- → CHART: projected churn rate over four years under two scenarios — scenario A: bundling intervention applied, churn appears suppressed years 1–3, then spikes in year 4 at contract expiry; scenario B: no intervention, churn declines gradually — student should see that the intervention produces a better-looking metric in the short term and a worse outcome at contract expiry, illustrating how feedback loops make period-one predictions misleading -->

---

## Chapter Summary

Before this chapter, you could read a correlation and reach for an action. That is how most organizational analytics is used, and it is why most organizational analytics fails in the specific way this chapter describes.

After this chapter, you have two new capabilities.

The first is diagnostic: you can look at a model recommendation and ask whether the recommendation is based on an observational relationship or a causal one — and identify the specific mechanism (confounding, scale, feedback) that might cause them to diverge. You can classify a model failure as concept drift, covariate shift, or intervention, and you understand why intervention requires different treatment than the other two.

The second is applied: you can work through the three diagnostic questions with a real case, identify whether the model's training regime covered the proposed intervention, identify confounders that would be severed by the intervention, and identify feedback loops that would undermine the model's validity in subsequent periods. You can do this before the money is committed.

**The one idea that matters most from this chapter:** A predictive model is a description of what happened when the world was left to its own devices. The moment you intervene — at scale, by fiat — you are not asking the world to continue on its prior trajectory. You are asking it to do something new. The model does not know what new looks like.

**The common mistake to watch for:** Treating statistical significance as causal license. A statistically significant correlation in a large dataset is not evidence that the intervention will work. It is evidence that, historically, situations with high $X$ tended to also have high $Y$. Whether that will hold when you set $X$ to its new value depends on the causal structure — which the correlation alone cannot reveal.

**The Feynman test:** Close the book and explain to someone why $P(Y \mid X)$ is not the same thing as $P(Y \mid do(X))$, using an example from your own industry. If you can do that — if you can name the confounder, sketch the causal graph, and say what would happen differently in the interventional regime — you have genuinely understood this chapter.

---

## Exercises

### Warm-Up

**W1.** *(Objective 1 — Distinguish observational from interventional; Difficulty: Low)*

A hospital analytics team observes that patients who receive more frequent nursing check-ins have lower rates of post-operative complications. They recommend increasing check-in frequency for all surgical patients. Identify: (a) what the observational distribution captures in this data, (b) what the interventional distribution would need to capture if the recommendation were implemented, and (c) one confounder that could explain the correlation without implying a causal benefit of check-in frequency.

**W2.** *(Objective 4 — Classify divergence mechanism; Difficulty: Low)*

For each scenario below, identify whether the primary failure mechanism is concept drift, covariate shift, or intervention — and give a one-sentence justification.

(a) A model predicting employee attrition, trained pre-pandemic, is applied in 2022 after widespread remote-work normalization.

(b) A credit model trained on a regional customer base is deployed in a new international market with different income distributions but similar default mechanisms.

(c) A retailer's pricing model, trained on data from periods when it held 4% market share, is used to set prices after the retailer acquires two competitors and now holds 22% market share.

**W3.** *(Objective 3 — Apply do-operator logic; Difficulty: Low)*

In your own words, explain what the do-operator "does" to a causal graph. Specifically: when you apply $do(X = x)$, what happens to the arrows in the graph that were pointing *into* $X$? Why does this matter for predicting outcomes?

---

### Application

**A1.** *(Objective 2 — Diagnose model usage; Difficulty: Medium)*

A subscription software company observes that customers using five or more product features within their first 30 days have 3× higher two-year retention than customers using fewer. The growth team proposes an onboarding intervention: guide all new customers to activate five features within their first 30 days.

Walk through the intervention diagnostic. For each of the three questions (scale, confounding, feedback), write a specific answer for this case — not a generic statement but an answer anchored to the details of this scenario. Then give your overall assessment: is the model's historical correlation likely to hold under the intervention? What additional information would you want before recommending the campaign?

**A2.** *(Objective 5 — Critique a model recommendation; Difficulty: Medium)*

Return to the telecommunications company case from the Integration section. The analytics team pushes back: "You're being too theoretical. Our data shows the correlation is robust across all customer segments — small business, mid-market, and enterprise. The bundling strategy will work." Construct a specific two-paragraph response that (a) acknowledges the empirical finding without dismissing it, and (b) explains precisely why segment-level robustness of the observational correlation does not resolve the interventional concern.

**A3.** *(Objective 6 — Apply the Innovator's Dilemma reframe; Difficulty: Medium)*

An incumbent firm in a market facing AI-native disruption has a five-year revenue forecast that shows moderate decline. The forecast was built using a model trained on historical disruption cases. A board member argues: "The model shows firms that invested heavily in response to disruption retained market share. We should invest heavily." Apply the intervention diagnostic to the board member's argument. What is the observational relationship the model found? What would the do-operator question require you to ask? What confounders might explain the historical correlation between heavy investment and market share retention without implying that this firm's heavy investment will produce the same result?

**A4.** *(Objectives 1, 3 — Construct a causal argument; Difficulty: Medium)*

Choose a business decision from your own industry or professional experience — a pricing change, a hiring initiative, a marketing campaign, an operational restructuring. Write a two-paragraph analysis that (a) describes the observational relationship the decision appears to rely on, and (b) identifies the intervention risk: what is being set by fiat, what upstream causes are being severed, and what confounders might explain the observational relationship without surviving the intervention.

---

### Synthesis

**S1.** *(Objectives 2, 4, 5 — Integrate all three divergence mechanisms; Difficulty: High)*

A financial services firm has deployed a customer lifetime value (CLV) model to guide retention spending. The model was trained on five years of data and has been in production for eighteen months. The firm's head of analytics reports three simultaneous anomalies: (1) the model's predictions have been systematically too high for customers acquired in the last six months; (2) the model performs well for traditionally employed customers but poorly for freelance customers, who represent a growing share of new acquisitions; (3) a recent initiative that doubled retention call frequency for high-CLV customers produced no improvement in actual retention.

For each anomaly, identify the divergence mechanism at work and explain what corrective action it implies. Then explain why it matters that all three are happening simultaneously — what does that combination suggest about the reliability of the model's current recommendations?

**S2.** *(Objectives 1, 6 — Connect causal structure to strategic response; Difficulty: High)*

The chapter argues that the Innovator's Dilemma is primarily an incentive and time-horizon problem, not a perception problem. Suppose you are a strategy consultant advising a firm that has correctly identified a disruptive entrant and wants to respond. Write a brief memo (two to three paragraphs) to the board that: (a) explains why the observational data on historical responses to disruption may not reliably predict the outcomes of their specific intervention, (b) identifies the organizational variables that would need to be part of the causal model for the prediction to be valid, and (c) recommends one structural change to the firm's incentive design that would reduce the gap between the executives' effective time horizon and the disruption horizon.

---

### Challenge

**C1.** *(Objectives 1, 2, 3, 4, 5 — Stress-test the framework; Difficulty: High)*

The chapter's central argument is that predictive models trained on observational data cannot reliably predict the consequences of interventions at scales or in regimes not represented in the training data.

Construct the strongest possible counterargument. Under what conditions would an observational model's prediction *reliably* survive an intervention? Be specific — give the conditions, explain why they would preserve the observational-interventional equivalence, and then assess how frequently those conditions actually hold in the organizational analytics cases you are likely to encounter. Your response should engage seriously with the chapter's argument, not simply assert that the chapter is wrong.

**C2.** *(Objectives 4, 6 — Forward-looking application; Difficulty: High)*

The chapter notes that disruption horizons appear to be compressing — from five to seven years in Christensen's classic cases to possibly six to twelve months in some AI-native disruption cases as of this writing. If this compression is real and continues, what are the implications for how the Living Model's time-horizon parameter should be set and updated? Specifically: (a) how would you empirically estimate the disruption horizon for a given industry and competitive context, (b) what would the Living Model need to track continuously to detect when the disruption horizon is shortening faster than expected, and (c) what are the limits of modeling this — where does the horizon compression itself become unpredictable, and what does the model owe the decision-maker when it reaches that limit?

---

## Connections Forward

This chapter has established the conceptual distinction between observation and intervention, and given you a diagnostic for catching intervention failures before they become $381 million write-downs.

The next chapter takes on a different but related failure mode: the abuse of the word "real-time." Many of the failures this chapter described are latency problems in disguise — not just that the model used observation when it should have used intervention, but that the model was working from data that was already stale when the intervention began. "Real-time analytics" is one of the most heavily marketed and least honestly defined terms in enterprise software. Chapter 3 argues for replacing it with "continually updated" — which is a precise claim that can be tested — and shows what the infrastructure for genuinely continual updating actually requires.

Chapters 5 through 9 develop the full mathematical apparatus for moving from the observational distribution to the interventional one: Pearl's *do*-calculus, structural causal models, the backdoor criterion, and the conditions under which causal effects can be identified from observational data at all. Chapter 10 handles the case where the apparatus runs out — latent confounders that cannot be measured, the limits of observational data even when correctly analyzed.

The Zillow case will return in Chapter 8, where I use it as the anchor example for the full backdoor criterion analysis. You will be able to reconstruct, from first principles, exactly which variables would have needed to be controlled for the model's estimates to survive the scale-up — and why those variables were not in the data Zillow had available.

---

**What would change my mind:** A documented case in which a predictive model trained on observational data correctly anticipated the consequences of an intervention at a scale not represented in the training data, and where the prediction was robust to a sensitivity analysis that varied the assumed causal structure. I have not seen such a case in mainstream enterprise practice. I would update if shown one.

**Still puzzling:** The empirical question of where, exactly, the disruption horizon is settling for AI-native business models. I see the compression but cannot yet say with confidence whether the new equilibrium is twelve months, six months, or something shorter. The Living Model's horizon parameter cannot be fixed until that empirical question is answered, and I do not yet know how to answer it without more cases on the table.

---

*Tags: observational vs interventional distribution, do-operator, Zillow Offers failure, concept drift vs covariate shift, Innovator's Dilemma incentive structure, disruption horizon compression, causal inference MBA, Living Model architecture*

---

###  LLM Exercise — Chapter 2: The Map That Doesn't Move

**Project:** Build Your Own Living Model

**What you're building this chapter:** The intervention diagnostic on your chosen decision — a ranked list of the three most dangerous intervention risks (training-scale, confounding, or feedback-loop) that surface why your current historical data may fail to predict what happens after you act.

**Tool:** Claude Project (continue in the same one from Chapter 1).

---

**The Prompt:**

```
Continuing my Living Model project. My chosen decision and current data sources are documented in the Decision Dossier in the Project context.

This chapter teaches that a predictive model trained on observational data — P(Y|X) — is not the same as one that can predict the consequence of an intervention — P(Y|do(X)). The do-operator severs the upstream causes of X. Three diagnostic questions surface intervention failures before they happen:

1. SCALE — Was your training data generated under a regime in which actors like you were intervening at the scale you are now considering? (Zillow's failure was scaling iBuying to a level no participant in the training data had ever operated at.)

2. CONFOUNDING — What is the conditional distribution of your action given the other variables in the data? Is your action confounded by something that also drives the outcome? (J.C. Penney's failure was treating segment-driven correlation as price-driven causation.)

3. FEEDBACK — If you ran the recommendation as an intervention, what would the world look like two, three, four periods later — and are those subsequent states inputs to the next decision your model will make? (Zillow's purchases moved local prices, which fed back into the next round of estimates.)

For my chosen decision, walk through all three questions specifically. Don't give me generic advice — anchor every answer to my actual decision, the data sources I named in the Decision Dossier, and the realistic confounders and feedback paths in my domain. Name the specific upstream variables you suspect are doing the work that the observational data credits to my candidate intervention.

Then classify the most likely deployment-divergence mechanism if I were to act on the current data:
- Concept drift (world changed on its own)
- Covariate shift (new input regions)
- Intervention (I am setting a variable by fiat)

End with: a numbered list of the THREE most dangerous intervention risks for my decision, ordered by severity, and one specific data-collection or experimental move I could make this quarter that would meaningfully reduce risk #1.
```

---

**What this produces:** A three-question intervention diagnostic written specifically for your decision, a divergence-mechanism classification, and a ranked list of the three most dangerous intervention risks with one concrete next move on the top one.

**How to adapt this prompt:**
- *For your own project:* If you skipped Chapter 1, replace the Dossier reference with: "My decision is [X], my domain is [Y], my current data sources are [Z]."
- *For ChatGPT / Gemini:* Works as-is.
- *For Claude Code:* Not needed yet.
- *For a Claude Project:* Just paste — the Project carries Chapter 1's Decision Dossier automatically.

**Connection to previous chapters:** Builds directly on the decision and data sources defined in Chapter 1. Without that anchoring this exercise produces generic answers.

**Preview of next chapter:** Chapter 3 audits the *time dimension* of the systems you currently use for this decision — whether they are continually updated in any meaningful sense, and which of three latencies (data, model, decision) is the actual bottleneck.

---

## 🕰️ AI Wayback Machine

The ideas in this chapter didn't appear from nowhere. **Trygve Haavelmo** was publishing *The Probability Approach in Econometrics* in 1944 to argue that observational econometric models cannot, by themselves, support claims about deliberate intervention decades before most people had heard of predictive models breaking under intervention. Here's a prompt to find out more — and then make it better.

![Trygve Haavelmo, c. 1950s. AI-generated portrait based on a public domain photograph (Wikimedia Commons).](images/trygve-haavelmo.jpg)
*Trygve Haavelmo, c. 1950s. AI-generated portrait based on a public domain photograph.*

**Run this:**

```
Who was Trygve Haavelmo, and how does his 1944 distinction between observational fit and intervention-supporting structure connect to why predictive models break when used to recommend deliberate action? Keep it to three paragraphs. End with the single most surprising thing about his career or ideas.
```

→ Search **"Trygve Haavelmo"** on Wikipedia after you run this. See what the model got right, got wrong, or left out.

**Now make the prompt better.** Try one of these:

- Ask it to explain the *autonomy of structural equations* in plain language, as if you've never seen an economic model
- Ask it to compare Haavelmo's structural-vs-reduced-form distinction to today's predictive-vs-causal model debate
- Add a constraint: "Answer as if you're writing the warning label on a forecast that will be used to set policy"

What changes? What gets better? What gets worse?

