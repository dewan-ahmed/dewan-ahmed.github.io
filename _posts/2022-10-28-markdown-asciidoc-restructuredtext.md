---
title: "Markdown, asciidoc, or reStructuredText - a tale of docs-as-code"
date: 2022-10-28T00:00:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/md-adoc-rst.jpg"
  alt: "Photo by Leah Kelley: https://www.pexels.com/photo/close-up-photo-of-gray-typewriter-952594/"
tags:
  - docs-as-code
  - technical writing
---

## Intro

Who writes the developer documentation for your product? During the early days of my tech career, I worked on a database replication product at IBM. My immediate team had developers and testers but not a single technical writer. I can speak for myself that I never bothered to know who keeps writing the lengthy and complex documentation for every release. Since then, many companies in tech (including companies like IBM) have taken a modern approach to documentation. This "modernization" started when more and more developers started writing documentation (not all developers hate writing docs 🤭). Having git running in their DNA, these developers wanted to bring in collaboration, version control, scripting, and easier bug tracking within developer documentation.

> :information_source: Note that I'm not using the term product documentation, rather developer documentation. If your users are developers, it's more likely that they would appreciate reading docs written by other developers. 

With the text files for documentation in version control system and changes going through the pull request process, the documentation is treated exactly like software code and hence docs-as-code. I see five stages in docs-as-code implementation:

1. Write: Use a markup language to write the docs
2. Extend (optional): Use a framework to extend features
3. Host: Host the markup files on some repository
4. Automate build (optional although highly recommended): Add a CI with spelling, prose, link checks, etc
5. Publish: Make your documentation live on platforms like GitHub (Pages) or Netlify. 

In the first part of my docs-as-code series, I'll talk about the choice of markup languages, the available frameworks, and do a comparison among markdown, asciidoc, and reStructuredText based on some use cases. I'll skip the history but in case you're interested, my friend and colleague Tibs delivered a talk on the [history of text markup languages](https://www.youtube.com/watch?v=P-7hwjocEpM). My hope is to provide you a detailed analysis among these choice of markup languages so that you can make an informed decision when selecting one for your next developer documentation project. If you don't need a brief intro and would like to skip the appetizer, you can go straight to the [Choose the right markup](#choose-the-right-markup) section. 

## The choices - md, adoc, rST

Before choosing a markup language, you'll need to choose either a desktop code editor like [VSCode](https://code.visualstudio.com/) or stay in your terminal and use something like [~Emacs~ VIM](https://www.vim.org/). Most code editors will have plug-ins for popular markup languages which will offer you syntax highlighting, preview, etc. If you're testing or learning a specific syntax, there are many online editor, such as, [StackEdit for Markdown](https://stackedit.io/). 

The main choices for document markup languages are:

- Markdown (md)
- Asciidoc (adoc)
- reStructuredText (rST)

### Markdown

The [original version](https://daringfireball.net/projects/markdown/) of markdown by John Gruber doesn't specify the syntax unambiguously. For example, it doesn't have table formatting. Due to the popularity of markdown and the limitation of the Gruber markdown, a plethora of variation, or "flavors", of markdown has been introduced. Due to this lack of standard, folks like [Eric Holscher](https://twitter.com/ericholscher) who are passionate about the community have been vocal about [not using markdown for documentation](https://ericholscher.com/blog/2016/mar/15/dont-use-markdown-for-technical-docs/). 

In 2014, a group of Markdown fans came together and established [CommonMark](https://commonmark.org/) - a standard, interoperable and testable version of Markdown. 

## Framework Framework Framework

## The (common) objections

## Choose the right markup

If you're a developer, learning the syntax and styling for a markup language is relatively easy compared to a programming language. When you learn the syntax of a markup language and can render an HTML page locally, you reach the `hello world` nirvana of documentation. However, the difficulty doesn't necessarily start when you write the first documentation file. Maintainability is the major pain for documentation and hence, the choice of a markup language that suits your use case is extremely critical.

## Outro