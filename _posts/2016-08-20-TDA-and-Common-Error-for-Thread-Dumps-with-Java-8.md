---
layout: post
title: TDA and Common Error for Thread Dumps with Java 8
categories: [TDA, regex, JAVA]
tags: [TDA, regex, sed, JAVA, thread]
fullview: true
comments: true
---

The external thread dumps are the best option if you want to investigate performance degradation of Java applications. Almost all geeks out there who are dealing with Thread Dumps are familiar with awesome tools called <b><a href="https://java.net/projects/tda/">TDA</a></b>. However, once in a while when you are generating your thread dumps for applications with Java 8, you will encounter to following error in console:

{% highlight java %}
Exception in thread "AWT-EventQueue-0" java.lang.NumberFormatException: For input string: "5 os_prio=0"
  at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
  at java.lang.Long.parseLong(Long.java:589)
  at java.lang.Long.<init>(Long.java:965)
  at com.pironet.tda.utils.ThreadsTableModel.getValueAt(ThreadsTableModel.java:80)
  at com.pironet.tda.utils.TableSorter.getValueAt(TableSorter.java:285)
 ....
{% endhighlight %}

TDA team investigated this issue and they belive it's related to fact that an _os_prio_ culmn exist between _prio_ and _tid_ and they fixed it <a href="https://github.com/irockel/tda/issues/6">here</a>. However, if you are not willing to apply the fix or changing your TDA version, you can clean up your Thread dumps with following <b>regex</b>:

### For Linux:

{% highlight regex %}
sed -i 's/prio=[0-9]\{1,2\} os_prio=-\?[0-9]\{1,2\}/prio=5/g' <filename>
{% endhighlight %}

## For Mac:

{% highlight regex %}
sed -i '' -E 's/prio=[0-9]{1,2} os_prio=-?[0-9]{1,2}/prio=5/g' <filename>
{% endhighlight %}

Credit of above solution goes to Atlassian Team _(<a href="https://confluence.atlassian.com/kb/generating-a-thread-dump-794369342.html">here</a>)_.

<a class="btn btn-default" href="https://github.com/irockel/tda">TDA is available in GitHub</a>
