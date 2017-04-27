---

layout: post
title: Let Git Do It for You
date: 2017-04-27 11:25:00 +0200

---
Few weeks ago on a conference I met my former colleague who recently had been promoted to a 
team leader. And he said he'd faced (of course :) ) new challenges. One of them was related 
to audit procedure. It should be said that that company has very strict project management
process including a lot of requirements and procedures including audits, code quality specification
and so on. When you are a team lead all those are treated as obstacles preventing you from
doing your job -- delivering a value. But for company having them in place is usually a good
thing because it reduces risks and increases the whole project predictability.

So that guy was complaining that he needed to participate in audit meetings, answering different
questinons, one of them was "why you don't specify JIRA task ID in git commit messages?". He asked 
me what I thought about that. I replied that in general it was a good thing. Just because it adds 
traceability. Having issue ID allows you to do a lot:
automate [issue resolution](https://docs.gitlab.com/ee/user/project/integrations/jira.html#closing-jira-issues),
track what issues have been included into a particular version and so on. 

However while being a teamleader / developer myself I did not always do that. The main
reason was my laziness. Really, it is a tedious work, you need to type the same ID all over again,
because you are in the zone and you're doing frequent commits. It's easy to miss it or make a typo.
If only it could be automated...

But we're developers and automating things is our job. Git has a concept of hooks allowing to 
add custom checks and pre/post processing. If you are using git, you're probably using some
gitflow like flow, and create feature branches. If the name of your branch contains the ID
of corresponding issue, hook can be used to extract it and add to your commit message. 

The good new is that you don't even need to create if from scratch. After quick googling I found few solutions and 
applied [this one](https://gist.github.com/robatron/01b9a1061e1e8b35d270). It did not work "as is"
though -- for some reason regular expression did not parse our JIRA IDs. It got solved by changing it
from `[A-Z0-9]{1,10}-?[A-Z0-9]+-\d+` to `[Pp][Dd]3-[0-9]+`. Our JIRA project has ID **PD3** and
that regex works really well.  
You can do it as well after hook is installed. Just type `notepad .\.git\hooks\commit-msg` in your
git repository and change the regex.

To test it you can use the following trick: open bash and use the following command:
{% highlight bash%}
echo "example of your commit message with jira ID " | grep -Eo "pattern" 
{% endhighlight %}

If it outputs needed ID -- you're done. Just update `commit-msg` and continue coding with less friction. 
All issue IDs will be set. Here is an example:

{% highlight bash%}
C:\Users\admin\work\my-repo [feature/PD3-805-issue ≡ +0 ~1 -0 !]> git commit -am 'my nice commit message'
JIRA ID 'PD3-805', matched in current branch name, prepended to commit message. (Use --no-verify to skip)
[feature/PD3-805-issue 6e5b755] PD3-805 my nice commit message
 1 file changed, 2 insertions(+)
C:\Users\admin\work\my-repo [feature/PD3-805-issue ↑]> git lg -1
* 6e5b755 - (HEAD -> feature/PD3-805-issue) PD3-805 my nice commit message (7 seconds ago) <Serhiy Kalinets>
C:\Users\admin\work\my-repo [feature/PD3-805-issue ↑]>
{% endhighlight %}

Happy coding! 