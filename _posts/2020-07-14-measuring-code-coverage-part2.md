---
layout:     post
title:      Measuring code coverage in .NET Core (part II)
date:       2020-07-14 20:42:00
summary:    With a little (and great) help of Coverlet, reportgenerator and github actions
categories: code-coverage testing dotnet-core
---

Generate code coverage reports and be able to comfortably and easily understand them is important for:
* Evaluate the quality of your test suite 📈 
* Identify not tested source code 📝
* Provide guidance to your project 🔭

## Generate code coverage reports

As explained in the previous post achieving code coverage reports in .NET core is possible with the responsibility of the open source, cross platform package [Coverlet](https://github.com/tonerdo/coverlet "Coverlet source repository").

```shell
dotnet test 
    --no-build 
    --logger "trx;LogFileName=TestResults.trx" 
    --results-directory ./BuildReports/UnitTests/
    /p:CollectCoverage=true 
    /p:CoverletOutput=./BuildReports/Coverage/
    /p:CoverletOutputFormat=cobertura
```

Using the command above ```dotnet test``` will create the file ```TestResults.trx``` file using the Visual Studio logger and put those test results in the defined ```/BuildReports/UnitTests``` folder. Coverlet will make sure the coverage test data is collected, create the ```/BuildReports/Coverage``` folder and generate it in the ```cobertura``` format.

## Read code coverage reports

The previously generated data report needs to be easily interpreted, to achieve this I will use another great open-source tool: [reportgenerator](https://github.com/danielpalme/ReportGenerator)

```shell
dotnet reportgenerator 
    -reports:/BuildReports/Coverage/coverage.cobertura.xml
    -targetdir:BuildReports/Coverage/
    -reporttypes:HTML;HTMLSummary
```

This command will run the report generator to create a HTML report with a summary in human readable information based on the foregoing ```cobertura``` data file. The report files are generated in the defined ```targetdir``` folder. 

When opening the summary report in your favourite browser you can already have an overview of all covered and uncovered lines or branches as well the number of assemblies, classes, lines and risk hotspots.

![dotnet reportgenerator summary](/images/posts/measuring-code-coverage/reportgenerator-summary.png "Summary screenshot of the dotnet reportgenerator report for an example project")

Furthermore is possible to navigate into classes and see coverage statistics about them. In the screenshot underneath you can see the example```Calculator.cs``` class I've used in the [sample test project](https://github.com/antao/learning-measuring-code-coverage) to support this post. The methods/properties navigation bar displayed in the right side is quite useful.

![dotnet reportgenerator class](/images/posts/measuring-code-coverage/reportgenerator-calculator-class.png "Calculator screenshot of the dotnet reportgenerator report for an example class")

## Publishing code coverage reports

Publishing your reports in a continous integration flow is as simple as adding a new task to your build script, nowadays there are multiple web services that provide collecting, integration and tracking code coverage history: SonarCloud, Coveralls or Codecov. 

For this example I've used Coveralls, please not that all this services provide you a free and paid account plans; The process of publishing the coverage reports to Coveralls is quite simple. I've used the [Coveralls GitHub Action](https://github.com/marketplace/actions/coveralls-github-action) since the test project I've wrote was already hosted in GitHub. The following code [example](https://github.com/antao/learning-measuring-code-coverage/blob/master/.github/workflows/dotnetcore.yml) action file performs a series of steps in which, the last, 'Publish coverage report to coveralls.io' publishes the coverage report to Coveralls. It relies on the GITHUB_TOKEN, which is available in GitHub Actions by default, no extra setup is needed for this. Also, the repository will be created in Coveralls automatically the first time theaction is executed.

![coveralls](/images/posts/measuring-code-coverage/coveralls-io.png "Coveralls main project page")

