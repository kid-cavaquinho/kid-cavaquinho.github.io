---
layout:     post
title:      Bitbucket pipelines and .NET Core 
date:       2018-09-06 18:41:00
summary:    "Automating your NuGet packages from test to production"
categories: continuous-delivery continuous-integration
---

I used to remember a time when implementing continuous delivery was not an easy task. Recently, I have found [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines) an integrated CI/CD product of Atlassian. The system promises and states at their official website a "trivial configuration to set up, automating your code from test to production", they are not lying.

What I wanted to achieve is a rather simple scenario - after each push to a feature branch the service restores needed packages, builds the code and runs tests. From there and after the necessary pull request reviews from your fellow team members when the branch is merged with master the service ships code.

Bitbucket Pipelines uses docker images to run the builds and the process of configuration goes through an YAML file [`bitbucket-pipelines.yml`](https://bitbucket.org/joaofilipeantao/bitbucket-pipelines-and-.net-core/src/master/bitbucket-pipelines.yml) where you can describe your needed build steps.

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

I introduced environment variables for specifying NuGet feed url, key and project names in my project repository, as you can see on the image below. You can add the values needed and apply them to your configuration by opening your repository and select **Settings** -> **Pipelines** -> **Environment Variables**.

![Screenshot displaying the repository environment variables used in the yaml configuration file](/images/posts/bitbucket-pipelines/environment_variables.png "Screenshot displaying the repository environment variables used in the yaml configuration file")

The BITBUCKET_CLONE_DIR it is the directory in which the repository was initially cloned and your artifacts will be produced. **There is no need to add this key to the environment variables**.

## Bitbucket Pipelines in action

Create your feature branch, add your awesome commits and push them. Once that is done, Pipelines should start automatically creating your build as you can see on the screenshot below.

![Screenshot of Bitbucket Pipelines restoring, building and testing the code automatic steps](/images/posts/bitbucket-pipelines/pipelines_default.png "Screenshot of Bitbucket Pipelines restoring, building and testing the code automatically steps")

Once you merge this branch with master the pack and push commands will be executed automatically. The example library is a light solution, in the screenshot you can see it takes about 17 seconds to execute this step. Because I did not specified a version in the pack command, the package is created with the version number defined in the configuration of your `.csproj` file. The push command will send the generated package to a NuGet feed, in this case I have used [MyGet](https://www.myget.org/).

![Screenshot of Bitbucket Pipelines packing and pushing the code automatic steps](/images/posts/bitbucket-pipelines/pipelines_master.png "Screenshot of Bitbucket Pipelines packing and pushing the code automatically steps")

I specified deployment towards production in the master branch build step. This will enable a visual indication of what is currently shipped in Deployments tab.

![Screenshot of deployement tab of Bitbucket Pipelines showing release number #16](/images/posts/bitbucket-pipelines/deployment.png "Screenshot of deployment tab of Bitbucket Pipelines showing release number #16")

## Conclusion

Bitbucket Pipelines provides a **easy to setup**, **flexible** and **integrated** solution for CI/CD in the Atlassian ecosystem. You can use it in the cloud or on premises with the following [pricing](https://bitbucket.org/product/pricing). It does contains some [limitations](https://confluence.atlassian.com/bitbucket/limitations-of-bitbucket-pipelines-827106051.html) you should be aware of. Enables integration with AWS, Google Clould, Kubernetes or Azure and supports repositories in many more programming languages. I've found great [documentation](https://confluence.atlassian.com/bitbucket/configuring-your-pipeline-872013574.html) support provided while writing this post.

You can find the source code I have written for this example [here](https://bitbucket.org/joaofilipeantao/bitbucket-pipelines-and-.net-core/src/master/).

No Git or Mercurial project repositories have been harmed while writing this post, actually they are both supported.
