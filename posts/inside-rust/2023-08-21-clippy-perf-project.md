---
layout: post
title: "Making Clippy blazingly fast by 2024"
author: Alejandra González
team: the Clippy team <https://www.rust-lang.org/governance/teams/dev-tools#Clippy%20team>
description: This is the start of a series of posts talking about optimizations done on the Clippy codebase. 
---

Clippy's codebase is designed to be beginner-friendly so new contributors can come and learn,
not optimized for performance. It was never a priority to make Clippy fast because *it never needed
to be.* But time passed and more lints were added, now we have [over 650 lints][lint_list], each one of them adding a bit of performance overhead. In big projects this can take over a minute to check all those lints, so the Clippy team have decided to do something about it.

> <span class="note"><svg class="note-icon" viewBox="0 0 16 16" version="1.1" width="16" height="16" aria-hidden="true"><path d="M0 8a8 8 0 1 1 16 0A8 8 0 0 1 0 8Zm8-6.5a6.5 6.5 0 1 0 0 13 6.5 6.5 0 0 0 0-13ZM6.5 7.75A.75.75 0 0 1 7.25 7h1a.75.75 0 0 1 .75.75v2.75h.25a.75.75 0 0 1 0 1.5h-2a.75.75 0 0 1 0-1.5h.25v-2h-.25a.75.75 0 0 1-.75-.75ZM8 6a1 1 0 1 1 0-2 1 1 0 0 1 0 2Z"></path></svg> Full disclosure</span>
>
> While some members are being sponsored by Embark Studios to continue / improve these efforts, these optimizations are and will be always free and open source. No brands attached.

The final goal of this project (and series of posts) is being able to run Clippy
quickly after save, making it a sort of IDE-companion, instead of a
totally external tool run in the CLI. Consuming less time / power on
CI checks would be a secondary goal, achieved collaterally
(but very much appreciated!).

## Careful analysis of the situation

Before opening your IDE, you must think "what is the biggest
bottleneck in my codebase's performance?". Do not start optimizing your
big functions that are only run once, optimize those small functions
that, even being small, clog your code. With some profiling we
discovered that the function [`clippy_utils::is_from_proc_macro`][from_proc_macro]
takes up *25%* of the runtime, even just being used in 32
occasions. This function will be our primary target for this period.

## Preparing strategies

The team held [this Zulip thread](zulip_thread) talking about the whole situation.

From that thread, a lot of optimization opportunities came up. It revived [this issue], and
[this PR][this_pr] not long after (maybe because of the thread, maybe not) 🎉, resulting in a significant performance boost.

## Crafting a cartography of our problem

So, we decided to focus more on this section of the Clippy project. By the end of 2024 the following goals should have accomplished the following goals:

- `is_from_proc_macro` use should be O(log n) time complexity instead of the current O(n).
- `rustc` and by proxy, Clippy, should not process allowed / disabled by default lints.
- Tools should be made to automatically benchmark every week or so the Clippy repo to check for regressions (we don't need per-commit benchmarking)

These 3 objectives are the goal for this period, the next period
an update will be released with how much performance increased from the
last update and what are our next objectives.

## A final note

The <span class="perf-project"><b>Performance project</b></span> is big, it
will (hopefully) be done by 2024 maybe it's delayed. I'll be writing
updates on this topic about every 1 or 2 months, so you (the
community) knows how it's going. Let's hope Clippy
is **blazingly fast** by 2024 🦀.

[lint_list]: https://rust-lang.github.io/rust-clippy/
[from_proc_macro]: https://doc.rust-lang.org/beta/nightly-rustc/clippy_utils/fn.is_from_proc_macro.html
[zulip_thread]: https://rust-lang.zulipchat.com/#narrow/stream/257328-clippy/topic/Clippy's.20performance/near/366555916
[this_pr]: https://github.com/rust-lang/rust/pull/114026
[this_issue]: https://github.com/rust-lang/rust/issues/106983

<style>
    .note {
        color: #2169DF;
    }

    .note-icon {
        fill: #2169DF;
        top: 2px;
    }

    .perf-project {
        text-shadow: 0px 2px 0px #ffc832;
    }
</style>