---
layout: post
title: Reflections on a Master’s, Part 3 - What Systems Science offers Data Science
excerpt_separator: <!--more-->
date: April 2, 2019
---

### Summary

In this third post of a three-part series, I reflect on my Master of Science in Systems Science degree. As mentioned in part 1, systems science isn’t a traditional avenue into data science, but I think it offers some distinct advantages over other fields. Here I list what strike me as three of the biggest. 

<!--more-->

------

#### Systems Science Aggregates Problems and Solutions

The general goal of systems science is a set of mathematical principles, broadly applicable to many scientific fields, which reveal insights between disparate domains. Because subjects of study differ between these domains, systems science takes the ‘stuff-free’ approach that finds commonalities related to their structure or behavior and not their physical composition. Understanding how one of these systems works means that similar systems are also understood. 


This process was described by Claude Shannon, the founder of information theory (one of the foundations of systems science), at a lecture at Bell Labs (1952) where he described his problem-solving process. One of his ideas was that of finding solved problems (P’ and S’ for the related problem and its solution) that were related to the current problem (P). Finding the exact relationship between P and P’ maps the related solution (S’) to the currently-unknown solution (S).


Likewise, this search for foundational concepts is echoed in [Data Science for Business](http://data-science-for-biz.com/) (2013) by Foster Provost and Tom Fawcett, who write that they “focus on the unifying concepts [such as expected value calculations], presenting specific tasks and algorithms as natural manifestations of them…” (pp. xiv) which is a natural and logical extension of Shannon’s thinking. This continues when they write “in 10 years’ time the predominant technologies [the * stuff * of data science] will likely have changed or advanced enough that a discussion here would be obsolete, while the general principles are the same as they were 20 years ago and likely will change little over the coming decades” (pp. 16). 


#### Systems Science Understands The World Differently


Ideas in complexity and chaos and nonlinear dynamics are generally isolated to systems science: in physics, for example, most phenomena have been historically treated as linear so they can be solved analytically; in systems science they’re modeled computationally so problems that are intractable analytically are straightforward to work with. Some categories of modeling frameworks are agent-based modeling, where the interactions between individual agents provide nonlinear, ‘emergent’ behavior; discrete event simulation, an operations research technique where events are drawn from probability distributions and are used to determine system performance; and system dynamics, a conceptual and computational framework where feedback loops are identified and their impact on behavior is identified. These models can be used to get insight about customer behavior and product adoption, for example. 


#### Systems Science Frames the Problem Context 


Data science is about generating insights from data and then acting upon them. Foster and Provost, again: “formulating data mining solutions and evaluating the results involves thinking carefully about the context in which they will be used” (pp. 15).   Not enough time is spent by data scientists thinking about the context of the problem, but this is a crucial step. For the work to be successful, the model needs to be built, but this model needs to be relevant, usable (can be used in production), and useful (meaningfully addresses the problem). 


Systems science offers relevant advice to all these issues, and most especially the third one. Systems thinking recognizes that cause-and-effect can be separated both by time and space, that the effects of an action take time to manifest (systems are slow to change), that nonlinear effects are more common than linear effects (a small change in input typically results in a non-proportional change in output). For example, a systems idea is that 

> Every action has side effects (it’s never possible to do just one thing)

This suggests that even relevant, usable, and useful models can still produce bad behavior. One example of this is the now-growing importance of equity in data science (without being careful, models can continue the pattern of discrimination as a side-effect of their decisions, e.g. Cathy O’Neill’s book [Weapons of Math Destruction](https://weaponsofmathdestructionbook.com/)). This principle also suggests that the implementation of an algorithm can change the patterns it identifies (for example, fraud changes as the algorithms to detect it change). 
