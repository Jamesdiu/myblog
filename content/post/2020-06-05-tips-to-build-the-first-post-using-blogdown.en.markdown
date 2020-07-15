---
title: Tips to build the first post using Blogdown
author: James Diu
date: '2020-06-05'
slug: tips-to-build-the-first-post-using-blogdown
categories:
  - R
tags:
  - blogdown
backtotop: no
toc: no
---
Yeah! I start blogging with `blogdown` finally! It took a longer time than I expected. Although some Youtubers are teaching how to build it within 15 min, I have used much more than that! I guess it is normal for someone like me without much experience with Hugo, Netlify, and markdowns. Since there are plenty of resources online, I would recommend reading the post from [Martin Frigaard](https://www.storybench.org/how-to-build-a-website-with-blogdown-in-r/) among all I read.  This post provides very clear and informative descriptions of the procedures that I followed. Indeed, it is also worth to go through the book [Creating Website with R Markdown](https://bookdown.org/yihui/blogdown/workflow.html), written by YiHui Xie, so that to understand more about the tools and techniques used in `blogdown`. 

Meanwhile, I would like to add some tips that I noticed. I hope it may help with other beginners;

1. Pay attention to **HUGO VERSION** required of the template used. When you are connecting your blog to Netlify, you have to provide the "Environment variables" such that to ensure the version is up-to-date enough to run your template.

2. Remember to change the **baseURL** in *config.toml* into the website address on Netlify. Otherwise, it would not be able to obtain all data from git, for example, the template design used.

3. Also for **baseURL**, make sure " / " is NOT missing at the end.

Above are the issues that locked me down and took me a few hours to find out. Hope these would help the next one to go smoothly!
