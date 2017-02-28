---
layout: post
title: "Git tricks"
categories: git
date: 2017-02-28 18:51 +02:00
---

There is no any doubt that git is the best source control management system. Only those who never
gave it a try can mumble something opposite. Moreover the whole power and beauty of git is exposed to
those brave guys who are using console. 

Using console in Windows is not that common across Windows guys, however it is not just possible but is very
productive and full of joy.

That feeling of creepy Windows console experience has been formed by [cmd](https://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/batch.mspx?mfr=true).
And here I should agree. It is old and poor version of command line from MS DOS, giving pain and distress to 
everyone who uses it. But there are much better options like Powershell or even bash. 

Here are some tricks I'm using in my everyday git activity. 

## Pretty logs
Default `git log` produces sub-optimal output:
![git log](/assets/ConEmu64_2017-02-28_13-58-10.png)

Every enty takes a lot of space and most information is useless. But it also can be like that:
![git lg](/assets/ConEmu64_2017-02-28_14-04-22.png)

Actually `git log` (like all other git commands) has [endless number](https://git-scm.com/docs/git-log)
of params and switches that can format it's output. Good news -- you don't need to remember them. You can
just google for _git gog pretty print_ and find dozens of ready combinations. Then you just need to add
ones you like to your list of [aliases](https://githowto.com/aliases). This is just a section of global `.gitconfig` file. For instance, 
mine aliases are here: 

{% gist skalinets/4003f113f6e997203066 %}

## Pull or rebase?
This might sound like a holly war: how to get changes from upstream? There are 2 options:

- `git pull` that is basically a `git fetch & git merge`. Usually this command will cause loops in the
history: 

![git pull](/assets/wish_2017-02-28_15-09-10.png)

- `git pull --rebase` that is a `git fetch & git rebase`. Rebase is used to eliminate loops. It just
moves your commits on the top of the history:

![git pull --rebase](/assets/wish_2017-02-28_15-04-30.png)

I personally tend to use the latter because I am sure that less loops make history cleaner and more easy to understand. 
To [save my keystrokes](http://keysleft.com/) I've added an alias `git pr`. It basically means _git pull rebase_. Besides
of that I added `--prune` option to that alias. It removes orphan remote branches -- branches that got deleted in remote
repo after you'd added them with `git pull'.

## Easy push
Our team uses sort of gitflow. So we create feature branches and push them to gitlab for code review. To push a newly
created branch `git push` is not enough:

{% highlight console  %}
C:\Users\admin\work\my-repo [my-branch]> git push
fatal: The current branch my-branch has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin my-branch

{% endhighlight %}

git is so kind to show you the command you need to type. You can (almost) easily copy & paste it, but there is more easy
and funny way. Meet [thefuck](https://github.com/nvbn/thefuck). This nice shell extension allows you to fix your errors
with one simple `fuck`:

{% highlight console  %}
    git push --set-upstream origin my-branch

C:\Users\admin\work\my-repo [my-branch]> fuck
C:\tools\python2\lib\site-packages\win_unicode_console\__init__.py:31: RuntimeWarning: sys.stdin.encoding == 'utf-8', whereas sys.stdout.encoding == None, readline hook consumer may assume they are the same
  readline_hook.enable(use_pyreadline=use_pyreadline)
git push --set-upstream origin my-branch [enter/↑/↓/ctrl+c]
remote: Permission to other/repo.git denied to skalinets.
....
{% endhighlight %}

It just works. _On my snippet I've got 403 because was trying to push to someone's else repository. And python for some 
reason is always complaining about `utf-8` encoding, but [that's Windows, you know](jekyll-windows-pain-p2).... We can
just ignore those errors._  

## Specifying commits

Most git commands accept revision or commit as a parameter. For example, `git log 12ab3` will show all commits that are
reachable from revision with hash starting with `12ab3`. You don't need to specify the full hash (that is long) but just
first unique digits. 

But there are other ways to tell what revision you need:

- __absolute__. As we mentioned before, hash is the most obvious way. Use first n unique digits instead of whole hash. 
Another way is to use branch name, tag name, or special link, like `HEAD`. 
- __relative__. Adding `^` to the end of revision means _parent of_. If you need parent of parent of parent -- just add 
3 `^`s: `^^^`. If you nth level of ancestor you can use `~n` where n is the level you need. To get 10th ancestor of 
commit `123abc` use `123abc~10`.
- __custom__. Sometimes you need to refer to commit by phrase or word from its commit message. You can do it with `:/phrase`
notation. For instance `git log :/fix` will show commits reachable from first commit having word _fix_ in its commit message.
This trick is very useful when you want to do quick fixup. 

## Squashing commits

Git provokes frequent commits. While working in your personal local branch you can relax your commit message discipline and 
use generic quick-to-type messages like 'fix' or 'one more fix'. 

When it comes to make a pull request, you might want to add some beauty to commits. Reword messages or reduce their number. 
This is where `git rebase -i` comes into play. `-i` means _interactive_ and by default it will open up your text editor with
the list of commits and you have ability to specify what actions you'd like to perform against them: reword, squash, remove etc. 
I will not repeat [nice tutorials](https://www.atlassian.com/git/tutorials/rewriting-history) -- just point out that it is
possible and not that scarry. Also check this nice post about [autosquash and fixups](https://robots.thoughtbot.com/autosquashing-git-commits)  

Have I missed anything? What do you use? Tell me in the comments / twitter / facebook. Good gitting!
