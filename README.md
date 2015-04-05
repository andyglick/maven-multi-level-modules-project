Multiple Level Maven Modules
============================

My friend Rob recently asked me if I knew how to execute a single test in a Maven module that was nested down below the 1st level below the parent pom and I
didn't know how to do that. I didn't have any visibility into his project's directory structure and sending emails back and forth wasn't really getting us
anywhere very quickly. Finally I bit the bullet and put together my own little example Maven project.

The parent pom
==============

There is a parent pom in the top level module, whose type is pom

The sub-parent pom
==================

Directly below the parent module there is another module with a parent pom. Let's refer to it as the sub-parent module containing a sub-parent pom, and its
type is also pom

The pom children
================

Layered under the sub-parent module are 2 child modules whose poms each have jar as their type. So if I wanted to access only 1 of the children from the
top-level directory I would have to give the -pl command with the suitable module name and that was the rub, I couldn't figure out how to determine what the
module name actually was for the purposes of getting maven to choose the right module.

It turns out that the GAV applies here. GAV is a Maven technical concept that resolves to GroupId, ArtifactId and Version.

So in the case of executing only the test phase in the maven-multiple-level-modules-first module the syntax would be:

    mvn test -pl org.zrgs.maven:maven-multiple-level-modules-first

To execute only the AppTest test class the syntax would be:

    mvn test -pl org.zrgs.maven:maven-multiple-level-modules-first -Dtest=AppTest

And to execute only the testOfCopyOfTestApp in the TestApp class the syntax would be:

    mvn test -pl org.zrgs.maven:maven-multiple-level-modules-first -Dtest=AppTest#testOfCopyOfTestApp

