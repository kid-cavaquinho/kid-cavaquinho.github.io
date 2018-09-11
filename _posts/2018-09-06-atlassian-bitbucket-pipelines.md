---
layout:     post
title:      Bitbucket Pipelines and .NET Core 
date:       2018-09-06 18:41:00
summary:    "Automating your NuGet packages from test to production"
categories: continuous-delivery continuous-integration dotnet nuget
---

I used to remember that time when implementing continuous delivery was not an easy task. Recently, I have found <a href="https://bitbucket.org/product/features/pipelines" target="_blank">Bitbucket Pipelines</a> an integrated CI/CD product of Atlassian. The system promises and states at their official website a "trivial configuration to set up, automating your code from test to production", they are not lying. 

What I wanted to achieve is a rather simple scenario - after each push to a feature branch the service restores needed packages, builds the code and runs tests. From there and after the necessary pull request reviews from your fellow team members when the branch is merged with master the service ships code. 

Bitbucket Pipelines uses docker images to run the builds and the process of configuration goes through an YAML file <a href="https://bitbucket.org/joaofilipeantao/bitbucket-pipelines-and-.net-core/src/master/bitbucket-pipelines.yml" target="_blank">`bitbucket-pipelines.yml`</a> where you can describe your needed build steps. 

In this case I used it to build, test, pack and push a .NET Core NuGet package. Let me show you how. 

## Bitbucket-pipelines.yml configuration
{% highlight yml lineanchors %}
image: microsoft/dotnet:sdk
pipelines:
  default:
    - step:
        name: Restore and Build
        caches:
          - dotnetcore
        script:
          - dotnet restore
          - dotnet build ${PROJECT_NAME}
    - step:
        name: Tests
        caches:
          - dotnetcore
        script:
          - dotnet test ${PROJECT_UNITTESTS_NAME}
  branches:
    master:
     - step:
         name: Pack and Push
         caches:
           - dotnetcore
         deployment: production 
         script: 
           - dotnet pack -c Release -v Normal
           - dotnet nuget push ${BITBUCKET_CLONE_DIR}/${PROJECT_NAME}/bin/Release/*.nupkg -s ${MYGET_URL} -k ${MYGET_APIKEY}
{% endhighlight %}   

## Add environment variables
I introduced environment variables for specifying NuGet feed url, key and project names, as you can see on the image bellow. You can add the values needed and apply them to your configuration by opening your repository and select <b>Settings</b> -> <b>Pipelines</b> -> <b>Environment Variables</b>. 

![alt text](http://localhost:4000/content/bitbucket-pipelines/environment_variables.png "Screenshot displaying my repository environment variables ")

The BITBUCKET_CLONE_DIR it is the directory in which the repository was initially cloned and your artifacts will be produced. <b>There is no need to add this key to the environment variables</b>. 

## Bitbucket Pipelines in action
Create your feature branch, add your awesome commits and push them. Once that is done, Pipelines should start automatically creating your build as you can see on the screenshot bellow.

![alt text](http://localhost:4000/content/bitbucket-pipelines/pipelines_default.png "Screenshot of Bitbucket Pipelines restoring, building and testing the code automatically steps")

Once you merge this branch with master the pack and push commands will be executed automatically. The example library is a light solution, in the screenshot you can see it takes about 17 seconds to pack in release configuration and push. Because I did not specified a version in the pack command, the package is created with the version number set in the package configuration. The push command will send the generated package to a NuGet feed, in this case <a href="https://www.myget.org/" target="_blank">MyGet</a>.

![alt text](http://localhost:4000/content/bitbucket-pipelines/pipelines_master.png "Screenshot of Bitbucket Pipelines packing and pushing the code automatically steps")

I specified deployment towards production in the master branch build step. This will enable a visual indication of what is currently shipped in Deployments tab.

![alt text](http://localhost:4000/content/bitbucket-pipelines/deployment.png "Screenshot of deployement tab of Bitbucket Pipelines showing release number #16")

## Conclusion
Bitbucket Pipelines provides a <b>easy to setup</b>, <b>flexible</b> and <b>integrated</b> solution for CI/CD in the Atlassian ecosystem. You can use it in the cloud or on premises with the following <a href="https://bitbucket.org/product/pricing" target="_blank">pricing</a>. It does contains some <a href="https://confluence.atlassian.com/bitbucket/limitations-of-bitbucket-pipelines-827106051.html" target="_blank">limitations</a> you should be aware of.
Enables integration with AWS, Google Clould, Kubernetes or Azure and supports repositories in many more programming languages. I've found great <a href="https://confluence.atlassian.com/bitbucket/configuring-your-pipeline-872013574.html" target="_blank">documentation</a> support provided while writing this post.

You can find the source code for the library <a href="https://bitbucket.org/joaofilipeantao/bitbucket-pipelines-and-.net-core/src/master/" target="_blank">here</a>. 

<small>No Git or Mercurial project repositories have been harmed while writing this post, actually they are both supported.</small>