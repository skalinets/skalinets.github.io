---
title: Running Spark Jobs in Desktop Kubernetes
---

Spark is a data processing framework that is considered to be a successor of Hadoop. It is faster
and eaiser to use, because it doen't need a complicated setup, like HDFS.

If you want to try it, there are a few options. I prefer to run stuff on my local Kubernetes cluster
that comes with [Docker for Desktop](https://docs.docker.com/docker-for-windows/edge-release-notes/).
And when it comes 

steps
1. Install Docker for Desktop Edge
2. Download and extract Spark 2.4
3. Copy 2 jars (kafka connect and logger stuff)
4. Create a Spark Docker image
5. Create a 