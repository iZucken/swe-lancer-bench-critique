# SWE-Lancer benchmark critique

"Prior art":
- [SWE-Bench+: Enhanced Coding Benchmark for LLMs](https://arxiv.org/pdf/2410.06992v1)

This is about:
- [SWE-Lancer: Can Frontier LLMs Earn $1 Million from Real-World Freelance Software Engineering?](https://arxiv.org/pdf/2502.12115)
- [Repository](https://github.com/openai/SWELancer-Benchmark)

### What is SWE

It is generally expected from SWE to be able to [Make It Work, Make It Right, Make It Fast](https://wiki.c2.com/?MakeItWorkMakeItRightMakeItFast).
An integration test that only checks if the feature seems to work as intended is not enough to address all of these aspects of SWE.

### Data source
Tasks are sourced from [Expensify](https://github.com/Expensify/App).
Expensify goes out of their way to make tasks easier to work with for external contractors.
Expensify evaluates such tasks with geometric scale, doubling bounties starting at $125.
TODO: investigate how bounty evaluation is done and escalated.
Selected tasks seem to span a lot of areas of expertise, but still are all in the context of a single contained project.

Analysis of the work should account for amount of paid work done to produce the benchmark and compute solutions:
- If claims to be believed (unverifiable) - a lot of engineering was done to pick and refine the tasks;
- A lot of managerial and other work already implicitly done by repository owners even before that;
- Does not account for compute/training costs;
- No mind at all is paid for failed tasks, as if managing invalid/falied work is somehow free. A 40% success rate worker would be fired;
- See further section on managerial work evaluation - bulk of "earnings" is made with a strange method;
- See [justifiable issues](https://github.com/openai/SWELancer-Benchmark/issues/13)

Suspect that sourcer of the data is complicit, by how results [are presented](https://use.expensify.com/blog/expensify-powers-openai-swe-lancer-project):
> Surprisingly, LLMs performed better at management tasks than direct coding, suggesting that AI may first integrate into software teams as an advisor rather than a coder.
See further why claims might be suspect. TODO: see if they are related.

There seems to be an undocumented by the papers issue with how the tasks are structured.

See sample issue [#14268](https://github.com/openai/SWELancer-Benchmark/blob/08b5d3dffd7beeae408033a059805adf569e9460/issues/14268/bug_reintroduce.patch) bug patch:
```diff
+    // Intentionally use raw character count instead of HTML-converted length
+    const validateCommentLength = (text: string) => {
+        // This will only check raw character count, not HTML-converted length
+        return text.length <= CONST.MAX_COMMENT_LENGTH;
+    };
```

[Consult the original issue thread](https://github.com/Expensify/App/issues/14268) - the actual [accepted solution](https://github.com/Expensify/App/pull/15501/files) diff is nothing like what benchmark contains.

This seems to be a common issue as noted by people on [hn thread](https://news.ycombinator.com/item?id=43099268)


### Published dataset

One more suspicious thing I see is that the paper states full dataset consists of ~1500 tasks estimated $1000000, while published set ("diamond") is only ~500 tasks, but worth half of that, so every task in diamond set is worth on average $1000, and every task in unpublished set is worth on average $500. Reported success rate does not seem to correlate.


### Reward system
At the time of writing, source project has ~450 issues marked as "external" (issues for which bounties are paid to successful external contractors).
Of them, ~100 are in "waiting for payment" state (the concerns are somewhat mixed) - should look into whom and how much the company actually pays.
~50 tasks are overdue.

We need to carefully address variance in dataset task rewards. Since iirc rewards scale from 250 to 30000, a few "lucky" upper bound solutions can heavily skew the overall "earning", making the solver appear much more competent. At some level of success it can be completely attributed to variance and mixing in of quiz questions.

### Difficulty claims
Claims are made on basis of resolution timings etc., but do not seem to account for the nature of highly atomized freelance work.
In an open bounty system, it does not seem unlikely that a lot of underqualified contractors might attempt a solution.
Ironically such would also be true for a LLM based agent.
In this context resolution timings would depend on a lot of factors:
- bounty size (this might be correlated with seemingly huge payout for unsubstatial work);
- talent availability at the time;
- talent engagement - in such open system company seems to be unbale to force anyone to resolve anything (anecdotally best talent might not be interested in working on trivial issues) - at the time of writing ~50 tasks are marked overdue;
Continuing to look at the challenge claims, we can observe that tasks are supplied with "best in class" reporting and are on average small and atomic, which is in common sense is unlikely to be classified as "challenging".
- [Confusion is understandable](https://github.com/openai/SWELancer-Benchmark/issues/8)




### Bias
The claims are maid about less bias in task selection; but stated the task curation criteria would suggest otherwise.
Bench authors only published part of the dataset to help "reduce contamination", but there is no proof benchmark authors avoided it themself.
This also does not account for the origin repository itself.
Some unclarified analysis of contamination effect is given in appendix 2, with significant deltas... Not only the cutoff needs to be considered, but also the size of corpus before/after.




### Scoring
Benchmark seems to inappropriately mix scorings for different kinds of work:
- IC SWE tasks which are actual programming tasks;
- SWE Manager tasks which are "soft" tasks;
- One extra point of consideration: if we evaluate the model for solving such tasks, the solution could be theoretically acquired supposedly almost instantly (compared to a human process), and therefore the tasks would never escalate to huge bounties in the first place.

#### IC SWE scoring issues
- As seen from an example bug, it is accompanied by a comment that clearly states the bug adjacent. This is not typical for real bugs. There are even stronger examples linked in related discussions.
- Target project uses a geometric bounty payout scaling (doubles starting at $125) which is very specific and does not seem reasonable as a baseline for this kind of evaluation. TODO: How exactly the tasks scale should be investigated - surface level checks show that trivial issues can escalate to thousands of dollars in bounties.
- TODO: Should check integration testing methodology, suspected that only specific integration tests are run for validation which does not seem sufficient at all, invalidating all results.
- Under original repository, discovery and testing are also parts of the reward structure. The benchmarking method ignores this.

#### SWE Manager issues
- verbatim "SWE Manager tasks require the model to choose between 4-5 proposals" - that means that a random guess would score 20-25%; we observe up to 30% success rate.
- An big assumtion is made that prior human managerial choices are a good ground truth. While automated tests can be made for programming tasks, nothing of sort is done for testing good managerial or architectural decisions. "Conflates retrospective alignment with decision-making quality".
- A strong bias for the eventual selected solution is likely since the repository is open and was incorporated into LLM data;
- Total task dataset "value" (and therefore max contribution) is for some reason higher than IC SWE tasks - 60/40;
- Somehow a price tag of the task is awarded to the manager, while in fact that sum was spent by the manager (and then some - some tasks spend more than stated amount actually, to award other participants for reporting bugs and adding extra tests, and sometimes other contributions - and sometimes in nontrivial amounts);
- TODO: It need further investigation, but it wouldn't be sane to compare working and nonworking solutions (or non-solutions). Need to see if managerial task account for this. From the paper: between "wait" non-solution proposal vs proposal with actual solution with code, "obviously" an actual solution should be evaluated?
- Need to see if there is bias for "later" solutions, which are logically might seem more likely to be accepted for a long running issue;
- From the dataset it seems that only a single price is given, e.g. for task #14958 only the "final" price is awarded. That means that long running issues award more just by default. See if proposals are at least "mixed" before evaluation.

### Double counting

The same issue ID may appear in sets of tasks for coding agent and the manager agent. That could drastically inflate the numbers, since "simple" tasks can now "award" money twice and further inflate scores.

### Conclusion


Preliminary verdict: paper verdict seems very tonque in cheek considering the numbers it proposes. Suspect an attempt to mislead people. Wording it as "earning" is misleading.

### For fun let an LLM critique this

Deepseek:
- "It is uniquely destructive because it weaponizes the language of rigor to legitimize a dangerous fantasy."
- "To salvage credibility in LLM evaluation: 1. Burn this benchmark."
- "Final Verdict: -∞/10 — A singularity of bad science, economic delusion, and ethical malpractice."


# Could a better benchmark exist at all?



