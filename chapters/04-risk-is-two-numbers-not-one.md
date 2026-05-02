# Chapter 4 — Risk Is Two Numbers, Not One

*Probability, impact, and the foundational metric of decision intelligence.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *Risk Is Two Numbers, Not One*
- *The Heat Map That Hides the Decision*
- *Expected Value of Intervention as Foundation*

**TL;DR.** Almost every enterprise risk framework collapses probability and impact into a single score, which destroys the information the decision-maker actually needs. The Living Model's foundational metric, Expected Value of Intervention, restores the distinction — and makes visible the cases where collapsing them produces decisions a reasonable person would never make.

---

## August 1998, and the Math That Did Not Save LTCM

In August 1998, Long-Term Capital Management's portfolio began experiencing daily losses that the firm's risk model said should occur once in millions of years. Russia had defaulted on its sovereign debt. Investors fled to safety. Spreads on the trades LTCM was long blew out simultaneously, in correlations the model had not contemplated. Over the next four months, the fund lost approximately $4.6 billion. A consortium of banks, organized by the Federal Reserve, supplied roughly $3.6 billion in capital to prevent a forced unwind that would have damaged the broader market.

The model was not stupid. It had been built by Nobel laureates. It estimated daily Value at Risk — the maximum loss the portfolio would experience with 99% probability over a one-day horizon. The estimate was not, in any narrow sense, wrong. The trades had been designed around statistical regularities that, on the data available, supported the position sizes the fund was running.

What the model did not do was distinguish between the *probability* of a tail event and the *conditional impact* of that event. It collapsed them. The single VaR number — a dollar amount with a probability attached — gave the firm's principals a sense that the position size was bounded. The bound it described was the typical bad day. The position size was not sized to the catastrophic day, because the framework had no separate language for catastrophic-day impact distinct from typical-day impact. By the time the catastrophic day arrived, the position size that had looked manageable on the typical-day metric was the position size that took the fund down.

This is the failure mode this chapter is about. It is not a failure of mathematics. It is a failure of representation. When you collapse two numbers into one, you destroy the information the decision-maker needs to size the response.

LTCM is the iconic case. The same error operates, less spectacularly, every day in the risk frameworks running in commercial use across enterprises. They are not killing companies one at a time. They are quietly producing decisions a reasonable person, given the original two numbers, would not make.

---

## The Two Numbers

Risk has two components, and they are not interchangeable.

**Probability** is the likelihood that a given outcome occurs. It has units of probability — a number between 0 and 1, or a percentage, or sometimes an odds ratio.

**Impact** is the magnitude of the outcome conditional on its occurrence. It has units of whatever the outcome is measured in — dollars, lives, reputation points, market share. Impact is conditional. It is not "the expected loss"; it is "if this outcome occurs, what is the loss?"

The two numbers are independent in principle. A 1% probability of $1 billion loss has the same expected value as a 50% probability of $20 million loss. Both are −$10 million in expectation. The risk frameworks that summarize each as "$10 million in expected loss" or "low-medium risk score" treat them identically. A reasonable decision-maker does not.

The reason a reasonable decision-maker does not treat them identically is that the *distribution* of the outcome matters, not just the expected value. A 1% probability of $1 billion loss has a heavy tail and a high variance. A 50% probability of $20 million loss has a different distribution, different reversibility, different implications for capital adequacy, different implications for what you can do next. If you are running a $100 million company, the first risk could end the company in one bad outcome; the second risk is a manageable cost of doing business that occurs predictably. The expected values are equal. The decisions are not.

The collapse to a single score wipes out the information that distinguishes them.

---

## How Frameworks Encode the Error

Most enterprise risk frameworks in commercial use operationalize the collapse. The risk register is the canonical artifact. Each row names a risk; each row has a probability rating (1 through 5, or low/medium/high), an impact rating (1 through 5, or low/medium/high), and a "risk score" that is the product. The output is sorted by score. The high scores get attention. The framework instructs the decision-maker to attend to risks with high products, regardless of which factor — probability or impact — is doing the work.

A 5×5 heat map is the visualization. Probability on one axis, impact on the other, color coded by product. Red in the corner. Green near the origin. The visualization is intuitive. It is also the place where the error becomes structural.

Consider two cells in the heat map. The cell at (probability = 5, impact = 1) is "happens often, doesn't matter much" — small repeating losses. The cell at (probability = 1, impact = 5) is "almost never happens, would be catastrophic if it did" — tail risk. Both cells have a product of 5. The heat map paints them the same color. The framework instructs you to treat them the same way.

A reasonable decision-maker would treat them differently. The first risk you absorb as a cost of doing business. The second risk you insure against, or accept only with capital reserves sized to survive it, or refuse altogether if the survival case requires capital you do not have. These are categorically different decisions. The heat map made them look identical.

The COSO ERM framework, ISO 31000, the FAIR model — each has its specifics, but the structural feature in common is that the operational primitive is a risk score, and the score is some function (often product) of probability and impact ratings. The frameworks have rich language for governance, accountability, ownership. They have impoverished language for the distinction this chapter argues is foundational.

The collapse is not always to a literal product. Sometimes it is a verbal summary — "high risk" — that has the same effect: the decision-maker is no longer in possession of the two numbers, only of a label. The label is what propagates upward. The board sees the label. The action is taken against the label. The information that would have changed the decision was lost three steps before.

---

## Expected Value of Intervention

The Living Model's foundational metric is **Expected Value of Intervention**, or EVI. It is defined as the difference between the expected outcome under a deliberate intervention and the expected outcome under the counterfactual where the intervention is not taken:

**EVI(X) = E[Y | do(X)] − E[Y | do(¬X)]**

Two things to note about this definition.

First, both terms use the *do*-operator. EVI is not asking "what is the expected outcome conditional on observing X?" — that's the observational distribution that Chapter 2 argued is the wrong distribution for decision-making. EVI is asking "what is the expected outcome when X is set by intervention, compared to the expected outcome when X is not, also set by intervention?" Both arms are interventional. The comparison is between two worlds, both of which are produced by deliberate action (or deliberate inaction).

Second, EVI does not collapse probability and impact. The expectation E[Y | do(X)] is computed as a sum over outcome states: sum over each possible outcome state s of (probability of outcome s under intervention X) × (value of outcome s). When the distribution is heavy-tailed — when the impact of catastrophic states is qualitatively different from the impact of typical states — the expectation is computed against the *actual distribution*, not against a collapsed score. The decision-maker sees the distribution, not just the mean.

EVI alone is still a single number. Its virtue is not that it solves the collapse. Its virtue is that it is computed *from* the underlying distributions and exposes them. A Living Model that ranks interventions by EVI also presents the variance, the tail behavior, the conditional impact in tail states, the assumed structural model. The decision-maker is asked to choose among interventions with full visibility into both numbers, not collapsed scores.

---

## The Worked Comparison

Two interventions, same expected value. The Living Model presents them so a decision-maker can see why they are not the same decision.

**Intervention A.** Probability of bad outcome: 50%. Impact if bad outcome occurs: $20 million loss. Expected loss: $10 million. Worst-case loss in any 12-month period: $20 million.

**Intervention B.** Probability of bad outcome: 1%. Impact if bad outcome occurs: $1 billion loss. Expected loss: $10 million. Worst-case loss in any 12-month period: $1 billion.

A risk framework that scored both as "$10 million expected loss" or "medium risk" would call them equivalent. They are not.

For a $100 million company, Intervention B can end the company in one outcome. Intervention A produces a $20 million loss with even probability — unpleasant, manageable, recoverable. The decisions have different shapes:

- *For Intervention A*, you assess whether you are willing to absorb the expected $10 million as a cost of operations. If you proceed, you should plan for the $20 million bad outcome to occur, because at 50% probability it will occur. You size capital reserves to absorb that. You can run this intervention repeatedly and the law of large numbers protects you over time.

- *For Intervention B*, the expected value calculation is misleading. Most years, the loss is zero. One year in a hundred, the loss is catastrophic. The law of large numbers does not protect you in any individual year. The decision is whether your firm can survive the 1% case at all. If survival requires capital reserves that exceed what your firm has, the EV calculation is irrelevant — the intervention is unviable on capital adequacy grounds even though its expected value is identical to A's.

Same expected value. Different decisions. The Living Model shows both numbers separately, computes EVI from the underlying distribution, and surfaces the tail behavior alongside the mean. The collapse to one number — whether by literal product, by heat map color, by verbal summary — would have hidden the distinction.

---

## The Heat Map's Specific Failure Mode

The 5×5 heat map deserves a paragraph of its own because its specific failure is structural and not appreciated.

When risk frameworks score probability and impact on 5-point ordinal scales, they implicitly compress real distributions. A "5" on probability might mean ">50%" or might mean ">10%" depending on the framework. A "5" on impact might mean ">$10M" or ">$1B" depending on the framework. The framework's calibration is set by whoever designed it; the mapping from ordinal score to underlying number is rarely visible to the consumer of the heat map.

The compression destroys exactly the tail information the decision requires. The catastrophic risk — 1% probability of $1B impact — gets a probability score of 1 (low) and an impact score of 5 (catastrophic), product 5. The routine risk — 50% probability of $20M impact — gets a probability score of 5 (high) and an impact score of 1 (low), product 5. They occupy different cells in the matrix, but the framework treats their score as informationally equivalent.

A more careful framework would never collapse the impact axis at the high end. The difference between $100 million and $1 billion is the difference between "we can survive this and learn from it" and "we cannot survive this." Both round to a "5" on a 5-point scale. The scale is dishonest at the place where honesty matters most.

The Living Model's resolution is not to fix the heat map. It is to stop using it. EVI, computed against the actual distribution, with tail behavior surfaced as a separate output, gives the decision-maker the information the heat map was destroying.

---

## When the Two Numbers Should Drive Different Actions

Two final cases worth naming, because they make the point more concrete.

**Catastrophic-but-rare** (low probability, high impact): The decision is dominated by tail behavior. Standard expected value is a misleading metric. The relevant question is whether your firm or system can *survive* the tail case at all, and if so, at what capital cost. If the answer is no, the intervention is rejected even when its expected value is positive. Cost-benefit on the mean is not the relevant calculation. This is the LTCM case. It is also the case for most of the consequential risks an organization actually faces: the data breach that destroys the brand, the regulatory action that ends the product line, the supply-chain failure that blocks revenue for two quarters. None of these are typical-day events. The framework that scores them on a typical-day axis has nothing useful to say about them.

**Routine-and-frequent** (high probability, low impact): The decision is dominated by mean behavior. Standard expected value is approximately correct. The relevant question is whether the cumulative cost of the routine outcomes exceeds the cumulative benefit of the intervention. The law of large numbers protects you. You can run the intervention many times. This is the case for most operational risks — fraud chargebacks, customer service incidents, small-scale failures in process. The expected-value math works because the distribution is concentrated near the mean.

These two cases require different decision rules. A framework that produces the same risk score for both pretends they require the same rule. They don't.

---

## Why the Collapse Persists

The honest question to ask, given how clearly the collapse fails, is why it persists. The frameworks have been around for decades. The failures are well-documented. The math is not hard.

Three reasons, none of them flattering.

*The collapse is communicable.* A heat map fits on one slide. A distribution does not. A risk register sortable by score is legible to a board in twelve minutes. A distribution-based analysis is legible to a board in forty-five minutes that they do not have. The collapse is a compression for an audience that has been trained on the compression. Replacing it requires retraining the audience, which requires a decision-maker willing to spend the political capital to insist on it.

*The collapse is defensible.* If the catastrophic outcome occurs, the risk officer can point to the score they assigned. The score was within range. The decision was made on the basis of the score. The accountability surface is the score, not the underlying judgment about the distribution. Replacing the score with the distribution shifts the accountability surface — the risk officer is now responsible for a more nuanced statement, and a more nuanced statement is harder to defend in a postmortem. The collapse is, in part, an artifact of the way blame attaches to risk decisions.

*The collapse is professionally entrenched.* An entire industry of frameworks, certifications, and consulting practices is built on the collapse. ISO 31000, COSO ERM, FAIR — each has a community of practitioners whose careers are built on the framework as constituted. The replacement is not just a methodological change. It is a change in the standard of practice. Standards of practice change slowly. The path of least resistance is to leave them where they are.

These reasons do not make the collapse correct. They explain why it is sticky.

---

## The Foundational Metric

The Living Model's commitment to Expected Value of Intervention as a foundational metric is not an aesthetic choice. It is the formal mechanism that prevents the collapse this chapter has been describing.

EVI is computed from the underlying distributions, not from collapsed scores. It compares the interventional outcome to the counterfactual, not to a baseline forecast. It is presented alongside the distributional features — variance, tail behavior, conditional impact in tail states — that determine whether the mean is the relevant statistic. It refuses, structurally, to be a single-number summary that hides the decision.

What a Living Model gives an executive is the EVI of each intervention under consideration, with its uncertainty bounds and its distributional features visible. What it does not give them is a heat map. The heat map is the artifact of the collapse. The collapse is the failure mode.

This is the foundational metric for the same reason the Ladder of Causation is the structural spine. Both refuse to collapse distinctions that the decision requires. The Ladder refuses to collapse association into intervention. EVI refuses to collapse probability into impact. Each is a structural commitment to the kind of clarity that decision-making under uncertainty actually requires.

---

## Looking Forward

Part Two of this book opens with Pearl's Ladder — the formal apparatus for the association/intervention/counterfactual distinction that has been the spine of Part One. With the Ladder in place, Chapters 6 through 9 build the mathematical machinery: causal graphs, structural causal models, Markov equivalence, the backdoor criterion, double machine learning, the abduction-action-prediction procedure for counterfactuals. Chapter 10 names what the apparatus cannot do — the limits of observational data, the latent confounder problem.

Part One has now made its case. Three decades of analytics have produced sophisticated hindsight and very little foresight. The reasons are structural: descriptive analytics cannot distinguish observation from intervention; predictive models trained on the past cannot reliably predict consequences of changing the past; "real-time" claims paper over latency failures that destroy the speed they advertise; risk scores collapse the two numbers that decisions actually require.

The mathematical apparatus of Part Two is the alternative. The architecture of Part Three is what gets built when the apparatus is operationalized. Both rest on the diagnosis Part One has now completed.

---

**What would change my mind:** A risk framework in commercial use that explicitly resists the collapse — that scores probability and impact separately, refuses to produce a single score, and surfaces tail behavior as a first-class output. The framework would be honest in the way I am arguing for. I have not seen one in mainstream commercial use. I would update on a single rigorous example.

**Still puzzling:** How to communicate the two-number nature of risk to a board that has been trained on heat maps for fifteen years. The framework is wrong; the visualization is intuitive; the audience is calibrated to it. I do not have a confident answer to the change-management problem and I suspect that is partly why heat maps persist.

---

**Tags:** Long-Term Capital Management 1998, Expected Value of Intervention, COSO ERM heat map failure, probability-impact collapse, fat-tail risk decision frameworks
