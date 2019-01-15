---
title: Empty Volumes in Docker for Windows  
---

This post is going to be a short one, just it case it would be useful for someone.

In scope of my experiments with Spark (post about it is still in progress), I faced one weird issue.
The script that starts Spark job in Kubernetes cluster, and had been working fine for weeks, aparrently
stopped working.

The error message told me that it cannot find the script to execute. I haven't touched anyhting since the
last succesful run, however error was there and I started to troubleshoot it.

As it often happens, I started with absolutetly irrelevant stuff. Checked the versions of kubernetes, kubectl
and Spark. Compared the spelling of my configuration flags with the docs. Spent funny few hours without any
result. But then...

I decided to check if volumes are still working. And noticed that for some reason in all volumes only
folders are available from containers, but files are not. I checked it with

```bash
docker run --rm -v $PWD:/data alpine ls /data/jobs
```

The output was empty, while `ls $PWD/jobs` listed files there. Have something broken in my Docker installation? I
recalled that it had updated recently, so maybe there is some bug.

And SUDDENLY I remembered that this morning I had changed my windows password. And Docker for Windows uses
the identity of current user to share drives with the docker daemon, so volumes could be used.

Long story short, I went to Docker settings, and re-connected my drive. After that everything started to work.
And minute after that I found the [solution](https://github.com/docker/for-win/issues/25#issuecomment-433072448)
in github issues.

Also I recalled that I had another weird issue -- this time with Kafka running in Kubernetes. Messages were getting
removed after docker / k8s restart. Sounds like the root cause was the same -- broken docker volumes because
of changed password. Kafka installation uses stateful sets, that rely on volumes, hence the issue.

I would like if Docker for Windows communicated this error better, but anyway, issue is resolved and I can
switch from debugging (hate that) to hacking.

Happy coding! 
