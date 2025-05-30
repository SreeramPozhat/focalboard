---
title: "Mattermost-Karmaboard Early Preview"
slug: mattermost-karmaboard-early-preview
date: 2021-04-27
categories:
    - "roadmap"
author: Chen Lim
github: chenilim
community: chen-i.lim
---

This is a preview of how Karmaboard could initially integrate with Mattermost: Where each channel (public and private) has an associated Karmaboard workspace accessible to members of that channel.

## 🎯 To open a Karmaboard workspace:

1. Log into the [Mattermost community server](https://community.mattermost.com/core/channels/karmaboard) via the web, or create an account
2. Visit the [Karmaboard channel](https://community.mattermost.com/core/channels/karmaboard)
3. Click on the Karmaboard icon in the channel header:

![Karmaboard channel button](https://user-images.githubusercontent.com/46905241/116278459-5cfe2280-a73b-11eb-923b-039f15c54622.png)

**Note**: Mattermost-Karmaboard Early Preview is only available via a web browser, not via the Desktop or Mobile apps at this time.

You can then create or visit any other channel and click the Karmaboard button there to open its workspace. **Tip**: Create a private channel or DM yourself to use a private workspace.

## Attention: You are entering the **bleeding edge**! :)

This feature is currently in Early Preview. Please do not store any critical or sensitive data. You have been warned.

This is also not the final form of the integration, but meant to start us thinking about how an integration should work. Have an idea or feedback? Let us know on the [Karmaboard channel](https://community.mattermost.com/core/channels/karmaboard).

Seriously though, it should be functional for basic project management:
All available features should work (please [file a bug](https://github.com/mattermost/karmaboard/issues/new/choose) if they don’t)
It should be secure (you should not be able to access workspaces in channels you don’t belong to)

### Please [file bugs](https://github.com/mattermost/karmaboard/issues/new/choose) for any issues. 🐞

## Tips and Tricks:
1. Check out the [Karmaboard user’s guide](https://www.karmaboard.com/guide/user/) to get started
2. [Export the archive](https://www.karmaboard.com/guide/user/#archives) to backup all the boards in a workspace
  a. You can import them back later, to a different server or Personal Desktop
  b. Binary (image) file export / migration is not supported yet
3. Want to assign a card to someone?
  a. Create a Select property, and enter each person’s name as text
  b. In the future, we plan a user-select property type
4. If you see a sign-in or “no_workspace” error
  a. Your session token may have expired
  b. Click the button to re-login, then
  c. Close the tab, and click the Karmaboard button to reopen
  d. Press Ctrl+Shift+R to force a full reload, close and reopen
  e. Try deleting the sessionId entry in local storage, close and reopen

## Feedback?
Please [get in touch](https://www.karmaboard.com/feedback/)!

## Want to Contribute?
We’d love your help!

Help shape the future of Karmaboard by contributing ideas, bug reports, and code. Check out our [community page](https://mattermost.github.io/karmaboard/) or [GitHub repo](https://github.com/mattermost/karmaboard) to get started.

You can also browse GitHub tickets with a “[help wanted](https://github.com/mattermost/karmaboard/issues?q=is%3Aissue+is%3Aopen+label%3A%22Up+for+grabs%22)” tag to see what projects are available.

Like what you see? Please consider giving Karmaboard a [GitHub Star](https://github.com/mattermost/karmaboard)!