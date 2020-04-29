# GitHub Review Bot

GitHub Review Bot is a sample [GitHub application](https://developer.github.com/apps/)
which demonstrates solutions to various challenges involved in writing real-world
applications for the platform.

GitHub provides excellent documentation on the available APIs and integration points
to their platform, but details about the application code are harder to come by. In
addition, many GitHub applications provide limited functionality which only touches
a subset of the APIs a real-world application would use. **GitHub Review Bot** is an
attempt to provide a working sample of a more realistic GitHub application, with
extensive documentation on the steps required.

We will be going into a lot of details about webhook payload validation, scoping
events and permissions for the application, authenticating as an application against
GitHub API and a lot more, using Python and a handful of libraries. If you do not
care about those details, or prefer writing code in Javascript, you might prefer
to check out [probot](https://probot.github.io/), an excellent framework
which does a lot of the heavy lifting involved in getting a GitHub application up
and running.

## Development Environment Setup

Before you can get started with the application development we need to set up
the development environment. We will be using
[Python 3.8.2](https://www.python.org/downloads/release/python-382/), the latest
version available at the time of writing this documentation. We will also use
[Python venv](https://docs.python.org/3/library/venv.html) to allow for easier
management of dependencies.

```sh
$ git clone https://github.com/undeadbeef/github-review-bot.git
$ cd github-review-bot
$ python -m .venv venv
$ source .venv/bin/activate
$ pip install -r requirements.txt
$ pip install -r dev-requirements.txt
$ pre-commit install
```

We will also need [smee.io](https://smee.io/) to enable webhook consumption while
running the GitHub application locally. **smee.io** requires
[node.js](https://nodejs.org/), which is beyond the scope of this documentation.
Once you have **node.js** up and running, you can install **smee.io** by simply
running:

```sh
$ npm install --global smee-client
```

## What is a GitHub application?

From https://developer.github.com/apps/about-apps/:

>GitHub Apps are first-class actors within GitHub. A GitHub App acts on its own behalf,
>taking actions via the API directly using its own identity, which means you don't need
>to maintain a bot or service account as a separate user.

>GitHub Apps can be installed directly on organizations and user accounts and granted
>access to specific repositories. They come with built-in webhooks and narrow, specific
>permissions. When you set up your GitHub App, you can select the repositories you want
>it to access. For example, you can set up an app called MyGitHub that writes issues in
>the octocat repository and only the octocat repository. To install a GitHub App, you
>must be an organization owner or have admin permissions in a repository.

In short, a GitHub application typically has two major parts to it:

* A webhook ingestion endpoint, which consumes GitHub events
* A GitHub API client, which performs actions on GitHub resources in response to events

### Webhook Ingestion

Webhook ingestion allows an application to be notified about relevant events happening
in GitHub. For example, the application might be interested in new pull requests being
opened, or on new releases being published.

The webhook ingestion endpoint is just a HTTP/HTTPS listener which can take ``POST``
requests with JSON content which holds information about events in GitHub. Request
headers hold information about the event as well as signature information, which we
can use to validate the content was published by GitHub for consumption by the
application. The request body contains a JSON payload, with a schema that depends
on the event type being published.
