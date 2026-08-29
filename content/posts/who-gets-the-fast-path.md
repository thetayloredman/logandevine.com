+++
title = "Who Gets The Fast Path?"
date = "2026-08-28"
authors = ["Logan Devine"]

[taxonomies]
tags = ["zirco", "swe"]
+++

I've been working on my hobby compiler [Zirco](/projects/zirco/) for the last few years, and one of its "selling points" for me is raw compile-time speed. Naive benchmarks show `zrc` compiles code about 4x faster than `clang` and `gcc`, while still using LLVM to produce binaries within 10% of C's runtime performance.

This is ironic, considering the zrc codebase is not designed great by any measure: the codebase is full of deeply nested structures and liberal allocation. Fortunately optimization can be external to the program itself, and we of course build the Rust code with aggressive optimization, a single codegen unit, and fat LTO. Naturally the next step is to make zrc _even faster_ using Profile Guided Optimization (PGO) and BOLT.

PGO removes a lot of guesswork from optimizations. Instead of guessing which parts of the codebase run regularly (the "hot" functions or branches), instrumentation and benchmarking is used to collect data on exactly how the program executes against some workload. The compiler can use that to optimize for those specific code paths, and you now have faster code with less guesswork. For a compiler however, determining what counts as an appropriate workload is the hard part.

# Follow the fast path

For the computational beasts they are, CPUs are shockingly good at guessing. Almost all modern CPUs contain mechanisms for branch prediction and speculative execution ([as good as they may be](<https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)>)). Compilers are aware of this, and arrange code so that the CPU is executing the most common instructions close together, often even linearly. This allows streamlined execution which reduces costly cache misses and mispredictions.

The next step up is to hint to the compiler what branches are frequent: Rust has the `likely_unlikely` experiment which provides functions like [`std::hint::likely`](https://doc.rust-lang.org/std/hint/fn.likely.html), and has stable support for [`std::hint::cold_path`](https://doc.rust-lang.org/std/hint/fn.cold_path.html) and the `#[cold]` function attribute. C similarly has `__builtin_expect`, and Zirco even has [the `unreachable` keyword](https://book.zirco.dev/reference.html#615-unreachable-statements) to indicate a section of unreachable code.

The difference can be substantial, but these optimizations should not be applied without proper benchmark data to back them, especially as a limited set can even introduce unsoundness if they are incorrect. In the case that the compiler is not adequately guessing the runtime behavior of your code, PGO steps in to allow real runtime behavior to make the same hints. BOLT takes this a step further by rearranging the final binary to push cold sections away from the most important code. The difference can be substantial: enabling PGO/BOLT in Zirco produced roughly a 20% improvement in compilation speed.

# Runtime information as a datasource

Static analysis can only do so much.

PGO changes that: you build an "instrumented" binary, run it on a workload (say, a benchmark or test suite) and then recompile the code using generated profiling information. For a simple web service, this is pretty easy: the "representative workload" is real user traffic, and the profiles can even be collected directly from a production server.

The question for Zirco however was, what counts as a representative workload? This is far less obvious. You could use a specifically designed (or even generated) set of programs that target the most complex portions of the compilation pipeline. You could compile Zirco's standard library. You could run the test suite. All of these target something in specific.

The problem is similar to "overfitting" in machine learning. The moment you are optimizing using PGO, you're biasing runtime speed to a specific "target audience" per se. Using the test suite for example, might bias optimizations in favor of small programs and startup code, as unit tests tend to be.

# There isn't one workload

Optimization is no longer deterministic transformation, it's now a product decision. _Who do I want to benefit most?_ And not just that, _who am I willing to incur slowdowns as a result of this optimization?_ Behavior differs between developers and environments: some might be compiling regularly as a result of a LSP integration, one might be building in CI, and one might just be writing small example programs to try the language out.

These are all valid use cases, and the issue is, not everyone wins.

For zrc, I started out by using the examples, the test suite, and the standard library. I felt like this gave adequate coverage: a mix of complete programs (the larger examples like `calculator`), smaller samples meant to demonstrate specific language features, alongside a comprehensive set of possible entrypoints. Again, however: this will vary for anyone who enables these optimizations. Always benchmark your code!

# How I actually implemented PGO/BOLT

[PR #776](https://github.com/zirco-lang/zrc/pull/776) enabled PGO, using [cargo-pgo](https://github.com/kobzol/cargo-pgo).

For many, this will fit pretty trivially into your workflow:

```bash,linenos
$ cargo pgo instrument build
$ cargo pgo instrument test  # run the test suite
$ ./target/x86_64-unknown-linux-gnu/release/zrc ./work
$ cargo pgo optimize build
```

And for BOLT, that last step is replaced with a _second_ BOLT optimization step:

```bash,linenos,linenostart=3
$ cargo pgo bolt build --with-pgo
$ ./target/x86_64-unknown-linux-gnu/release/zrc-bolt-instrumented ./work
$ cargo pgo bolt optimize --with-pgo
```

# CI has become a factory

With a full PGO+BOLT release build, the code is now compiled 3 times. Once to instrument for PGO, once to build a PGO-optimized binary instrumented for BOLT, and once again to build the final binary. I consider this worthwhile for the 20% speed boost, but this meant my slowest release build (Intel macOS) had tripled in length, bringing my pipeline up to 23 minutes. For a simple PR, I felt this was unacceptable, so I altered my pipeline to only build with these heavy optimizations for nightly builds and final releases, instead of as part of the normal CI pipeline.

# A note on benchmarking

Remember the overfitting problem from earlier? The same is true with your benchmarks. Don't let your PGO workload be the same as your benchmarks: it may overestimate just how much improvement you're getting. In fact, if you press too hard to optimize against your benchmark, you might even slow things down for the average user. The 20% measurement in speed increase was done on something entirely different than the examples and test suite we optimize for, as it was a full length program for a prime number sieve.

It is easy to mislead yourself and your team: be careful as to what's being benchmarked and make sure you have some real user benchmarks that aren't in your profiling workload.

# Who gets the fast path?

Ultimately, this is what makes PGO so interesting (and difficult) to adopt. Unlike mechanical optimizations such as most loop unrolling or function inlining, it requires choosing exactly who your program is for, and intentionally biasing performance in that direction.

There is no one fast path, only those which you decided were worth speeding up.
