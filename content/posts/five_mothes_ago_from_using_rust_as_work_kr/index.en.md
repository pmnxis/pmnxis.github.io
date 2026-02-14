---
title: "About Five Months After Using Rust at Work"
date: 2022-11-27T01:05:52+09:00
draft: false
categories: ["Etc"]
tags: ["회고록", "Rust", "Korean_Article"]
---

I have been using Rust at the company I recently joined. Five months have passed since I switched jobs, and I would like to describe what I have felt so far.
Setting aside the detailed syntactic advantages, I will simply describe my general impressions.

## Pros
1) If there is a developer who knows the domain well and at least two people conduct thorough code reviews for each other, you can code safely.
2) Compared to C, there are many conveniences.
3) As someone who only worked with C and firmware, Rust feels more familiar compared to other modern languages, and most behaviors/designs feel reasonable.
4) In my personal opinion, Rust can be applied to many areas except frontend. [Firmware, OS-dependent utilities, backend]
5) You continuously encounter study/challenge opportunities related to pure CS, rather than just business logic.
6) Regardless of what target architecture (CPU, Operating System) comes along, it is very convenient to adapt.

## Cons
![Menuconfig Rust Support](img/five_mothes_ago_from_using_rust_as_work_kr/entire_stack.jpeg)
> Can we go beyond FullStack and become EntireStack developers?

1) Each difficulty point fundamentally requires a large base of knowledge.
1) Occasionally, the study/challenge demands spill over into personal time. [This is not a con for me, but I think some people may feel it is.]
3) There are significant limitations when hiring.
4) When asked "What are the advantages of this language?", the scope of required knowledge inevitably becomes very broad. And that scope of knowledge may lie far beyond most people's areas of interest.
5) If you use it in embedded and look at the domestic developer pool, it is difficult to find embedded developers who will use Rust together, and the range of additional skills required of embedded developers grows almost exponentially. Even without considering Rust, the pool of embedded developers is simply too small, although embedded development is not my primary work.
6) The chip I want to use always has ambiguous Rust embedded support. (The answer to this is for me to contribute myself.) The community is still more focused on chips suitable for toy projects rather than chips that are practical in terms of cost/lead time during the chip shortage situation.

## Miscellaneous
1) I was very strict about code reviews at my previous company and was worried about how things would be at my current company. My initial impression was that since Rust's syntax is more sophisticated than C, individual coding styles varied too much, and I thought this would be a bottleneck during reviews.
However, clippy handles a lot of that, and as long as there is typo checking and a reasonable level of agreed-upon tests, reviews are not a problem.
2) We can naturally have healthy discussions about CS topics with each other.

## Proper Support for Multiple Architectures
![Menuconfig Rust Support](img/five_mothes_ago_from_using_rust_as_work_kr/some_cross_compile.jpeg)
 In theory and concept, pure **interpreted languages** are advantageous for multi-platform support. However, my actual experience with Rust has led me to feel differently.
No matter how theoretically advantageous **interpreted languages** are supposed to be, for truly and properly supporting all architectures (various CPU architectures and multiple OSes), Rust was extremely convenient.

 First of all, properly supporting all architectures is one of the key values of **systems programming** or **firmware programming**. So what languages were traditionally used for this kind of programming? That would be **C** and **C++**. However, to quickly get started (`Getting Start`) with these languages, you first had to set up `Makefile` or `CMake` configurations, and every time a new architecture was added, you had to accommodate it. On top of that, setting up the compiler, development environment, and libraries for each architecture was a separate task entirely.

 Let me compare with another language. Currently, almost nobody compares **Go-Lang** and **Rust**, but five years ago, many people did. When comparing the systems programming domain only on top of an OS, both are excellent languages. What I am about to say is quite a stretch, but in Rust's conceptual `no-std` scenario -- where there is no OS or the OS is very different from a typical one -- it is difficult to adapt. [The author previously worked in firmware development and is also evaluating whether production-grade firmware development is feasible.]

## Rust Is Difficult for Quick Prototyping Right Away
![Rust Learning Curve](img/five_mothes_ago_from_using_rust_as_work_kr/rust_difficulty.png)
 In the previous sections, I praised **Rust**'s advantages, but in this section, I will describe some slightly disappointing aspects.
 Rust is a difficult language to write. More precisely, it is very difficult to develop in a Rust-like way that maximizes Rust's strengths.
 Realistically, it would be very difficult with any language to fully leverage its strengths during development. However, if **Rust** is chosen for commercial purposes (as a language/framework for a company/development team), personal preferences should be set aside in favor of the company's perspective.

 > 1) Is it a medium that can realize what we want to develop?
 > 2) Can we adequately recruit developers?
 > 3) How much development time does it require?
 > 4) Does it run fast and correctly?
 > 5) Do the team members want to use it? (In other words, preference.)

 **Rust** is likely to score low on items **(2)** and **(3)**, and I believe this is absolutely the case for **(2)** in particular.

 If **Rust** must be chosen despite these drawbacks, it would largely be due to **(4)** and **(5)**. In that case, the development team and developers would need tangible events or results that highlight **Rust**'s advantages in order to sustain its use, whether voluntarily or otherwise.

 _Even though you can just use whatever language the company assigns, if you have a choice and want to keep using **Rust**, you would want to highlight its merits. (This is a somewhat difficult topic to articulate, as it overlaps with emotional territory.)_

 Then, to develop in a way that maximizes **Rust**'s strengths, a great deal of knowledge is required. In other words, the learning curve becomes steep.
 Depending on how you look at it, this can be either digging your own grave or creating an opportunity.

## But an Opportunity to Gain Tremendous Intellectual Value
![Example of Modern Memory Arch - NUMA](img/five_mothes_ago_from_using_rust_as_work_kr/NUMA.png)
 When C was first created, concepts like `multi-processing/processors` (SMP), caching, and `GPGPU` did not exist, and in many details, it was an era with a different memory model from today's. And to this day, C is used to handle these aspects. When you find yourself in a situation where you must develop with these concepts in mind, there are many difficulties.

 However, **Rust** has infrastructure in place to overcome these challenges to some degree, with room for more to come. And while this is an ambiguous statement, through **Rust**, it feels like opportunities are created to more closely examine the difficult architectural designs of computers and operating systems, albeit indirectly. This can be gained through **Rust** compiler's safety constraints and warnings, and I believe it is also driven by the influence of a community populated by many expert-level systems programmers. Speaking a bit more about multi-platform support, I also believe that the strength of the community is why multi-platform support is handled so well.

## When Can We Say We Have Become Proficient Rust Developers?
![When Is Good Time For Say Rust Engineer](img/five_mothes_ago_from_using_rust_as_work_kr/whenWhenWhen.jpg)
 People occasionally say things like "I am a **____** developer." So when can we say that we have become proficient **Rust** developers?

There is no definitive answer.

However, in my personal opinion, if you can explain the advantages that a given framework or language provides in a way that others can understand, then perhaps you can call yourself a **____** developer.

But unfortunately, with **Rust**, this is very difficult to articulate. You have to start by explaining ownership. And to do that, you first need to understand the stack and heap of a running process.
> Reference: [4.1 What is Ownership - The Rust Programming Language Korean Translation](https://rinthel.github.io/rust-lang-book-ko/ch04-01-what-is-ownership.html)

Because of this, you either need the ability to explain a great deal to the person you are trying to convince, or the listener must be at a high level, requiring precise and very deep explanations.

It is unfortunate that the difficulty level of explanation starts at **HARD MODE** from the get-go, but we do not develop alone. If we truly want to submit a Pull Request, I think it is necessary to explain well to the reviewer, or to write code in a way that makes explanations easier.

Through that process, you can build the knowledge base and communication skills needed to explain things,
 and going further, I think you can develop the ability to explain any given technology as described above.

The next time (or the time after) I write a retrospective, I plan to cover the actual gains and losses from using **Rust** at work.
