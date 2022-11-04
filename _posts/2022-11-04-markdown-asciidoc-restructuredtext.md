---
title: "Markdown, Asciidoc, or reStructuredText - a tale of docs-as-code"
date: 2022-11-04T00:00:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/md-adoc-rst.jpg"
  alt: "Photo by Leah Kelley: https://www.pexels.com/photo/close-up-photo-of-gray-typewriter-952594/"
tags:
  - docs-as-code
  - technical writing
  - devrel
---

Who writes the developer documentation for your product? During the early days of my tech career, I worked on a database replication product at IBM. My immediate team had developers and testers but not a single technical writer. I can speak for myself when I say that I never bothered to know who keeps writing the lengthy and complex documentation for every release. Since then, many companies in tech (including companies like IBM) have taken a modern approach to documentation. This "modernization" started when more and more developers started writing documentation (not all developers hate writing docs 🤭). Having git running in their DNA, these developers wanted to bring in collaboration, version control, scripting, and easier bug tracking within developer documentation.

> :information_source: Note that I'm not using the term "product documentation", rather "developer documentation". If your users are developers, it's more likely that they would appreciate reading docs written by other developers. 

With the text files for documentation in version control system and changes going through the pull request process, the documentation is treated exactly like software code and hence docs-as-code. I see five stages in docs-as-code implementation:

1. Write: Use a markup language to write the docs.
2. Extend (optional): Use a framework to avail features that are not part of the basic markup.
3. Host: Host the markup files on some repository. This could be self-hosted or a managed provider.
4. Automate the build (optional but highly recommended): Add a CI with spelling, prose, link checks, etc.
5. Publish: Deploy your documentation live using platforms like GitHub (Pages), Vercel, or Netlify. 

In the first part of my docs-as-code series, I'll talk about the choice of markup languages, the available frameworks, and do a comparison among Markdown (md), Asciidoc (adoc), and reStructuredText (rST) based on some use cases. I'll skip the history but in case you're interested, my friend and colleague Tibs delivered a talk on the [history of text markup languages](https://www.youtube.com/watch?v=P-7hwjocEpM). My hope is to provide you with a detailed analysis of these choices of markup languages so that you can make an informed decision when selecting one for your next developer documentation project. If you don't need a brief intro and would like to skip the appetizer, you can go straight to the entrée - [Choose the right markup](#choose-the-right-markup) section. 

## The choices - md, adoc, rST

Before choosing a markup language, you'll need to choose either a desktop code editor like [VSCode](https://code.visualstudio.com/) or stay in your terminal and use something like ~Emacs~ [VIM](https://www.vim.org/). Most code editors will have plug-ins for popular markup languages which will offer you syntax highlighting, preview, etc. If you're testing or learning a specific syntax, there are many online editors, such as, [StackEdit for Markdown](https://stackedit.io/). 

The main choices for document markup languages are:

- Markdown (md)
- Asciidoc (adoc)
- reStructuredText (rST)

### Markdown (md)

The [original version](https://daringfireball.net/projects/markdown/) of Markdown by John Gruber doesn't specify the syntax unambiguously. For example, it doesn't have table formatting. Due to the popularity of Markdown and the limitations of Gruber Markdown, a plethora of variations, or "flavors", of Markdown have been introduced. Due to this lack of standard, folks like [Eric Holscher](https://twitter.com/ericholscher), who are passionate about the community, have been vocal about [not using markdown for documentation](https://ericholscher.com/blog/2016/mar/15/dont-use-markdown-for-technical-docs/). 

In 2014, a group of Markdown fans came together and established [CommonMark](https://commonmark.org/) - a standard, interoperable and testable version of Markdown. In March 2017, GitHub published a formal spec for GitHub-Flavored Markdown (GFM); based on CommonMark. In the [accompanying blog post](https://github.blog/2017-03-14-a-formal-spec-for-github-markdown/) when releasing GFM, GitHub addressed many of the limitations that Eric Holscher raised, things like how many spaces are needed to indent a line, or how many empty lines you need to break between different elements. GFM is, by far, the most popular flavor of Markdown.

If you're just starting out with Markdown, check out the [syntax](https://www.markdownguide.org/basic-syntax/) and write some hello-world docs [online](https://markdownlivepreview.com/).

### Asciidoc (adoc)

You started with Markdown, but then you needed to embed a YouTube video. So you embed HTML within your Markdown doc. As your project grows, there will be  more HTML and some JavaScript embedded in your Markdown files. With out-of-the-box features like tables, admonitions (tips, notes, warnings, etc.), and table of contents, Asciidoc is a popular choice for documentation projects that want to avoid gluing HTML and JavaScript for advanced use. Asciidoc has no flavors so there's really just a single standard. You might have heard about GitHub Flavored Asciidoc (GFA). This is not a flavor per se, rather GitHub using Asciidoctor in safe mode to render files with the extension `.adoc`, `.ad`, and `.asciidoc`. More on Asciidoctor under the [framework section](#framework-framework-framework). [asciidocLIVE](https://asciidoclive.com/) is a great place to start if you're testing waters with Asciidoc and would like to learn the syntax.

### reStructuredText (rST)

If you're in the Python community, you might be using reStructuredText for documentation. A popular site that uses this markup language is [WriteTheDocs](https://www.writethedocs.org/). The reStructuredText parser is a component of [Docutils](https://docutils.sourceforge.io/index.html) and  is the default markup language used by [Sphinx](https://www.sphinx-doc.org/en/master/index.html). reStructuredText can be extended using [custom directives](http://docutils.sourceforge.net/docs/ref/rst/directives.html) that can satisfy a wide variety of documentation needs. [Here](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html) is a quick primer on reStructuredText if you're getting started. You can play with the reStructuredText syntax using [this online editor](https://livesphinx.herokuapp.com/).  

## Framework Framework Framework

For a serious documentation project, you're almost certainly using a framework for specific features and to extend the limit of the vanilla markup. In this section, let's check out some of the popular frameworks for Markdown, Asciidoc, and reStructuredText.

### Markdown frameworks

Considering that the end result of a documentation project is often a static site, the words **static site generator (SSG) tool** and Markdown framework can be used interchangeably. Here are my top three picks:

1. [Hugo](https://gohugo.io/) is probably the most popular open-source static site generator. Written in Go, Hugo builds sites at a blistering speed. It has a native support for a variety of content types, menus, and dynamic API-driven content. These don't require you to use an additional plugin. You can select a theme that suits your project from [a wide selection](https://gohugo.io/showcase/). Check out [some of the project websites](https://gohugo.io/showcase) that are built with Hugo.

2. Released in 2022, [Markdoc](https://markdoc.dev/) is a relatively new Markdown-based authoring framework. The Markdoc project is [open-source](https://github.com/markdoc/markdoc) and it powers [Stripe's documentation](https://stripe.com/docs). Their website has a [live edit](https://markdoc.dev/) button which makes the website a playground for you to give Markdoc a try. Documentations created with Markdoc will automatically render with your [React](https://markdoc.dev/docs/examples/react) app and using `@markdoc/next.js` for your [Next.js](https://markdoc.dev/docs/nextjs) app.  

3. Configured with a single YAML file, [MKDocs](https://www.mkdocs.org/) is the third Markdown framework on my list, which falls in the category of SSG.  Although there are not as many as in Hugo, MKDocs offers a [few official themes](https://www.mkdocs.org/user-guide/choosing-your-theme/) and a number of [third party themes](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes). As a Python-based framework, you can use `pip` to install [MKDocs plugins](https://www.mkdocs.org/dev-guide/plugins/). You can follow [this getting started guide](https://www.mkdocs.org/getting-started/) for your first MKDocs project.

### Asciidoc frameworks

1. [Asciidoctor](https://asciidoctor.org/) is a Ruby-based text processor for parsing AsciiDoc into a document model and converting it to HTML5, PDF, EPUB3, and other formats. Built-in converters for HTML5, DocBook5, and man pages are available in Asciidoctor. Asciidoctor has an out-of-the-box default stylesheet and built-in integrations for MathJax (display beautiful math in your browser), highlight.js, Rouge, and Pygments (syntax highlighting), as well as Font Awesome (for icons). Although Asciidoctor is written in Ruby, that does not mean you need to know Ruby to use it. Asciidoctor can be executed on a JVM using [AsciidoctorJ](https://docs.asciidoctor.org/asciidoctorj/latest/) or in any JavaScript environment (including the browser) using [Asciidoctor.js](https://docs.asciidoctor.org/asciidoctor.js/latest/).

2. Unlike Asciidoctor, [Antora](https://antora.org/) is a true framework for Asciidoc that can store, retrieve, and aggregate all Asciidoc content from multiple git repositories. Antora’s page referencing system isn’t coupled to filesystem paths or URLs. You are able to cross reference pages across a local machine, a staging environment, and a production environment. To generate a site with Antora, you need the [Antora CLI](https://www.npmjs.com/package/@antora/cli) and [Antora site generator](https://gitlab.com/antora/antora). 

### reStructuredText Framework

If you've noticed the usage of "framework" instead of "frameworks" in the above heading, you can guess that there's only one framework for reStructuredText. [Sphinx](https://www.sphinx-doc.org/en/master/index.html), originally created for Python documentation, takes plain-text files in reStructuredText format and transforms it into HTML, PDF, and any output formats. A few well-known projects that use Sphinx for documentation are [Django](https://docs.djangoproject.com/) and [ReadTheDocs](https://docs.readthedocs.io/en/latest/index.html). 

## Choose the right markup

If you're a developer, learning the syntax and styling of a markup language is relatively easy compared to a programming language. When you learn the syntax of a markup language and can render an HTML page locally, you reach the `hello world` nirvana of documentation. However, the difficulty doesn't necessarily start when you write the first documentation file. I [asked on Twitter](https://twitter.com/DewanAhmed/status/1583465169006448640?s=20&t=kOdJ_vOoZiZ_TJK0vJDK9A) about the choice among Markdown, AsciiDoc, and reStructuredText and this section will summarize the findings.

### Markdown - simple and easy to get started

#### Flavors

Yes, there are [A LOT](https://github.com/commonmark/commonmark-spec/wiki/markdown-flavors) of flavors. Despite all eleventy-zillion flavors, Markdown is loved for developer documentation due to its simplicity. The argument against all the unknown Markdown flavors and incompatibility doesn't stand that much since most projects would choose a common flavor like GFM. 

#### Purpose

For many projects, documentation is about a solid README and some files under the `docs` folder on GitHub. They don't need macros or custom tags in their documentation. Since GitHub renders multiple flavors of Markdown nicely, Markdown is an easy choice for many projects.

#### Syntax

Let's look at a few syntax comparisons to understand how simplicity helps Markdown win many projects. 

Link in reStructuredText:

```
This is `Dewan's blog <https://www.dewanahmed.com>`_.
```

Link in Asciidoc:

```
https://www.dewanahmed.com[Dewan's blog]
```

Link in Markdown:

```
[This is `Dewan's blog](https://www.dewanahmed.com)
```

Let's take a look at header complexity in reStructuredText: 

```
This is a header
~~~~~~~~~~~~~~~~
```

```
======================
This is also a header
======================
```

The list of valid header characters in reStructuredText:

```
! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~
```

Compare the above insanity to Markdown and Asciidoc:

```
# This is a header in Markdown

## This is a sub-header in Markdown
```

```
= This is the Document Header (Level 0) in Asciidoc

== This is the Section Title (Level 1) in Asciidoc
```

Setting aside the annoyance of having to hold down the “=”, "~", or another underline-like character a bunch of times, the big problem with rST headers is that header characters in rST have no single mapping onto the header hierarchy.

#### Extendability

Many tech companies love the rich ecosystem of [Remark](https://github.com/remarkjs) to transform Markdown with plugins. Still, **extendability** is the subject where Markdown scores the lowest. Most folks mention that Markdown is not suitable for serious documentation projects. If your project uses some form of workflow and you'd like to run some scripts within your Markdown, your choice of scripting language might vary from JavaScript (Jekyll or Markdoc) to GoLang (Hugo) to Python (MKDocs). Some teams might be tempted to include custom HTML and CSS within Markdown and then get stuck in a messy situation during migration to another framework. I can see that some frameworks like [Hugo offer localization](https://gohugo.io/content-management/multilingual/) but from the survey responses, Markdown isn't the first choice when folks think of complex documentation projects involving localization. 

### reStructuredText - mostly for Python-based projects that require extendability

#### Python

![Why rST](/assets/images/why-rst.png)

The first line says it all! Almost every single question of "Why rST?" is answered with "because we have a bunch of Python devs...". 

#### The power of directives

reStructuredText offers a number of useful [directives](https://docutils.sourceforge.io/docs/ref/rst/directives.html) out-of-the-box. For example, admonitions ("safety messages" or "hazard statements") can appear in rST like this:

```
.. DANGER::
   Do not review/merge your own PR!
```

You can also use [sidebar](https://docutils.sourceforge.io/docs/ref/rst/directives.html#sidebar), [math](https://docutils.sourceforge.io/docs/ref/rst/directives.html#math), and a [number of other useful directives](https://docutils.sourceforge.io/docs/ref/rst/directives.html#body-elements). 

#### Love/hate Sphinx

[Sphinx](https://www.sphinx-doc.org/en/master/index.html) is incredibly powerful and can offer a table of contents, automatic links for functions, automatic code highlighting using [Pygments](https://pygments.org/), and other capabilities using [built-in](https://www.sphinx-doc.org/en/master/usage/extensions/index.html#builtin-extensions) or [third-party](https://www.sphinx-doc.org/en/master/usage/extensions/index.html#third-party-extensions) extensions. If you'd like to use (a flavor of) Markdown with Sphinx, you can do so using [MyST-parser](https://myst-parser.readthedocs.io/en/latest/) - a Sphinx and Docutils extension to parse [MyST](https://jupyterbook.org/en/stable/reference/cheatsheet.html). 

However, [many folks complained](https://twitter.com/choldgraf/status/1212054861132521472?s=20&t=SiZ3Pr5NnPbID0kNLYcisg) about the difficulty in surfacing and debugging errors that happen in the Sphinx build process. The fact that Sphinx build is significantly slower than other SSG frameworks, the development and writing flow takes a hit in the process.

### Asciidoc - a perfect balance of simplicity and extendability

Reading the above analysis of Markdown and rST, **simplicity** and **extendability** are two criteria for evaluating a markup language. Asciidoc seems to provide a nice middle-ground - simple enough from the syntax angle but advanced enough from the extendability point of view. 

Critics say that AsciiDoc is less familiar to engineers and technical writers. AsciiDoc is said to have a number of syntactic idiosyncrasies,  such as the use of [multiple leading asterisks](https://docs.asciidoctor.org/asciidoc/latest/lists/unordered/#nested-unordered-list) in order to express nested bulleted lists or [varying the length](https://docs.asciidoctor.org/asciidoc/latest/blocks/delimited/#nesting) of the delimiter line to nest delimited content blocks.

The Asciidoc's fans outweigh its critics by a number of magnitudes. You can choose one of three Asciidoctor processors (Ruby, JavaScript, Java/JVM) and get the same experience. From [build automation using Maven tools](https://docs.asciidoctor.org/maven-tools/latest/) to powerful static documentation site generation using [Antora](https://antora.org/), Asciidoc provides a solid ecosystem with the comfort of syntax simplicity and extension richness.

### Other factors when considering a markup

If you're migrating your documentation project from one markup to another, you'll want to know about projects like [Pandoc](https://pandoc.org/). If your build process includes a process to convert markups, Pandoc can be [run with GitHub Actions](https://github.com/pandoc/pandoc-action-example).

If you need to maintain different versions of your documentation based on product releases, you need to carefully plan in advance. Copying file snapshots after every release is a BAD IDEA. This does not scale, and pretty soon you'll exhaust your resources. The ideal way is to use docs-as-code and use git to manage the versions of your documentation.

Do you have parts of the documentation behind a firewall that require authentication to access them? You'll need to set up an identity and access management (IAM) service in front of your static website. Unless IAM is your jam, it's better to avail of a managed service to tackle it.

## Wrap up

In this blog, I covered three popular markup languages, some frameworks, and my opinion on the choice of markup language based on use cases. If there's one thing you want to take away from this blog, it will be to find the balance between the ease of getting started (simplicity) and the support for future growth (extendability) when choosing a markup language. If you liked this blog or have any feedback, please [reach out](https://twitter.com/DewanAhmed).  