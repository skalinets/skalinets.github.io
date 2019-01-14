---
title: Running Spark Jobs in Desktop Kubernetes
---

Spark is a data processing framework that is considered to be a successor of Hadoop. It is faster
and eaiser to use, because it doen't need a complicated setup, like HDFS.

If you want to try it, there are a few options. I prefer to run stuff on my local Kubernetes cluster
that comes with [Docker for Desktop](https://docs.docker.com/docker-for-windows/edge-release-notes/).
And when it comes 

Download spark-redis jar from [Maven](https://mvnrepository.com/artifact/com.redislabs/spark-redis/2.3.0)

steps
1. Install Docker for Desktop Edge
2. Download and extract Spark 2.4
3. Copy 3 jars (kafka connect and logger stuff) to `spark-runtime/jars`
4. Create a Spark Docker image
5. Create a 

create a zip file for redis:
1. pip install redis
2. pip show redis
https://stackoverflow.com/questions/32274540/write-data-to-redis-from-pyspark




how to enable intelli sense for spark python
https://github.com/DonJayamanne/pythonVSCode/wiki/PySpark