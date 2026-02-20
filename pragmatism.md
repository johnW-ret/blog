---
title: "The Pragmatic Case Against \"Pragmatism\""
description: "Humility over pragmatism"
createdOn: 2025-06-04
---

One of my least favorite programming "ism"s has got to be pragmatism.

Invocations of pragmatism are so prevalent because they beseech the listener to consider the reality of the world we live in: trade-offs. You might want that RTX 5090 for your next PC build, but it is going to cost you.

Pragmatism offers a level of sophistication above the base form of argument that pervades the comments of Hacker News and Reddit: preference. "We have considered the available options and decided that X best fits the project's requirements" sounds a lot better than "Y sucks" in a PowerPoint presentation.

To be clear, there's nothing wrong with pragmatism per se. I would much rather be part of a system migration lead by an executive who thinks they make pragmatic decisions than one who makes decisions on a whim.

The problem with pragmatism, as often applied, is that the speaker assumes a sufficiently clear view of the problem space and uses the sophistication and reasonableness of "being practical" to "skip a step" to assume their presuppositions are generally applicable principles.

Pragmatism works very well in cases where the problem space is fairly settled because one's decisions can be better founded in logic, rather, there are fewer subjective matters to be "pragmatic" about. Consider the decision of which collection data structure to employ. The "pragmatic" approach would be to choose the structure whose performance characteristics best conform to the requirements. Think "time-complexity vs space-complexity with respect to hardware constraints", "maximum possible response time in a mission critical system", etc.

While "developing your own data structure" might be necessary in some cases or while trying to do so could be an informative exercise, it would rarely be considered a "pragmatic" use of time or resources. As there are proofs about the lower-bounds of time complexity possible for certain popular algorithms, it likely wouldn't be a logical decision in most scenarios.

Problem spaces with fewer settled facts allow for more subjective judgement. Humans don't like subjective judgement, so this creates a desire for impassionate, pragmatic arguments to fill the void, even if the number of possible solutions is enormous.

Consider the popular truism: "dynamically-typed languages are best for rapid prototyping  and statically-typed languages are best for slow and careful development". There's something about this statement that feels like a fundamental principle if all you have written is C++ and Python. You have to write the target data serialization class, or you don't, right? That is... unless a compiler writes one for you using a schema from real data with something like a C# source generator, an F# type provider, or a Template Haskell library.

What about domain modeling? Expressing program state and choices via unions and records in a language like TypeScript might seem like a nice idea in principle except for all the tags you have to write to differentiate object types due to duck typing. If all your program state is defined by switching on `string`s anyway, why not just slap attributes to dynamic objects or entries to dictionaries directly? If the only language you've used with abstract data types doesn't support nominal typing, if you have never seen a

```fsharp
type LoadStatus<TData, TError> = NotStarted | Progress of double | Success of TData | Failed of TError
```

, this is a perfectly reasonable thing to think.

Furthermore, languages like Python and LISP that offer an interactive development experience through REPLs used to solely be associated with dynamic typing. The only compilers that were fast enough to support an interactive development experience were interpreters, which themselves were not fast enough to support static type checking on every evaluation. Today, this has largely changed, where even languages like C# which were never designed to be run in an interpreter can be run in a REPL or notebook with reasonable performance characteristics.

To be clear, no matter how much I really, really think this means you should try F# at least once, none of this is truly dispositive for which language you should use for which task. If you only need a single property out of a JSON object, you don't need a type provider. But the idea that the evidence is not always dispositive and the choice is not obvious is kind of the point. You won't know about these features (and many more I have not listed and myself don't know about due to my inexperience) unless you look for them.

You can grok research with AI, but it will recommend the most mainstream tools by design. That's not necessarily a bad thing - as the most mainstream tools are often the most broadly suitable and reliable due to their ubiquity. But what's more important than becoming aware of a tool is knowing what works well, if it's suitable for your use case, and what the _pain points_ are - things that are hard to know until you _try it_.

I'd like to emphasize that [I do value historical precedent](https://en.wikipedia.org/wiki/G._K._Chesterton#Chesterton's_fence). Building a new system from scratch to replace an old, terrible one can - in some cases - lead one to acquire a new respect for the choices made by the designers of the old system. But the key element that lends following the tried and true its prudence is _humility_ - the humility that there are innumerable things out there that don't work but at least one thing someone in the past found that did.

That humility is often what's lacking in invocations of "pragmatism". The passionate advocate lays bare their biases and preferences. The pragmatist hides their biases and assumptions as settled principles which become the assumed presuppositions to their analyses.

"Pragmatism" is one of those words has an _unassailable positive valence_ ([Thanks ChatGPT!](https://chatgpt.com/share/683fb625-fee0-800c-8651-0dc86b155c98)) like "realism" or "common sense" that has a sort of 'non-falsifiable' element to it because - after all - who wouldn't want to be pragmatic? It feels a lot like how "progressive" used to be or how "modern" is effectively used as a synonym to just mean "good". Among programmers, "pragmatism" often just means "thing I like" or "the most complicated thing I am willing to try to understand".

It's very easy for me to get all high-minded about this concept - but it's actually something that's very natural. I once asked someone who wrote a book on immutable architecture why he uses C# over F# and he responsed with something to the effect of "C# has everything I need so I don't really need F#". I really like F#, but something about higher-kinded types and type classes rub me the wrong way as pointless abstraction. It also just so happens that I tend to write a lot more F# than I do Haskell. I am sure there are some Haskell programmers out there who grumble about how few people try Haskell but think of Idris and Lean as esoteric research projects.

The point is, the tendency to have high affinity for the things we're familiar with but feel alienated from the things we aren't is an impenetrable human characteristic. In general, it's fair to assume that people don't keep using things that suck, so it's not too crazy to assume that the most mainstream things are things that suck the least (do not let JavaScript developers read this sentence). This isn't a horrible heuristic for landing on something that's "just OK" for most occasions, but sometimes the "just OK" thing sucks a little bit. Eventually that suck-yness will gnawl away at a pragmatist's soul and make them a ["pragmatist in pain"](https://ericsink.com/entries/fsharp_chasm.html#:~:text=Moore%20calls%20a%20%22-,pragmatist%20in%20pain,-%22.), searching for a cure. Sometimes there is [a readily available cure](https://www.postgresql.org/docs/current/functions-json.html) for [the pain](http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/), but sometimes the situation is much more complicated than it appears on the surface.

So if you shouldn't be a pragmatist, then what should you be? A less charitable interpretation of this argument is that "you can't 'know' anything and can't form judgments about what you haven't tried". Certainly, it's possible to relate things you have experienced to things you haven't and do some intelligent guess work to fill in the gaps. If you had to try everything before making a decision then you'd never get anything done! As stated above, there's nothing wrong with pragmatism per se, but this is where humility is really important. It's hard to know whether someone with strong opinions has valuable experience or not, and even if they do if they are being honest and rational or not, but people who don't have any opinions at all usually just don't know or don't care. I think the hardest part is allowing yourself to be opinionated with what you have experience in or facts to back up, while being ever mindful to let your opinionatedness rapidly recede as you approach areas you're less familiar with and furthermore, keeping an open mind to let yourself be proven wrong.

Ultimately, it's possible to take shots at one aspect of how "pragmatism" is used or instead to define it really broadly to mean "rational decision making". If you weren't pragmatic about the cost-benefit analysis of, say, practicing for the Daytona 500 on Trail Ridge Road in Rocky Mountain National Park, you wouldn't be around for very much longer. But I think it's also important to let yourself "waste time" in order to learn. If you make a generalizing statement about technology, phrases like "as far as I know" can go a long way. And if someone introduces you to a technology that violates your principle, that's an opportunity to learn.

All in all, go build what you want to build and don't let internet people and consultants dissuade you from filling the code-shaped hole in your heart that only naive discovery can fill. Happy coding!
