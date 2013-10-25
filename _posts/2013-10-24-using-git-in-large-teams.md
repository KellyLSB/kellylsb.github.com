---
layout: post
title: "Using Git in Large Teams"
description: "Why rebasing is important"
category: startups
tags: [git, startups, optimization]
---
{% include JB/setup %}

Git IMHO the best source control software there is, and now that we have GitHub it is being utilized everywhere. Unfortunately most people don't really know all it's ins and outs and again thanks to GitHub most people don't need to. GitHub handles branching, merging, issues, pull requests and now even provides a minified IDE. But we all want to be successful startup founders someday and that means that our companies are going to grow fast and then using the GitHub provided shortcuts don't work too well anymore. So this post is about using Git in a large team environment that makes merge conflicts relatively easy and small while making the code review process on the managers and engineers as simple as possible.

So for the purposes of this example and methodology we are going to have 1 primary branch, 1 secondary branch and 4 branch-tags and a tag group along side master.

    Master
    ├── Develop
    │   ├── feature/*
    │   ├── bugfix/*
    │   ├── optimization/*
    │   ├── security/*
    │   │
    │   ├── pairing/feature/*/[user]
    │   ├── pairing/bugfix/*/[user]
    │   ├── pairing/optimization/*/[user]
    │   └── pairing/security/*/[user]
    │
    ├── v1.0 (slinky)
    ├── v2.0 (torque)
    └── v2.1 (avacado)

Master is what is live, it is the release branch, every merge into master should be tagged with a version number and name (this is for rollback/history purposes). A changelog is recomended during this merge. A generic one could be as simple as compiling a list of all the commit messages and their authors and timestamps (and optionally changed files; lines of code etc). This process can be automated by scraping the Git log for commits between the current state of the master and the current state of develop before making the merge and bumping the version. Optionally you can integrate the Master branch with a CI (Continuous Integration) server and have production automatically deployed for you if the tests pass (which they should, come on it's master!)

Develop is the working branch, all the work will be merged into develop. You should never ever make a change in develop; only merge in changes from other branches! There are exceptions to this rule, but generally you should never commit directly to develop. All branches (feature, bugfix, optimization, security, pairing branches) should be branched off from develop and merged back into develop when complete.

Work branches (listed above at feature, bugfix, optimization and security) are where work should be done! The * in the branch name should be the ID of the ticket/story that the engineer is working on. If the engineer is going to be pairing with someone then they should have a new parent branch with the format `pairing/<type>/<story>` that their personal work branches merge into (i.e. If I'm pairing on the GitHub issue #4 and it is a bugfix, then I would create the branch `pairing/bugfix/GH-4` and then branch again off that with `pairing/bugfix/GH-4/KellyLSB`). When my personal branch off the pairing branch is ready, I would rebase off the pairing branch then submit a PR or merge into the pairing branch (when using a PR that the other engineers I'm pairing with would review my changes before merging into the pairing branch). When all the engineers are ready to submit the pairing branch then we would rebase off Develop and merge or submit a PR from the pairing branch back into Develop.

## Rebase/Merge Process

Rebasing is a Git technique that rewinds your branch back to the base from which you branched off from your parent branch and then replays the changes one by one back into your repository; allowing you to resolve conflicts along the way. The process I recomend (from that to finish) is outlined as follows.

### Without Pairing

1. Branch off develop
  - `git checkout develop`
  - `git checkout -B feature/GH-2`
  - `git push -u origin feature/GH-2`
2. Write Code (Commit often; every file change, spec pass, etc...)
  - Don't forget to use `git status`, `git diff` and selectively add your files as necessary.
  - Do not squash your commits (it makes it hard to retroactively makes changes later)!
  - Do not force push to your branch (unless it is necessary)
3. When you are ready to merge back to develop, rebase off develop
  - `git checkout develop`
  - `git pull origin develop`
  - `git checkout feature/GH-2`
  - `git rebase develop`
4. If there is a conflict resolve it. (If not skip to #6)
  - Do not change any files other than the conflict resolution!
  - A merge tool such as MeldMerge or Kaliedoscope may be helpful.
5. Once you have resolved your conflict add the changes and continue the rebase
  - `git add ./path/to/changed/file [./more/files, ...]`
  - Don't forget to check git status
  - `git status`
  - `git rebase --continue`
  - If another conflict emerges go back to step #4
6. Congratulations you have successfully rebased. You may now submit a PR or merge back into develop
  - Submit a PR on GitHub or...
  - `git checkout develop`
  - `git merge feature/GH-2`
7. Throw a party!

### With Pairing

1. Branch off develop
  - `git checkout develop`
  - `git checkout -B pairing/feature/GH-2`
  - `git push -u origin pairing/feature/GH-2`
  - `git checkout -B pairing/feature/GH-2/KellyLSB`
  - `git push -u origin pairing/feature/GH-2/KellyLSB`
2. Write Code (Commit often; every file change, spec pass, etc...)
  - Don't forget to use `git status`, `git diff` and selectively add your files as necessary.
  - Do not squash your commits (it makes it hard to retroactively makes changes later)!
  - Do not force push to your branch (unless it is necessary)
3. When you are ready to merge back to the pairing branch, rebase off the paring branch
  - `git checkout pairing/feature/GH-2`
  - `git pull origin pairing/feature/GH-2`
  - `git checkout pairing/feature/GH-2/KellyLSB`
  - `git rebase pairing/feature/GH-2`
4. If there is a conflict resolve it. (If not skip to #6)
  - Do not change any files other than the conflict resolution!
  - A merge tool such as MeldMerge or Kaliedoscope may be helpful.
5. Once you have resolved your conflict add the changes and continue the rebase
  - `git add ./path/to/changed/file [./more/files, ...]`
  - Don't forget to check git status
  - `git status`
  - `git rebase --continue`
  - If another conflict emerges go back to step #4
6. Congratulations you have successfully rebased. You may now submit a PR or merge back into the pairing branch
  - Submit a PR on GitHub or...
  - `git checkout pairing/feature/GH-2`
  - `git merge pairing/feature/GH-2`
7. Collaborate with the other engineers you are pairing with.
  - Is the branch ready to be rebased back into develop?
  - Is there more changes that need to be made?
  - If the branch is ready continue otherwise checkout your personal pairing branch again and continue with step #2.
8. Ready to merge back to develop, rebase of develop
  - `git checkout develop`
  - `git pull origin develop`
  - `git checkout pairing/feature/GH-2`
  - `git rebase develop`
9. If there is a conflict resolve it. (If not skip to #11)
  - Do not change any files other than the conflict resolution!
  - A merge tool such as MeldMerge or Kaliedoscope may be helpful.
10. Once you have resolved your conflict add the changes and continue the rebase
  - `git add ./path/to/changed/file [./more/files, ...]`
  - Don't forget to check git status
  - `git status`
  - `git rebase --continue`
  - If another conflict emerges go back to step #9
11. Congratulations you have successfully rebased. You may now submit a PR or merge back into develop
  - Submit a PR on GitHub or...
  - `git checkout develop`
  - `git merge pairing/feature/GH-2`
12. Throw a party!
