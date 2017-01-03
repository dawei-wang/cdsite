---
title: Continuous Integration
---

Combining the work of multiple developers is hard. Software systems are complex, and an apparently simple, self-contained change to a single file can easily have unintended consequences which compromise the correctness of the system. As a result, some teams have developers work isolated from each other on their own branches, both to keep trunk / master stable, and to prevent them treading on each other's toes.

However, over time these branches diverge from each other. While merging a single one of these branches into mainline is not usually troublesome, the work required to integrate multiple long-lived branches into mainline is usually painful, requiring significant amounts of re-work as conflicting assumptions of developers are revealed and must be resolved.

Teams using long-lived branches often require code freezes, or even integration and stabilization phases, as they work to integrate these branches prior to a release. Despite modern tooling, this process is still expensive and unpredictable. On teams larger than a few developers, the integration of multiple branches requires multiple rounds of regression testing and bug fixing to validate that the system will work as expected following these merges. This problem becomes exponentially more severe as team sizes grow, and as branches become more long-lived.

The practice of continuous integration was invented to address these problems. CI (continuous integration) follows the XP (extreme programming) principle that if something is painful, we should do it more often, and bring the pain forward. Thus in CI developers integrate all their work _into_ trunk (also known as mainline or master) on a regular basis (at least daily). A set of automated tests is run both before and after the merge to validate that no regressions are introduced. If these automated tests fail, the team stops what they are doing and someone fixes the problem immediately.

Thus we ensure that the software is always in a working state, and that developer branches do not diverge significantly from trunk. The benefits of continuous integration are very significant---[research](#resources) shows that it leads to higher levels of throughput, more stable systems, and higher quality software. However the practice is still controversial, for two main reasons.

First, it requires developers to break up large features and other changes into smaller, more incremental steps that can be integrated into trunk / master. This is a paradigm shift for developers who are not used to working in this way. It also takes longer to get large features completed. However in general we don't want to optimize for the speed at which developers can declare their work "dev complete" on a branch. Rather, we want to be able to get changes reviewed, integrated, tested and deployed as fast as possible---and this process is an order of magnitude faster and cheaper when the changes are small and self-contained, and the branches they live on are short-lived. Working in small batches also ensures developers get regular feedback on the impact of their work on the system as a whole---from other developers, testers, customers, and automated performance and security tests---which in turn makes any problems easier to detect, triage, and fix.

Second, continuous integration requires a fast-running set of comprehensive automated unit tests. These tests should be comprehensive enough to give a good level of confidence that the software will work as expected, while also running in a few minutes or less. If the automated unit tests take longer to run, developers will not want to run them frequently, and they will become harder to maintain. Creating maintainable suites of automated unit tests is complex and is best done through test-driven development (TDD), in which developers write failing automated tests before they implement the code that makes the tests pass. TDD has several benefits, the most important of which is that it ensures developers write code that is modular and easy to test, reducing the maintenance cost of the resulting automated test suites. But TDD is still not sufficiently widely practiced.

Despite these barriers, helping software development teams implement continuous integration should be the number one priority for any organization wanting to start the journey to continuous delivery. By creating rapid feedback loops and ensuring developers work in small batches, CI enables teams to build quality into their software, thus reducing the cost of ongoing software development, and increasing both the productivity of teams and the quality of the work they produce.

### Resources ###

* [Martin Fowler's canonical article on Continuous Integration](http://www.martinfowler.com/articles/continuousIntegration.html).
* My personal favourite introduction, James Shore's [CI on a dollar a day](http://www.jamesshore.com/Blog/Continuous-Integration-on-a-Dollar-a-Day.html).
* Paul Hammant has [a series of detailed articles](http://paulhammant.com/categories.html#Trunk_Based_Development) describing how Amazon, Google, Facebook and others practice trunk-based development.
* A video in which Google's John Penix describes how [Google practices CI at scale](http://www.infoq.com/presentations/Continuous-Testing-Build-Cloud).
* [Paul Duvall's book on Continuous Integration](http://www.amazon.com/dp/0321336380?tag=contindelive-20).

### FAQ ###

*How do I know if my team is really doing CI?*

I have a simple test I use to determine if teams are really practicing CI. 1) are all the engineers pushing their code into trunk / master (not feature branches) on a daily basis? 2) does every commit trigger a run of the unit tests? 3) When the build is broken, is it typically fixed within 10 minutes? If you can answer yes to all three questions, congratulations! You are practicing continuous integration. In my experience, less than 20% of teams that think they are doing CI can actually pass the test. Note that it's entirely possible to do CI without using a CI tool, and conversely, _just because you're using a CI tool does not mean you are doing CI!_

*Don't modern tools such as Git make CI unnecessary?*

No. Distributed version control systems are an incredibly powerful tool that I fully endorse (and have been using since 2008). However like all powerful tools, they can be used in a multitude of ways, not all of them good. It's perfectly possible to practice CI using Git, and indeed recommended for full-time teams. The main scenario where trunk-based development is not appropriate is open-source projects, where most contributors should be working on forks of the codebase rather than on master. There's more discussion of CI and DVCS in a [blog post I wrote on the topic](/2011/07/on-dvcs-continuous-integration-and-feature-branches/).

*Does trunk-based development mean using feature toggles?*

No. [Feature toggles](http://martinfowler.com/bliki/FeatureToggle.html) are a new-fangled term for an old pattern: configuration options. In this context, we use them to hide from users features that are not "ready", so we can continue to check in on trunk. However feature toggles are only really necessary when practicing continuous _deployment_, in which we release multiple times a day. For teams releasing every couple of weeks, or less frequently, they are unnecessary. There are, however, two important practices to enable trunk-based development without using feature toggles. First, we should build out our APIs before we create the user interface that relies on them. APIs (including automated tests running against them) can be created and deployed to production "dark" (that is, without anything calling them), enabling us to work on the implementation of a story on trunk. The UI (which should almost always be a thin layer over the API) is written last. Second, we should aim to break down large features into small stories (1-3 days' work) in a way that they build upon each other [iteratively, not incrementally](http://www.agileproductdesign.com/blog/dont_know_what_i_want.html). In this way, we ensure we can continue to release increments of the feature. Only in the cases where an iterative approach is not possible for some reason (and this is less often than you think, given sufficient imagination) do we need to introduce feature toggles.

*What about GitHub-style development?*

In general, I am fine with [GitHub's "flow" process](https://guides.github.com/introduction/flow/)---provided branches don't live longer than a day or so. If you're breaking features down into stories and practicing incremental development (see previous FAQ entry), this is not a problem. I also believe that code review should be done in process---ideally by inviting someone to pair with you (perhaps using screenhero if you're working on a remote team) when you're ready to check in, and reviewing the code then and there. Code review is best done continuously, and working in small batches enables this. Nobody enjoys reviewing pages and pages of diff that are the result of several day's work because it's impossible to reason about the impact of large changes on the system as a whole.

*What tools should I use?*

Tool choice is a complex topic, and in many cases (unless you use
something wholly unsuitable) tool choice is not the critical factor in
success. I recommend doing some research to whittle down a shortlist
based on what technologies your team is familiar with, what has the
widest level of usage, and what is
under active development and support, and then setting a short-term goal and trying
to achieve it using each of the tools on your shortlist.

