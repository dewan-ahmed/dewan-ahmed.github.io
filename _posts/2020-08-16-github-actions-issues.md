---
title: "How to make a GitHub issue creator follow your rules?"
categories:
  - Blog
tags:
  - GitHub Actions
---

One of my friend sets up build infrastructure for his team and the process is initiated from a GitHub issue. Although the issue description states the required reading each user needs to do before using this infrastructure, many don’t take the time to actually read the docs and start messaging my friend with questions. In this tutorial-style blog, I offer my friend help through GitHub Actions where the issue creator is forced to check-off a checklist (of the required reading) or else their issues would be tagged with Incomplete Checklist label.

This tutorial has two parts:

- To [configure issue templates](https://docs.github.com/en/github/building-a-strong-community/configuring-issue-templates-for-your-repository) for your repository
- Adding [issue-checklist-checker](https://github.com/marketplace/actions/issue-checklist-checker) GitHub Actions on your repository

## Configuring Issue Template

On your issue-enabled GitHub repository, create a markdown file under .github/ISSUE_TEMPLATE and add a checklist in the description. Take a look at the following sample:

```markdown
---
name: Request access setup for your team
about: Request access setup for your team.
title: "[Access Setup Request]"
labels: 'access request'
assignees: 'dewan-ahmed'

---

Pre-requisites ::: You must check the following list before your issue gets addressed

- [ ] I have read the docs
- [ ] I have watched the videos
- [ ] I have done my research

* name: team-name
* team-admin-emails: email1, email2, ...
* team-user-emails: email1, email2, ...
```
You can get creative with the checklist and need to make changes to the above sample to fit your need.

## Configuring GitHub Actions

On your GitHub account, create a [Personal Access Token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) from Settings –> Developer Settings –> Personal access tokens.

On the specific repo where you have created the issue template, add the personal secret token under Settings –> Secrets. Keep a note of the name of the secret which you’ll need to provide in the following yml. Under .github/workflows directory, create main.yml and add the following:

```markdown
name: Issue Checklist Checker

on: issues

jobs:
  checkIssue:
    name: Check Issue for Incomplete Checklist
    runs-on: ubuntu-latest
    steps:
      - uses: adamzolyak/checklist-checker-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.PAT }}
```

I named my secret **PAT** so that’s what you see above. If you have a different name for your secert, be sure to make the necessary change.

That’s it! With this particular GitHub Action in place, any incomplete checklist item will result in Incomplete Checklist label on that issue. To make things even more interesting, if the issue is closed with incomplete checklist items, the action will reopen the issue and comment on the issue 😏.

You can check-out [this sample repo](https://github.com/dewan-ahmed/github-actions-issue-checker) for reference.

