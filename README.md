Attempt a comprehensive critique of the latest SWE benchmark by OpenAI.

["Prior art"](https://arxiv.org/html/2410.06992v1)

[Paper](https://arxiv.org/pdf/2502.12115)
[Repository](https://github.com/openai/SWELancer-Benchmark)



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
- No mind at all is paid for failed tasks, as if managing invalid/falied work is somehow free;
- See further section on managerial work evaluation - bulk of "earnings" is made with a strange method;
- See [justifiable issues](https://github.com/openai/SWELancer-Benchmark/issues/13)



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


### Reward system
At the time of writing, source project has ~450 issues marked as "external" (issues for which bounties are paid to successful external contractors).
Of them, ~100 are in "waiting for payment" state (the concerns are somewhat mixed) - should look into whom and how much the company actually pays.
~50 tasks are overdue.


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

#### IC SWE scoring issues
- Target project uses a geometric bounty payout scaling (doubles starting at $125) which is very specific and does not seem reasonable as a baseline for this kind of evaluation. TODO: How exactly the tasks scale should be investigated - surface level checks show that trivial issues can escalate to thousands of dollars in bounties.
- TODO: Should check integration testing methodology, suspected that only specific integration tests are run for validation which does not seem sufficient at all, invalidating all results.

#### SWE Manager issues
- verbatim "SWE Manager tasks require the model to choose between 4-5 proposals" - that means that a random guess would score 20-25%; we observe up to 30% success rate.
- An big assumtion is made that prior human managerial choices are a good ground truth. While automated tests can be made for programming tasks, nothing of sort is done for testing good managerial or architectural decisions.
- A strong bias for the eventual selected solution is likely since the repository is open and was incorporated into LLM data;
- Total task dataset "value" (and therefore max contribution) is for some reason higher than IC SWE tasks - 60/40;
- Somehow a price tag of the task is awarded to the manager, while in fact that sum was spent by the manager (and then some);
- TODO: It need further investigation, but it wouldn't be sane to compare working and nonworking solutions (or non-solutions). Need to see if managerial task account for this. From the paper: between "wait" non-solution proposal vs proposal with actual solution with code, "obviously" an actual solution should be evaluated?
- Need to see if there is bias for "later" solutions, which are logically might seem more likely to be accepted for a long running issue;
- From the dataset it seems that only a single price is given, e.g. for task #14958 only the "final" price is awarded. That means that long running issues award more just by default. See if proposals are at least "mixed" before evaluation.

### Double counting

The same issue ID may appear in sets of tasks for coding agent and the manager agent. That could drastically inflate the numbers, since "simple" tasks can now "award" money twice and further inflate scores.

### Conclusion


Preliminary verdict: paper verdict seems very tonque in cheek considering the numbers it proposes. Suspect an attempt to mislead people.
