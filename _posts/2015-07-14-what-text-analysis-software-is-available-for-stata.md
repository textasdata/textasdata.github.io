---
id: 110
title: What text analysis software is available for Stata?
date: 2015-07-14T08:41:50+00:00
author: Kenneth Benoit
layout: post
guid: http://www.textasdata.com/?p=110
permalink: /2015/07/what-text-analysis-software-is-available-for-stata/
tags:
  - General
  - quanteda
  - R
---
A lot of text analysis packages exist for R, such as [quanteda](https://github.com/kbenoit/quanteda), [tm](http://cran.r-project.org/web/packages/tm/index.html), [qdap](https://github.com/trinker/qdap), and [korPus](http://reaktanz.de/?c=hacking&s=koRpus). But these are only useful if you are proficient in R programming. What about users of alternative statistical packages, such as Stata?

Turns out that recent versions of Stata have made huge strides in this area. As of Stata version 13, Stata introduced a [new data type of &#8220;long string&#8221;](http://www.stata.com/stata13/long-strings/) &#8211; _strL_ &#8211; that can be of virtually unlimited length. Combined with a variety of string handling tools (such as _tokenize_) Stata has many built-in functions and tools for manipulating text. As of version 14, Stata (finally!) [supports Unicode text](http://www.stata.com/stata14/unicode/).

Recently there are two promising new developments on the Stata text analysis front.

The best-of-breed consumer text analysis package, [WordStat/QDAMiner](http://provalisresearch.com/products/content-analysis-software/), now works with Stata. See [the announcement from the Provalis page](http://provalisresearch.com/products/content-analysis-software/wordstat-for-stata/). Previously, I recommended (and have taught with) this package to anyone with text analysis needs but who is not comfortable programming. Now, it could be worth looking at again for programmers too. QDAMiner is fantastic if you have qualitative text analysis needs (like general annotation or applying content analysis codes to text) and also works superbly with dictionary-based applications.

There is a relatively new text analysis package for Stata called txttool, see [the article](http://www.stata-journal.com/article.html?article=dm0077) here. I have not tried it in action, but the article makes it sound quite good for basic stuff. This can be installed easily using

```
. net install txttool
```

from Stata.

If I ever get around to it, I will fix up the creaky my own ancient [Stata package “Wordscores”](http://www.tcd.ie/Political_Science/wordscores/software.html) originally written for Stata 7.0. Miraculously, it still works, but of course [quanteda](https://github.com/kbenoit/quanteda) is better, approximately like comparing a spaceship to a bicycle.