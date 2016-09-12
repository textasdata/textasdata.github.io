---
id: 89
title: 'Encoding headaches, emoticons, and R&#8217;s handling of UTF-8/16'
date: 2015-02-05T09:40:50+00:00
author: Kenneth Benoit
layout: post
guid: http://text.kbenoit.webfactional.com/?p=89
permalink: /2015/02/encoding-headaches-emoticons-and-rs-handling-of-utf-816/
tags:
  - quanteda
  - R
---
<p class="p1">
  I was recently asked for help from a colleague (<a href="https://twitter.com/kmmunger">@kmmunger</a>) who was experiencing a choke on cleaning the tokenized texts from Twitter data. The tweets were in the JSON format that comes from the Twitter API, in what we thought was UTF-8 encoding. Turns out these tweets used some emoticons from the nosebleed section of the Unicode maps, and these were not being read properly into R, as <a href="http://github.com/kbenoit/quanteda">quanteda</a> was being used for processing this text.
</p>

<p class="p1">
  The error message looked something like this:
</p>

```R
Creating dfm from corpus ...
   ... indexing documents</span></p>
   ... tokenizing texts, found 2,426 total tokens</span></p>
   ... cleaning the tokens</span></p>
Error in tolower(x) : invalid input 'í ½í¸”' in 'utf8towcs'
```

<p class="p1">
  The solution I proposed turned out to be tricky and ultimately, not very satisfactory. The problem seems to be built in to R and the way it handles byte-encoded Unicode text. The situation in essence: The tweet texts included emoji, which use a code point in Unicode that is pretty high up in the table. It’s so high, it seems, that the variable byte encoding used to represent the code point gets represented by two code “surrogates” in whatever source encoded them. Now you could say that this is a fault in whatever encoded the texts, and that&#8217;s probably now wrong, since R handled the emoji when entered directly as UTF-8 (see below). However it&#8217;s also clear that R tries to read each byte of the surrogate pair as a separate Unicode character, which produces an incorrect result. Because the lowercase operation is undefined for this incorrect result, the clean() function produces an error when it attempts the conversion to lowercase.
</p>

<p class="p1">
  You can read <a href="http://en.wikipedia.org/wiki/Universal_Character_Set_characters#Surrogates">more about surrogates here</a>, but beware, you are starting down the rabbit hole.
</p>

<p class="p1">
  <span class="s1">To test this, I focused on a single tweet (but one of many) that was causing problem. In the plain text form, this was represented by Unicode escape characters. \u201c is an easy one: This is the left &#8220;curly quote&#8221;. It&#8217;s not until later in the text that the single character emoji are represented by a two-byte </span>\uXXXX sequence. In plain text, it looked like this:
</p>

```
\u201c@AyeBeKnowin: No words \ud83d\ude14 RT @NinTANDO_Me: @AyeBeKnowin the Knicks .... Go ahead say suttin\u201d\ud83d\ude1e\ud83d\ude1e\ud83d\ude1e
```

<p class="p1">
  My interpretation of what is happening: The text is plain ascii but with the non-ascii unicode character represented by their \uXXXX escape codes.  For characters that R can handle, like curly quotes, this will be \u201c, a left curly quote, which is the first character in tweet 12.  Additional characters exist as TWO unicode \uXXXX points, such as the \ud83d\ude14 which is actually the encoding using surrogates of a single unicode emoji character, with code point U+1F614. See <a href="http://apps.timwhitlock.info/emoji/tables/unicode"><span class="s2">http://apps.timwhitlock.info/emoji/tables/unicode</span></a>.
</p>

<p class="p1">
  <span class="s1">Along the process of reading in the tweets and converting them to what R understands as UTF-8, it is converting this into the byte representation (in hexadecimal) as &#8220;\xed\xa0\xbd\xed\xb8\x94”, when it should in fact be &#8220;\xF0\x9F\x98\x94”.  This works fine, as you can see if try in R:</span>
</p>

```R
> tolower("\xF0\x9F\x98\x94")</span></p>
<p class="p1"><span class="s1">[1] "\U0001f614"
```

<p class="p1">
  <span class="s1">it returns the unicode code point as \U without converting it. I am not sure why R chose this byte conversion, or indeed whether this is really an error or if there is something I have got wrong here.</span>
</p>

<p class="p1">
  <span class="s1">The reason it’s encoded originally as &#8220;\ud83d\ude14” in the source file <strong>appears</strong> to be is because this is the surrogate representation, as you will see if you  click on the link from the above page directly here: <a href="http://apps.timwhitlock.info/unicode/inspect/hex/1F614"><span class="s2">http://apps.timwhitlock.info/unicode/inspect/hex/1F614</span></a>. It turns out that this character is the &#8220;pensive face&#8221; emoticon (see below for what this looks like). </span>I qualify this statement because I am not an expert on Unicode encodings and certainly not on the surrogate byte &#8220;hack&#8221; that appears to have been one of the reasons UTF-16 is discouraged.
</p>

<p class="p1">
  <span class="s1">Laboring through the tweets, it was possible to check the byte representations in the input text using the tool from <a href="http://rishida.net/tools/conversion/"><span class="s2">http://rishida.net/tools/conversion/</span></a>.  Pasting the first of the above tweets into the green box and hitting convert, the emoticon emerges. Note here again that the actual string with the \u escape codes is pure ASCII, but the byte encoding is contained in the \uXXXX sequences, which R converts (wrongly, apparently) into hex byte sequences represented as \xXX.</span>
</p>

<p class="p1">
  <span class="s1">The short-term solution to this was to simply use iconv() in a way that eliminated the untranslateable multibyte codes. But this is hardly a solution, since it is discarding the emoticons that might make interesting textual features &#8212; especially here, since hte application related to sentiment analysis. </span> But this will remove the offending characters, and make the call to clean() work fine:
</p>

```R
iconv(tweets$text, "ASCII", "UTF-8", sub="")
```

<p class="p1">
  I also suspect that the problem could be solved if the input text were in an actual UTF-8 encoding, rather than being in an ASCII formatted JSON file that represents non-ASCII characters as hexadecimal escape sequences. On my Mac&#8217;s R console, for instance, this worked and even displayed properly.
</p>

<p class="p1">
  <a href="{{ site.baseurl }}/images/post_images/emoji.png"><img  src="{{ site.baseurl }}/images/post_images/emoji.png" alt="emoji" width="998" height="284" sizes="(max-width: 998px) 100vw, 998px" /></a>
</p>

<p class="p1">
  Here it does not display as ann emoticon in the output from topfeatures() but it does have the correct code point reference. R can display this on my output device using cat() but not using the default print method for a named character vector.
</p>

<p class="p1">
  I would love to read comments from those with more experience with encoding than I have, since it&#8217;s quite possible I got some of the details wrong here.
</p>
