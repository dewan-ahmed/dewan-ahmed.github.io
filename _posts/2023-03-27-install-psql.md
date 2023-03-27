---
title: "How to install PostgreSQL client psql on macOS, Linux (Ubuntu), and Windows?"
date: 2023-03-27T00:00:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/2023/psql.jpg"
  alt: "Photo by David Clode on Unsplash"
tags:
  - postgresql
  - tips
---

The installation process for the PostgreSQL client, **psql**, may vary depending on your operating system. Here are the general steps for some popular operating systems:

## Linux (Ubuntu)

1. Open the terminal and run the following command to update the package list:

```
sudo apt-get update
```

2. Install the PostgreSQL client by running the following command:

```
sudo apt-get install postgresql-client
```

## macOS

1. Install Homebrew by running the following command in Terminal:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

2. Once Homebrew is installed, run the following command to install PostgreSQL:

```
brew install postgresql
```

Alternatively, you could also use homebrew to install **libpq** that gives you **psql**, **pg_dump** and a whole bunch of other client utilities. 

Unfortunately, since it provides some of the same utilities as are included in the full PostgreSQL package, brew installs it "keg-only" which means it isn't in the PATH by default. 

## Windows

1. Download the PostgreSQL installer for Windows from [the official website](https://www.postgresql.org/download/windows/).
2. Run the downloaded installer and follow the prompts to install PostgreSQL.

After completing the installation process, you should be able to run **psql** in the terminal/command prompt to connect to your PostgreSQL database.

