# CI Pipeline for .NET (Core) Application

## Table of Contents

- [CI Pipeline for .NET (Core) Application](#ci-pipeline-for-net--core--application)
  * [Table of Contents](#table-of-contents)
  * [Description](#description)
  * [Branching Strategies for Git](#branching-strategies-for-git)
    + [Git Flow](#git-flow)
      - [Advantages](#advantages)
      - [Disadvantages](#disadvantages)
    + [GitHub Flow](#github-flow)
      - [Advantages](#advantages-1)
      - [Disadvantages](#disadvantages-1)
    + [GitLab Flow](#gitlab-flow)
      - [Advantages](#advantages-2)
      - [Disadvantages](#disadvantages-2)
    + [One Flow](#one-flow)
      - [Advantages](#advantages-3)
      - [Disadvantages](#disadvantages-3)
  * [Semantic Versioning](#semantic-versioning)
    + [Advantages of SemVer](#advantages-of-semver)
    + [Points to keep in mind](#points-to-keep-in-mind)
  * [Git Tag](#git-tag)
    + [Creating a Tag](#creating-a-tag)
    + [Annotated Tags](#annotated-tags)
    + [Lightweight Tags](#lightweight-tags)
  * [CI Pipelines Flow](#ci-pipelines-flow)
  * [Example from Real world](#example-from-real-world)
    + [Required Tools](#required-tools)
    + [Build Steps](#build-steps)
      - [1. Generate Build Number](#1-generate-build-number)
        * [PowerShell](#powershell)
        * [Bash](#bash)
      - [2. Get Environment Variables](#2-get-environment-variables)
      - [3. Install NuGet 4.3.0 on Build Agent](#3-install-nuget-430-on-build-agent)
      - [4. Restore NuGet packages](#4-restore-nuget-packages)
      - [5. Prepare SonarQube analysis step](#5-prepare-sonarqube-analysis-step)
      - [6. Clean & Build Solution with MSBuild](#6-clean---build-solution-with-msbuild)
      - [7. Use Visual Studio Test Platform Installer for preparing platform for running Unit test](#7-use-visual-studio-test-platform-installer-for-preparing-platform-for-running-unit-test)
      - [8. Run Unit Tests](#8-run-unit-tests)
      - [9. Run Code Analysis](#9-run-code-analysis)
      - [10. Publish Quality Gate Result](#10-publish-quality-gate-result)
      - [11. Create NuGet package](#11-create-nuget-package)
      - [12. Push NuGet file to Azure feed repository](#12-push-nuget-file-to-azure-feed-repository)
- [CD pipeline](#cd-pipeline)
  * [Description](#description-1)
  * [Deployment Steps](#deployment-steps)

## Description

Continuous Integration (CI) is the practice of automating the integration of code changes from multiple contributors into a single software project. The CI process is comprised of automatic tools that assert the new code’s correctness before integration. A source code version control system is the crux of the CI process. The version control system is also supplemented with other checks like automated code quality tests, syntax style review tools, and more.

## Branching Strategies for Git

### Git Flow

The Git Flow is the most known workflow on this list. It was created by [Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/) in 2010 and it is based in two main branches with infinite lifetime:

- **master** — this branch contains production code. All development code is merged into master in sometime.
- **develop** — this branch contains pre-production code. When the features are finished then they are merged into develop.

During the development cycle, a variety of supporting branches are used:

- **feature-\*** — feature branches are used to develop new features for the upcoming releases. May branch off from develop and must merge into develop.
- **hotfix-\*** — hotfix branches are necessary to act immediately upon an undesired status of master. May branch off from master and must merge into master anddevelop.
- **release-\*** — release branches support preparation of a new production release. They allow many minor bug to be fixed and preparation of meta-data for a release. May branch off from develop and must merge into master anddevelop.

#### Advantages

- Ensures a clean state of branches at any given moment in the life cycle of project
- The branches naming follows a systematic pattern making it easier to comprehend
- It has extensions and support on most used git tools
- It is ideal when there it needs to be multiple version in production

#### Disadvantages

- The Git history becomes unreadable
- The master/develop split is considered redundant and makes the Continuous Delivery and the Continuos Integration harder
- It isn’t recommended when it need to maintain single version in production

Read more about [Git Flow branching strategy](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/GitFlow.png)
> Git Flow

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/GitFlowDetailed.png)

### GitHub Flow

The GitHub Flow is a lightweight workflow. It was created by [GitHub in 2011](http://scottchacon.com/2011/08/31/github-flow.html) and respects the following 6 principles:

1. Anything in the master branch is deployable
2. To work on something new, create a branch off frommaster and given a descriptively name(ie: new-oauth2-scopes)
3. Commit to that branch locally and regularly push your work to the same named branch on the server
4. When you need feedback or help, or you think the branch is ready for merging, open a pull request
5. After someone else has reviewed and signed off on the feature, you can merge it into master
6. Once it is merged and pushed to master, you can and should deploy immediately

#### Advantages

- It is friendly for the Continuous Delivery and Continuous Integration
- A simpler alternative to Git Flow
- It is ideal when it needs to maintain single version in production

#### Disadvantages

- The production code can become unstable most easily
- Are not adequate when it needs the release plans
- It doesn’t resolve anything about deploy, environments, releases, and issues
- It isn’t recommended when multiple versions in production are needed

### GitLab Flow

The GitLab Flow is a workflow created by [GitLab in 2014](https://about.gitlab.com/blog/2014/09/29/gitlab-flow/). It combine [feature-driven development](https://en.wikipedia.org/wiki/Feature-driven_development) and feature branches with issue tracking. The most difference between GitLab Flow and GitHub Flow are the environment branches having in GitLab Flow (e.g. *staging* and *production*) because there will be a project that isn’t able to deploy to production every time you merge a feature branch (e.g. SaaS applications and Mobile Apps)
The GitLab Flow is based on 11 rules:

1. Use feature branches, no direct commits on master
2. Test all commits, not only ones on master
3. Run all the tests on all commits (if your tests run longer than 5 minutes have them run in parallel).
4. Perform code reviews before merges into master, not afterwards.
5. Deployments are automatic, based on branches or tags.
6. Tags are set by the user, not by CI.
7. Releases are based on tags.
8. Pushed commits are never rebased.
9. Everyone starts from master, and targets master.
10. Fix bugs in master first and release branches second.
11. Commit messages reflect intent.

#### Advantages

- It defines how to make the Continuous Integration and Continuous Delivery
- The git history will be cleaner, less messy and more readable (see [why devs prefers squash and merge, instead of only merging, on this article](https://softwareengineering.stackexchange.com/questions/263164/why-squash-git-commits-for-pull-requests))
- It is ideal when it needs to single version in production

#### Disadvantages

- It is more complex that the GitHub Flow
- It can become complex as Git Flow when it needs to maintain multiple version in production

### One Flow

The One Flow is a proposed alternative in article GitFlow considered harmful by Adam Ruka, written in 2015. The main condition that needs to be satisfied in order to use OneFlow is that every new production release is based on the previous release. The most difference between One Flow and Git Flow that it not has *develop* branch.

#### Advantages

- The git history will be cleaner, less messy and more readable (see why devs prefers squash and merge, instead of only merging, on this article)
- It is flexible according to team decisions
- It is ideal when it needs to single version in production

#### Disadvantages

- It isn’t recommended for projects with Continuous Delivery or Continuous Deploy.
- The feature branches make it harder the Continuos Integration
- It isn’t recommended when it needs to maintain single version in production

## Semantic Versioning

Semantic versioning (also referred as **SemVer**) is a versioning system that has been on the rise over the last few years. It has always been a problem for software developers, release managers and consumers. Having a universal way of versioning the software development projects is the best way to track what is going on with the software as new plugins, addons, libraries and extensions are being built almost everyday.
Read more about [Semantic Versioning](https://semver.org/).

Semantic Versioning is a 3-component number in the format of X.Y.Z, where :

- **X** stands for a major version.
- **Y** stands for a minor version.
- **Z** stands for a patch.

So, SemVer is of the form Major.Minor.Patch.

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/SemVer.png)

**Working:** The goal of SemVer was to bring some sanity to the management of rapidly moving software release targets. As discussed above, 3 numbers i.e, Major, Minor and Patch are required to identify a software version. For example, if we take version 5.12.2, then it has a major version of 5, a minor version of 12 and a patch version of 2. Below given are the scenarios when you should bump the value of X, Y and Z.

Bump the value of **X** when breaking the existing API.
Bump the value of **Y** when implementing new features in a backward-compatible way.
Bump the value of **Z** when fixing bugs.

Valid identifiers are in the set **\[A-Za-z0-9\]** and cannot be empty. Pre-release metadata is identified by appending a hyphen to the end of the SemVer sequence. Thus a pre-release for version 1.0.0 could be 1.0.0-alpha.1. Then if another build is needed, it would become 1.0.0-alpha.2, and so on. Note that names cannot contain leading zeros, but hyphens are allowed in names for pre-release identifiers.

### Advantages of SemVer

- You can keep track of every transition in the software development phase.
- Versioning can do the job of explaining the developers about what type of changes have taken place and the possible updates that should take place in the software.
- It helps to keep things clean and meaningful.
- It helps other people who might be using your project as a dependency.

### Points to keep in mind

- The first version starts at 0.1.0 and not at 0.0.1, as no bug fixes have taken place, rather we start off with a set of features as first draft of the project.
- Before 1.0.0 is only the Development Phase, where you focus on getting stuff done.
- SemVer does not cover libraries tagged 0.*.*. The first stable version is v1.0.0.

For the **our projects** we are modified semantic versioning to **X.Y.Z.W**

- **X** - *Major*, from added annotated tag;
- **Y** - *Minor*, from added annotated tag;
- **Z** - *Commit* (patch), this will be incremented automatically by Git, after adding new commit;
- **W** - *Build number*, this is input parameter from build system (Azure DevOps, TeamCity, etc)

This version will be like *correlation id* for our CI and CD pipelines.
You have to be 100% sure that commit was deployed to Production or have answer on this question in 5 sec It prevent mess during release phase and etc.

## Git Tag

This part will discuss the Git concept of tagging and the git tag command. Tags are ref's that point to specific points in Git history. Tagging is generally used to capture a point in history that is used for a marked version release (i.e. v1.0.1). A tag is like a branch that doesn’t change. Unlike branches, tags, after being created, have no further history of commits. For more info on branches visit the git branch page. This document will cover the different kind of tags, how to create tags, listing all tags, deleting tags, sharing tags, and more.

### Creating a Tag

To create a new tag execute the following command:

```cmd
git tag <tagname>
```

Replace *\<tagname\>* with a semantic identifier to the state of the repo at the time the tag is being created. A common pattern is to use version numbers like git tag v1.4. Git supports two different types of tags, annotated and lightweight tags. The previous example created a lightweight tag. Lightweight tags and Annotated tags differ in the amount of accompanying meta data they store. A best practice is to consider Annotated tags as public, and Lightweight tags as private. Annotated tags store extra meta data such as: the tagger name, email, and date. This is important data for a public release. Lightweight tags are essentially 'bookmarks' to a commit, they are just a name and a pointer to a commit, useful for creating quick links to relevant commits.

### Annotated Tags

Annotated tags are stored as full objects in the Git database. To reiterate, They store extra meta data such as: the tagger name, email, and date. Similar to commits and commit messages Annotated tags have a tagging message. Additionally, for security, annotated tags can be signed and verified with GNU Privacy Guard (GPG). Suggested best practices for git tagging is to prefer annotated tags over lightweight so you can have all the associated meta-data.

```cmd
git tag -a v1.4
```

Executing this command will create a new annotated tag identified with *v1.4*. The command will then open up the configured default text editor to prompt for further meta data input.

```cmd
git tag -a v1.4 -m "my version 1.4"
```

Executing this command is similar to the previous invocation, however, this version of the command is passed the -m option and a message. This is a convenience method similar to git commit -m that will immediately create a new tag and forgo opening the local text editor in favor of saving the message passed in with the -m option.

Read more about git [Annotated tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

### Lightweight Tags

```cmd
git tag v1.4-lw
```

Executing this command creates a lightweight tag identified as v1.4-lw. Lightweight tags are created with the absence of the *-a*, *-s*, or *-m* options. Lightweight tags create a new tag checksum and store it in the *.git/* directory of the project's repo.

For our purposes we will use Annotated Tags for generation Semantic Version in projects.
**How is it working?**
Create Git repository and put there project.
Add appropriate tag (like **“v0.1”**) to your current branch.
Try to run next command in git cmd for your project.

```cmd
git describe --tags --long
```

Take a look on result. You should receive something like this: **v0.1-0-xxxxxx**.
After that add one more commit to Contoso project (just change something) and again run the command **git describe --tags --long** and you have to receive the result: **v0.1-1-xxxxxx**. So for the future we will use this **3** figures for versioning.

## CI Pipelines Flow

The main focus of next figure is to provide detailed view on CI pipepline definition.

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/CIFlow.jpg)

> CI pipeline design

The build system is designed to implement the CI/CD Process which is shown on the following figure:

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/CI_CDProcessforNetApplication.jpg)

> CI/CD process for .Net Application

## Example from Real world

This part will show you on example "How to" create CI pipeline for .NET application.
For our purposes we will use [ASP.NET MVC Application Using Entity Framework Code First](https://code.msdn.microsoft.com/ASPNET-MVC-Application-b01a9fe8).
More details you can find [here](https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application).
[Download](https://code.msdn.microsoft.com/ASPNET-MVC-Application-b01a9fe8/file/169473/2/ASP.NET%20MVC%20Application%20Using%20Entity%20Framework%20Code%20First.zip).

### Required Tools

- Git
- PowerShell
- MSBuild or MS Visual Studio
- Azure DevOps

**TODO:**

1. Create Git repository in Azure DevOps.
2. Put downloaded content there.
3. Add annotated tag to your branch. (for example v0.1)
4. Push tag to remote repository.
5. For checking your tag in the repository, execute next command **git describe --tags --long** in command line.You have to receive someting like **v0.1-0-xxxxxx**.
6. So we are ready for the [Build Step #1 Generate Build Number](#1.-generate-build-number)

### Build Steps

#### 1. Generate Build Number

Use any scripts for generating *Build Number*, but powershell is preferable. Create directory **BuildScripts** in solution folder. Put your script there (for the future all scripts releated to build, you have to put in this folder). Add *PowerShell* build step in AzureDevops and call your script from there. Do not forget to update ***assemblyInfo.cs*** files (globalAssemblyInfo.cs) in your script. Below you can find examples on *PowerShell* and *Bash*.
For **Deployment script** create separate folder with name **DeplomentScripts**.

##### PowerShell

```powershell
Param (
    [string]$build_num = "1",
    [string]$version_file_mask = "AssemblyInfo.cs"
)

function Get-GitVersion
{ 
    $git_version = (git describe --tags --long --match v?.? | Select-String -pattern '(?<major>[0-9]+)\.(?<minor>[0-9]+)-(?<seq>[0-9]+)-(?<hash>[a-z0-9]+)').Matches[0].Groups
    return $git_version
}

function Update-AssemblyVersion
{
    Param (
        [string]$version
    )

    foreach ($o in $input) 
    {
        Write-Host "Updating  '$($o.FullName)' -> $version"

        $assemblyVersionPattern = 'AssemblyVersion\("[0-9]+(\.([0-9]+|\*)){1,3}"\)'
        $fileVersionPattern = 'AssemblyFileVersion\("[0-9]+(\.([0-9]+|\*)){1,3}"\)'
        $assemblyVersion = 'AssemblyVersion("' + $version + '")';
        $fileVersion = 'AssemblyFileVersion("' + $version + '")';

        (Get-Content $o.FullName) | ForEach-Object  { 
           % {$_ -replace $assemblyVersionPattern, $assemblyVersion } |
           % {$_ -replace $fileVersionPattern, $fileVersion }
        } | Out-File $o.FullName -encoding UTF8 -force
    }
}

# Retrive Git Version Number
Write-Host "Resolving Git Version"
$git_version = Get-GitVersion

$git_describe = $git_version[0].Value 
Write-Host "Git Version is '$git_describe'"

# Prepare version numbers
$version = [string]::Join('.', @($git_version[1], $git_version[2].Value,$git_version[3].Value, $build_num))

# TFS - update the running package version
Write-Host ("##vso[task.setvariable variable=BUILD.BUILDNUMBER;]$versionTFS")
Write-Host "BUILD_BUILDNUMBER: " $env:BUILD_BUILDNUMBER

Write-Host ("##vso[build.updatebuildnumber]$versionTFS")
Write-Host "BUILD_BUILDNUMBER: " $env:BUILD_BUILDNUMBER

# Update AssemblyInfo.cs file
$fileRootFolder = (get-item $PSScriptRoot ).parent.FullName
Write-Host "Searching '$fileRootFolder'"
Get-ChildItem $fileRootFolder -recurse |? {$_.Name -eq $version_file_mask} | Update-AssemblyVersion $version
```

##### Bash

```sh
#!/bin/bash

# Get git version
git_version=$(git describe --tags --long --match 'v?.?')
echo "Current git version is: $git_version"

SEMVER_REGEX="(0|[1-9][0-9]*)\\.(0|[1-9][0-9]*)\\-(0|[1-9][0-9]*)(\\-[0-9A-Za-z-]+(\\.[0-9A-Za-z-]+)*)?(\\+[0-9A-Za-z-]+(\\.[0-9A-Za-z-]+)*)?$"

function error {
  echo -e "$1" >&2
  exit 1
}

function validate-version {
  local version=$1
  if [[ "$version" =~ $SEMVER_REGEX ]]; then
    # if a second argument is passed, store the result in var named by $2
    if [ "$#" -eq "2" ]; then
      local major=${BASH_REMATCH[1]}
      local minor=${BASH_REMATCH[2]}
      local patch=${BASH_REMATCH[3]}
      local prere=${BASH_REMATCH[4]}
      local build=${BASH_REMATCH[5]}
      eval "$2=(\"$major\" \"$minor\" \"$patch\" \"$prere\" \"$build\")"
    else
      echo "$version"
    fi
  else
    error "version $version does not match the semver scheme 'X.Y-Z(-PRERELEASE)(+BUILD)'."
  fi
}

# Split git version
validate-version "$git_version" parts
major="${parts[0]}"
minor="${parts[1]}"
patch="${parts[2]}"

echo "Major = $major"
echo "Minor = $minor"
echo "Commit = $patch"
echo "BuildID = $BUILD_BUILDID"

version="$major.$minor.$patch.$BUILD_BUILDID"

# Update VSTS variables
echo "##vso[task.setvariable variable=BUILD_BUILDNUMBER]$version"
echo "BUILD_BUILDNUMBER: $BUILD_BUILDNUMBER"

echo "##vso[build.updatebuildnumber]$version"
echo "BUILD_BUILDNUMBER: $BUILD_BUILDNUMBER"
```

After execution of your script you have to see next picture in Azure DevOps. It's really importambe in **BuildNumber** column to see generated **Build Number**.

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/azureDevOpsVersioning.JPG)

### 2. Get Environment Variables

This step is optional. It will save your time during debugging CI process. Use PowerShell script build step for running  script below. The script return your environment variables, value for secure variables will be showed like ******.

```powershell
$environmentVars = get-childitem -path Env:*
foreach($var in $environmentVars)
{
 $keyname = $var.Key
 $keyvalue = $var.Value
 
 Write-Host "${keyname}: $keyvalue"
}
```

### 3. Install NuGet 4.3.0 on Build Agent

If you use Build Agents provided by Microsoft, better to add this step. If you have configured your own build agent - you can skip it. Use Nuget 4.3.0 tool installer (or later).

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/useNuGet430Installer.JPG)

### 4. Restore NuGet packages

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/NuGetRestore.JPG)

During application development developers use 3rd party libraries. It can be NuGet, NPM and etc. For Contoso solution we also have to restore all dependencies. In our case it's NuGet packages.

**NuGet** is a [free and open-source package manager](https://en.wikipedia.org/wiki/Package_manager) designed for the Microsoft development platform (formerly known as NuPack). Since its introduction in 2010, NuGet has evolved into a larger ecosystem of tools and services.
NuGet is distributed as a Visual Studio extension. Starting with Visual Studio 2012, NuGet comes pre-installed by default. NuGet is also integrated with SharpDevelop. NuGet can also be used from the command line and automated with scripts.
It supports multiple programming languages, including:
.NET Framework packages
Native packages written in C++,[5] with package creation aided by CoApp

How to find Nuget console:
In **Visual Studio**: Tools -> **Nuget** Package Manager -> Package Manager Console.
Download **NuGet.exe** file and play with cmd.All detailed info about Nuget you can find [here](https://docs.microsoft.com/en-us/nuget/tools/nuget-exe-cli-reference).

### 5. Prepare SonarQube analysis step

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/prepareAnalysisSQ.JPG)

This step is optional.
Fill the gaps with next build variables:
![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/SQcfg.JPG)

- **Project Key** - SonarQube project unique key,
- **Project Name** - SonarQube project name, here better to combine variables like on screen. You will be able to keep track of all changes in one branch,
- **Project Version** - project version in SonarQube, use variable which was created in build step #1.

### 6. Clean & Build Solution with MSBuild

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/buildMSBuild.JPG)

Why is important to clean build?

When you clean a build, all intermediate and output files are deleted, leaving only the project and component files. From the project and component files, new instances of the intermediate and output files can then be built. The library of common tasks that is provided with MSBuild includes an [Exec](https://docs.microsoft.com/en-us/visualstudio/msbuild/exec-task?view=vs-2019) task that you can use to run system commands. For more information on the library of tasks, see [Task reference](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-task-reference?view=vs-2019).

You can add *Clean* like a separete step before build step or set a flag in appropriate field.

**More info about MSbuild**
The Microsoft Build Engine is a platform for building applications. This engine, which is also known as MSBuild, provides an XML schema for a project file that controls how the build platform processes and builds software. Visual Studio uses MSBuild, but it doesn't depend on Visual Studio. By invoking msbuild.exe on your project or solution file, you can orchestrate and build products in environments where Visual Studio isn't installed.
Visual Studio uses MSBuild to load and build managed projects. The project files in Visual Studio (.csproj, .vbproj, .vcxproj, and others) contain MSBuild XML code that executes when you build a project by using the IDE. Visual Studio projects import all the necessary settings and build processes to do typical development work, but you can extend or modify them from within Visual Studio or by using an XML editor. Read [here](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild?view=vs-2019).

**MsBuild Targets**
Targets group tasks together in a particular order and allow the build process to be factored into smaller units. For example, one target may delete all files in the output directory to prepare for the build, while another compiles the inputs for the project and places them in the empty directory. For more information on tasks, see [Tasks](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-targets?view=vs-2019).

**How to: Specify which target to build first**
Read [here](https://docs.microsoft.com/en-us/visualstudio/msbuild/how-to-specify-which-target-to-build-first?view=vs-2019)

**How to: Clean a build**
Read [here](https://docs.microsoft.com/en-us/visualstudio/msbuild/how-to-clean-a-build?view=vs-2019)

**MSBuild command-line reference**
Read [here](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-command-line-reference?view=vs-2019)

Specify next parameters for build with MSBuild:

- Configuration have to be **Release** (/p:Configuration=Release)
- Platform have to be **Any CPU** (/p:Platform="Any CPU")
- Target discuss with DEV team (/t:Build)

### 7. Use Visual Studio Test Platform Installer for preparing platform for running Unit test

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/VSTestPI.JPG)

This Step is optional.
If you use Build Agents provided by Microsoft, better to add this step. If you have configured your own build agent - you can skip it.

### 8. Run Unit Tests

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/VStest_testAssemblies.JPG)

If project already has Unit tests add this build step.

Fill the gaps with next build variables:

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/RunUnitcfg.JPG)

**Test Files** - list of Test files
**Search Folder** - folder, where tests will be run

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/RunUnitcfg2.JPG)

Do not forget to enable Code Coverage.

### 9. Run Code Analysis

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/RunCodeAnalysis.JPG)

This Step is optional.

### 10. Publish Quality Gate Result

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/PublishQGResult.JPG)

This Step is optional.

### 11. Create NuGet package

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/NuGetPack.JPG)

As Build result we will have NuGet package. So for NuGet package creation add this build step.
Technically speaking, a NuGet package is just a ZIP file that's been renamed with the .nupkg extension and whose contents match certain conventions. This topic describes the detailed process of creating a package that meets those conventions. For a focused walkthrough, refer to [Quickstart: create and publish a package](https://docs.microsoft.com/en-us/nuget/quickstart/create-and-publish-a-package-using-visual-studio?tabs=netcore-cli).
[Package versioning](https://docs.microsoft.com/en-us/nuget/reference/package-versioning)

NuGet package have to contain **build artifacts** + **DeploymentScripts** folder.

### 12. Push NuGet file to Azure feed repository

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/NuGetPush.JPG)

Use Azure Feed like artifactory storage (storage for the build artifacts). Push already created NuGet package (build artifact) to Feed.
**Do NOT use drop for this purposes!**

# CD pipeline

## Description

Continuous deployment is a strategy for software releases wherein any code commit that passes the automated testing phase is automatically released into the production environment, making changes that are visible to the software's users.

Continuous deployment eliminates the human safeguards against unproven code in live software. It should only be implemented when the development and IT teams rigorously adhere to production-ready development practices and thorough testing, and when they apply sophisticated, real-time monitoring in production to discover any issues with new releases.

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/CDFlow.jpg)

**Release** - set of commands, which explain how to deploy package to environment (consist of package + deployment configuration).

Pipeline have to take NuGet package from Azure Feed and push it to Azure Web App (App Service) for example.  
Release name have to be: **Name_packageVersion_$(rev:r)**

Example: *ndMail_AutoTests_$(BUILD.BUILDNUMBER)_$(rev:r)*

- **ndMail_AutoTests** - package Name
- **packageVersion** - package version, was generated during build
- **$(rev:r)** - number of deployments for build package (you can deploy one package with different confgirations a several times)

Use secret variables for storing sensitive data, like key/password for Service Principals.
For passing secret variables to deployment use script arguments.
Create Release all time, when **NEW NuGet package** appears in Azure Feed Repository.

## Deployment Steps

1. Get package from Feed (automatically)
2. Get All variables (optional) - will help you debug deployment process
3. Unpack NuGet package (optional)
4. Push to Azure App Service

![](https://github.com/CoE-GitHub/azure-best-practices/blob/ososhenko/images/PushToAppService.JPG)
