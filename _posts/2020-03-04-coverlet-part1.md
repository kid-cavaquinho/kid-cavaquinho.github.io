---
layout:     post
title:      Measuring code coverage in .NET Core (part I)
date:       2020-04-04 10:42:00
summary:    With a little (and great) help of Coverlet
categories: code-coverage testing dotnet-core
image:      '/images/posts/coverlet/pcoverage.png'
---

Measuring the code coverage of your test suite is a key metric of any software engineering project and tell us how much of the source is tested.
While there are several tools to achieve coverage measurements in different languages or frameworks in this post I will focus on the .NET Framework and on a open source, cross platform package named [Coverlet](https://github.com/tonerdo/coverlet "Coverlet source repository").

### Coverage criteria

Several criteria could be used to determine how the source is exercised or not during testing, with Coverlet the code coverage is calculated in: line, branch and method. These are related, but also very distinct.  

- **Line**: How many lines of source code have been tested.
- **Branch**: How many branches of the control structures (eg. conditional statements or loops) have been tested.
- **Method**: How many of the methods defined have been called.

These metrics are calculated in percentage and average values, in my humble opinion none of these metrics is more important then other, we should rely on all of them depending on how critical your source code is.

### Drivers

An important aspect of Coverlet is that it can be used within three different drivers:

- VSTest engine integration
- MSBuild task integration
- .NET global tool

#### VSTest engine integration

Install [coverlet.collector](https://www.nuget.org/packages/coverlet.collector "Coverlet Collector package nuget.org") package in your test projects with the package manager console or in the command line with the following:

```dotnet add package coverlet.collector```

In case you did not notice the latest .NET Core test templates, such as xUnit or MSTest contain this package already instaled by default, run it with the following command:

```dotnet test --collect:"XPlat Code Coverage"```

This will generate a [cobertura](https://cobertura.github.io/cobertura/) file report by default containing coverage results and will be published to a directory ```TestResults``` under your test project. Read further about advanced VSTest configurations [here](https://github.com/tonerdo/coverlet/blob/master/Documentation/VSTestIntegration.md).

#### MSBuild task integration

Install [coverlet.msbuild](https://www.nuget.org/packages/coverlet.msbuild "Coverlet MSBuild package nuget.org") package in your test projects with the package manager console or in the command line with the following:

```dotnet add package coverlet.msbuild```

You can now collect your coverage with the following command and parameter:

```dotnet test /p:CollectCoverage=true```

The results will be published by default to a ```json``` file in the root folder of your test project and also displayed in the command terminal. Read further about advanced MSBuild configurations [here](https://github.com/tonerdo/coverlet/blob/master/Documentation/MSBuildIntegration.md).

#### .NET Global tool

Install [coverlet.console](https://www.nuget.org/packages/coverlet.console/ "Coverlet console package nuget.org") globally and use the following command:

```dotnet tool install --global coverlet.console```

You can now collect your coverage with the following command and parameters:

```coverlet /path/to/test-assembly.dll --target "dotnet" --targetargs "test /path/to/test-project --no-build"```

 Read further about advanced global tool configurations [here](https://github.com/tonerdo/coverlet/blob/master/Documentation/GlobalTool.md).

## What percentage should you aim for

It depends, right? It's important to understant that **good coverage does not mean good tests** and aiming for a 100% coverage might turn to be costly. I do not believe in a code coverage silver bullet; you should always discuss and identity together with your team what are the critical parts of your application, start with making those robust and aim for a goal together. Use the reports wisely and strive for your desired coverage, this might be an interesting and challenging goal to achieve.

## Coverage reports

In addition, reports are generated in several formats, such as ```json```, ```lcov```, ```opencover```, ```teamcity``` or others. You can consume them later on even hook this calculation to your CI/CD pipelines or a static code analysis tool.

## TLDR

Coverlet rocks, thank you shout out to [Toni Solarin-Sodara](https://github.com/tonerdo), [Peter Liljenberg](https://github.com/petli) and [Marco Rossignoli](https://github.com/MarcoRossignoli). Currently there are a few know issues, read [here](https://github.com/tonerdo/coverlet/blob/master/Documentation/KnownIssues.md) and [here](https://github.com/tonerdo/coverlet/issues?page=1&q=is%3Aissue+is%3Aopen), consider reaching out and help contributing to this open source project. I've wrote a sample .NET Core application with an xUnit test project to support this post, you can find it [here](https://github.com/antao/learning-measuring-code-coverage).

In the next posts I will explain how to publish the generated coverage reports on build and make them useful to your CI/CD pipeline.
