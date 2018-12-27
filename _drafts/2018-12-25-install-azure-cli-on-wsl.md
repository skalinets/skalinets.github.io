---
title: Troubleshooting Installation of Azure CLI on WSL
---

Azure CLI is a cross-platform command line tool from Microsoft to manage your Azure resources.
It could be easily installed on Windows Subsystem for Linux (WSL) using instructions from 
[Microsoft](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest).

However when I tried to do it on my machine, I faced an error 

```text
Executing: /tmp/apt-key-gpghome.1NUDW4YfF8/gpg.1.sh --keyserver packages.microsoft.com --recv-keys BC528686B50D79E339D3721CEB3E94ADBE1229CF
gpg: connecting dirmngr at '/tmp/apt-key-gpghome.1NUDW4YfF8/S.dirmngr' failed: IPC connect call failed
gpg: keyserver receive failed: No dirmngr
```

```csharp
var t = new String();

```



https://docs.microsoft.com/en-us/cli/azure/run-azure-cli-docker?view=azure-cli-latest
