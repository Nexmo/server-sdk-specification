# Adopting conventional commits
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Problem

When releasing an OSS package, it is customary to include a changelog detailing what has been fixed, added or modified. The current process is manual, which requires devs on the tooling team to neglect changes or forget that may be one step in t is easy to process, it is easy to ignore changes or forget what is included in the update. This can be compounded if the release has been staged for months and the developer has to go through the git log to determine what has changed. By structuring commits (or PR messages as described below), automatically generating changelogs becomes a breeze and will require minimal effort to build out. This RFC proposes using [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) as the de facto standard followed by the tooling team.

## Duration
16 Sep 2022 - 23 Sep 2022

## Current State
Proposed

## Proposers
Chuck Reeves

## Detail
Conventional commits follow a structure for commit messages that many tools can quickly parse. To summarize the [entire specification](https://www.conventionalcommits.org/en/v1.0.0/#specification), a commit message is broken into three parts: **header**, **body** and **footer**, each separated by two newline characters (_\n_). **header** REQUIRED while the **body** and **footer** are OPTIONAL

The **header** consists of a REQUIRED _noun_(see this list below of proposed nouns), OPTIONAL _scope_ (see below for proposed scopes), or exclamation point "!", REQUIRED to be terminated with a colon "_:_" then a REQUIRED short description. The header is essential as it is used to determine the changelog message and MAY be used to trigger release actions in GitHub.

The **body** can provide expanded information for the commit. Note: the **body** MAY contain two new line characters as the **header** and **footer** require the use of the colon after the first word. Keep that in mind when writing out **bodies**,

One or more **footer** is used to create references (Link to a GitHub issue or Jira ticket) along with helping out SemVar (if the footer contains the word "BREAKING CHANGE", it denotes that merging in this commit should bump the major version.

## Proposed Solution
Before implementing a solution, a merge strategy would need to be decided on as there are different tools and GitHub actions to accomplish the goal.

### Merge Strategy

#### Merge Commits

If we want to continue using merge commits, then each commit message MUST follow the format of the conventional commits. Tools are configured to ignore merge commits and will use each commit to build out change logs. This would require a GitHub action to check the commit message when pushed up. If it fails, the commit message MUST be amended and force-pushed.

#### Squash

With squashing, the PR title and description will become a commit message (making it easier to conform to the standard). This can help keep change logs cleaner as fewer messages will be in the change log (also reduce [this](https://xkcd.com/1296/) famous example). This would require the PR to be linted to follow the standard.

### Nouns and Scopes

Once we decide on the merge strategy, we must determine which _nouns_ and _scopes_ we should use. This may seem trivial, but they help automation tools determine what to do and when. For example, if the message follows "docs: expanded description for property". We can filter out that commit from publishing a review candidate as no code has changed.

#### Nouns (or types)

The Conventional Commits spec only describes the nouns: "feat" and "fix". Which can be somewhat restrictive when trying to optimize build pipelines. [The contributing guide](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#type) for Angular expands the nouns to include

-   **build**: Changes that affect the build system or external dependencies
-   **ci**: Changes to our CI configuration files and scripts
-   **docs**: Documentation only changes
-   **feat**: A new feature
-   **fix**: A bug fix
-   **perf**: A code change that improves performance
-   **refactor**: A code change that neither fixes a bug nor adds a feature
-   **style**: Changes that do not affect the meaning of the code (white space, formatting, missing semi-colons, etc.)
-   **test**: Adding missing tests or correcting existing tests

That list is pretty comprehensive; however, I would suggest adding:

-   **release:** Changes are part of a release package

##### Proposed List

Below is the proposed list of nouns to be voted on

-   **build**: Changes that affect the build system or external dependencies
-   **ci**: Changes to our CI configuration files and scripts
-   **docs**: Documentation only changes
-   **feat**: A new feature
-   **fix**: A bug fix
-   **perf**: A code change that improves performance
-   **refactor**: A code change that neither fixes a bug nor adds a feature
-   **style**: Changes that do not affect the meaning of the code (white space, formatting, missing semi-colons, etc.)
-   **test**: Adding missing tests or correcting existing tests
-   **release:** Changes are part of a release package

#### Scopes

Scopes provide context to a commit and are OPTIONAL. However, it is good to define what scopes are to be used. I would recommend that scope be tied to a product line (voice, messages, video). Scopes can then help the group commits to the release of the product.

##### Proposed list:

-   server
-   video
-   voice
-   message
-   sms
-   auth
-   account
-   application
-   number
-   verify
-   ai
-   report
-   conversation
-   audit
-   client

### Tools

Several tools can help facilitate enforcing conventional commits:

-   [Commit lint](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional)
-   [Release please](https://github.com/googleapis/release-please)
-   [Conventional commit GitHub action](https://github.com/marketplace/actions/conventional-commit-checker)
-   [Action to check PR title](https://github.com/marketplace/actions/conventional-pr-title)
-   [Changelog builder](https://github.com/conventional-changelog/conventional-changelog)

## Record of Votes
Since there are many options, add yourself to the table with how you vote. Please specify if you want to have squash or merge as the strategy, along with your approval of the noun and scope list.

| Person |  Accept proposal | Squash Or Merge | Noun List | Scope List |
| ------ | ---------------- | --------------- | --------- | ---------- |
| Chuck Reeves | ✅ | Squash | ✅ | ✅

## Resolution