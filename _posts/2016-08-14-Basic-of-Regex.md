---
layout: post
title: Basic of Regex
categories: [grep, regex, Linux]
tags: [linux, grep, regex]
fullview: true
comments: true
---

The first topic that I've chosen to export it from my mind is **Regex**. While **Regex** is handy and you can pick it up quickly, it causes tons of frustration. You can use it everywhere when you want to search for a specific part of your file, no matter if it's a **Sublime Text Editor** or a **grep** command in Linux.

In this article, I show some basic examples of **Regex** with **grep** in a Linux Terminal. For those who may not know what's grep for: grep is a single command in Linux for searching a text data with a specific patterns and it uses as following:

~~~~~~~~
grep [options] PATTERN [file]
~~~~~~~~

If you want to know available options of grep, you can easily go through the *grep --help* *(in Ubuntu)* since it contains all options that you have for using this command. Anyway, let's go back to the topic:

We use Regext for finding the pattern that we want. However, Regex uses special expressions in combination with any of the following:

<h4>Literal</h4>
<p>Any character used in a search or matching expression</p>
<h4>Metachracter</h4>
<p>One or more special characters with special meaning</p>
<h4>Escape Sequence</h4>
<p>Use of metachracters as a literal</p>

Most difficult part of using *Regex* is how to build a pattern for finding what we want! Following are some basic options of *Regex*:

~~~~~~~~
[ ]     Matches anything inside brackets. It can be either number or letters but.
        Be aware that, it's case sensitive.

-    This option creates a range. For example, 1-9 means between 1 to 9 OR a-z means between a to z.
    Be aware that, it's case sensitive and you shouldn't use space between.

.    Find any character in its position.

*    Matches any character zero or more times

^    Negates a search when used inside brackets.
    The caret is used outside brackets to find only lines that begin with a given string.

( )    Combine multiple patterns.

|    Find left or right values. Mostly used with () option.

$    Find line base on their ending character. It's Similar to caret outside brackets.
~~~~~~~~

Well, we're the generation of UI so it's mind blowing to go through all of them without any example. Let's assume that we have following text file:

~~~~~~~~
Here is my sample text file
This text file contains 10 lines

Let's search for my name "saleh"
Love my wife
love my life

I born in 1987
November is the best month

1987 + 11 + 18 = 2016

WoW, my birthday equation is the year that I posted this!
~~~~~~~~

Now we want to find:

<h4>All lines that start with either L or l</h4>

~~~~~~~~
saleh@Saleh:~/Personal Project$ grep ^[Ll]. saleh.txt 
~~~~~~~~
Result:

~~~~~~~~
Let's search for my name "saleh"
Love my wife
love my life
~~~~~~~~

<h4>Find saleh:</h4>
~~~~~~~~
saleh@Saleh:~/Personal Project$ grep saleh saleh.txt
~~~~~~~~
Result:

~~~~~~~~
Let's search for my name "saleh"
~~~~~~~~

<h4>Find all Lines start with L</h4>
~~~~~~~~
saleh@Saleh:~/Personal Project$ grep ^L saleh.txt 
~~~~~~~~
Result:

~~~~~~~~
Let's search for my name "saleh"
Love my wife
~~~~~~~~

*Hope she visits this site and see this*

<h4>Find Numbers Only</h4>
~~~~~~~~
saleh@Saleh:~/Personal Project$ grep -v [A-Za-z] saleh.txt 
~~~~~~~~
Result:

~~~~~~~~
1987 + 11 + 18 = 2016
~~~~~~~~

**-v prints non-matching lines and it's one of the grep options**

<h4>Find any lines that has number</h4>
~~~~~~~~
saleh@Saleh:~/Personal Project$ grep [0-9] saleh.txt 
~~~~~~~~
Result:

~~~~~~~~
This text file contains 10 lines
I born on 1987
1987 + 11 + 18 = 2016
~~~~~~~~

You can always use the combination of above for building your search patterns. However, don't forget that prior to going through the patterns you need to know what you want to find. Hope it helps.
