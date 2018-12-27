---
title: How to Run Jekyll in Docker
---

It was [one of my first posts]({{ site.baseurl }}{% post_url 2017-01-31-jekyll-windows-pain-p1 %}) in 
this blog where I described how to setup Jekyll on Windows. Recently I have changed my laptop
and found myself not willing to perform that setup again. That post contains word
**PAIN** in its title for a reason :)

Instead I decided to follow someonne's feedback on that post and setup stuff running in Docker.
It was a joy but, as always, with strings attached.

Lets start with the good news. There is a [docker image](https://hub.docker.com/r/jekyll/jekyll/)
for Jekyll with clean [usage example](https://github.com/envygeeks/jekyll-docker/blob/master/README.md#usage),
that was easy to run, but I got the following error instantly:

```text
... Could not find minitest-5.11.3 in any of the sources (Bundler::GemNotFound)
```

It turned out that we need to perform `bundle install` before running build. Someone advised to
run `bundle exec jekyll build` instead just `jekyll build`, but in my case that resulted just in
different error: 

```text
... Could not find concurrent-ruby-1.1.4 in any of the sources (Bundler::GemNotFound)
```

So I desided to try running `bundle install` in the container:  

```bash
export JEKYLL_VERSION=3.8
docker run --rm \
  --volume="$PWD:/srv/jekyll" \
  -it jekyll/jekyll:$JEKYLL_VERSION \
  bundle install
```

It almost worked, but got an error:

```text
Could not verify the SSL certificate for https://rubygems.org/.
```

This error has nothing to either Docker or Jekyll, it's just our corporate network. We have a company's CA certificate,
that is not recognized by WSL and Docker and other non-Windows stuff. You need to either bypass SSL or add that 
certificate to your linux boxes. Quick and dirty way would be changing `source "https://rubygems.org"` to 
`source "http://rubygems.org"`, but it turned out that things would break on next steps anyway. So we have no choice 
but install the CA certificate.

To install the CA to a Docker container you can either start the container and install the certificate with 
startup command, or build a new image bundled with that certificate. I decided to begin with check if adding the
certificate would even help, so I copied that certificate to the blog directory, started the container and jumped into it:

```bash
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:3.8 -- /bin/bash
```

If you google for "how to install a CA certificate to linux", you will find something like that:

```bash
sudo cp mycert.crt /usr/local/share/ca-certificates/extra/
sudo update-ca-certificates
```

`sudo` did not work in that container, but when if I removed it, I got the following error:

```text
Warning! Cannot copy to bundle: /usr/local/share/ca-certificates/extra
WARNING: ca-cert-extra.pem does not contain exactly one certificate or CRL: skipping
WARNING: ca-certificates.crt does not contain exactly one certificate or CRL: skipping
```

It [took me a while](https://github.com/gliderlabs/docker-alpine/issues/260#issuecomment-351128828)
to find out that in Alpine Linux you need to copy the certificate to the `ca-certificates` directory, not
to its subdir. After I changed my `cp` command to `cp mycert.crt /usr/local/share/ca-certificates`
it got installed and `bundle install` started to work.

Next step was adding that certificate to the image. It's rather easy (thanks, Docker!). I just crafted a 
script:

```bash
#!/bin/sh
cp  /../path_to_my_cert.crt . -f
docker build -t jekyll-with-ca .
```

and a `Dockerfile`:

```Dockerfile
FROM jekyll/jekyll:3.8
COPY *.crt /usr/local/share/ca-certificates/
RUN update-ca-certificates
WORKDIR /srv/jekyll
EXPOSE 4000 80
```

Well, we just need to copy the certificate from some location and add installation steps to the docker file.
Easy!

Then, we can run our jekyll site with the following script:

```bash
docker run --rm -it -p 4000:4000 -v="$PWD:/srv/jekyll" jekyll-with-ca \
  bash -c "bundle install; jekyll serve --drafts --config _config.yml,_config.dev.yml"
```

We use `bash` to [run several](https://stackoverflow.com/a/28490909/229949) commands here.

So far so good, however now it is installing gems on the start and it might take few minutes. Can we improve that? 

Of course we can. We just need to add bundle install step to our `Dockerfile`:

```Dockerfile
FROM jekyll/jekyll:3.8
COPY *.crt /usr/local/share/ca-certificates/
RUN update-ca-certificates
WORKDIR /srv/jekyll
COPY Gemfile* ./
RUN bundle install
EXPOSE 4000 80
```

There is [another possible solution](https://github.com/markkimsal/docker-jekyll_plus#install-gems), but I
found pre-build docker image more user friendly. 

And that's it! I must admit that even it took some time, the overall experience was the way
better than with Windows.