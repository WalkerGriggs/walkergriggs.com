+++
title = "Digital Homesteading"
author = ["Walker Griggs"]
date = 2021-12-08
categories = ["writing"]
draft = true
creator = "Emacs 27.2 (Org mode 9.4.4 + ox-hugo)"
weight = 2003
+++

Tech is [changing](https://trends.google.com/trends/explore?cat=5&date=2011-01-01%202021-01-01&q=%2Fm%2F011spz0k,%2Fg%2F11b7lxp79d,%2Fm%2F0wkcjgj). Whether you want to call it horitzontal scaling, scaling out, distributing, or deconstructing; the ways we structure, write, test, and deploy systems is changing.

I argue though, that the ways we learn, however, are not changing but should be. Learning the latest, individual abstraction isn't enough anymore. Side projects are often too narrow and our exposure to "enterprise systems" is limited. We, as engineers, are not adequately tailoring our "studies" to fit the needs of industry... but we could be.

I propose a form of continuous learning called "digital homesteading" which emphasizes composition and encourages self-sufficiency.


## Homesteading, generally {#homesteading-generally}

Homesteading is synonymous with the Homestead Act of 1862 which granted US citizens a western tract of 160 acres should they be willing to settle and farm the land. Generally, the US governemnt used homesteading to incentive western expansion, but we'll ignore the political machinations. Fast forward to the 1970, the Back to the Land Movement worked to similar effect, where people took up rural smallholdings in search of increased autonomy and self-sufficiency.

Homesteading, in this context, is analogous to self-sufficiency. More often than not, homesteaders were physically seperated from society or relied on a small, local community. They grew their own crops, hunted their own game, built their own shelters, and mended their own fences. If their roof leaked, they patched it. If their clothes tore, they patched them just the same. The list goes on, but in every example, they had to rely on their own intuition and experience to solve their daily problems. If they didn't have experience with a particular trade, they were pretty well incentivised to learn.


## Continuous learning, generally {#continuous-learning-generally}

As is common in ever-changing industries, engineers need to constantly onboard new material, practice our craft, and mind our information diet. Most importantly, we need to learn in ways which compliment idustry needs.

Speaking to my own experience, my typical "learning lifecycle" is fairly sequential and well deliniated. I'll think up a project which suits the some topic. If I'm feeling ambitious, I might even blend two new topics; a language and paradigm, for example. From there, I'll start reading documentation, lay the groundwork, and mould the core behaviors. More often than not though, that project ends up on the private-repo pile after a few iterations or I feel sufficiently versed on the topic. I forget about it, and move on to the next topic.

I'd be willing to bet this pattern is pretty common. This method isn't "bad" or ineffectual, but there are some areas for improvement.

First, like cramming for a test, we don't retain a lot of that info. I'm especially guilty of searching through old projects for a pattern or practice I found useful, but couldn't reproduce.

Perhaps more importatnly, these projects also exist in a vacuum. We understand the bounds of the topic in isolation, but don't always see the interaction between two systems. Think of this like unit testing vs integration testing; one isolates behaviors and mocks the bounderies, the other encapulates behavior and instead focuses on interacton.

See again: "we need to learn in ways which compliment industry needs".


## Homesteading meets continuous learning {#homesteading-meets-continuous-learning}

So far we've touched on homesteading and continuous learning in practice. Let's bridge that gap by first reviewing examples of what I consider to be digital homesteading in practice, and then using those examples to derrive a few characteristics of digital homsteading in theory.

The most approachable example is a homelab (note the shared root: "home"). An average homelab might be a few rasberry pis as "compute nodes", an old laptop repurposed as a NAS, or maybe a desktop as a router. You, as the "homesteader", might run KVM or ESXI (type 1 hypervisors) on a makeshift server. You might run Telegraf, InfluxDB, and Grafana to collect, store, and visualize hardware metrics. You might also setup a home network with Pfsense and stream movies with Plex. Slowly, you're building out an ecosystem of systems and services.

Another example. Say you're in the market for a new graphics card, but are having trouble following the various stock trackers, raffles, and notifications. You might write a web app that lets you define alerts through a simple domain specific language. Of course, your friends on discord or slack or IRC want to use that app too. Everyone loves a good chatbot and there are lots of off the shelf solutions, but maybe you want to write your own. You'll want to understand the bots failure modes, so you setup Rollbar or Sentry to error tracking. Maybe you'll even want to push soft touch alerts to your home, so you write a Philip Hue integration. The possibilities are endless.

In both examples,

-   We're building an ecosystem. We're layering services or systems which interact with and complement eachother.
-   Our services persistent, but not production.
-   The individual components span multiple layers.
-   Each service provides useful but not vital functionality
-   We're self sufficient along at least one vertical.


### Ecosystem {#ecosystem}

We're not just considering how an individual component behaves, but how multiple systems interact. Enterprise servces are transitioning to horizontal systems of scale, and we need to factor that into our design process.

Consider your digital homestead. Where is the barn in relation to the fields? The food cellar? Have we considered how the three systems work in concert? With regards to our more tangible example: have we considered how our discord bot pulls information from the web app? Are they tighly coupled? Does the webapp implement any business logic, or just expose the DSL? Do the latest stock alerts need to be persisted, or only cached?


### Persistent {#persistent}

Digital homesteads should run around the clock. According to the 2020 Stack Overflow Survey, DevOps and Site Reliability Engineers are value multipliers in enterprise environments.

> Site reliability engineers and DevOps specialists remain among the highest paid individual contributor roles. 80% of respondents believe that DevOps is at least somewhat important, and 44% work at organizations with at least one dedicated DevOps employee.

Persistent homesteads go beyond SRE though. When we take responsibility for supporting every stage of software development -- when we're product owners responding to feature requests, senior leadership driving priority, on-call operators triaging downed systems, SRE debuggig service blips, and DevOps implementing resilient runtime environments -- we're service owners.

Service ownership is overlooked in the majority of continuous learning projects, despite it being such a critical facet of successful enterprise services.


### Span multiple layers {#span-multiple-layers}

It's important to think about where and how things are run. This diversity adds perspective


### Useful but not vital {#useful-but-not-vital}

This bullet ties back to the "persistent but not production" mantra. You're only going to resent your digital homestead if you rely on it for "business critical" tasks. These systems will be flawed, they will take time, they will break, and you will need to fix them.

Hosting an SMTP server for your professional email or writing a React clone for an enterprise service is objectively a bad idea. In the end of the day, we're not looking to reinvent the wheel, but to instead understand why the wheel is fabulous, how the wheel is fallable, and how the wheel can be leveraged to great success.

If we give our homestead value, we'll stay invested. If we rely on our homestead to feed the neighborhood, we risk a famine.


### Self-sufficient {#self-sufficient}

In self-sufficiency, we find the most valuable lessons. If something isn't readily available, we can write it ourselves. If we aren't immediately sure how to write it ourselves, we can learn through trial and error.

Of course you could follow this rule to an extreme -- I'm not suggesting we write our own compilers (though you certainly could challenge yourself). I'm suggesting that in an industry of higher order abstarctions, we might consider our own Back to the Land Movement.
