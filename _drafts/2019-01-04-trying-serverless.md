---
title: Trying serverless framework
---

```bash
sudo npm i -g serverless
```

Then we do 

```bash
serverless login
```

You should see a login page in the browser where you can sign up or login. Being a cool thing,
Serverless support 2FA via Google and Github If you try to run this from WSL it may not work. 
See [my post]({{ site.baseurl }}{% post_url 2018-12-20-starting-linkerd-with-wsl %})for potential fix.

Once you log in, it will ask you to create a new project, just enter some cool name and you're done.

Serverless supports different platform, this time I have chosen Google. First, I executed steps from
their [official documentation](https://serverless.com/framework/docs/providers/google/guide/quick-start/).

I always create a git repository in directories when doing such exersices. it allows me to step back if
needed. 

Got this error:

```
Serverless: WARNING: Missing "tenant" and "app" properties in serverless.yml. 
Without these properties, you can not publish the service to the Serverless Platform.
```

I was asked to enable google functions API