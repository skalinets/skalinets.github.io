---
title: Troubleshooting Installation of Azure CLI on WSL
---

Azure CLI is a cross-platform command line tool from Microsoft to manage your Azure resources.
It could be easily installed on Windows Subsystem for Linux (WSL) using instructions from 
[Microsoft][1].

However when I tried to do it on my machine, I faced an error:

```text
Executing: /tmp/apt-key-gpghome.1NUDW4YfF8/gpg.1.sh --keyserver packages.microsoft.com --recv-keys BC528686B50D79E339D3721CEB3E94ADBE1229CF
gpg: connecting dirmngr at '/tmp/apt-key-gpghome.1NUDW4YfF8/S.dirmngr' failed: IPC connect call failed
gpg: keyserver receive failed: No dirmngr
```

I started to google and didn't find anything specific during the first five minutes. Then I recalled
[this post]({{ site.baseurl }}{% post_url 2018-09-05-one-line-fix %})), when I fisrt had spent some fair
amount of time to investigate the stuff and finally realized that there was a solution in vendor's KB.
I scrolled down [installation instructions][1] and yes, it contained some relevant information. Unfortunately,
it was not exactly my issue and didn't help too much.

Further investigation have led me to the discussion of [WSL issue][3], where deeply in [comments][3] I
found the solution. Here it is:

```bash
wget https://packages.microsoft.com/keys/microsoft.asc
sudo apt-key add microsoft.asc
rm microsoft.asc
sudo apt-get update
```

After that `sudo apt-get install azure-cli` worked fine and I got `az` installed.

PS. And yes, maybe it would be better to try an [az Docker image][4], but it would result in less interesting
and much shorter blog post :)

[1]: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest
[2]: https://github.com/Microsoft/WSL/issues/3286#issuecomment-436999269
[3]: https://github.com/Microsoft/WSL/issues/3286
[4]: https://docs.microsoft.com/en-us/cli/azure/run-azure-cli-docker?view=azure-cli-latest