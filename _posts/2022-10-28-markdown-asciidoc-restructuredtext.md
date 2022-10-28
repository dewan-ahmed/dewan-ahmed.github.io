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

1. Write: Use a markup language to write the docs.
2. Extend (optional): Use a framework to avail features that are not part of the basic markup.
3. Host: Host the markup files on some repository. This could be self-hosted or a managed provider.
4. Automate build (optional although highly recommended): Add a CI with spelling, prose, link checks, etc.
5. Publish: Deploy your documentation live using platforms like GitHub (Pages), Vercel, or Netlify. 

In the first part of my docs-as-code series, I'll talk about the choice of markup languages, the available frameworks, and do a comparison among markdown, asciidoc, and reStructuredText based on some use cases. I'll skip the history but in case you're interested, my friend and colleague Tibs delivered a talk on the [history of text markup languages](https://www.youtube.com/watch?v=P-7hwjocEpM). My hope is to provide you a detailed analysis among these choice of markup languages so that you can make an informed decision when selecting one for your next developer documentation project. If you don't need a brief intro and would like to skip the appetizer, you can go straight to the entrée - [Choose the right markup](#choose-the-right-markup) section. 

## The choices - md, adoc, rST

Before choosing a markup language, you'll need to choose either a desktop code editor like [VSCode](https://code.visualstudio.com/) or stay in your terminal and use something like [~Emacs~ VIM](https://www.vim.org/). Most code editors will have plug-ins for popular markup languages which will offer you syntax highlighting, preview, etc. If you're testing or learning a specific syntax, there are many online editors, such as, [StackEdit for Markdown](https://stackedit.io/). 

The main choices for document markup languages are:

- Markdown (md)
- Asciidoc (adoc)
- reStructuredText (rST)

### Markdown

The [original version](https://daringfireball.net/projects/markdown/) of markdown by John Gruber doesn't specify the syntax unambiguously. For example, it doesn't have table formatting. Due to the popularity of markdown and the limitation of the Gruber markdown, a plethora of variation, or "flavors", of markdown has been introduced. Due to this lack of standard, folks like [Eric Holscher](https://twitter.com/ericholscher), who are passionate about the community, have been vocal about [not using markdown for documentation](https://ericholscher.com/blog/2016/mar/15/dont-use-markdown-for-technical-docs/). 

In 2014, a group of Markdown fans came together and established [CommonMark](https://commonmark.org/) - a standard, interoperable and testable version of Markdown. In March 2017, GitHub published a formal spec for GitHub-Flavored Markdown (GFM); based on CommonMark. In the [accompanying blog post](https://github.blog/2017-03-14-a-formal-spec-for-github-markdown/) when releasing GFM, GitHub addressed many of the limitations that Eric Holscher raised, things like how many spaces are needed to indent a line, or how many empty lines you need to break between different elements. GFM is, by far, the most popular flavor of markdown.

If you're just starting out with Markdown, check out the [syntax](https://www.markdownguide.org/basic-syntax/) and write some hello-world docs [online](https://markdownlivepreview.com/).

### Asciidoc

You started with Markdown but then you needed to embed a YouTube video. So you embed HTML within your Markdown doc. As your project grows, it's **Markdown + X** and so on. With out-of-the-box features like tables, admonitions (tips, notes, warnings, etc.), and table of contents, Asciidoc is a popular choice for documentation projects that plan to scale and have complex requirements. Asciidoc has no flavors so there's really just a single standard. You might have heard about GitHub Flavored Asciidoc (GFA). This is not a flavor per se, rather GitHub using Asciidoctor in safe mode to render files with the extension `.adoc`, `.ad`, and `.asciidoc`. More on Asciidoctor under the [framework section](#framework-framework-framework). [asciidocLIVE](https://asciidoclive.com/) is a great place to start if you're testing waters with Asciidoc and would like to learn the syntax.

### reStructuredText

If you're in the Python community, you might be using reStructuredText for documentation. A popular site that uses this markup lanuage is [WriteTheDocs](https://www.writethedocs.org/). The reStructuredText parser is a component of [Docutils](https://docutils.sourceforge.io/index.html) and  is the default markup language used by [Sphinx](https://www.sphinx-doc.org/en/master/index.html).  reStructuredText can be extendible using [custom directives](http://docutils.sourceforge.net/docs/ref/rst/directives.html) that can satisfy a wide variety of documentation needs. [Here](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html) is a quick primer on reStructuredText if you're getting started. You can play with the reStructuredText syntax using [this online editor](https://livesphinx.herokuapp.com/).  

## Framework Framework Framework

For a serious documentation project, you're almost certainly using a framework for specific features and the extension of the vanilla markup. In this section, let's check out some of the popular frameworks for Markdown, Asciidoc, and reStructuredText.

### Markdown frameworks

Considering that the end result of a documentation project is often a static site, the words **static site generator (SSG) tool** and Markdown framework can be used interchangeably. Here are my top three picks:

1. [Hugo](https://gohugo.io/) is probably the most popular open-source static site generator. Written in Go, Hugo builds sites at a blistering speed. It has a native support for a variety of content types, menus, and dynamic API-driven content. These don't require you to use an additional plugin. You can select a theme that suits your project from [a wide selection](https://gohugo.io/showcase/). Check out [some of the project websites](https://gohugo.io/showcase) that are built with Hugo.

2. Released in 2022, [Markdoc](https://markdoc.dev/) is a relatively new Markdown-based authoring framework. The Markdoc project is [open-source](https://github.com/markdoc/markdoc) and it powers [Stripe's documentation](https://stripe.com/docs). Their website has a [live edit](https://markdoc.dev/) button which makes the website a playground for you to give Markdoc a try. Documentations created with Markdoc will automatically render with your [React](https://markdoc.dev/docs/examples/react) app and using `@markdoc/next.js` for your [Next.js](https://markdoc.dev/docs/nextjs) app.  

3. Configured with a single YAML file, [MKDocs](https://www.mkdocs.org/) is the third Markdown framework on my list which falls in the category of SSG. Although not as many as Hugo themes, MKDocs offers a [few official themes](https://www.mkdocs.org/user-guide/choosing-your-theme/) and a number of [third party themes](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes). As a Python-based framework, you can use `pip` to install [MKDocs plugins](https://www.mkdocs.org/dev-guide/plugins/). You can follow [this getting started guide](https://www.mkdocs.org/getting-started/) for your first MKDocs project.

### Asciidoc frameworks



## The (common) objections

## Choose the right markup

If you're a developer, learning the syntax and styling for a markup language is relatively easy compared to a programming language. When you learn the syntax of a markup language and can render an HTML page locally, you reach the `hello world` nirvana of documentation. However, the difficulty doesn't necessarily start when you write the first documentation file. Maintainability is the major pain for documentation and hence, the choice of a markup language that suits your use case is extremely critical.

## Outro