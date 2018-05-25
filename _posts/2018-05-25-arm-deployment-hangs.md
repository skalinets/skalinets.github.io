---
title: ARM Deployment Fails after 1 Hour with DeploymentFailedCleanUp Error
---

> TLDR; If your ARM deployments take an hour and then fails with  DeploymentFailedCleanUp, try to switch it to [Incremental][1] deployment mode to
see if that will help.

It's was a Friday and we decided to start it with the deployment to prod. Our build had been tested
during few days before that, we used CI, CD and ARM deployment and were pretty sure that it would go
smoothly.

If you not aware what ARM is, the good place to start is this [article](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-create-first-template).

So I opened the release in VSTS portal, clicked "deploy to production" button and send something like
"deployment started, should take ~5 minutes to complete" to our team channel. After that I switched to
another task, anticipating that once deployment is done, notification in another would pop up and let
us know that we've done yet another successful release.

However, after 40 minutes I came out from [the zone](https://en.wikipedia.org/wiki/Flow_(psychology)) and noticed that there were no
any notification about completed deployment. To my surprise it was still running. Running at "Deploy ARM" step. Since usually it takes a minute or two to complete that step, something definitely went wrong. 

The obvious step was just to cancel deployment and run it again. But it did not help. Well, this is something that should be investigated.

At some point I opened "deployments" section in our production resource group
and noticed that there are failed deployment. It's error was:

```
The deployment operation failed, because it was unable to cleanup. Please see https://aka.ms/arm-debug for usage details. (Code: DeploymentFailedCleanUp)
```

"Cool", I thought, then browsed that link and found 0 mentions for that code. Then I tried to google and found ~100 documents, most of them did not even contain that word and others were just useless. 

First I tweeted about that: 

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">The way <a href="https://twitter.com/Azure?ref_src=twsrc%5Etfw">@Azure</a> tells you &quot;go f#ck yourself&quot;: &quot;The deployment operation failed, because it was unable to cleanup. Please see <a href="https://t.co/6wumrDo3Ma">https://t.co/6wumrDo3Ma</a> for usage details. (Code: DeploymentFailedCleanUp)&quot;. It failes after 1 hour. Provided link does not have any mention of that code.</p>&mdash; Serhiy Kalinets (@skalinets) <a href="https://twitter.com/skalinets/status/999964965560836096?ref_src=twsrc%5Etfw">May 25, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Then I tried different things, like removing old deployments, but they did not help. I was going to start crying, but eventually found the solution.

Few days ago we did some manual tweaks with app services in prod resource group. They were related to DNS change, setting up SSL and so on. We haven't have needed assets, line DNS name for non-prod environments, and decided to
start with setting things manually for prod and then generalize our automated
deployment for other envs.

But our ARM deployment was configured in [Complete mode][1]. And that was the issue. Looks like ARM is not always happy with cleaning up stuff that was added manually.

So we changed deployment mode to Incremental and after 4 minutes our prod got updated. By the way, if you (like us) use VSTS for deployment and want to update a deployment step, make sure to create a new release, because existing
one will still use previous configuration. That is done intentionally, since it helps to get reproducible releases, but we just need to keep it in mind.

The outcome is:

> Don't change manually resources in resource groups that are managed by ARM Templates.

Hope, this will help someone. Happy coding!

[1]:  https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy#incremental-and-complete-deployments

