+++ 
draft = true
date = 2021-09-14T13:24:07-04:00
title = "Faking the Filesystem in Python"
description = "Using pytest and pyfakefs for unit tests"
slug = ""
authors = ["Tom"]
tags = ["Python","pytest","pyfakefs"]
categories = []
externalLink = ""
series = ["python"]
+++

##### Is stubbing the filesystem worth it?

I've heard some differing opinions on this from developers with expertise in different languages.
The more pedantic-leaning might claim that (_emphasis_ mine)...

{{< url-quotation source="Michael Feathers" url="https://www.artima.com/weblogs/viewpost.jsp?thread=126923">}}
A test is not a unit test if:

* It talks to the database
* It communicates across the network
* _It touches the file system_
* It can't run at the same time as any of your other unit tests
* You have to do special things to your environment (such as editing config files) to run it
{{< /url-quotation >}}

There are a couple of reasons for reserving these external pieces for things like integration and end-to-end tests.

The first is speed.
A slow test is one that is rarely run, and in many cases would not have much value for local development, or even branch builds.
Having a suite of tests that provide near-instant feedback lets me take a "try it and see what breaks" approach so I can focus on the creative aspects of coding.

The other is isolation.
If we incorporate the external pieces, often called _collaborators_, in our unit tests, we are testing more than our project's code.
In contrast to these collaborators, when developing I am concerned with testing _the code under test_, that project's code.
I don't care if the filesystem is working as expected for these tests.

However, while things like a networked database might require test doubles for speed, and eliminate external overhead like a networked database, how much does it really matter about the _filesystem_?
If may be annoying for a developer to run a local database, but what development environment isn't going to have a filesystem?
How many projects use the filesystem that much?

Well it is not necessarily a bad practice to take a more pragmatic approach, as [martinfowler.com](https://martinfowler.com/) notes (again _emphasis_ mine)...

{{< url-quotation source="Ham Vocke, _The Practical Test Pyramid_" url="https://martinfowler.com/articles/practical-test-pyramid.html#SociableAndSolitary">}}
Some argue that all collaborators (e.g. other classes that are called by your class under test) of your subject under test should be substituted with mocks or stubs to come up with perfect isolation and to avoid side-effects and a complicated test setup. Others argue that _only collaborators that are slow or have bigger side effects_ (e.g. classes that access databases or make network calls) should be stubbed or mocked.
{{< /url-quotation >}}

So in many cases, the filesystem is the least of our concerns.
But so far I've haven't addressed a key variable out of the cost side of the cost-benefit ratio.

##### How difficult is it to mock the filesystem?

