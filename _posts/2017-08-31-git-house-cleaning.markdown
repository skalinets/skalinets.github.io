---

layout: post
title: "Git House Cleaning"
date: 2017-08-31 12:00 +02:00
categories: git powershell

---

If you're using git, most likely a lot of branches are being created. Some of them are pushed
to remote and finally merged into the main branch. Branches are very lightweight in git (they're
just [pointers to commits](https://git-scm.com/book/en/v1/Git-Branching-What-a-Branch-Is)) and
branching is highly encouraging. 

However there is _a time to cast away stones, and a time to gather stones together_. While creating
branches is a joy, removing them is a routine. What is worse is that every pushed branch multiplies
to 3 (three) entities that should be removed some day. Those are:

- local branch (__my-branch @ your repo__)
- remote branch (__my-branch @ remote server__)
- ref to remote branch (__origin/my-branch @ your repo__)

Each of those beasts has unique way of passing away. Let's see.

Local branches can be removed via `git branch -d` or `git branch -D`. The difference is that former
would not remove a branch that has not been merged into your HEAD (or the branch you're
currently on, if you wish). But you can _shout at_ git using capital `D` and then branch will 
be gone.

Next in our list is a branch on the remote server. Usually it gets removed by clever monkeys
living in smart software. For instance, when you merge pull request in github/gitlab/bitbucket, latter
can delete that branch for you. But you still can do it yourself. The `git` command is not very
intuitive, but you'll get used to it after few months or so. Here it is: `git push origin :my-branch`. Yes, we're pushing the branch, but when it's prepended
with colon, it means _I want to remove that branch on the server_. I have no idea
where it comes from and now [everyone in internet](https://git-scm.com/book/id/v2/Git-Branching-Remote-Branches#_delete_branches) says that you should use
`git push origin --delete my-branch`, but using colons is so much more fun...

The last item (refs to remote branchs) is easy. It has been covered in [post about git tricks](git-tricks) -- you can use `--prune` to remove remote refs during `git fetch` or `git pull`. 

Okay, we covered all delete scenarios and looks like that both remote branches and refs to them usually are being removed automatically. Your only area of concern is removing local branches having been merged. You can do it manually if you're not laze or you can use the script. My favorite is this one:

{% highlight powershell %}
git branch --merged HEAD  | `
  ? { $_ -notmatch 'develop' -and $_ -notmatch 'master'  } |`
  %{git branch -d $_.ToString().Trim()}
{% endhighlight %} 

It gets all merged branches that are neither master nor develop and terminates 
them with `git -d`. 

I have this script in my home folder and from time to time, when I want to do some
destruction, I invoke it like `~\clean-branches.ps1` and see how branches are [dying](https://gist.github.com/skalinets/8df45e65287ed8c4f2b54012be50d1f2).