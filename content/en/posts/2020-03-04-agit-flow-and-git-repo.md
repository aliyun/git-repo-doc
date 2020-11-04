---
title: "Agit-Flow and git-repo"
slug: "agit-flow-and-git-repo"
date: 2020-03-04T08:00:00+08:00
draft: false
---

## About myself

Due to the coronavirus outbreak in China, I was unable to attend the [Git Merge 2020](https://git-merge.com) conference to give [a speech on "AGit and git-repo"](https://git-merge.com/#xin-jiang) in Los Angeles on March 4, 2020.  It's a pity that I can't communicate face to face with developers all over the world this time. I will share the content of my speech on this blog.

A brief introduction to myself:

* I am Jiang Xin, a software engineer in Alibaba, China. 

* You might be familiar with the feature I contributed to Git: interactive git clean, and I recommend you to use `git clean -i` instead of `git clean -n/-f`.

* I am also the Git l10n coordinator helping 14 localization teams to improve multilingual for Git.

* I wrote a book on Git in Chinese and sent copies to Junio and Linus 10 years ago.

## The secret of Git's success

Git is the best choice of SCM and becomes the infrastructure for many popular source code management platforms. What is the secret behind Git's success?

Linus, the founder of Git and Linux, revealed the secret of Git's success in an interview on Git's 10th anniversary:

> The big thing about distributed source control is that it makes one of the main issues with SCM’s go away – the politics around “who can make changes.”

The keyword in the above statement is "politics". When using traditional SCM, administrators could only add write permissions to core members, so that lots of potential contributors could not make contributions to the project freely, and that's not good for open-source software. 

The distributed nature of Git makes it possible for a project to have a more flexible workflow. Not only core members but also read-only users can contribute to the project in a more elegant way. For example, GitHub invented a "Fork + Pull Request" development model, which is the most classic and widely-used workflow so far. Instead of pushing the commits directly to a branch of the remote repository, a contributor can push his/her commits to his/her own forked fully-controlled repository and create a code review (Pull Request) later via the web interface.


## The two most commonly-used Git workflow

There are two popular Git workflows that are introduced by GitHub and Gerrit. The two workflows have these common features:

1. A simpler authorization model for repositories. Do not need to add write permissions to developers one by one; All read-only users can make contributions to repositories.

2. High code quality because of code review. Contributors create code reviews instead of push commits to branch directly.

3. A simpler branch model. Do not need to create a feature branch as a temporary place to create a code review.

{{< figure src="/images/agit-flow/github-gerrit-comparisons-en.png" width="750" caption="Fig: Comparisons of GitHub and Gerrit" >}}

Differences between these two workflows:

1. Different models for code review.

    Code review in GitHub is called "pull request", and each feature has a pull request.

    Code review in Gerrit is called "change", and each commit generates a change.

2. Different types of workflow.

    The workflow of GitHub is a distributed workflow. Contributors work in their own repositories.

    The workflow of Gerrit is a centralized workflow. All users work in the same repository.

3. Different technical details.

    GitHub workflow is "fork + pull request", and its backend is CGit.

    Gerrit users should install a special hook "commit-msg" inside their local repositories so that every commit generated in local repositories has a unique "Change-Id". Gerrit uses JGit as its backend.

4. Pros:

    GitHub users use standard git command, so GitHub is easy to get going. Users have full control of their forked repositories. GitHub is the largest open-source community with lots of projects, and one project can be shared to others.

    Gerrit repositories are manageable for administrators. Android has a better solution for multi-repositories management than git-submodule.

5. Cons:

    GitHub workflow is too heavy for one-time contribution and is hard to use with a multi-repository project like Android.

    Gerrit is managed by one or more administrators, and a user cannot create their own repositories freely. So, one Gerrit server always hosts repositories for one project or one team. It is hard for Gerrit to form a software community.


## AGit-Flow Usage

### What is AGit-Flow?

Can we combine these two Git workflows to create a new one with all their advantages? (not like GerritHub)

Inspired by Gerrit workflow, we created a centralized git workflow based on CGit (instead of JGit) with a minor changed git-core and several APIs. We call the workflow as “AGit-Flow”, and implemented it in Alibaba's internal source code platform. With the help of "AGit-Flow", it's unnecessary to fork a repository or create many feature branches inside a repository. Users can use `git push` command to create code review (pull request) directly.

{{< figure src="/images/agit-flow/agit-flow-overview-en.png" width="750" caption="Fig: AGit-Flow relationship with GitHub and Gerrit" >}}

In Alibaba, we like pull requests, we like to create or update code reviews directly from the command line, and we like CGit. We don't like the commit-msg hook to mangle our commits, and we are not willing to have multiple code platforms (each of which hosts part of our repositories).

We developed a command-line tool named "git-repo", so we do not need to type long and complicated `git push` command. "git-repo" can be used for Gerrit, AGit-Flow and AGit-Flow alike workflows.


### Work with AGit-Flow

See the following graph for a typical AGit-Flow workflow for a single repository:

{{< figure src="/images/agit-flow/agit-flow-diagram-en.png" width="600" caption="Fig: AGit-Flow diagram for single repository" >}}

In the above graph, there are two roles. One is a developer, and the other is a committer.

The developer creates and updates a pull request with the following steps.

1. Clone the remote repository to local.

2. Work in the local worktree, and create a new commit.

3. Run `git pr` command in local worktree to push the local commit to the remote server.

4. Create a new pull request on the server side. (e.g., pull request #123).

5. After receiving comments from reviewers, the developer makes a change in the worktree and creates a new commit.

6. Run `git pr` command for a second time, which will push the new commit to the remote server.

7. The remote server finds that the same user pushes commits to the same target branch with the same session (local branch), and will update an existing pull request, instead of creating a new one.


The committer can write comments for the pull request; what's more, the committer can update the pull request with a new commit.

8. The committer runs command `git download 123` to download pull request #123 to the local repository.

9. The committer makes modifications and creates a new commit in worktree. Then, run command `git pr --change 123` to send local commit to the remote server.

10. The remote server detects this special `git push` command and will update the existing pull request which created by the developer.

11. The committer (or administrator) merge the pull request to the target branch.


The following demo is AGit-Flow on a single repository of Alibaba code platform:

{{< figure src="/images/git-repo-single.gif" width="750" caption="Fig: Demo of AGit-Flow on a single repository" >}}


## AGit-Flow Implementation

A special `git push` command is used to trigger AGit-Flow workflow. This special command has a special refspec, e.g.:

    $ git push origin HEAD:refs/for/<target-branch>/<session>

1. The target reference name of the special refspec has a prefix "refs/for/". Next to the prefix is "&lt;target-branch&gt;", which points to a corresponding branch in the remote repository. The last part in the special refspec is "&lt;session&gt;",  and we always use name of the local branch as "&lt;session&gt;". If the second `git push` command has the same target branch, the same session and from the same user, will update the already existed pull request generated by the first command instead of creating a new one.

2. There are other special prefixes for AGit-Flow, such as "refs/drafts/", and "refs/for-review/".

    The prefix "refs/drafts/" has the same format and usage as the prefix "refs/for/", but will create a pull request in draft mode. A draft mode pull request cannot be merged.

    The prefix "refs/for-review/" follows a pull request ID and can be used to update specific pull request which may be generated by others.

Below is a diagram of modules of the AGit-Flow server:

{{< figure src="/images/agit-flow/agit-flow-impl-en.png" width="750" caption="Fig: modules of AGit-Flow server" >}}

### Front-end authentication

The `git push` command first reaches the front-end module of the server. The front-end module will check user permissions, and only allow a user with write permission to push.

As we said in the beginning, a "politically correct" Git workflow should allow read-only users to "write" to the repository. So how can we make a proper authorization system for the front-end to allow a user with write permissions to run normal `git push` (updating heads or tags), and to allow a user with read-only permissions to run special `git push` command of AGit-Flow?

The solution is a special environment variable or HTTP header sent to the server like follows.

{{< figure src="/images/agit-flow/impl-1-front-end-en.png" width="700" caption="Fig: Front-end authentication for AGit-Flow" >}}

The client sends a special environment for SSH protocol or adds a special HTTP header for the special `git push` command to the server.

+ For the HTTP protocol, the `git push` command from the client side has a special option `-c http.extraHeader=AGIT-FLOW: <agent-version>`, which will add a special HTTP header in the request to the HTTP front-end.

+ For the SSH protocol, the client side will use a special SSH command defined by the "GIT_SSH_COMMAND" environment, and the special SSH command with the option `-o SendEnv=AGIT_FLOW` will send a special environment to the SSH front-end.

+ When the front-end finds a special environment (SSH protocol) or a special HTTP header (HTTP protocol), it will use a loose authorization rule (only checking for read-only permissions).

+ Warning: Please check write permission for the repository in the "pre-receive" hook to prevent a read-only user with a forged environment.


### Patched git-core and proc-receive hook

Next, requests from the client side will be delivered to the "git-receive-pack" process.

{{< figure src="/images/agit-flow/impl-2-git-core.png" width="320" caption="Fig: Original git-receive-pack diagram" >}}

The original "git-receive-pack" process works like the follows:

1. Requests of the client (commands and packfile) are sent one after another to the "git-receive-pack" process.

2. Packfile is saved (packed or unpacked) into a temporary directory for quarantine. Commands will be sent to the "pre-receive" hook after parsed.

3. If the "pre-receive" hook failed to execute, the quarantine directory will be deleted, and quit.

4. Move files in quarantine directory to objects directory.

5. Pass commands to the internal "execute_commands" function, execute commands (create, update, or delete references).

6. Run the "post-receive" hook at last for notifications.


We have contributed our patched "git-receive-pack" for AGit-Flow to the Git community, See:

* [https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/](https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/)

The following diagram is our patched "git-receive-pack" for AGit-Flow:

{{< figure src="/images/agit-flow/impl-2-git-core-patched.png" width="600" caption="Fig: Patched git-receive-pack for AGit-Flow" >}}

The changes we made for "git-receive-pack" to support AGit-Flow are:

1. Add a filter for commands sent from users to "git-receive-pack".

2. The filter helps to divide commands into two groups. One group of commands performs the original process, and the other group of commands does not execute the internal `execute_commands` function, but executes an external "proc-receive" hook instead.

3. Send an extended status report from "receive-pack" to cliend-side ("send-pack").

See the following sections.


#### Git config variable: receive.procReceiveRefs

{{< figure src="/images/agit-flow/proc-receive-1.png" width="320" >}}

Push requests (commands) are sent to the server (git-receive-pack）one by one through its standard input. Each command has the following format:

    <old-oid> <new-oid> <reference>

Reference names of commands of AGit-Flow style `git push` have different prefixes other than "refs/heads/" and "refs/tags/". We introduced a new git config variable "receive.procReceiveRefs" for Git to recognize the special commands for AGit-Flow. For example, we use the following settings for Alibaba code platform:

    git config --system --add receive.procReceiveRefs refs/for
    git config --system --add receive.procReceiveRefs refs/drafts
    git config --system --add receive.procReceiveRefs refs/for-review

The above git commands add three values for config variable "receive.procReceiveRefs". Commands from clients matched either value of this config variable will be marked with a particular tag to be handled differently later.


#### New proc-receive Hook

The commands marked with the particular tag will not be sent to the internal `execute_commands` function. They will be executed by a new external hook: "proc-receive".

This hook executes once for the receive operation.  It takes no arguments, but uses a pkt-line format protocol to communicate with "receive-pack" to read commands, push-options and send results.

{{< figure src="/images/agit-flow/proc-receive-2.png" width="800" >}}

1. Version negotiation between "receive-pack" and "proc-receive".

   First, "receive-pack" sends the protocol version and capabilities to the "proc-receive" hook. The hook will send back the version it supports. Currently supported capablilities include: push options, atoms, etc.

2. Send commands and push-options from "receive-pack" to the "proc-receive" hook.

   Each command will be sent from "receive-pack" to "proc-receive" in one pkt-line of the format:

        <old-oid> <new-oid> <reference>

   Commands will end with a flush-pkt.

   Only when both "receive-pack" and "proc-receive" support "push-options"，"receive-pack" will send push-options to "proc-receive".

3. The "proc-receive" hook will call an external API to execute the commands. In Alibaba, this API is used to create or update a pull request.

4. The "proc-receive" hook sends status report to "receive-pack". The following formats of report are supported.

    + `ok <ref>`

      Successfully create or update `<ref>`.

    + `ng <ref> <reason>`

      Fail to update `<ref>`, and the error message is given by `<reason>`.

    + `alt <ref> [<alt-ref>] [old-oid=<oid>] [new-oid=<oid>] [forced-update]`

      Expect to update `<ref>`, but update an alternamte reference (`<alt-ref>`).  `old-oid`、`new-oid` and other attributes can be given by optional key-value pairs.

    + `ft <ref>`

      Fall through, let 'receive-pack' to execute it.


#### Extended status report for client-side ("send-pack")

The "proc-receive" hook may receive a command for a pseudo-reference with a zero-old as its old-oid, while the result of the hook may point to an alternate reference and the reference may exist already with a non-zero old-oid.  We extended the format of report from "receive-pack" to client-side ("send-pack"), so git client can report correctly.

{{< figure src="/images/agit-flow/proc-receive-3.png" width="800" >}}

For example, the following `git-push` command has a pseudo-reference in its refspec:

    $ git push origin HEAD:refs/for/master/topic

New version of Git can report the actual reference updated:

    To <URL/of/upstream.git>
     + 263ea37...e5a9ada  HEAD -> refs/pull/123/head (forced update)


### Public API: ssh-info

Gerrit HTTP service provides a `ssh_info` API, which returns the IP address and port of the SSH server. Android repo will use this API to connect the remote server using SSH passwordless authentication.

AGit-Flow makes an extension for this `ssh_info` API. The return value of the API is not plain text anymore but is a JSON including protocol type (such as "agit") and protocol version. And the extended `ssh_info` is not an abbreviation of "Secure Shell information", but is the abbreviation of "Smart Submit Handler information". This API can be provided not only from the HTTP service but also from the SSH service.

To see different `ssh_info` API results and various corresponding `git push` commands, see the following chart.

{{< figure src="/images/agit-flow/impl-4-ssh-info-en.png" width="750" caption="Fig: `ssh_info` - the Smart Submit Handler information API" >}}


## git-repo

git-repo is a command-line tool for centralized workflow, can work with Gerrit, AGit-Flow compatible servers. It is written in Golang, and it can be installed easily without further dependency. It provides an easy-to-use solution for multiple repositories which is introduced by Android repo first, and it can also work with a single repository.

git-repo is an open-source software on GitHub.

+ Source code: [https://github.com/alibaba/git-repo-go](https://github.com/alibaba/git-repo-go/) 
+ Website: [https://git-repo.info](https://git-repo.info/)

### Installation

1. Download the latest release of "git-repo" from [https://git-repo.info/en/download/](https://git-repo.info/en/download/).

   Choose the proper package according to your OS (Linux, Mac, or Windows) and architecture (AMD or 386), download and extract the package.

2. Make "git-repo" executable before install on Linux or macOS:

        $ chmod a+x git-repo

3. Move "git-repo" into an executable PATH (e.g., `/usr/local/bin` for Linux or macOS, `C:\Windows\system32` for Windows):

        $ sudo mv git-repo /usr/local/bin/


### Execute git-repo for the first time

git-repo will perform some initialization tasks on the first execution. See: [git config extensions](../../gitconfig). Running any command will trigger the initialization tasks.

    $ git repo --version

Note: git-repo will be recognized as git subcommand so that we can execute `git repo` (without a dash).

The following alias commands are installed by the initialization tasks:

{{< figure src="/images/agit-flow/git-repo-aliases-en.png" width="450" caption="Fig: Alias commands of git-repo" >}}


Show help message of git-repo:

    $ git repo -h

Show help message of subcommands of git-repo, see the following examples:

    $ git repo help init
    $ git repo help sync


### Working on a single repository

git-repo provides several alias commands for git workflow on a single repository:

* `git peer-review` or `git pr`: publish local commits to create a code review.
* `git download`: download a code review from remote to the local repository.
* `git abandon`: prune already published local branch.

Demo for git workflow on a single repository:

{{< figure src="/images/git-pr-demo.gif" width="750" caption="Fig: Demo of git pr" >}}


### Working on multiple repositories

Git built-in solution for multiple repositories is git-submodule. It is OK for the release of the source code of all related repositories, but it is bad for development because all sub repositories pin to specific commits.

Android project introduced a better solution for the management of multiple repositories that the relationship of repositories is defined in an XML file in a manifest project. Android also introduced a new command-line tool named repo, which helps manage repositories.


git-repo is compatible with Android repo and can work on multiple repositories managed by a manifest repository.

1. Create a workspace.

        $ mkdir workspace
        $ cd workspace

2. Initialize workspace from a manifest project (repository), which has an XML file (e.g., default.xml) to describe the relationship of the sub-projects.

        $ git repo init -u <manifest repository>

3. Fetch all sub-projects and checkout to the local workspace.

        $ git repo sync

4. Create a local branch on all projects for development.

        $ git repo start --all <branch/name>

5. Working in the workspace and make new commits.

6. Run the following commands to send local commits to the remote server to create code reviews (pull requests).

        $ git repo upload

The following example is how to manage multiple repositories using git-repo:

{{< figure src="/images/git-repo-manifest.gif" caption="Fig: Demo of git-repo for multiple repositories" >}}


### git-repo is extensible

git-repo is extensible, can work with Gerrit, AGit-Flow, and other compatible protocols.

You can add new protocol support in git-repo by:

+ Method 1: Add a new protocol helper, which implements the `ProtoHelper` interface in the `helper` directory of git-repo.

+ Method 2: Add an external program, which should support `--upload`、`--download` parameters, as the helper for your protocol.

{{< figure src="/images/agit-flow/impl-git-repo-extension-en.png" width="750" caption="Fig: git-repo protocol extension" >}}


## Summary

AGit-Flow is a centralized git workflow inspired by Gerrit, but with CGit, and use pull requests for code review.

git-repo is a client for AGit-Flow, developed using Golang, and is compatible with Android repo.

Open source on GitHub:

* https://github.com/alibaba/git-repo-go (Give me a ⭐️)

* https://github.com/alibaba/git-repo-go-doc (For website https://git-repo.info)

* [https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/](https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/) (Patches on git-receive-pack for AGit-Flow)

Implement your own "AGit-Flow":

* Assign a name for your AGit-Flow compatible protocol. (github-flow?)

* Make your front-end ready for read-only users to push.

* Install patched git-core in your server and set the special config variable to turn on the feature.

* Write your own "proc-receive" hook and internal code review API.

* Add a public `ssh_info` API, which returns a JSON response for service detection.

* Add a new helper to git-repo to support your public Git service.

----

Edit:

+ 2020/4/27: Hook is renamed from "execute-commands" to "proc-receive".
+ 2020/4/27: Use a pkt-line format protocol between "receive-pack" and "proc-receive".
