# History and Use of Sagas in React-Redux Applications
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [History and Use of Sagas in React-Redux Applications](#history-and-use-of-sagas-in-react-redux-applications)
	- [History of the term "Saga"](#history-of-the-term-saga)
		- [Links](#links)
	- [Early Use of the term "Thunk"](#early-use-of-the-term-thunk)
		- [Links](#links)
	- [The Trouble with Thunks](#the-trouble-with-thunks)
	- [What Are Sagas Going to Do For Us?](#what-are-sagas-going-to-do-for-us)
	- [The `redux-saga` Layout and Idioms](#the-redux-saga-layout-and-idioms)

<!-- /TOC -->

A brief talk about the history of the terms we throw around to describe these patterns, and how to use them in the context of our React-Redux browser application.

## History of the term "Saga"

The first known use of the term "saga" to describe a software pattern comes from a 1987 ACM paper by Hector Garcia Melena and Kenneth Salem of Princeton, where the term meant simply "LLT" or "Long Lived Process":

> As its name indicates, a long lived transac- tron 1s a transactlon whose execution, even without interference from other transactions, takes a substantial amount of time, possibly on the order of hours or days A long lived transac- tion, or LLT, has a long duration compared to the malorlty of other transactions either because it accesses many database obJects, it has lengthy computations, it pauses for inputs from the users, or a combmatlon of these factors Examples of LLTs are transactions to produce monthly account statements at a bank, transactions to process claims at an insurance company, and transactions to collect statistics over an entire database [etc.]

By "Sagas",they were referring to a low-cost version of the these days-long processes. Poetically, "saga" like a Homeric epic.

It was relatively recently, but forever ago in our terms, that the author of Redux-Saga connected the pieces with his previous CQRS-ES experience with Sagas managing a large dataset and applied it to a Redux middleware (the original conversation is available on Github).

The borrowed term means exactly the same thing; using Javascript generators and a UNIX-like vocabulary (`fork` and `spawn`), manage long-running Redux processes with a light-weight as possible idiom (lighter than observables/rx, at least).

### Links
* ["https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf"](SAGAS)

## Early Use of the term "Thunk"

Thunks similarly have a completely independent history that predates Facebook. The term "thunk" is as old as functional programming.

As Wikipedia says (at the moment), thunks are "a subroutine used to inject an additional calculation into another subroutine [...] primary used to delay a calculation until its result is needed [...], which is exactly what we (ideally) use it for; instead of paying the total upfront cost of computation, we delay the function's execution until needed.

Btw, its total age is unknown, but its use has been seen at least back to the creation of Algol 60 (and the BNF).

### Links
* [https://en.wikipedia.org/wiki/Thunk]("Thunk", Wikipedia)

## The Trouble with Thunks

Thunks are handy and a fundamental unit of function programming, but there is one problem with their use in Redux. In fact, this problem led Dan Abramov and company to remove thunks from Redux and place them in a separate library; they were always meant to be a quick way to get started and were never meant to scale up.

The difficulty is that there is no built-in method for thunks to communicate with each other. Therefore, we must either invent or add a library to "stitch" together when they should be parallel, when they should be serial, when they should wait on a side-effect (user IO, for example), etc. Both options are less than ideal.

The Redux team always meant for users (programmers) to solve the problem with middleware, and instead of by hand (naturally), two idioms emerged. "Reactive" (the fact the React is not particularly reactive is a topic for another day) and something like IPC or busses (which became Sagas). Options like `redux-promises` were on the table, but had the same drawback as thunks: they weren't aware of each other unless you took action, which got complex quickly.

For us, the only practical solution is sagas (IMHO). Rx is much steeper learning curve with less applicability in other environments (although observables are neat).

## What Are Sagas Going to Do For Us?

This may seem clear to some of us from other projects, but it bears examination. What do sagas do to improve the codebase? What, specifically, does `redux-saga`'s API offer?

In the most abstract sense, it offers what sagas offered large (for the period) non-CRUD data stores in 1987. It allows us to decide what should be serial and what can be parallelized (a _critical_ question with the pattern of bugs that we see), the ability to wait for external conditions (customer or vendor IO), the ability to _cancel_ (!) (something Promises can't offer us), and tools to manage operations with need a long-running overseer, such as routing, network IO (i.e., "API"), and even potentially authn/authz or other processes which require retries.

One more level of abstraction down, because its API is modeled on a familiar paradigm (`fork`/`spawn`), we'll have an easy way to manage communication between concurrent processes (something like IPC). "Forks" even automatically unroll themselves when canceled or when one of its children throw an uncaught error. At the level of the "root" saga, we have a central view of all of our application's concurrency.

And at the most concrete layer that this presentation covers, we'll stop and prevent bugs that are _ultimately_ caused by race conditions, even though we frequently get caught in a cycle of treating symptoms rather than causes. Every ad-hoc delay (even those that BE has put in just to Make It Work) is a symptom. These bugs also create a higher-than-necessary toll on servers, where unnecessary repetition or buggy retry mechanisms spam.

## The `redux-saga` Layout and Idioms

Sagas are "first-class"; i.e., they're at the top-level and apply to the whole codebase.
