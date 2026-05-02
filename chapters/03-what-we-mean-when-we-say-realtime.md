# Chapter 3 — What We Mean When We Say "Real-Time"

*Three latencies, and what "continually updated" actually requires.*

**Nik Bear Brown**
**Draft — 2026-05-01**

---

*Suggested titles:*

- *What We Mean When We Say "Real-Time"*
- *The Three Latencies*
- *Continually Updated: The Honest Term*

**TL;DR.** "Real-time" is the most abused term in enterprise software, papering over three distinct latencies — data, model, and decision — any one of which can render the others meaningless. The honest replacement is *continually updated*, which is not a marketing claim but a set of technical, organizational, and epistemic requirements.

---

## A Word That Means Something Different to Everyone Who Uses It

A high-frequency trading firm runs its execution loop in microseconds. When its engineers say "real-time," they mean a price tick has reached the matching engine within a measurable fraction of a millisecond and a quote has been generated, evaluated, and dispatched before the next tick arrives. The latency budget is so small that the speed of light through optical fiber is part of the engineering decision. Co-location matters. Switch design matters. The word "real-time" is doing precise, technical work.

An enterprise dashboard running on a data warehouse refreshed every six hours can also call itself "real-time." So can an analytics product where the underlying ETL job runs nightly. So can a "live" KPI that is, in fact, a cache of a query that was run twelve hours ago. The word, here, is doing no technical work at all. It is decorative.

The consequence is not minor. When an executive is told a system operates in real time, they form an expectation about what the system can answer and how quickly. When they discover, sometimes mid-decision, that the data on screen is from yesterday — or that the model that scored it is from last quarter — the system's apparent authority does not transfer to the actual decision. They are making a real-time decision against a six-hour-old window of the world.

This chapter is about the gap between the marketing term and the actual requirement. The argument has three moves: name the three latencies that "real-time" elides, show what "continually updated" actually demands, and identify where most enterprises are failing at it.

---

## The Three Latencies

A decision system has at least three independent time scales. Each can be short or long. They tend to be discussed as one because they tend to be marketed as one. They are not.

**Data latency.** The time between an event happening in the world and the data describing the event being available to the system. A credit card transaction happens at the point of sale. The transaction record reaches the bank's processing system. The record is loaded into a feature store, joined with the cardholder's history, and made available to a model. Each hop costs something. A genuine real-time credit card fraud system might have a data latency of fifty milliseconds — fast enough to score the transaction before the merchant's terminal has confirmed it. A self-described real-time customer analytics system might have a data latency of hours, because the ingestion pipeline batches.

**Model latency.** The time between the system having access to current data and the model that scores it being itself current. A model trained on data through last quarter, deployed today, has a model latency of a quarter even if the data flowing through it has a latency of milliseconds. The model is current to its training cutoff. The world has moved since.

**Decision latency.** The time between the model producing a recommendation and a human (or another system) acting on it. A fraud system that scores a transaction in fifty milliseconds and routes the alert to a queue that an analyst reviews twice a day has a decision latency of hours. The technical infrastructure was built for milliseconds. The decision authority was not.

A "real-time" claim that addresses only one of these — usually data latency — and silently ignores the other two is a category of marketing failure that has consequences. Knight Capital's August 2012 disaster, mentioned in Chapter 1, was a case of all three latencies being short. The execution loop was milliseconds. The models were current. The decision was automated. There was no decision latency at all. That is what made the system catastrophic — it had no governor. The opposite case is more common: a "real-time" customer dashboard with multi-hour data latency, models trained months ago, and decision latency measured in quarterly reviews. Each component pretending to be the same thing the marketing said.

The honest version of "real-time" requires you to specify which latency you mean. Usually you mean all three. Usually the system delivers only one.

---

## What "Continually Updated" Actually Requires

I argue in this book for replacing "real-time" with **continually updated** whenever the claim is about a system's relationship to changing conditions in the world. The replacement is not stylistic. *Continually updated* is a load-bearing term. It says something specific about three layers of the system, and unless all three are present, the claim is not honest.

**Technical layer.** A continually updated system requires streaming infrastructure, not batch. Events are captured at source — a Kafka or Pulsar topic, a Kinesis stream, an event bus — and processed as they arrive, by something like Flink or Spark Streaming, into a feature store that the model can query at low latency. The model itself must be capable of either online learning (its parameters update incrementally as new data arrives) or sliding-window retraining (a fresh fit on the most recent N observations runs frequently enough to keep model latency low). The decision pipeline must be wired in such a way that the model's output reaches the decision-maker — human or automated — without batching that defeats the speed of everything upstream.

This stack is mature. It is not hard to find. Most enterprises have already paid for it. They are not running it correctly.

**Organizational layer.** The technical capability is meaningless if the people authorized to act on the model's outputs cannot act faster than the slowest link in the human chain. A continually updated system requires decision rights pushed down to where the model's output lands. The pricing system that retrains hourly but cannot change a price without a weekly committee review is not continually updated. It is a continually updated input to a weekly batch decision. The fraud system that scores transactions in milliseconds but routes alerts to a queue reviewed by an analyst twice a day is the same. The technical layer is doing its job. The organizational layer is unfinished.

**Epistemic layer.** The hardest part. A continually updated system requires the people operating it to act on incomplete evidence, accept that they will be wrong some of the time, and calibrate their confidence rather than waiting for the certainty that batch reporting culture conditions them to expect. Continually updated decision-making is *not* faster decision-making. It is decision-making that has accepted the irreducible uncertainty of acting on conditions that are themselves changing. The reports waited for in a quarterly review have a kind of comfort: they describe what already happened. The continually updated dashboard describes what is happening, with acknowledged uncertainty about what will be true an hour from now. Many organizations have never learned to operate in that posture. They keep waiting for the report that is coming. By the time it arrives, they have lost the window.

If any one layer is missing, the system is not continually updated. A streaming infrastructure feeding a quarterly report is not. A modern dashboard that nobody is allowed to act on without committee approval is not. A continually updated dashboard read by an executive who waits for the monthly review before deciding is not.

---

## The Honest Working Definition

Consolidate. A system is continually updated to the degree that:

- *Its data latency* — from event to availability — is short relative to the decisions it supports.
- *Its model latency* — from training to deployment — is short relative to the rate at which the underlying data-generating process changes.
- *Its decision latency* — from model output to action — is short relative to both of the above and to the window in which the action remains effective.

Notice the relativizations. "Real-time" implies an absolute speed. *Continually updated* implies a calibrated relationship to the rate at which the world is moving. A weather forecast updated every six hours is continually updated for clothing decisions. The same forecast is not continually updated for a hurricane evacuation. The decision tells you what "continually" needs to mean.

This is the honest engineering requirement that the abuse of "real-time" papers over. A vendor who claims their system is real-time should be asked: real-time relative to what decision? With what data latency? What model latency? What decision latency? If they cannot answer, the claim is decorative.

---

## A Worked Example: Fraud Detection Done Three Ways

Three credit card fraud detection systems, all marketed as "real-time," all serving the same nominal use case. Their actual capabilities differ by orders of magnitude.

**System A.** Data latency: 50 milliseconds. Model latency: 6 weeks (the model is retrained quarterly, with ad-hoc updates between). Decision latency: automated, 100 milliseconds. The execution loop is fast. The model is sometimes ahead of new fraud patterns and sometimes weeks behind. When fraudsters discover a new attack vector and operationalize it, the system catches the early cases that look like prior patterns and misses the variant cases that don't. The transactions where the model is wrong are scored confidently because the model does not know it is wrong.

**System B.** Data latency: 50 milliseconds. Model latency: 6 hours (the model retrains every six hours on the most recent rolling window, with online updates between). Decision latency: 100 milliseconds (automated). This system is continually updated by the working definition above. New fraud patterns get incorporated within hours. The cost is an engineering operation — model training pipelines that run six times a day, monitoring for performance degradation, fallback procedures when a retrain produces a model that is worse than the prior one.

**System C.** Data latency: 50 milliseconds. Model latency: 6 hours. Decision latency: variable, because suspicious transactions are queued for analyst review and the analyst queue averages four hours. This system has the technical capabilities of System B and the operational capabilities of System A. The decision latency is the bottleneck. By the time an analyst reviews the alert, the fraud has often completed. The system catches it for forensics, not for prevention.

All three call themselves "real-time." Only System B is continually updated. The others are technical systems serving a non-continually-updated decision process — the worst of both worlds.

The example generalizes. Whenever you find a "real-time" claim, look for the three latencies. Look for the layer where the marketing term is doing decorative work.

---

## What This Costs

The cost of using "real-time" decoratively is not aesthetic. It is the gap between what executives believe they are buying and what they are operating with. This gap shows up in several recognizable failure modes.

*The dashboard mismatch.* An executive making a decision believes the data on screen is current. The data is six hours old. The decision is made on a six-hour-old window of the world; the world has moved. Six hours is sometimes fine, sometimes catastrophic, and the executive does not know which they are operating under.

*The model staleness mismatch.* The data is current. The model was trained six weeks ago. New patterns have emerged that the model has no language for. The model continues to score transactions, customers, opportunities — confidently, with calibrated probabilities relative to the world it was trained on, miscalibrated relative to the world it is operating in. No one tells the executive the model is stale because the model does not announce it.

*The decision-rights mismatch.* The technical pipeline is fast. The committee that has to approve the action meets weekly. The window of opportunity is two days. The window closes. The system is functioning correctly. The organization is using it incorrectly.

These failures share a structure. None of them announces itself. All of them are visible in retrospect. Most of them are diagnosed as technical problems when they are organizational problems, or vice versa. Until the three layers — technical, organizational, epistemic — are all named and audited, the failures will keep happening.

---

## Why "Continually Updated" Is the Honest Term, Not Just a Less Slick One

It is tempting to read this chapter as a complaint about marketing language. It is not. The argument is structural.

A system that calls itself real-time encourages the consumer to stop asking questions. The label does the work. *Real-time*, the consumer thinks, *means I can act on what I see right now.* The system has no obligation to specify what "right now" means, because the label has already been accepted as sufficient.

A system that calls itself continually updated cannot get away with this. The label invites the question: continually updated *for what*? At what rate? Calibrated to which decision? The honest term forces the conversation that the marketing term suppresses. It refuses to be a single-word substitute for what is actually a four-part specification.

This is the same structural move the book makes at every layer. Pearl's Ladder refuses to collapse association into intervention. Expected Value of Intervention, in the next chapter, refuses to collapse probability into impact. Continually updated refuses to collapse three latencies into one buzzword. Each refusal is a commitment to making the underlying distinction visible — because the distinction is what the decision actually requires.

---

## Closing

Call things what they are. A system with low data latency and high model latency is not real-time; it is current data being scored by an out-of-date model. A system with low data latency and low model latency that feeds a quarterly review is not real-time; it is a continually updated input to a batch decision. A system with everything wired correctly through the technical layer that has no decision authority pushed down to where the output lands is not real-time; it is theater.

The honest term, and the one this book will use throughout, is *continually updated*. It implies a relationship to a decision and to a rate of world change, not an absolute speed. It demands work in three places, not one. And it cannot be claimed by a vendor; it has to be earned, layer by layer, by the organization that wants to act on conditions that are themselves changing.

The next chapter takes on a different one-number error — risk. The framework that collapses probability and impact into a single score makes a category of decision impossible to make well. That collapse, like the abuse of "real-time," is so common that it has come to seem like the discipline rather than its failure mode.

---

**What would change my mind:** A system marketed as "real-time" that, on inspection, turns out to have data latency, model latency, and decision latency all calibrated appropriately to the decision it serves — and that calls itself real-time without that being the most useful claim about it. I expect to find such systems in narrow domains (HFT, payments, ad bidding). I do not expect to find them in mainstream enterprise analytics. I would update if shown enterprise cases.

**Still puzzling:** Whether the epistemic layer — the organizational willingness to act on incomplete evidence — is something that can be trained, or whether it requires a different kind of leadership than the leadership produced by quarterly reporting culture. I lean toward the second answer. I am not sure how to test it without a longitudinal study I do not have access to.

---

**Tags:** real-time abuse enterprise software, three latencies decision systems, continually updated streaming infrastructure, fraud detection decision latency, organizational decision rights data systems
