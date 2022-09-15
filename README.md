[![Build Status](https://travis-ci.org/andyglick/maven-multi-level-modules-project.png)](https://travis-ci.org/andyglick/maven-multi-level-modules-project)
# Invoking builds into Multiple Level Maven Module Structures

My friend Rob recently asked me if I knew how to execute a single test in a Maven module that was nested 2 or more levels below the parent pom and I
didn't know how to do that, and none of the suggestions that I made worked for him in his environment. I didn't have any visibility into his project's
directory structure and sending emails back and forth wasn't really getting us anywhere very quickly. Finally I bit the bullet and put together my own little
example Maven project. In hopes that others might learn something useful from it, I published it.

This is the directory structure of the project

    maven-multiple-level-modules-parent
      maven-multiple-level-modules-sub-parent
        maven-multiple-level-modules-first
        maven-multiple-level-modules-second

There is a pom in each module directory. In the 3 modules below the root module, each references the one above it as it's parent. The 2 parent modules each
reference their respective children in its own *modules* section.

## The parent pom

There is a parent pom in the top level module, whose type is pom

## The sub-parent pom

Directly below the parent module there is another module with a parent pom. Let's refer to it as the sub-parent module containing a sub-parent pom, and its
type is also pom

## The pom children

Layered under the sub-parent module are 2 child modules whose poms each have jar as their type.

## The Problem

So if the goal was to execute the test phase of only 1 of the pom children, and beyond that to fire only 1 test class from that module, or to be even more
discriminating, to execute only 1 test from the chosen test class, then at a minimum the *-pl* option would have to be used to name the module. But what I
couldn't figure out was how to specify the name of the module correctly so that the Maven engine would recognize it and run the surefire plugin on it. In the
end, and what I often find to be the case when munging around with Maven internals, *-X* is your friend. What I saw in the Maven run log was:

    [INFO] Scanning for projects...
    [INFO] ------------------------------------------------------------------------
    [INFO] Reactor Build Order:
    [INFO]
    [INFO] maven-multiple-level-modules-parent
    [INFO] maven-multiple-level-modules-sub-parent
    [INFO] maven-multiple-level-modules-first
    [INFO] maven-multiple-level-modules-second
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building maven-multiple-level-modules-parent 1.
    [INFO] ------------------------------------------------------------------------
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building maven-multiple-level-modules-sub-parent 1.0
    [INFO] ------------------------------------------------------------------------
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building maven-multiple-level-modules-first 1.0
    [INFO] ------------------------------------------------------------------------
    [DEBUG] === REACTOR BUILD PLAN ================================================
    [DEBUG] Project: org.zrgs.maven:maven-multiple-level-modules-parent:pom:1.0
    [DEBUG] Tasks:   [test]
    [DEBUG] Style:   Regular
    [DEBUG] -----------------------------------------------------------------------
    [DEBUG] Project: org.zrgs.maven:maven-multiple-level-modules-sub-parent:pom:1.0
    [DEBUG] Tasks:   [test]
    [DEBUG] Style:   Regular
    [DEBUG] -----------------------------------------------------------------------
    [DEBUG] Project: org.zrgs.maven:maven-multiple-level-modules-first:jar:1.0
    [DEBUG] Tasks:   [test]
    [DEBUG] Style:   Regular
    [DEBUG] -----------------------------------------------------------------------
    [DEBUG] Project: org.zrgs.maven:maven-multiple-level-modules-second:jar:1.0
    [DEBUG] Tasks:   [test]
    [DEBUG] Style:   Regular
    [DEBUG] =======================================================================

So it turns out that the GAV for each module applies here. GAV is a Maven technical concept that resolves to GroupId, ArtifactId and Version. And what you
can see on each of the "[DEBUG] Project:" lines is in fact the GAV of the module to be executed.

Why this important piece of information is hidden in the DEBUG output and isn't included in the default INFO text is beyond me. It is another "interesting"
design decision on the part of the Maven developers.

## The Solution

It may be worth saying that when I looked at this evening I no longer remembered what the Maven -pl option 
indicated, it turns out that -pl is an abreviation for --projects

So in the case of executing the test phase only in the maven-multiple-level-modules-first module the syntax turned 
out to be:

    mvn test -pl org.zrgs.maven:maven-multiple-level-modules-first

To execute only the AppTest test class during that test phase the syntax is:

    mvn test -pl org.zrgs.maven:maven-multiple-level-modules-first -Dtest=AppTest

And to execute only the testOfCopyOfTestApp in the TestApp class the syntax is:

    mvn test -pl org.zrgs.maven:maven-multiple-level-modules-first -Dtest=AppTest#testOfCopyOfTestApp

For completists, I was able to craft the syntax to explicitly execute multiple tests from a single test class 
using the "+" as the between test delimiter. Surefire is now capable of handling that, it no longer throws exceptions

The pom file uses maven-surefire-plugin:3.0.0-M7 and junit:4.13.2 

    mvn test -pl org.zrgs.maven:maven-multiple-level-modules-first -Dtest=AppTest#testOfCopyOfTestApp+testApp

I removed all of the references to the late and not especially lamented versioneye service from any and all 
pom files
