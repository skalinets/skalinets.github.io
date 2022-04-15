--- 
title: Building Telegam Bot with Python and Google Cloud Functions
---

Oookay, let’s build yet another telegram bot using python. I know there are
quazillion of similar tutorials in internet, why bother with another one?

We will use the following technologies:

* **Language**: python. Because why not? I think that everyone should know
python. And in this post I will provide with some guidance of how to start
development.

* **Library**:
[python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot).
I did a very brief investigation (in fact, just quick googling and picking top
result). It seems to be simple and powerful — that is all we need.

* **Hosting**: [Google Cloud Functions][cloud-functions]. This is a very cost
effective solution, at least for small traffic (once our bot becomes popular we
might consider switching to something else). Everyone is doing their cloud stuff
in either AWS or Azure and completely misses the Google platform. We will fill
that gap. Also keep in mind that upon registration you can get free $300 for 180
days.

[cloud-functions]: https://cloud.google.com/functions/

* **Automation**. Github Actions. Of course with python and Google Cloud you can
write your bot in the Google Cloud UI, but this is not our way. I am strong
believer in automation — writing code in browser becomes annoying very
quickly. So instead we will spend some time building a CICD pipeline — this is
a fun that will save us huge amount of time in the future. And we might learn
something new.

## Register the Bot

First things first. We need to register a bot. And this is the easiest part.
Just talk to [@BotFather](https://t.me/BotFather) in telegram – he will handle
everything. You just need to invent a name and a tag for you bot (one of the
hardest programming problems).

The result should be a bot's HTTP token, that should be stored in some secret
place (note that you can aways take it from the chat with `BotFather`. Keep that
in mind when sharing screen, etc.)

## Setting up Google Cloud

1. First you need to register in [the Google Cloud Console][cloud-console]. It's
free and also you'll get [$300 credit for 90 days][cloud-credis].
1. [Create a project][create-project]
1. Enable the Cloud Functions API. To do that just navigate to functions and try
to create one. It will ask about enabling a bunch of needed APIs, enable all of them.
1. [Create a service account][create-service-account].
1. [Assign required roles][service-account] to the service account from the
previous step.
1. Finally, [create and download the API Key][create-api-key].

Now Google Cloud is ready to adopt our telegram bot.

[create-api-key]: https://cloud.google.com/iam/docs/creating-managing-service-account-keys#iam-service-account-keys-create-gcloud
[create-service-account]: https://cloud.google.com/iam/docs/creating-managing-service-accounts
[service-account]: https://cloud.google.com/functions/docs/reference/iam/roles#additional-configuration
[cloud-credis]: https://cloud.google.com/free/docs/gcp-free-tier/#free-trial
[cloud-console]: https://console.cloud.google.com
[create-project]: https://developers.google.com/workspace/guides/create-project

## Setting Up Our Development Environment

I am not a python guru, so might be you. If that is the case, this section
might be helpful to get started with this amazing language.

To start development with python you need only a text editor. There is a big
chance that python has already been installed on your machine. But you can never
be sure what version is installed, and if that version differs from the prod’s one,
you can step to “works on my machine” trap. To overcome this issue python
community introduced *virtual environments*. For me its like a docker image where
you can fix the version (of versions) of python and needed dependencies for you
project.

For some reason there are more than one way and tool to manage these
environments. I did some googling, found this [this post][python-envs] and took
pyenv from there. Pyenv allows you to keep several versions of python side by
side so that every application will use the proper python version. So I
installed [it via home brew][brew] and then did [some crazy black
magic][pyenv-install] to my zsh profile (as suggested in the docs):

[python-envs]: https://gillesfabio.com/blog/2022/03/03/set-up-a-python-development-environment-on-macos/
[pyenv-install]: https://github.com/pyenv/pyenv#basic-github-checkout
[brew]: https://brew.sh/

``` bash
brew install pyenv
echo 'eval "$(pyenv init --path)"' >> ~/.zprofile
echo 'eval "$(pyenv init -)"' >> ~/.zshrc 
```

`pyenv` does a great job maintaing different versions of Python, but to allow it
to manage virtualenvs, a separate plugin is needed
I also installed a plugin [brew](https://github.com/pyenv/pyenv-virtualenv):

``` bash
brew install pyenv-virtualenv
```

Long story short, we got our environment ready for some action, lets
do that action.

## Start of Development

First we need to make sure that we use the right version of python. At the
moment of this writing the highest version Google Cloud Functions supports is
3.9. So lets install it and create a virtual env:

``` bash
pyenv install 3.9.11                # install python 3.9.11
pyenv virtualenv 3.9.11 venv.3.9.11 # create virtual env
pyenv activate venv.3.9.11          # activate that virtual env
```

Then create a folder, init a git repo and create a GitHub repo:

``` bash
mkdir ./image-bot
cd ./image-bot
git init
gh repo create --clone image-bot --private
```

BTW I use ~~arch~~ [GitHub CLI](https://cli.github.com/) — this is yet another
tool that reduces cases when you need to leave the console. It allows to do
almost anything you need in GitHub. But if for some reason you need a web UI, it
could be opened via  `gh browse`.

I am using [Neovim](https://neovim.io/) for all my development and setup for
python is one of the easiest ones. Just install
[coc.nvim](https://github.com/neoclide/coc.nvim) and
[coc-pyright](https://github.com/fannheyward/coc-pyright) and you are good to
go. To make pyright aware of `pyenv virtualenv`’s you need to add a small piece
of configuration to `pyrightconfig.json`, like this:

``` json
{ 
  "venv" : "venv.3.9.11",
  "venvPath" : "/Users/serhii.kalinets/.pyenv/versions"
} 
```

It could be added manually, or you can use [yet another pyenv
plugin](https://github.com/alefpereira/pyenv-pyright):

``` bash
pyenv pyright
```

## Code

Let’s use a very simple “hello world” bot implementation. In bots world it
usually just sends back a message sent to it, acting like a [echo linux
command][echo], hence the name `echobot` Python-telegram-bot has an
[examples][telegram-examples] directory that contains [echobot][echobot] example
as well. But it is too complicated and won't work with Cloud Functions "As Is",
so we'll start with something easiser.

[echo]: https://www.man7.org/linux/man-pages/man1/echo.1.html
[telegram-examples]: https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples
[echobot]: https://github.com/python-telegram-bot/python-telegram-bot/blob/master/examples/echobot.py

We need 2 files.

#### requirements.txt

``` txt
python-telegram-bot
```

#### main.py

``` python
import os
import telegram
import requests


def bot(request):
    bot = telegram.Bot(token=os.environ["TELEGRAM_TOKEN"])
    if request.method == "POST":
        update = telegram.Update.de_json(request.get_json(force=True), bot)
        if update:
            chat_id = update.message.chat.id
            message = update.message.text
            response = requests.get("https://meme-api.herokuapp.com/gimme")
            if response.status_code == 200:
                url = response.json()["url"]
                bot.sendMessage(chat_id=chat_id,
                                text=f"got your request... you sent: {message}\n"
                                + f"here is your meme: {url}")

    return "this should never happen"
```

Here we defined a function `bot(request)` that is going to be invoked by Cloud
Functions runtime. We create an instance of the `Bot` class, using the telegram
token. Note that we don't hardcode that token, but read it from the environment
variable, conforming to 12 Factor Apps Principles Then we check if the request
is in fact *POST*, convert it to the `Update` type, get `message`, `chat_id`, and
send reply back with the message and also a random meme from
[Reddit](https://reddit.com) (kudos to the author of [this API][meme-api]).

[meme-api]: https://github.com/D3vd/Meme_Api

## Automation

Before adding more features, we need to build a delivery pipeline. Of
course it is possible to develop for Cloud Functions exclusively in the web UI,
or copy & paste code from the editor to that UI, or even use [gcloud][gcloud]
CLI tool to update the function definition from your machine, but all these are
just repetitive tasks that become annoying rather quickly. It would be much
better to delegate that job to our CICD tool, that is GitHub Actions.

[gcloud]: [https://cloud.google.com/sdk/docs/install]

I am using the following workflow definition.

{% raw %}

``` yaml
name: Deploy

on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
      
    - id: auth
      uses: google-github-actions/auth@v0
      with:
        service_account: ${{ secrets.GC_SERVICE_ACCOUNT }}
        credentials_json: ${{ secrets.GC_TOKEN }}
    - id: deploy
      uses: google-github-actions/deploy-cloud-functions@v0
      with:
        name: bot
        runtime: python39
        source_dir: ./src
        env_vars: TELEGRAM_TOKEN=${{ secrets.TELEGRAM_TOKEN }}
    - id: set-url
      run: curl https://api.telegram.org/bot${{ secrets.TELEGRAM_TOKEN }}/setWebhook?url=${{ steps.deploy.outputs.url }}
```

{% endraw %}

Here I use [google cloud auth][google cloud auth] action to authenticate in
Google Cloud and [deploy-cloud-functions][deploy-cloud-functions] action to do
the actual deployment. The last step registers a webhook with our telegam bot.
It is rather handy that we can reuse the address of our cloud function as an
output of the previous step.

Note that the [GitHub Secrets][GitHub Secrets] are used for service account
name, deployment token and telegram token. They can be set up via Web UI or
(what is preferred) GitHub CLI:

``` bash
gh secret set GC_TOKEN < service_account_token_from_google_cloud.json
gh secret set TELEGRAM_TOKEN -b '<token_provided_by_@botfather>'
gh secret set GC_SERVICE_ACCOUNT -b service_name@project_id.iam.gserviceaccount.com
```

Here we see that deployment token is a JSON file while the rest are string
literals.

[GitHub Secrets]: https://docs.github.com/en/actions/security-guides/encrypted-secrets
[deploy-cloud-functions]: https://github.com/marketplace/actions/cloud-functions-deploy
[google cloud auth]: https://github.com/marketplace/actions/authenticate-to-google-cloud

The deployment takes some time (up to 2 minutes in my observation), and this is
a wonderful reason of why we should always strive to do the automation. Instead
of running a deployment manually and then waiting until it is over during few
minutes, we can push our changes and switch to something else.

## Conclusion

Ok, at this point we got a working bot with a delivery pipeline. Each change
gets deployed within few minutes after it was pushed. In the next posts we will
add more checks to ensure that our bot is not broken by some bug.

Stay tuned!
