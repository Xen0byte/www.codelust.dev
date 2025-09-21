+++
title = "The Only Meaningful Measure Of Code Quality ..."
date = "2025-09-07T12:25:34+01:00"
author = "Aleksander Reinhardt"
tags = ["code quality", "metrics", "clean code", "clean architecture"]
keywords = ["only", "meaningful", "measure", "code", "quality"]
description = "... Is The Effort Required To Modify It"
showFullContent = false
hideComments = false
+++

We spend a lot of time talking about "code quality". Ask ten developers what it means, and you'll get ten different answers: low defect rate, consistent styles, high readability, high test coverage, alignment to SOLID principles, deployable on commit, low cyclomatic complexity, using clean architecture, or maybe just "code that I like writing", but when you boil it all down, there's only one measure of quality that really matters, **how easily can we change the code tomorrow**, and everything else feeds into this metric and only matters in so far as it increases (or decreases) our ability to adapt the codebase when change is inevitably required.

---

{{<toc>}}

---

## Why Change Is The Real Measure

Software doesn't live in a vacuum. Requirements shift, features evolve, teams grow, technologies come and go, and mistakes need fixing. The one constant is change. If your code is beautiful but rigid, it's a liability. If it's ugly but malleable, that's a good start. If it's both beautiful and malleable, that's the place to be, and I would argue further that beauty is a side-effect of good malleability.

When it comes to code hygiene, some examples of common mistakes include ...

**Over-Engineering**

Future-proofing is important, but building abstractions and layers of indirection "just in case" can weigh a project down. Every extra generic, factory, or configuration knob becomes another thing to maintain. Trying to anticipate every future need adds unnecessary weight, and when the time comes to adjust you’ll find yourself dragging that weight along.

**Code Golf**

A clever one-liner might impress your peers, but if nobody can understand or debug it, or safely refactor it six months later, is it really high quality? That's rhetorical; the answer is "no".

**Muddle-Through Debugging**

When faced with a defect, some developers scatter fixes everywhere without understanding the root cause. It might get the system working temporarily, but it leaves behind brittle and unpredictable code. Imprecise debugging makes future changes dangerous, because nobody really knows which line of duct tape is holding things together.

**Cargo-Cult Programming**

Strict adherence to patterns can feel disciplined but, if they're applied dogmatically and without a good measure of critical thinking, it can trap you in a maze of abstractions that makes change harder, rather than easier. To put it a different way, with an example inspired by a saying about fruit salads: knowledge means knowing that an empty interface has a perfect [dependency score](https://en.wikipedia.org/wiki/Software_package_metrics), while wisdom means understanding that an empty interface is an invalid contract.

---

## The Illusion Of Preventive Metrics

We latch onto preventive metrics because they're easy to measure in the moment: linting results, test coverage reports, static analysis scores. These metrics provide valuable insights, but they measure process attributes rather than the outcome that truly matters: how easily the code can be modified.

For example:

- A codebase with 100% test coverage might be thoroughly tested but painfully rigid. Every small change could break dozens of brittle, implementation-dependent tests.
- You can achieve perfect cyclomatic complexity scores by extracting every decision into its own method, but end up with a spaghetti of tiny functions that no one can follow.
- Code can pass every linter rule while still embodying a fundamentally poor design that resists change.
- Low coupling scores might hide the fact that your modules communicate through global state or rigid protocols that make real-world changes difficult.

These metrics are useful guardrails, but we shouldn't mistake them for the destination. I've seen many teams celebrate perfect static analysis scores while dreading any requirement change because they know how fragile their faux-perfect codebase actually is.

---

## DORA Metrics As Reactive Signals Of Changeability

[DORA metrics](https://dora.dev/guides/dora-metrics-four-keys/) can serve as a dashboard of secondary effects produced by (or degraded by) the effort required to modify code. When a codebase is easy to change, deployment frequency rises naturally because changes can be sliced thin and deployed independently. Lead time for changes shrinks because developers spend less time navigating through incidental coupling. Change failure rate drops because modifications remain localized and their intent is clear. Mean time to restore improves because the system's behaviour is observable and its seams are explicit.

When any of these metrics trend in the wrong direction, they signal that hidden friction is accumulating within the codebase. Tests may have begun obstructing refactors instead of enabling them. Architectural boundaries might have become leaky. Abstractions could be defending past decisions rather than facilitating new ones. The value in these metrics isn't in chasing them directly, but in mining them for early warnings that the real asset, changeability, is eroding.

---

## Designing For Change

So, how do we actually measure and design for changeability? A few guiding principles:  

1. **Localise Decisions** – Changes should ripple as little as possible. If adding a feature forces edits across ten files in unrelated modules, your design is resisting change.

2. **Lean On Tests That Enable Refactoring** – Tests should make it *safer* to change code, not punish you for trying. Favour higher-level integration tests over brittle mocks that freeze implementation details.

3. **Readability Is For Future-You** – You don't write code for the compiler; you write it for the humans who will modify it next (likely including yourself, after you've forgotten everything).

4. **Avoid Premature Optimisation And Abstraction** – Over-abstracted code makes small changes expensive. Over-optimized code is often calcified code.

5. **Practice Continuous Refactoring** – Small, ongoing cleanups keep the codebase supple. Waiting until it's "bad enough" usually means it's already too late.

6. **Measure The Right Thing** – Track how long changes take and why. When a "simple change" takes days, investigate the barriers that made it difficult rather than just accepting the cost.

---

## A Mental Shift

When we review code, we should stop asking "Is this correct?" or "Does this look nice?" Those matter, but they're secondary. The real question is: **"If requirements change tomorrow, how painful will it be to adapt this?"**  

That's the north star of code quality.  

---

## Conclusion  

Code quality is often treated like an aesthetic judgment or a scorecard of best practices. But in practice, the only thing that truly matters is **changeability**. Can we adapt, extend, and evolve this code without fear? If yes, the code is high quality. If not, it isn't—no matter how elegant it looks today.  

In the end, our job as developers isn't just to write code that works. It's to write code that *keeps working, even as the world around it changes.*  

## A Mental Shift

Code reviews often center around concerns which address the code only as it exists today. The more critical matter, and the one that should guide our evaluations, is: **"When requirements inevitably change, how painful will it be to adapt this code?"**.

This shift in perspective transforms how we evaluate and write software. It prioritises clarity over cleverness, appropriate abstractions over rigid frameworks, and thoughtful design over mindless adherence to patterns. Changeability becomes the north star of code quality by which we navigate all technical decisions.

---

## Conclusion

We too often mistake the aesthetics of code for its quality, celebrating elegant solutions that become tomorrow's maintenance nightmares. We confuse adherence to best practices with actual value delivery. But genuine code quality has a simpler measure: **changeability**. 

Can the code be safely extended when new features arrive? Can it be refactored without cascading failures? Can it evolve alongside shifting business needs? If yes, that code is high quality, regardless of whether it follows every pattern in the book. If not, no amount of syntactic sugar can redeem it.

The mark of professional software development isn't creating code that works today. It's creating code that continues to serve its purpose as the world around it transforms, code that bends but doesn't break under the constant pressure of change.
