+++
title = "dispatch no. 001"
date = "2026-11-26"
authors = ["Logan Devine"]

[taxonomies]
tags = ["dispatches"]
+++

Well, hey! It's been a few months since I've had a chance to write anything just because I wanted to, so I felt like I'd try this new style out, thanks to a recommendation from Nex who recently [wrote something similar](https://nexy.blog/2026/09/19/nexy-digest-edition-1/).

So, hopefully every once in a while when I don't have anything else to write about, I can just give an update on what I've been up to lately. I doubt this will be _too_ regular, but who knows!

All that aside, let's get into the elephant in the room if you've spoken with me recently:

# Undergrad applications

College applications have taken up the overwhelming majority of my free time since August. People warned me about this process, but holy shit, I didn't expect it to be this much busywork.

Although the thousands of words of writing is difficult enough on its own, a surprising majority of the time has just gone to thought. Fitting my last four years of experiences, which are convoluted enough as they are, into a few short prompts that someone might only spend a few minutes reading... eesh.

I've also had to think for a substantial amount of time about what I might even want out of college. (Mainly, because one of these supplementals needs me to answer that question.) Who really knows, but I do hope to study more about the deeper systems and theoretical questions underlying the software I use and write every day.

By this point, many of my applications are submitted, but there's a few of the tough ones remaining to get done by January. I told myself I wouldn't touch the blog until all of the applications are submitted, but I suppose I lied... :P

Excited to have this all over, and yet anxious. The waiting game until March isn't going to be fun.

# Coursework

The majority of my senior year coursework has been taken at our local community college (thanks California), and it's been a mix of hell and really fun.

Currently, those two courses are General Physics I and American Sign Language Level I. On the physics side of things, that class has been a mess to deal with because of the way it's structured, and by this point in the course over 70% of the class has dropped, but I'm still managing to hang on (and doing quite well, actually)!

ASL has also been a really cool class. I didn't have much of an idea of what it would be like walking in, but it has genuinely been one of the more enjoyable courses I've taken&mdash;the instructor is the funniest guy ever. I still have to do a final paper for that class, which I'm not very excited for given how busy this coming week will be, but it's probably time to lock in there.

Finals are next week, and I am not prepared, but generally my approach here has been to YOLO it and see how I do. It hasn't failed me yet. It sure will one day.

# Matrix

Alright fine, obligatory Matrix section. I can't put it off for too long. Welcome to a soup of links...

## Presence v2

To the unaware, I've been working among [lveneris](https://matrix.to/#/@lveneris:kludgecs.com), [stratself](https://matrix.to/#/@stratself:muoi.me), and [way too many others](https://ispresencefixedyet.com/contributors) to write a series of proposals we've collectively called Presence v2.

In the last few months, I've:

- Helped author a [complementary post](https://ispresencefixedyet.com/analysis) showing how one of our proposals, [MSC4532: Revised Social Presence][MSC4532], will affect the complexity of presence distribution (seriously, check it out!)
- Successfully [landed](https://github.com/ruma/ruma/pull/2593) [support](https://github.com/ruma/ruma/pull/2634) [for](https://github.com/ruma/ruma/pull/2546) that same proposal, alongside [Selective Presence][MSC4495], and a [presence sliding sync extension][MSC4559] in Ruma!

And others have made implementations in Sable and Continuwuity! Things are moving, and it's nice to see given having published our proposals means things feel far slower.

[MSC4495]: https://github.com/matrix-org/matrix-spec-proposals/pull/4495
[MSC4532]: https://github.com/matrix-org/matrix-spec-proposals/pull/4532
[MSC4559]: https://github.com/matrix-org/matrix-spec-proposals/pull/4559

## Other spec work

Besides Presence v2, I've got a few other proposals which need care and watering from time to time. Recently:

- I've got a [unmerged implementation](https://github.com/gomuks/gomuks/pull/744) of [MSC4441: User Profile Annotations ("Profile Notes")][MSC4441] for gomuks web, which has been running as part of my patches and been really sweet to use! That proposal also had all of its encryption capabilities split out into [MSC4483].
- I've also been working on implementations for [MSC4527: State event power levels][MSC4527] and its "alternative," [MSC4583: Minimum power level to send state][MSC4583].

[MSC4441]: https://github.com/matrix-org/matrix-spec-proposals/pull/4441
[MSC4483]: https://github.com/matrix-org/matrix-spec-proposals/pull/4483
[MSC4527]: https://github.com/matrix-org/matrix-spec-proposals/pull/4527
[MSC4583]: https://github.com/matrix-org/matrix-spec-proposals/pull/4583

## Introducing Nebu!

I've yet to really announce this publicly, but a few months ago I also started working on [Nebu](https://github.com/matrix-nebu/nebu), which is a new work-in-progress TypeScript SDK for the client-server API. Nebu aims to bring a level of structure familiar to Discord.js users that I personally didn't believe other SDKs provided, alongside eventually providing a higher level piece-based bot framework along the lines of [Sapphire](https://sapphire.js.org/). The goal here was to create such a framework, but we realized we needed a better SDK to make that possible.

Right now, Nebu has a well-designed REST handler, and can send messages. There's also a work-in-progress sync engine, although it isn't very useful yet.

Once I get a little more free time, I plan on closing up the first phase of the roadmap and actually announcing the project. In the meantime, take a look, contributions & discussion are welcome in [#nebu:zirco.dev](https://matrix.to/#/#nebu:zirco.dev)!

# Zirco

On the [Zirco](https://zirco.dev/) front, it's been much quieter, but some long needed changes have been made:

- libzr's legacy `shl_X` and `shr_X` bit-shift functions were [dropped](https://github.com/zirco-lang/zrc/pull/786).
- We switched to [LLVM 23](https://github.com/zirco-lang/zrc/commit/64c820e66ff11ab0bf32cdc13215ddb61712d972).
- I finally _actually_ fixed aarch64 builds in CI.
- And, for the Nix consumers, there's now a [binary cache by Cachix](https://github.com/zirco-lang/zrc/commit/a6c0548c939f3cda043507c6d2b6696225c9aabe)!

# Infrastructure & Nix

## New Borealis

Remember when I was [raving about Debian alongside Nix being the best](/posts/debian-plus-nix-stability-meets-reproducibility/)? Well, I lied.

Recently, I switched both of my home machines, Redwood (workstation) and Borealis (the laptop which I call a "homelab") over to NixOS proper. Borealis has also picked up a couple new roles, notably hosting my Matrix client and keeping a database archive for my Synapse server. Another thing that was particularly tough about the migration: getting Proxmox Backup Server working again. I eventually settled on a janky OCI-based setup, but that might change to be native in the future.

It's all been quite the experience, and with my primary goal being "learn Nix better," I'd say I've attained that.

My [NixOS configuration](https://github.com/thetayloredman/nixos) repository has everything exactly as I'm running it: configuration for every host, deployment settings via Colmena, my gomuks patchset, all organized in the Dendritic pattern.

## Nixpkgs contributions

Of course, now that I'm committed to NixOS, I'll naturally want to package a few things I find missing. This has taken me down a bit of a rabbithole with Nixpkgs, where I've now made about 60 PRs. Currently, I maintain the gomuks set of packages, alongside DCP-o-Matic, Continuwuity, and a couple of the Mautrix bridges. It's been especially interesting to see the software lifecycle from a packager's perspective: sometimes, instead of "just fix the bug!" it's a game of backporting a far more complex patch.

I also participated in NixOS 26.11 Zokor's "Zero Hydra Failures" cycle this month, where contributors from around Nixpkgs work together to stabilize the release branch and ensure packages aren't broken/failing to build.

Some of the recent contributions I'm proud of are:

- Packaging [gomuks-desktop](https://github.com/NixOS/nixpkgs/pull/543940), gomuks' electron frontend
- Packaging [DCP-o-Matic](https://github.com/NixOS/nixpkgs/pull/554828), a tool for generating Digital Cinema Packages like I mentioned in my [post on DCI security](/posts/dci-security/)
- Hunting down a [long-standing typo in packaging for gccNG](https://github.com/NixOS/nixpkgs/pull/587757)
- And packaging [Nexus Client](https://github.com/NixOS/nixpkgs/pull/589765) (although this one isn't merged yet).

# Actual job stuff (I know, right)

Stuff at work has been going well, albeit chaotic. With the big releases of Doomsday and Dune coming up, I'm sure it's in to be even busier.

We've had some hardware fail here and there, which has meant some long nights, and the rain around here in NorCal has meant a lot of flooding and leaks.

I'm writing this myself on Thanksgiving day before I head in to work a double. Don't you just love the service industry?

# Looking ahead

There's quite a bit on the horizon.

Once applications season is over, although the waiting period will be a project of its own, I'm looking forward to finding out where I'll be spending the next several years. I'm also anxious, but excited, to start aiding the transition into my successor at work. The free time will be nice.

With that time, hopefully you'll see more of me here, and Nebu + my other projects can progress a bit more :)

That's about it. Hopefully I'll work on another one of these before too long!
