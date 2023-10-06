---
title: How to delete obsolete remote branches in git and get them back
---

If you work with something like GitHub, you probably have a lot of local branches that you no longer use because the PRs have already been merged. They don't really get in the way, but they can be a bit annoying, cluttering up the UI and just hanging around.

You can delete them one by one, of course, but if there are too many, just the thought of it can be a bit daunting. If you use a GUI, there might be a button to delete them, or you can select all and delete via `cmd-A`, or you can just resign yourself to the situation and write a comment like "they don't really bother me" before reading this post.

In the case of the CLI, there are more options. I, for example, use the solution from ([Stack Overflow](https://stackoverflow.com/a/17029936/229949)). Of course, you could ask Chat GPT, but I prefer Stack Overflow because there are likes and comments from people for whom it worked. The command is long, but it can be copied and executed in a couple of seconds. After that, all branches really disappear (this somehow doesn't happen after `git fetch -p`, but I'm too lazy to dig it further).

So, I did it this way and was satisfied with myself, started writing this post, and then suddenly remembered that I had one local branch where I was working on something intermittently but never created a PR or even pushed it to the server. Whether it was a feeling that it wasn't worth pushing, or I wanted to surprise my colleagues - I don't remember anymore. But the soulless script doesn't care about my concerns; it saw that the remote branch was missing and deleted it. Days of work vanished into thin air... Or did they?

Of course not. This is another reminder that nothing disappears in Git, and branches are just pointers. If you delete a branch, the pointer is removed, but the commit itself remains in place.

Okay, that's good news, but how do you find that commit? And here's where a Git time machine called "reflog" comes to our rescue. It's a tool that shows all movements and changes in the commit tree. So, you run it and look for a mention of your branch. Within a day or two, you can find it if you're attentive.

Or within a couple of seconds if you have a computer. I'm lucky to have one, and it has the `grep` command (actually, I have `rg`, but it's not essential). So, I run something like this:

```bash
git reflog | rg my-cool-branch
```

I look for something like "moving to my-cool-branch," then use git checkout to switch to that revision, and finally, I do:

```bash
git checkout -b my-cool-branch
```

After that, I push it and continue writing this post. 😊
