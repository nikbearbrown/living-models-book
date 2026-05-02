# Living Models

*Causal Intelligence for the Decisions That Actually Matter*


**Author:** Chris Selland & Nik Bear Brown
**Publisher:** Bear Brown, LLC
**Status:** Draft
**Started:** 2026-05-02


> The data was never the problem. It was always the question.

---

A textbook on the architecture of decision systems that can reason about deliberate intervention rather than merely describe the past.

The argument is short. Three decades of enterprise analytics have produced sophisticated hindsight and very little foresight. Dashboards describe what happened. Predictive models extrapolate from it. Both are useful; neither answers the question every consequential decision actually requires — *what would happen if we did this, and what would have happened if we had done that differently?* These are causal questions, not statistical ones, and they live on rungs of Pearl's Ladder of Causation that the dominant tools cannot reach. *Living Models* describes the apparatus that can reach them, the architecture that operationalizes the apparatus in production, and the disciplines that keep it alive after deployment.

The book is for technical leaders, strategists, and engineers who build, evaluate, or rely on decision systems in fast-moving environments. It assumes elementary probability and basic regression. It teaches the do-operator from first principles in Chapter 5 and builds from there.

---

## Status

All twenty chapters drafted as of May 2026. The drafts are first-pass and explicitly not finished. Each chapter sits at roughly 2,500–5,000 words; the workshop spec calls for 5,000–8,000, and the deeper worked examples and graduated exercises are still to come. The foreword (March 2026) is finalized.

Front matter, glossary, appendices, and Parts Four through Six remain to be written. See [`outline.md`](outline.md) for current chapter status.

---

## How the book is structured

**Part One — The Problem.** Why three decades of analytics produced sophisticated hindsight and very little foresight. The dashboard that lied. The map that doesn't move. The abuse of "real-time." Why risk is two numbers, not one.

**Part Two — The Theory.** The mathematical foundations of causal intelligence, made readable. Pearl's Ladder. DAGs and structural causal models. The equivalence problem. Estimating effects. The counterfactual. Confounders, colliders, and the limits of observational data. Treatments and the gold standard. The plumber's objection.

**Part Three — The Architecture.** How to build systems that actually use the theory. The four properties that define a Living Model. The expert in the room and how experts get causation wrong. The machine that interviews the expert. Resolving the graph. From graph to decision. The Causal Brain Executive Report. Keeping the model alive.

**Parts Four through Six** — to be written — apply and extend the architecture: Christensen and Damodaran as inputs to causal models rather than models themselves; worked cases on pricing, supply chain, and disruption; the frontier of latent confounders, network interference, agentic systems, and causal digital twins.

---

## How to read it

Readers who want the diagnosis can read Part One alone. The four chapters establish what is wrong with the analytical estate most organizations have built and why the problem is structural rather than incidental.

Readers who want the apparatus need Parts One and Two together. The mathematics of Part Two depends on the failure modes Part One catalogs; without the diagnosis, the apparatus reads as academic; with it, the apparatus reads as the response to specific, named problems.

Readers who want to build something need all three parts. Part Three's architecture is operationally serious — eight chapters on elicitation, resolution, decision, reporting, and maintenance — and it depends on the conceptual apparatus Part Two builds.

Each chapter follows the same structure: cold open in a specific scene, mechanism developed from first principles, worked example, synthesis, and a closing pair of *what would change my mind* and *still puzzling* sections that name the chapter's open questions honestly. Skim by reading the openings.

---

## Voice

Feynman-flavored. Mechanism visible. Pretension stripped, jargon translated or cut. The voice of someone working through the architecture out loud, not lecturing on settled doctrine. First person where I am taking a position; "you" to invite the reader to think along; "we" to work through the harder mechanisms together.

The book takes positions. Where I think the literature is wrong I say so; where I am uncertain I say that too. The closing sections of each chapter — *what would change my mind* and *still puzzling* — are commitments that the rest of the chapter is not the final word.

---

## What this book is not

Not a survey. The literature on causal inference is enormous and continues to grow rapidly; this book selects and synthesizes rather than catalogs. Pearl, Rubin, Hernán, Robbins, Greenland, Athey, Wager, Imbens, Cunningham, and many others have written authoritative treatments at greater technical depth, and readers who want the full picture should read them.

Not a programming guide. The book describes architectures and disciplines; the engineering implementation is left to libraries (DoWhy, EconML, CausalML, PyMC) and to the reader's own judgment about how to integrate the architecture with the systems she already operates.

Not neutral. The book argues that most current enterprise analytics is structurally inadequate to the questions it is asked to answer, and that the architectural shift to Living Models is consequential. Readers who want a balanced both-sides treatment should look elsewhere.

---

## Why this exists

The framework began with an email. Someone reached out with a concrete problem about how to build intelligence that keeps pace with decisions in fast-moving environments. The first-principles answer became the book.

The full origin is in [`chapters/2026-03-16-foreword.md`](chapters/2026-03-16-foreword.md).

---

## Repository layout

```
books/living-models/
├── README.md          # this file
├── book.md            # per-book CLAUDE.md — audience, scope, voice, cadence
├── outline.md         # chapter-by-chapter status (to write / drafted / finalized / killed)
├── chapters/          # chapter drafts, dated YYYY-MM-DD-chapter-N-slug.md
├── essays/            # chapter-adjacent longreads
├── narratives/        # narrative-nonfiction pieces in this book's voice
├── bookmaps/          # source-book analyses for this project
└── style/             # per-book voice samples (overrides workshop default)
```

---

## Feedback

This is a working draft. Issues, pull requests, marginalia, and arguments-by-email are all welcome. The closing sections of each chapter name the questions I am most uncertain about; those are the most useful places to push back.

---

*Drafts authored by Nik Bear Brown with substantial drafting assistance from Claude. The byline is mine; the responsibility for any errors is mine. The chapter-by-chapter provenance — which sections are mine verbatim, which were drafted by Claude against my source material, which were revised by me after — is logged in the repository history.*

## Publish

Upload `output/living-model.epub` to [KDP](https://kdp.amazon.com).
