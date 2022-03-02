---
title: "10 steps to a better Dockerfile"
date: 2021-11-10T22:50:00-04:00
tags:
  - devops
  - containers
---

The journey to the cloud typically starts with containerizing your apps. One of the first challenges developers face is writing the blueprint for those container images—aka a Dockerfile. This article guides you through ten steps to writing better Dockerfiles. The basis for our example is a  [popular Spring application](https://developers.redhat.com/courses/spring-boot).

![containers](../assets/images/containers.jpg)

## Pre-requisites

* [Java 11 or newer](https://openjdk.java.net/install/)
* [Maven](https://maven.apache.org/install.html)
* [Docker](https://docs.docker.com/engine/install/)

I'm also assuming that you have some understanding of Java, Docker, and Dockerfile (i.e. given a Dockerfile, you are able to build the image and run a container using that image).

At first, clone [this repository](https://github.com/spring-projects/spring-petclinic) by executing the following command:

```
git clone https://github.com/spring-projects/spring-petclinic.git
```

The following steps assume that you have a pre-compiled binary.

### Step 0

Contrary to popular belief, image size is not your first concern when writing a Dockerfile. If you're used to **apt-get**, you don't need to switch to a different package manager to convert your 700MB Ubuntu image to a 10MB Alpine image. The first step is to just get the Dockerfile work; preferably using the same build commands that you're used to. 

Add the following Dockerfile at the root of the project.

```
FROM debian
RUN apt-get update


COPY . /app

# Install OpenJDK-11
RUN apt-get update && \
    apt-get install -y openjdk-11-jdk && \
    apt-get install -y ant && \
    apt-get clean;

# Fix certificate issues
RUN apt-get update && \
    apt-get install ca-certificates-java && \
    apt-get clean && \
    update-ca-certificates -f;

# Setup JAVA_HOME -- useful for docker commandline
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64/
RUN export JAVA_HOME


ENTRYPOINT ["java","-jar","/app/target/app.jar"]
```

This Dockerfile works, but it could be improved upon. Just like a 100-year-old house, we must do some renovations with this Dockerfile. 

### Step 1

In the first improvement step, we'll tackle caching. "What is caching?" you might ask. 

Each instruction in the Dockerfile results in a new image layer being created and added to your local image cache. That image then becomes the "read-only" parent for the image created by the next instruction. Layers stack on top of each other, adding functionality incrementally. Once Docker caches an image layer for an instruction it doesn't need to be rebuilt. Caching reuses existing image layers and helps save expensive network calls. To read more about caching, visit [this blog](https://www.docker.com/blog/intro-guide-to-dockerfile-best-practices/).


Two things to keep in mind in regards to caching:

* Order your steps in the Dockerfile from least to most frequently changing steps to optimize your caching
* When copying files into your images, be very specific in what you want to copy. Because any change in the files you're copying will break the cache 

```
# Note: Order matters for caching
FROM debian
RUN apt-get update

## Improvement 1A. Move copy command from here...

## Improvement 1B. Have cacheable units cached together. The apt-get update and install should happen together or not at all

# Install OpenJDK-8
RUN apt-get update && \
    apt-get install -y openjdk-11-jdk && \
    apt-get install -y ant && \
    apt-get clean;

# Fix certificate issues
RUN apt-get update && \
    apt-get install ca-certificates-java && \
    apt-get clean && \
    update-ca-certificates -f;

# Setup JAVA_HOME -- useful for docker commandline
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64/
RUN export JAVA_HOME

## Improvement 1A. ...to here
## This is because we're building from a pre-compiled application and don't need to copy the source files

COPY target/spring-petclinic-2.4.5.jar /app

ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]

```

### Step 2

In this step of the improvement, try to eliminate dependencies that you don't need in your image. You can always add those later if you need them.

```

FROM debian
RUN apt-get update

## Improvement 2: Remove unnecessary dependencies using --no-install-recommends flag

# Install OpenJDK-11
RUN apt-get update && \
    apt-get install -y --no-install-recommends openjdk-11-jdk && \
    apt-get install -y ant && \
    apt-get clean;

# Fix certificate issues
RUN apt-get update && \
    apt-get install ca-certificates-java && \
    apt-get clean && \
    update-ca-certificates -f;

# Setup JAVA_HOME -- useful for docker commandline
ENV JAVA_HOME /usr/lib/jvm/java-11-openjdk-amd64/
RUN export JAVA_HOME

COPY target/spring-petclinic-2.4.5.jar /app

ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]

```

### Step 3

These are possibly THE MOST IMPORTANT improvements (for security and maintainability reasons):

* Use official images whenever possible (vetted for vulnerabilities)
* DO NOT, under any circumstances, use the **latest** tag. Just DON'T (how to distinguish between old "latest" and new "latest"?)

If you’re tagging an image with “latest”, that’s all the information you have apart from the image ID. Operations like deploying a new version of your app, or rolling back are simply not possible if you don’t have two distinctly tagged images that are stable.

```
# Use official image and specific image tags 

## Improvement 3: Using an official image that already has Java and using a specific version
FROM openjdk:11
RUN apt-get update

COPY target/spring-petclinic-2.4.5.jar /app

ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]
```

Suddenly the Dockerfile looks less ugly. Don't get too happy as it'll get complicated again ;)

### Step 4

Look for minimal flavors that will do the job. 

> :warning: Word of caution: It's a balance between convenience and image size

That means your favorite _alpine_ image will no longer let you **apt-get**.

```
## Improvement 4: Look for minimal flavors
FROM openjdk:17-alpine

## Warning: You're not going to get apt-get anymore
# RUN apt-get update

COPY target/spring-petclinic-2.4.5.jar /app

ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]
```

## Step 5

In this step, we focus on building from source in a consistent environment. So far we have been using a pre-built binary and only copying that .jar file. However, if the Dockerfile is indeed the blueprint, the _source of truth_ should be the source code; not the build artifact. 

```
## Improvement 5A: Build from source in a consistent environment
FROM maven:3.8.1-ibmjava-8-alpine

WORKDIR /app

COPY pom.xml .

COPY src ./src

RUN mvn -e -B package

ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]

```

But now there's a problem! Every time you make a code change, the dependencies will be fetched. The solution is to make the dependency resolving a separate step.

```
## Improvement 5A: Build from source in a consistent environment
FROM maven:3.8.1-ibmjava-8-alpine

WORKDIR /app

COPY pom.xml .

## Improvement 5B: Separating dependency from build step for caching
RUN mvn -e -B dependency:resolve

COPY src ./src

RUN mvn -e -B package

ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]

```

## Step 6

So far you've made great progress. But... the image again has become a lot bigger. You also have added your development and build tools in the final image. _Multi-stage Builds_ to the rescue. Learn more about multi-stage builds [here](https://docs.docker.com/develop/develop-images/multistage-build/).

```
## Improvement 6: Multistage build
FROM maven:3.8.1-ibmjava-8-alpine AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
RUN mvn -e -B package

FROM openjdk:17-alpine AS release
COPY --from=builder /app/target/spring-petclinic-2.4.5.jar /
ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]
```

Not only do you have separation of concerns using different stages; only the **release** stage is what you'll be pushing to the registry and the files from the previous stages will not be shared. You can build just the **release** stage using the following command:

```
docker build -t <tag-name> . --target release
```

## Step 7

This next improvement is more of a best practice. **ARG** is the only instruction that may precede **FROM** in the Dockerfile. They are only available from the moment they are *announced* in the Dockerfile with an ARG instruction up to the moment when the image is built. Running containers can’t access values of ARG variables. Defining an ARG variable before the first **FROM** makes it a global variable that can be referenced from all stages.

```
## Improvement 7: Using global ARG

ARG flavor=alpine

FROM maven:3.8.1-ibmjava-8-$flavor AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
RUN mvn -e -B package

FROM openjdk:17-$flavor AS release
COPY --from=builder /app/target/spring-petclinic-2.4.5.jar /
ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]

```

## Step 8

**-DskipTests** is a way overused flag and this final improvement tip is for those. You can run the tests as separate stages and your build stage can skip running the tests.

```
ARG flavor=alpine

FROM maven:3.8.1-ibmjava-8-$flavor AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
## Improvement 8: Run tests as separate stages. You can skip the tests here
RUN mvn -e -B package -DskipTests

FROM builder AS unit-test
RUN mvn -e -B test

FROM openjdk:17-$flavor AS release
COPY --from=builder /app/target/spring-petclinic-2.4.5.jar /
ENTRYPOINT ["java","-jar","/app/spring-petclinic-2.4.5.jar"]

FROM release AS integration-test
RUN apk add --no-cache curl
# If you have an actual integration test script, refer to that below. You'll also need to copy that script from the build stage
# RUN ./test/int-test.sh

```

## Step 9

Caching is not always ideal. For example, when cloning a Git repository, the `git clone` command will possibly never change, but the repo will. The simplest solution to avoid these issues is to just not use the cache at all for such scenarios. 

[On that note, [here](https://forums.docker.com/t/best-practices-for-getting-code-into-a-container-git-clone-vs-copy-vs-data-container/4077) is a nice discussion on the best practices for getting code into a container (git clone vs. copy vs. data container).]

The no-cache argument will completely discard the cache, always executing all steps of the Dockerfile. For security patches/updates and use-cases like the one above, it's helpful to periodically use the --no-cache argument.

The **FROM** instruction is the only line that is not affected by the no-cache argument. If the base image is present in the machine, it won’t be pulled again.

---

The final version of the Dockerfile looks nowhere as slim (in terms of the number of lines; not image size) as what we had in Step 3. However, we've made lots of improvements in terms of **reproducibility**, **maintainability**, and **consistency**. If you've enjoyed this blog or have any feedback, please [give me a shout-out](https://www.linkedin.com/in/diahmed/).