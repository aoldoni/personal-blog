---
title: "Basic principles when managing ongoing tasks"
description: ""
tags: ["team management", "project management", "team leadership", "software engineering management"]
categories: [ "Management" ]
draft: false
date: 2018-03-25
---

## Software is complex

Building new software is challenging[^1]. Even when you take examples considered simple, such as building a new website or a basic CMS, note how they will likely contain at least one of the following:

- Since technology moves fast, you will need to include a new piece of software or code library that you never used before;
- Even for the code dependencies you used before, there are now new versions that you will likely need to utilise instead, generally for performance or security reasons. It means that, since you never used this newer version, there might be unknown code changes in it that affect system behaviour;
- Because of normal team rotation, you are likely to be working with engineers, UX/designers and other people within the company that you never worked before;
- The client or sponsor of the initiative, either internal or external, is likely different or a new person that you might not have previously interacted with;
- You were assigned to build this new software because no one ever built it before, otherwise the existing software would obviously be already in use. This is uncharted territory by definition, and will inherently incur in more uncertainty. It also means you are tackling a technology or a business domain that is not necessarily well defined yet.
  
The above list is incomplete, but each of its elements above builds up on the level of risk. It calls for the manager of this project and its team to respect its complexity.

It is important to characterize the mindset needed for this role (either called a project manager, or a software engineering manager, or, in a more general way, a team leader) as it is pinned by this underlying responsibility of making this complexity clear to the relevant people, and to minimise it in a continuous, restless, graceful and structured way.  


## The loaded context and the larger goal

During the execution, there is at least 2 contexts in which management needs to be paying attention to. In an analogy with how computers work, one context is what I call the _loaded context_, which is like the RAM (Random-access memory) of the project. It includes the tasks you are working on right now (at this very moment), and it's mostly fresh in the team members' brains.

The second context, or the _larger goal_, means planning or assessing things that we will have to do, but are sitting in the HD (Hard Disk). Conversely it could be a task in the backlog that needs to be done, but not currently in execution. It may also include things like anticipating information for tasks that you are still not working on, or making sure that the general need for the software you are building still exists.

These contexts require different techniques and approaches to be managed properly.


## Some things to watch for when managing the loaded context

So, as discussed, the loaded context is what you are working on right now, or your ongoing tasks. In order to tackle its complexity, beware with the following.
  
  
#### Words to never say

I just recently read the book **Rework**[^2], by Jason Fried and David Heinemeier Hansson from Basecamp, and noted how, amongst dozens of other knowledge pearls, it beautifully list the "words you should never use in business". I will take a few sentences from the book and group them together in the single quote below:

> They're **_need_**, **_can't_**, **_must_**, **_easy_**, **_just_**, **_only_**, and **_fast_**. And also watch out for their cousins: **_everyone_**, **_no one_**, **_always_**, and **_never_**. **_ASAP_** is poison.

Clearly saying things like "Everyone will need this feature" brings a sense of absoluteness that might not be exactly true in practice, or might not be corroborated by data. It goes against focusing on what's really needed. Moreover, saying something like "Can you add this feature in? Just this time, it's really fast!" can become addictive and cause scope creep to quickly become a problem. It shows that there is no prioritization. One might even go further and say that this is an "easy" task, but, as also noted in the book, what is "easy" anyway? The definition differs for everyone.

These words go against the objective of a team manager, as it adds to the existing complexity instead of removing it. It brings imprecision and bluntness to a context in which reality is already sufficiently rough.
  
  
#### Issues need owners

Another point is that a manager needs to be constantly paying attention on where the team spends their time on, and what are the issues affecting productivity.

A behaviour that I observed as damaging is when problems are discussed only in an informal way. These can be conversations between engineers in which they clearly describe an ongoing challenge, most of the times even proposing a vision on how it could be resolved, but end the conversation without formally raising the issue. This is done with an implication that _someone_ will eventually resolve the problem. The manager needs to make sure these issues affecting the team are properly registered, and then assign them to an owner and give it proper allocation.

For an example, suppose someone says: "We've just set this database up with all-default settings and, at some point, before we go to production, someone needs to fine-tune and optimise it". This information _as it is now_ is, unfortunately, just adding to the complexity of the project. It will likely not get tackled if not rapidly recorded and tracked. If it is captured then it can be assigned, properly planned, and subsequently followed up for completion.

Tying with the previous point, the word "ASAP", in my experience, is unfortunately used a lot in this context. Imagine the confusion of having an issue that is mentioned to be resolved "ASAP", but is not even recoded somewhere, and has no owner.
  
  
## Final notes

I strongly recommend reading **Rework** for more insights on building software products. It is a "classic" that I took a while to get my hands on, but that is constantly mentioned in any technologist reading list. Not everything in there struck me as correct in general, but it is clear that it worked for Basecamp (or 37signals, at the time) and describes their culture in practice.

[^1]: https://en.wikipedia.org/wiki/Software_entropy
[^2]: https://basecamp.com/books/rework