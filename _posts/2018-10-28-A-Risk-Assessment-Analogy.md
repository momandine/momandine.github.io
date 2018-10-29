---
layout: post
title: "A Risk Assessment Analogy"
description: "How do you know when enough safety is enough?"
category: 
tags: ["testing"]
---
{% include JB/setup %}

I was reading a book recently where a character scuba dives while pregnant. The book is set in the past, so I decided to look up modern medical information on whether this is a good idea.

The answer is “No”. There is no medical reason why the answer is no. To the best of our knowledge, none of the biological systems required for gestation are significantly affected by scuba diving. But most people think about scuba diving as risky, and if they were to guess without doing research, they’d conclude… maybe it’s not a great idea to do while pregnant. So even though the medical establishment agrees that it should theoretically be fine, if something goes wrong, psychologically it is very destructive to look back and see something you could have done differently.

Why am I talking about this in an engineering blog? Because it’s an analogy to failure scenario generation, safety, and verification in any discipline. Many engineers have shipped a piece of code, having told themselves it’s fine to the best of their knowledge, only to see if fall over or emit terrible bugs in production. This is why we often push to a small percentage of real traffic, do bug-bashes and conduct pre-mortems where we imagine different types of failures and what would have caused them. We’re trying to smoke out the unknown unknowns that cause issues. It’s a type of thinking I am actively learning how to lean into. As an optimist, someone who tends to seek out nuance, and a person who has a strong bias towards action, I tend to chafe against risk-aversion without a clear threat model. The term “Cover Your Ass” gets thrown around to describe extreme end of this - wasteful carefulness.

I am now on a team that is business-critical, and one of the lowest layers of the engineering stack. We have many customers for the code we write, including ourselves. There are some kind of things we can break without any consequence - low-priority monitoring, for example. There are things that if they break, it will be obvious, and someone can step in and fix it - like an auto-remediation service that can replaced by hand-operations. 

Then there failures that are very hard to reverse, are hard to detect, and that heavily affect other teams and customers - like breaches in our durability and consistency guarantees. If something is permanently deleted, there is no undoing. If an inconsistency is introduced, there may not be repercussions until much later, when system invariant check begin to fire or users notice. These are the areas where defaulting to risk-aversion is the right thing to do. 

You can’t guarantee nothing will go wrong, of course. And a good post-mortem will take goodwill into account. But there’s a rule of thumb you can you use: if there’s a serious outage, how would you justify your actions to engineering leadership? If you look at the testing and rollout you’ve planned, and think about ways in which the system might fail, and you can sincerely tell your boss’s boss’s boss that you’ve made reasonable tradeoffs even if something goes wrong, then you’re good to go.

It’s a balance, like all things. There are probably situations where there is an urgent reason to metaphorically scuba dive that is worth the unknown-unknown nature of the risk, even with high stakes. People’s intuitions and risk-friendliness also vary based on personality, and how they’ve seen things fail in the past. A lot of growing as an engineer is fine-tuning that initial response to design decisions. Now when I see code that has a lot of side effects, that relies on implicit state, or that is clever but not clear, it makes me uncomfortable, because I emotionally remember having to debug similar issues in the post. Sometimes the situation is different, and the scary things really are justifiable, or less scary. That discomfort is worth exploring with an open mind, rather than fighting the last war over and over. Going forward, I hope to expand my intuition for system design as well. 

