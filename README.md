Multiple Level Maven Modules
============================

My friend Rob recently asked me if I knew how to execute a single test in a Maven module that was nested down below the 1st level below the parent pom and I
didn't know how to do that. I didn't have any visibility into his projects directory structure and sending emails back and forth wasn't reallu getting us
anywhere very quickly. Finally I bit the bullet and put together my own little example program.

The parent pom
==============

There is a parent pom, whose type is pom

The sub-parent pom
==================

Directly below the parent pom there is another parent pom. Lets refer to it as the sub-parent pom, and its type is pom

The pom children
================

Layered under the sub-parent module directory are 2 child poms whose type is jar. So if I wanted to access only 1 of the children from the top-level directory
I would have to give the -pl command with the suitable module name and that was the rub, I couldn't figure out how to determine what the module name actually
was for the purposes of getting maven to choose the right module.