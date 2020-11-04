---
title: "Quick start"
draft: false
weight: 11
enableToc: false
---

## Quick Start

### Install git-repo

You can download `git-repo` from: [https://git-repo.info/en/download/](https://git-repo.info/en/download/).

Choose the proper package according to your OS (Linux, Mac, or Windows), and architecture (amd or 386), download and extract the package.

Copy the binary into executable PATH (e.g. `/usr/local/bin` on Linux or macOS) to complete the installation.


### Run git-repo for the first time

Run any subcommand of `git-repo`, will perform some initialization tasks. e.g.:

    $ git repo version


### Working on a single repository

Make sure to setup a remote tracking branch correctly using:

    $ git branch -u origin/master

Create code review (pull request) from command line:

    $ git pr


### Working on multiple repositories

`git-repo` is compatible with Android `repo`, and can work on multiple repositories managed by a manifest repository.

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

6. Run the following commands to send local commits to remote server to create code reviews (pull requests).

        $ git repo upload
