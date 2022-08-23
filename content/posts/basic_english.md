+++
title = "Basic English"
author = ["Walker Griggs"]
date = 2022-08-03
draft = false
creator = "Emacs 28.1 (Org mode 9.5.2 + ox-hugo)"
weight = 2001
+++

> It takes only 400 words of Basic to run a battleship; with 850 words you can run the planet.
>
> Ivor Armstrong Richards

I'm terrible at learning foreign languages. In fact, I studied latin for 8 years -- a dead language for all intents and purposes -- and hardly remember a thing. Recently I tried learning Italian; that fell by the wayside too.

My experience with foreign languages could probably be summed up in one word: overwhelming. Gerunds and gerundives. Participle. Present perfect imperatives. Yet, somehow, there's a sizable population of polyglots out there who learn languages, or at least the basics, in just a few weeks. How? Enter: Basic English.

Basic English is a controlled language, or a whittled down version of a language meant to reduce complexity and improve comprehension. Charles Ogden and Ivon Richards designed Basic English as a tool for those learning English as a second language. Odgen believed that the fastest path to become conversational in any language was to learn only the most used words.

Of the hundreds of thousands of words in the English language, Basic is only 850. Britches, breeches, bell-bottoms, blue jeans -- who cares, so long as you can say "pants".

Of course, this got me thinking about my experience learning to code, or work with computers more generally. Honestly, Basic English is not far off.

In highschool, we wrote hundreds of lines on paper well before we typed a single character into a text editor. Before we learned loops, we learned about variables. Before variables: types. The syllabus was condensed to 850 words (or whatever the programming equivalent is), and we kept to it. Our diction was limited, and we drilled those core principles home.

Jump forward however many years, and my experience learning Rust was vastly different. I dove straight into traits and borrowing and async, and I ultimately failed to learn the language. I don't know Rust any better than I know Italian. I didn't limit myself to 850 words.

My initial revision of this essay proposed (or at least attempted to) a model to evaluate programming languages. My reasoning was that math, philosophy, and computer science are fundamentally just syntaxes to express logic, arguments, and reasoning. A well designed language, so I reasoned, <span class="underline">wasn't</span> a language with many bells in whistles. Instead, it applied routine, boring, consistent, trite syntax to great effect.

That train of thought is a logical fallacy though: a faulty parallel construction. Controlled languages don't help <span class="underline">evaluate</span>, they just improve legibility for non-native speakers. Rust isn't a bad language by any stretch, and English isn't either -- they're just difficult to grok for the first-time speaker.

So what can we learn from controlled languages as programmers, architects, or designers?

****1) Learn slowly to learn quickly****

What did my experience with Rust teach me? You're never too experienced, smart, or savvy to start from square one. The core contributors of Rust literally wrote [a book on getting started](https://doc.rust-lang.org/stable/book/) for a reason.

This takeaway is the more obvious of the two, but we willingly walk ourselves into a trap when we jump straight to complex features, patterns, or idioms. We push well past those 850 words, and sabotage our learning process.

****2) Simple code is empathetic code****

I <span class="underline">love</span> writing list comprehensions in Python! My caveman brain releases endorphins when I realize how much I can do in only one line. Paradoxically, though, list comprehension can be... incomprehensible.

We need to write code with the understanding that someone in a galaxy far far away will need to read it.

In my case, maybe that person is a colleague who isn't familiar with Python. Maybe they're a contractor who knows Python, but it's been a while. Or maybe I've switched companies, and am not around to answer their questions. By saving myself a few keystrokes, I've cost someone valuable minutes; I'm not respecting their time.

Of course, list comprehension is a small example, but this principle applies just as well to complex patterns and sprawling systems. Simplicity is empathy.

All in all, controlled languages are an interesting theory and intuitively make so much sense. I likely won't be fluent in Italian any time soon, but I'll certainly remind myself to slow down and keep it stuipid simple. I might even revisit Rust and do it right this time.