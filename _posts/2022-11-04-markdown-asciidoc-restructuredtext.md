---
title: "Markdown, Asciidoc, or reStructuredText - a tale of docs-as-code"
date: 2022-11-04T00:00:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/md-adoc-reST.jpg"
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
2. Extend (optional but almost a requirement): Use a framework to avail features that are not part of the basic markup.
3. Host: Host the markup files on some repository. This could be self-hosted or a managed provider.
4. Automate the build (optional but highly recommended): Add a CI with spelling, prose, link checks, etc.
5. Publish: Deploy your documentation live using platforms like GitHub (Pages), Vercel, or Netlify. 

In the first part of my docs-as-code series, I'll talk about the choice of markup languages, the available frameworks, and do a comparison among Markdown (md), Asciidoc (adoc), and reStructuredText (reST) based on some use cases. I'll skip the history but in case you're interested, my friend and colleague Tibs delivered a talk on the [history of text markup languages](https://www.youtube.com/watch?v=P-7hwjocEpM). My hope is to provide you with a detailed analysis of these choices of markup languages and frameworks so that you can make an informed decision when selecting one for your next developer documentation project. If you don't need a brief intro and would like to skip the appetizer, you can go straight to the entrée - [Choose the right markup](#choose-the-right-markup) section. 

# The choices - md, adoc, reST

Before choosing a markup language, you'll need to choose either a desktop code editor like [VSCode](https://code.visualstudio.com/) or stay in your terminal and use something like [~~Emacs~~VIM](https://www.vim.org/). Most code editors will have plug-ins for popular markup languages which will offer you syntax highlighting, preview, etc. If you're testing or learning a specific syntax, there are many online editors, such as, [StackEdit for Markdown](https://stackedit.io/). 

The main choices for document markup languages are:

- Markdown (md)
- Asciidoc (adoc)
- reStructuredText (reST)

## Markdown (md)

The [original version](https://daringfireball.net/projects/markdown/) of Markdown by John Gruber doesn't specify the syntax unambiguously. For example, it doesn't have table formatting. Due to the popularity of Markdown and the limitations of Gruber Markdown, a plethora of variations, or "flavors", of Markdown have been introduced. Due to this lack of standard, folks like [Eric Holscher](https://twitter.com/ericholscher), who are passionate about the community, have been vocal about [not using markdown for documentation](https://ericholscher.com/blog/2016/mar/15/dont-use-markdown-for-technical-docs/). 

In 2014, a group of Markdown fans came together and established [CommonMark](https://commonmark.org/) - a standard, interoperable and testable version of Markdown. In March 2017, GitHub published a formal spec for GitHub-Flavored Markdown (GFM); based on CommonMark. In the [accompanying blog post](https://github.blog/2017-03-14-a-formal-spec-for-github-markdown/) when releasing GFM, GitHub addressed many of the limitations that Eric Holscher raised, things like how many spaces are needed to indent a line, or how many empty lines you need to break between different elements. GFM is, by far, the most popular flavor of Markdown.

If you're just starting out with Markdown, check out the [syntax](https://www.markdownguide.org/basic-syntax/) and write some hello-world docs [online](https://markdownlivepreview.com/). 

For Markdown, there are [A LOT](https://github.com/commonmark/commonmark-spec/wiki/markdown-flavors) of flavors. Despite all eleventy-zillion flavors, Markdown is loved for developer documentation due to its simplicity.

## Asciidoc (adoc)

You started with Markdown, but then you needed to embed a YouTube video. So you embed HTML within your Markdown doc. As your project grows, there will be  more HTML and some JavaScript embedded in your Markdown files. With out-of-the-box features like tables, admonitions (tips, notes, warnings, etc.), and table of contents, Asciidoc is a popular choice for documentation projects that want to avoid gluing HTML and JavaScript for advanced use. Asciidoc has no flavors so there's really just a single standard. You might have heard about GitHub Flavored Asciidoc (GFA). This is not a flavor per se, rather GitHub using Asciidoctor in safe mode to render files with the extension `.adoc`, `.ad`, and `.asciidoc`. More on Asciidoctor under the [framework section](#framework-framework-framework). [asciidocLIVE](https://asciidoclive.com/) is a great place to start if you're testing waters with Asciidoc and would like to learn the syntax.

## reStructuredText (reST)

If you're in the Python community, you might be using reStructuredText for documentation. A popular site that uses this markup language is [WriteTheDocs](https://www.writethedocs.org/). The reStructuredText parser is a component of [Docutils](https://docutils.sourceforge.io/index.html) and  is the default markup language used by [Sphinx](https://www.sphinx-doc.org/en/master/index.html). reStructuredText can be extended using [custom directives](http://docutils.sourceforge.net/docs/ref/reST/directives.html) that can satisfy a wide variety of documentation needs. [Here](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html) is a quick primer on reStructuredText if you're getting started. You can play with the reStructuredText syntax using [this online editor](https://livesphinx.herokuapp.com/).  

reStructuredText offers a number of useful [directives](https://docutils.sourceforge.io/docs/ref/rst/directives.html) out-of-the-box. For example, admonitions ("safety messages" or "hazard statements") can appear in reST like this:

```
.. DANGER::
   Do not review/merge your own PR!
```

You can also use [sidebar](https://docutils.sourceforge.io/docs/ref/rst/directives.html#sidebar), [math](https://docutils.sourceforge.io/docs/ref/rst/directives.html#math), and a [number of other useful directives](https://docutils.sourceforge.io/docs/ref/rst/directives.html#body-elements). 

# Framework Framework Framework

For a serious documentation project, you're almost certainly using a framework for specific features and to extend the limit of the vanilla markup. In this section, let's check out some of the popular frameworks for Markdown, Asciidoc, and reStructuredText.

## Markdown frameworks

Considering that the end result of a documentation project is often a static site, the words **static site generator (SSG) tool** and Markdown framework can be used interchangeably. Here are my top five picks:

1. [Jekyll](https://github.com/jekyll/jekyll), the engine behind GitHub Pages, was the most popular SSG until Hugo came out. Written in Ruby, Jekyll is a great choice for writing blog sites. Jekyll can take your content written in Markdown and Liquid templates and render them to a static website deployed to a server of your choice. The site you're currently on is [built using Jekyll and GitHub Pages](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll). 

2. [Hugo](https://gohugo.io/) is probably the most popular open-source static site generator. Written in Go, Hugo builds sites at a blistering speed. It has a native support for a variety of content types, menus, and dynamic API-driven content. These don't require you to use an additional plugin. You can select a theme that suits your project from [a wide selection](https://gohugo.io/showcase/). Check out [some of the project websites](https://gohugo.io/showcase) that are built with Hugo.

3. Released in 2022, [Markdoc](https://markdoc.dev/) is a relatively new Markdown-based authoring framework. The Markdoc project is [open-source](https://github.com/markdoc/markdoc) and it powers [Stripe's documentation](https://stripe.com/docs). Their website has a [live edit](https://markdoc.dev/) button which makes the website a playground for you to give Markdoc a try. Documentations created with Markdoc will automatically render with your [React](https://markdoc.dev/docs/examples/react) app and using `@markdoc/next.js` for your [Next.js](https://markdoc.dev/docs/nextjs) app.  

4. Configured with a single YAML file, [MKDocs](https://www.mkdocs.org/) is the third Markdown framework on my list, which falls in the category of SSG.  Although there are not as many as in Hugo, MKDocs offers a [few official themes](https://www.mkdocs.org/user-guide/choosing-your-theme/) and a number of [third party themes](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes). As a Python-based framework, you can use `pip` to install [MKDocs plugins](https://www.mkdocs.org/dev-guide/plugins/). You can follow [this getting started guide](https://www.mkdocs.org/getting-started/) for your first MKDocs project.

5. Last, but certainly not the least, of my favorite framework(s) is the family of frameworks based on [MDX](https://mdxjs.com/). Before that, let's understand what is MDX and how does it vary from MD. Quoting from [MDXJS](https://mdxjs.com/):

> [JSX](https://reactjs.org/docs/introducing-jsx.html) is an extension to JavaScript that looks like HTML but makes it convenient to use components (reusable things). MDX allows you to use JSX in your markdown content. You can import components, such as interactive charts or alerts, and embed them within your content.

With over 40K GitHub starts, [Docusaurus](https://docusaurus.io/) is undeniably one of the most popular modern documentation frameworks based on MDX. Docusaurus orignates from Meta Open Source (f.k.a Facebook) and is packed with content-centric features (docs, blog, pages, versioning, i18n, a11y, SEO...). As a **React + Node.js static site generator**, you can either use MD to get started easily or use MDX to make your docs look interactive. [This showcase](https://docusaurus.io/showcase) of documentation sites using Docusaurus is enough to convince me that the UX and DX of a documentation site is as important as the content itself.  

From the creators of [NextJS](https://nextjs.org/), the next SSG based on MDX is [Nextra](https://nextra.site/). Nextra offers advanced syntax highlighting, ease of i18n creation, out-of-the-box full text search, and Markdown link and image converted to [Next.js Link](https://nextjs.org/docs/routing/introduction#linking-between-pages) and [Next.js Image](https://nextjs.org/docs/basic-features/image-optimization#local-images). With Nextra, you can write a blog or docs using the themes available. [Here](https://nextra.site/showcase) are the sites that are built using Nextra.

Besides these five frameworks, [Notaku](https://notaku.so/) gets an honorable mention that uses [Notion](https://notaku.so/) as CMS to create documentation site. With SEO and site performance optimization out-of-the-box, Notaku can be a great choice for teams those are already using Notion.

## Asciidoc frameworks

1. An implementation of the docs-as-code approach, [docToolchain](https://github.com/docToolchain/docToolchain) is a collection of scripts that makes it easy to create and maintain powerful technical documentation. It is a popular open-source project that uses [jBake](https://jbake.org/) under the hood as the SSG. docToolchain can [publish  to Confluence](https://doctoolchain.org/docToolchain/v2.0.x/015_tasks/03_task_publishToConfluence.html), [generate PDF](https://doctoolchain.org/docToolchain/v2.0.x/015_tasks/03_task_generatePDF.html) using an Asciidoctor plugin, and [more](https://doctoolchain.org/docToolchain/v2.0.x/015_tasks/03_tasks.html). 

2. [Asciidoctor](https://asciidoctor.org/) is a Ruby-based text processor for parsing AsciiDoc into a document model and converting it to HTML5, PDF, EPUB3, and other formats. Built-in converters for HTML5, DocBook5, and man pages are available in Asciidoctor. Asciidoctor has an out-of-the-box default stylesheet and built-in integrations for MathJax (display beautiful math in your browser), highlight.js, Rouge, and Pygments (syntax highlighting), as well as Font Awesome (for icons). Although Asciidoctor is written in Ruby, that does not mean you need to know Ruby to use it. Asciidoctor can be executed on a JVM using [AsciidoctorJ](https://docs.asciidoctor.org/asciidoctorj/latest/) or in any JavaScript environment (including the browser) using [Asciidoctor.js](https://docs.asciidoctor.org/asciidoctor.js/latest/). You can choose any one of three Asciidoctor processors (Ruby, JavaScript, Java/JVM) and get the same experience. You can also use the [Asciidoctor Maven Plugin](https://docs.asciidoctor.org/maven-tools/latest/) to convert your Asciidoc documentation using Asciidoctor from an Apache Maven build.

3. Unlike docToolchain or Asciidoctor, [Antora](https://antora.org/) is a true framework for Asciidoc that can store, retrieve, and aggregate all Asciidoc content from multiple git repositories. Antora’s page referencing system isn’t coupled to filesystem paths or URLs. You are able to cross reference pages across a local machine, a staging environment, and a production environment. To generate a site with Antora, you need the [Antora CLI](https://www.npmjs.com/package/@antora/cli) and [Antora site generator](https://gitlab.com/antora/antora). 

## reStructuredText Framework

If you've noticed the usage of "framework" instead of "frameworks" in the above heading, you can guess that there's only one framework for reStructuredText. [Sphinx](https://www.sphinx-doc.org/en/master/index.html), originally created for Python documentation, takes plain-text files in reStructuredText format and transforms it into HTML, PDF, and any output formats. A few well-known projects that use Sphinx for documentation are [Django](https://docs.djangoproject.com/) and [ReadTheDocs](https://docs.readthedocs.io/en/latest/index.html). 

[Sphinx](https://www.sphinx-doc.org/en/master/index.html) is incredibly powerful and can offer a table of contents, automatic links for functions, automatic code highlighting using [Pygments](https://pygments.org/), and other capabilities using [built-in](https://www.sphinx-doc.org/en/master/usage/extensions/index.html#builtin-extensions) or [third-party](https://www.sphinx-doc.org/en/master/usage/extensions/index.html#third-party-extensions) extensions. If you'd like to use (a flavor of) Markdown with Sphinx, you can do so using [MyST-parser](https://myst-parser.readthedocs.io/en/latest/) - a Sphinx and Docutils extension to parse [MyST](https://jupyterbook.org/en/stable/reference/cheatsheet.html). 

However, [many folks complained](https://twitter.com/choldgraf/status/1212054861132521472?s=20&t=SiZ3Pr5NnPbID0kNLYcisg) about the difficulty in surfacing and debugging errors that happen in the Sphinx build process. The fact that Sphinx build is significantly slower than other SSG frameworks, the development and writing flow takes a hit in the process.

# Choose the right markup and framework

If you're a developer, learning the syntax and styling of a markup language is relatively easy compared to a programming language. When you learn the syntax of a markup language and can render an HTML page locally, you reach the `hello world` nirvana of documentation. However, the difficulty doesn't necessarily start when you write the first documentation file. As you try more advanced tasks for your developer documentation project, you lean more on the ~~framework~~ community behind the framework.

I [asked on Twitter](https://twitter.com/DewanAhmed/status/1583465169006448640?s=20&t=kOdJ_vOoZiZ_TJK0vJDK9A) about the choice among Markdown, AsciiDoc, and reStructuredText and the first version of this blog summarized the findings. When I  published this blog back in November 2022, it got quite a few attentions and was trending on the [first page of HackerNews](https://news.ycombinator.com/item?id=33468213). Besides the satisfaction from the blog's success, I felt terrible for missing out tools like Jekyll, docToolchain, and Docusaurus. Thanks to [Dan Moore](https://twitter.com/mooreds), [Ralf D. Müller](https://twitter.com/RalfDMueller), [Sebastien Lorber](https://twitter.com/sebastienlorber), and the HackerNews commenters, I've updated this blog to include the missing tools.

## Scripting language preference

![Why reST](/assets/images/why-reST.png)

The first line says it all! Almost every single question of "Why reST?" is answered with "because we have a bunch of Python devs...". 

Next is Markdown. If your project uses some form of workflow and you'd like to run some scripts within your Markdown, your choice of scripting language might vary from JavaScript (Jekyll, Markdoc, or Docusaurus) to GoLang (Hugo) to Python (MKDocs).

Unlike reStructuredText or Markdown frameworks, Asciidoc's Antora framework provides extensive feature set out-of-the-box so that you don't need to add scripting languages like Python, Ruby, or JavaScript within your documentation. Full disclaimer that my take is based on Antora's website. An advanced user of the framework might be able to confirm this claim. 

## Syntax

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

Setting aside the annoyance of having to hold down the “=”, "~", or another underline-like character a bunch of times, the big problem with reST headers is that header characters in reST have no single mapping onto the header hierarchy.

Similar to reST, critics say that AsciiDoc is less familiar to engineers and technical writers. AsciiDoc is said to have a number of syntactic idiosyncrasies,  such as the use of [multiple leading asterisks](https://docs.asciidoctor.org/asciidoc/latest/lists/unordered/#nested-unordered-list) in order to express nested bulleted lists or [varying the length](https://docs.asciidoctor.org/asciidoc/latest/blocks/delimited/#nesting) of the delimiter line to nest delimited content blocks.

## Extendability

Many tech companies love the rich ecosystem of [Remark](https://github.com/remarkjs) to transform Markdown with plugins. Still, **extendability** is the subject where Markdown historically scored the lowest. Most folks mention that Markdown is not suitable for serious documentation projects.  Some teams might be tempted to include custom HTML and CSS within Markdown and then get stuck in a messy situation during migration to another framework. 

Let's think for a moment here 🤔 ... 

How often do you migrate your docs? Migrating any mature documentation project that uses some sort of framework is never easy. If you choose the wrong framework or overload your project with custom HTML/CSS/JS without taking the advantage of a framework's built-in feature, Markdown is not to blame here. 

If you're migrating your documentation project from one markup to another, you'll want to know about projects like [Pandoc](https://pandoc.org/). If your build process includes a process to convert markups, Pandoc can be [run with GitHub Actions](https://github.com/pandoc/pandoc-action-example). If you're planning on [converting Asciidoc files to Markdown](https://codeahoy.com/q/q502/How-to-convert-asciidoc-to-Markdown), you'll have to take a detour via the XML route. The proces will be: `Asciidoc ---> XML ---> Markdown`. 

[MyST-parser](https://myst-parser.readthedocs.io/en/latest/), which is both completely compatible with CommonMark, and also supports all ReStructured Text directives, can be used to convert reST to Markdown. 

## Internationalization (i18n)

Internationalization (i18n) pain for a documentation project is a process problem, not a feature gap. Documentation framework are not meant to translate your developer docs for you in the language of your choice. Some frameworks might offer i18n **support**, like the [Crowdin support in Docusauraus v2](https://docusaurus.io/docs/i18n/crowdin). With Jekyll, you have to pick a theme [like this one](https://github.com/kurtsson/jekyll-multiple-languages-plugin). I doubt if reST or adoc frameworks would differ much from this process. Besides the framework support, you'll need a localization vendor like [lokalise](https://lokalise.com/) or a community driven platform like [Crowdin](https://crowdin.com/) to actually **do** the translation. Sébastien goes into great details in this [Docusaurus RFC](https://github.com/facebook/docusaurus/issues/3317) when discussing i18n support in Docusaurus v2.  

## Other factors when considering a markup

If you need to maintain different versions of your documentation based on product releases, you need to carefully plan in advance. Copying file snapshots after every release is a BAD IDEA. This does not scale, and pretty soon you'll exhaust your resources. The ideal way is to use docs-as-code and use git to manage the versions of your documentation.

Do you have parts of the documentation behind a firewall that require authentication to access them? You'll need to set up an identity and access management (IAM) service in front of your static website. Unless IAM is your jam, it's better to avail of a managed service to tackle it.

## Authoring Experience ⚖️ User Experience

Let's look at some documentation sites with great UX:

| Jekyll  | Docusaurus  | Antora  | Sphinx  |
|---|---|---|---|
| [Sketch](https://sketch.com/)  | [Algolia](https://docsearch.algolia.com/)  | [Apache Solr](https://solr.apache.org/guide/solr/latest/)  | [ReadTheDocs](https://readthedocs.org/)  |
| [Spotify](https://developer.spotify.com/)  | [Hasura](https://hasura.io/docs/)  | [Apache Cassandra](https://cassandra.apache.org/_/development/documentation.html)  | [WriteTheDocs](https://github.com/writethedocs/www)  |
| [Twitch](https://dev.twitch.tv/)  | [Supabase](https://www.supabase.io/docs)  | [Cloudbees](https://docs.cloudbees.com/)  | [Aiven](https://docs.aiven.io/)  |

Humble brag that the last one in the list ([Aiven](https://docs.aiven.io/) 🦀) won [2022 DevPortal Awards](https://devportalawards.org/nominees/2022/aiven-developer) in the category **Best DevPortal Beyond REST Platforms**. If I added a list of great docs websites but didn't include the OG, it would be a crime. Check out one of THE BEST documentation out there - [Stripe Docs](https://stripe.com/docs), built using [Markdoc](https://markdoc.dev/). 

What makes a great user experience for a documentation site? The factors affecting your users are:

1. General look and feel of your website
2. Proper use of interactive components (although this is completely optional)
3. Ease of finding information (including the correctness of the information)
4. Features like **copy code to clipboard**, syntax highlighting on code, etc. 
5. How less annoying your site is with things like pop-ups, tracking, etc.

I like that you care deeply about your users. How about the folks who work really hard to build these docs? If the emoji is loading properly, you'll see that I put a scale icon between authoring experience and user experience on the section heading. Here are the factors affecting your authors:

1. Ease of creating their first PR (you are using docs-as-code approach, right? RIGHT?)
2. Maturity of the content architecture (Can I easily guess which sub-folder my new file will go under?)
3. Level of scripting knowledge required to contribute (do I need to be a react dev to contribute to the docs?!)
4. Automation within the content review/publication process to catch linting, spelling and other errors (this is not related to the markup/framework)
5. Frequency of large migration projects (ideally "zero")

Points 2, 4 and 5 don't relate to the choice of the markup language or framework. Depending on points 1 and 3, your docs contribution will include almost anyone from your company or only the DevRel team and technical writers. 

Writing documentation is different from writing code.

**Writing documentation is different from writing code.**

Yes, I needed to repeat that line. As a user, I love the visually appealing documentation sites built using an MDX-based framework. While MDX affords users more power and flexibility, it comes at the cost of complexity. Content can quickly become as complex as regular code, which can lead to maintainability complications or a more difficult authoring environment. You can choose an MDX-based framework for other features and stay away from overscripting. But the temptation (to add more scripts) is harder to resist than one might think. 

With the above discussion, can we form a middle ground that balances both user experience and authoring experience? While all 10 factors are critical, some of the factors I mentioned above don't relate to your choice of markup and framework (for example, content architecture or if you add too many pop-ups). I'll highlight five questions you can ask when selecting a markup and framework.

Question 1: Do you have a strong reason for NOT using Markdown? 

Question 2: How much do you care about the appearance of your docs site? Do you prefer simple and clean docs or beautiful and interactive ones?

Question 3: What are the features that matter to your users and authors? Search? Internationalization? Versioning? Pick the framework that delivers (most of) these features.

Question 4: Do you have a special need like building docs from multiple git repositories or having authentication on certain parts of the documentation? 

Question 5: Is the framework open-source? 

# Wrap up

In this blog, I covered three popular markup languages, some very popular frameworks, and my opinion on the choice of markup language and framework based on a few key factors. If there's one thing you want to take away from this blog, it will be to find the balance between the ease of getting started (authoring experience) and the ease of finding information (user experience). If you liked this blog or have any feedback, please [reach out](https://twitter.com/DewanAhmed).  