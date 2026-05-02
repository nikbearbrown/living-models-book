# Chapter 2 — The Map That Doesn't Move

*Why predictive models fail under intervention.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *The Map That Doesn't Move*
- *When Strategy Meets a Model Trained on the Past*
- *The Innovator's Dilemma Is a Time-Horizon Problem*

**TL;DR.** A predictive model is a map of a world generated under specific conditions; the moment you intervene at scale, the world the map describes ceases to exist. The Innovator's Dilemma is not primarily about causal blindness — it's about an incentive structure that rewards the quarterly action while the disruption operates on a horizon that is itself getting shorter.

---

## The Algorithm That Bought Too Many Houses

On November 2, 2021, Zillow announced it was winding down Zillow Offers, the iBuying business it had built around an algorithmic pricing model. The Q3 2021 loss for the segment was $381 million. The company had bought roughly 27,000 homes through the program; thousands of those were now flagged for sale at losses. About a quarter of Zillow's workforce — around 2,000 people — would be laid off.

The model was not broken in the way a buggy program is broken. It had been built carefully, validated against history, and deployed at scale. For several years it had worked. It estimated the price at which Zillow could buy a home today and resell it three to six months later, accounting for renovation costs and market drift. The estimates were, by ML standards, accurate.

Then they stopped being accurate. And the reason they stopped is the subject of this chapter.

The model had been trained on a world in which Zillow was a small participant — observing prices, occasionally buying a home, mostly watching. The training data captured the relationship between home features, neighborhood signals, market conditions, and three-to-six-month resale prices *given that participants were behaving the way they had been behaving*. When Zillow began buying tens of thousands of homes, it stopped being a participant and started being an actor. Its purchases moved local prices. Its inventory backed up. The renovations took longer than the model had assumed, in part because Zillow's own demand for contractors was driving up labor cost. The model had no language for any of this. It kept estimating prices with the confidence intervals that had served it well when its predictions did not have to survive the market consequences of its own decisions.

The technical name for what happened is a violation of the assumption that the training distribution and the deployment distribution are the same. The plain-language version: Zillow was using a map of a country that existed before Zillow had reorganized it.

This is not a Zillow story. It is the structure of nearly every consequential failure of predictive analytics in organizational decision-making. A model trained on observation cannot reliably predict the consequences of intervention. The mathematical version of this claim is precise. The organizational version is consequential.

---

## Two Distributions, One Word

Statisticians distinguish between two probability distributions that look similar and behave differently.

The first is the **observational distribution**: P(Y | X). The probability of outcome Y given that X is observed in the data. This is what every supervised learning algorithm fits. You observe historical pairs of (X, Y), you fit a function, you use the function to predict Y when you next observe X. Every dashboard, every regression, every churn predictor, every credit risk model in production is operating on the observational distribution.

The second is the **interventional distribution**: P(Y | do(X)). The probability of outcome Y when X is *set by intervention*, rather than observed. This is what every consequential decision actually requires. You are not observing X passively. You are deciding to make X happen. You raise prices. You send the email campaign. You acquire the company. You buy 27,000 homes.

The two distributions are different whenever something connects X to Y other than the causal mechanism the decision is invoking. Confounders are the textbook case — a variable that influences both X and Y produces a correlation in the observational distribution that disappears when you intervene on X by fiat. Selection effects are another case — the historical data records the decisions someone chose to make, which were not random, which means the conditions under which X was high are not the conditions you'll be operating in when you intervene to make X high.

There is a third case that is not always named explicitly and which Zillow's failure illustrates exactly: *the act of intervention changes the system itself*. When Zillow bought at small scale, the price the model estimated was a price the market would clear at three months later. When Zillow bought at large scale, Zillow itself became a part of the market the model was trying to estimate. The mechanism Zillow's intervention triggered did not exist in the training data because Zillow had not been intervening at that scale in the training data.

Pearl's *do*-operator captures this distinction precisely. P(Y | X) reads "the probability of Y given that X is observed." P(Y | do(X)) reads "the probability of Y given that X is set by intervention." Notationally, the *do*-operator removes incoming arrows to X in the causal graph — when you set X by intervention, the things that previously caused X are no longer determining its value. You are.

This sounds like a technical distinction. It is the difference between $381 million and zero.

---

## What This Looks Like When You Run the Numbers

Consider a stripped-down example. A pricing team observes that customers who pay higher prices for the company's product also have higher retention. The correlation in the historical data is strong. The team proposes raising prices across the board, expecting that retention will rise.

Here is the observational distribution. Higher price → higher retention. Correlation: 0.4, p < 0.001, n = 50,000 customer accounts.

Here is what is actually happening underneath. The company sells to two segments: small businesses (price-sensitive) and enterprise customers (price-insensitive, who buy more features). Enterprise customers pay higher prices because they buy more features; they have higher retention because their switching costs are higher. **Segment** is a confounder — it influences both the price paid and the retention. In the observational distribution, segment is doing all the work the correlation is appearing to attribute to price.

Now consider the intervention. The company raises prices across all customers by 15%. The do-operator wipes out the connection between segment and price — both small businesses and enterprises now pay the new higher rate. Segment still influences retention (enterprise switching costs are still high). But price now operates on a different mechanism — it triggers churn in the price-sensitive segment, the segment that previously paid lower prices and stayed.

In the observational data, the conditional probability of retention given price is high. In the interventional distribution, the conditional probability of retention given that you have raised price is much lower — because the customers most likely to leave at the new price were precisely the customers who had been keeping the observational correlation high by paying lower prices and staying.

The historical model said: raising prices will raise retention. The actual world says: raising prices in a segmented market without segment-level customization will surface churn in the segment that was holding the correlation up. The math here is not subtle. The math is exactly the *do*-operator doing its job — the moment you intervene on X, you sever the relationship between X and the variables that were determining X in the historical data, which means the conditional distributions you previously estimated no longer apply.

A predictive model that does not distinguish between these two distributions is, at best, a description of what will happen if nothing changes. The moment you act, the model's relevance to the decision degrades — sometimes gracefully, sometimes catastrophically.

---

## What "Trained on the Past" Actually Means

Return to the technical claim and make it precise. A predictive model is a function fit to historical pairs of inputs and outputs. The function captures statistical regularities present in the historical data. Whether it generalizes — whether it makes useful predictions when deployed — depends on whether the deployment world is statistically similar to the training world. Three things can break that similarity, and they are usually conflated.

**Concept drift**: the underlying data-generating process changes. A churn model trained on 2019 customer behavior meets 2020 customer behavior, where stay-at-home orders have restructured everything the model had learned. The model is not making different predictions. The world is producing different ground truth. The model's accuracy degrades because the world has moved.

**Covariate shift**: the distribution of inputs changes, but the relationship between inputs and outputs is stable. A credit risk model trained on a borrower population that included few self-employed applicants now meets a population where self-employed applicants are 30% of the book. The relationship between the features and default is still what it was; the new applicant pool is just a different sample. Models can sometimes be reweighted to handle covariate shift.

**Intervention**: someone — possibly the model's deployer — deliberately changes the value of an input that the model had been treating as exogenous. The pricing team raises prices. Zillow scales up its buying. The change is not drift; it is a *do*. The training distribution included the regime where this input was being set by whatever process had been setting it historically. The deployment distribution includes the regime where this input is being set by you, by fiat, at the scale you are setting it.

A predictive model has no way to distinguish between these three on its own. From its perspective, the deployment data is just new data. If the new data looks like the training data, predictions stay calibrated. If it doesn't, predictions degrade — and the model does not announce the degradation. This is the silent failure mode from Chapter 1, in its predictive-stage form.

The Living Model architecture's first defensive move is to make the *intervention* case explicit. When the system is asked "what happens if we set X to this value?", it does not answer by interpolating from the historical relationship between X and Y. It answers by simulating the intervention through a causal structure that distinguishes the variables that were caused by upstream factors in the data from the variables you are now setting by fiat. The simulation is allowed to differ from the historical correlation. Often it has to.

---

## The Innovator's Dilemma Is Mostly About Time

Clayton Christensen's *The Innovator's Dilemma* is often read as a story about strategic blindness — incumbents fail to see the disruption coming because they are looking at the wrong customers. The popular reading is that Kodak missed digital, Blockbuster missed streaming, and the lesson is to listen to the early adopters of the inferior technology that will eventually eat your market.

The popular reading underestimates the incumbent executives. Most of them see it. The Kodak engineers built a digital camera in 1975. Blockbuster's executives knew about Netflix; they were offered the chance to acquire it in 2000 for $50 million, and declined. The fact that incumbents do not act on what they see is the part of the story that is harder to explain by reference to causal blindness — and it is the part that matters most for how a Living Model has to be built.

The reframe: the Innovator's Dilemma is primarily a problem of organizational incentives and time horizons, not of perception.

Consider the executive who sees the disruptive entrant coming. They are running a profitable, large business. Their compensation is tied to the next four to eight quarters of revenue and margin in that business. The incumbent product is doing fine. The disruptive entrant is small, growing, and unprofitable. To respond — to start cannibalizing their own product, to invest in a thin-margin offering for non-consumers, to redirect engineering effort to a market that will not produce meaningful revenue for two to three years — they would have to take an action that hurts their measured performance now in service of an outcome that may not arrive within their tenure.

The executive's choice is not "see the disruption" versus "miss the disruption." It is "act on the disruption and absorb the personal cost" versus "manage the disruption optically and let the next executive deal with it." The rational, incentive-aligned answer is the second one. Most executives choose it. Some of them are eventually displaced. The system was working as designed.

This reframe matters because it tells you what a Living Model has to model that conventional strategic analysis does not. A model that captures only the technological substitution dynamic — the entrant's performance trajectory crossing the incumbent's customer requirements — predicts the displacement but offers no traction on whether the incumbent will respond. A model that captures the incentive structure, the executive's time horizon, the boardroom incentive to defer, the analyst expectations that punish quarterly margin compression for long-term repositioning — that model can rank actual interventions, including interventions on the incentive structure itself.

The other thing the reframe forces is an honest reckoning with how fast this is now happening. Christensen's classic cases — disk drives, steel mills — operated on disruption horizons of five to seven years. The compute, retail, and SaaS disruptions of the last decade have run on horizons of eighteen to thirty months. The AI-native disruption arriving as I write this appears to be running on horizons of six to twelve months. The compression is the news. An executive whose incentive horizon is four quarters and whose disruption horizon used to be five years had time. An executive whose incentive horizon is four quarters and whose disruption horizon is twelve months does not.

The Living Model, built right, includes the time-horizon dimension as a first-class input. It does not just rank interventions by expected causal effect. It ranks them by expected causal effect *over a horizon that is itself a variable* — and shows the executive what the trajectory looks like under their current incentive structure versus a hypothetical structure that aligns with the disruption horizon their business is actually facing.

This is an uncomfortable output to receive. It tells a board, sometimes, that the reason their company is not responding to the disruption is the way they are paying their CEO.

---

## The Practical Test

Before trusting any model with a decision of consequence, three questions are worth asking. They will not certify the model — nothing certifies a model — but they will surface the failure mode this chapter has been describing.

*Was your training data generated under a regime in which actors like you were intervening at the scale you are now considering?* If Zillow had asked this in 2019, the answer would have been no. The training data captured a world where iBuying was a small share of national volume. Scaling Zillow's purchases to a meaningful share of local markets was an intervention the data did not contain.

*What is the conditional distribution of your action given the variables in the data?* If your action — the X you are about to set — is heavily determined by some variable in the data, then the historical relationship between X and Y is reflecting that variable's influence as much as X's own. Intervening to set X by fiat will sever that connection and produce a different relationship.

*If you ran your model's recommendation as an intervention, what would the world look like one period later, two periods later, three?* A useful answer engages the feedback loops — the way the intervention reshapes the inputs the model will see next time. The Living Model's continual updating capability is, in part, a response to this question: the only honest answer to "what does the world look like after my intervention" is "let's see the data and update."

A model that cannot answer these questions is not necessarily wrong. It is operating in a regime where it cannot tell you whether it is wrong. That is the Zillow position.

---

## Looking Forward

The architecture that handles intervention rather than only prediction is the subject of Part Three. The mathematical apparatus — Pearl's *do*-calculus, structural causal models, the backdoor criterion, identification — is the subject of Chapters 5 through 9. Chapter 10 handles the case where the apparatus runs out: latent confounders, the limits of observational data.

The next chapter, though, takes on a different abuse. Zillow's failure was not a latency problem; the data was current. The failure was that even with current data, a model trained on the prior regime could not handle the new one. But many of the failures of organizational analytics are *also* latency failures, hiding inside marketing language that has been gutted of meaning. "Real-time" is one of the most abused terms in enterprise software. The next chapter argues for the honest replacement — *continually updated* — and shows what it actually requires.

---

**What would change my mind:** A documented case in which a predictive model trained on observational data correctly anticipated the consequences of an intervention at a scale not represented in the training data, and where the prediction was robust to a sensitivity analysis that varied the assumed causal structure. I have not seen such a case in mainstream enterprise practice. I would update if shown one.

**Still puzzling:** The empirical question of where, exactly, the disruption horizon is settling for AI-native business models. I see compressions but I cannot yet say with confidence whether the new equilibrium is twelve months, six months, or something shorter. The Living Model's horizon parameter cannot be fixed until that empirical question is answered, and I do not know yet how to answer it without more cases on the table.

---

**Tags:** Zillow Offers 2021 wind-down, observational vs interventional distribution, Innovator's Dilemma incentive structure, horizon compression disruption, do-operator predictive failure
