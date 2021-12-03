---
layout: post
title: Stepping back in BS
date: '2021-12-03'
categories: oyster
tags: bismark
---

Deeping into Cv DNA methylation data and I was trying to develop bedgraphs of libraries. I noted that in fact I did not have a complete set [here](https://gannet.fish.washington.edu/seashell/bu-mox/scrubbed/071521-cvBS/). Having also recalled (and seen via .sh files) it took me at least 3 jobs to "complete" the effort that did not complete. So now I question everything. And disappointed that I failed to document the botch of an effort. Well today I am older and wiser and determined not to make a similar mistake. I have decided to cross my fingers and pull deduplicated bams back into mox and run downstream code.
This of course presumes my bismark alignment and dedup was done properly.

Thus
```
wget -r \
--no-check-certificate \
--no-directories --no-parent \
-P . \
-A *deduplicated.bam \
https://gannet.fish.washington.edu/seashell/bu-mox/scrubbed/071521-cvBS/
```
into `/gscratch/scrubbed/sr320/120321-cvBS`
