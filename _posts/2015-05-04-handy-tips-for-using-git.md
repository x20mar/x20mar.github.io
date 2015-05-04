---
title: "Handy tips for using Git"
layout: "post"
---

Over the years using Git I found a few tips and tricks on using Git. Rather than have an article here and there, I figured I should write down all the things I know somewhere...

### Aliases

{% highlight bash %}
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.br branch
git config --global alias.hist 'log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short'
git config --global alias.type 'cat-file -t'
git config --global alias.dump 'cat-file -p'
git config --global alias.ss stash save
git config --global alias.sp stash pop
{% endhighlight %}

### Linux Bash

In Linux (this doesn't work in OS X) bash you can modify the bashrc so that you can see the git branch like this:

{% highlight bash %}
oali@laptop:~/Documents/Code/github/x20mar.github.io (master)$
{% endhighlight %}

To get this modify the .bashrc to be like this:

{% highlight bash %}
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w`__git_ps1`\[\033[00m\]\$ '
{% endhighlight %}

### Syncing a fork

Github has good documentation on this so I'll just point to their documentation on this

[Syncing a fork](https://help.github.com/articles/syncing-a-fork/)