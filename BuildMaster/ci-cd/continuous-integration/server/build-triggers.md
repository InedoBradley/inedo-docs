---
title: Automatic Build Triggering
show-headings-in-nav: true
sequence: 400
---

BuildMaster supports the automatic creation of builds with customizable, pre-defined criteria. This functionality is collectively known as "build triggers”. Build triggers either monitor your [source control repository](/docs/buildmaster/builds/continuous-integration/source-control) or execute at certain times of the day. Their proper use can drastically decrease the amount of time it takes organizations to get from concept through to actual deployment into production ('lead time'). 

BuildMaster supports three general types of build triggers:

{.docs}
 * **[Source Control Repository Monitors](/docs/buildmaster/builds/continuous-integration/build-triggers-and-monitors/repository-monitors)**  will watch a Git or Subversion repository for new commits
 * **[Webhook Monitors](/docs/buildmaster/builds/continuous-integration/build-triggers-and-monitors/repository-hooks)** will listen for webhook events that you configure in your GitLab or GitHub account; see [GitLab Webhook Monitors](https://github.com/Inedo/inedox-git/wiki/configuring-gitlab-hooks) and [GitHub Webhook Monitors](https://github.com/Inedo/inedox-git/wiki/configuring-github-hooks)
 * **Scheduled Build Trigger** will execute on a daily, weekly, or custom interval

The default behavior of each of these triggers is to create a new build in either all active releases, the latest release, or a specific release (which could be a number or a variable, such as a application-defined `$TriggerReleaseNumber`).

Triggers can be set at the application level, or at the system level if you have many applications with similar conventions. For example, [BuildMaster's public instance](https://buildmaster.inedo.com/) configures a number of its applications using a shared GitLab webhook trigger and has some advanced selection logic to determine which application to create a build in: 

````
Log-Debug "Webhook received for $Repository repository (Branch=$Branch)";
if $Repository == inedo-docs
{
    Create-Build
    (
        Application: Docs.Inedo.Com,
        Variables: %(Branch: $Branch)
    );
}
else if $Branch == master
{
    if $MatchesRegex($Repository, ^inedox-.`$*)
    {
        set $appName = $Substring($Repository, 7);
        Log-Debug "Looking for $appName application...";
    
        Log-Debug `$ApplicationExists(`$appName) = $ApplicationExists($appName);
    
        if $ApplicationExists($appName)
        {
            Log-Debug "$appName application found. Creating build...";
        
            Create-Build
            (
                Application: $appName
            );
        }
    }
    
    if $Repository == upack || $Repository == Inedo.UPack
    {
        Create-Build
        (
            Application: $Repository
        );
    }
    
    if $Repository == inedo-docs-jp
    {
        Create-Build
        (
            Application: InedoDocs-jp,
            Variables: %(Branch: $Branch)
        );
    }
}
````

## Customizable & System-level Build Triggers

In contrast to the limitations of continuous integration platforms like Jenkins/TeamCity that force you into a "every commit gets a build" workflow, BuildMaster's build triggers are fully customizable and can be used either individually or in combination across any time horizon. This provides flexibility to customize their potential use cases to meet unique business needs.

For example, you can use both repository and webhook triggers to capture new commits, in real-time, across siloed code being worked on by different development teams. You could then create an additional scheduled build trigger that will check for changes across all the teams at some regular interval, compile them, and test it.

In addition to workflow efficiency, these types of customizable build triggers eliminate the anxiety that comes from the creation of an excessive number of builds. Instead of having dozens or even hundreds of unnecessary builds, developers and management teams can focus their attention on a commit/build workflow that is pre-defined for efficiency.

## Executing Custom Build Trigger Logic Using OtterScript

BuildMaster uses OtterScript, a domain-specific language, to allow you to add custom, complex logic to your build triggers.

Typically, the execution of build trigger function will create a build. However, there are use cases for when you would want additional automation to occur. Any additional context from the commit will be available via deployment variables. Common use-cases are:

    Automatically creating a release & build when a new branch is created by combining the Ensure-Release and Create-Build operations 
    Sending an email notification when a change is detected using the Send-Email operation 
    Create builds for multiple releases or spanning multiple applications
    
OtterScript's complex logic gives you flexibility that, in turn, allows you to make better use of known best practices such as branched builds. Platforms like Jenkins and TeamCity are rigid and require you to create a build against specific branches. In contrast, OtterScript can be used to execute different build plans depending on which branch a source control change is on. This allows BuildMaster to automatically ensure commits go to any branch, for any purpose, for any reason.

For example, you can use OtterScript to use legacy methods to build legacy releases of your software, while simultaneously releasing commits for the current version into production for testing. Another example would be using time/date specific logic to change the workflow for commits based on the time of day. 

### Example OtterScript 
The following OtterScript is essentially what will execute when you configure a Git repository monitor that creates builds against the latest release, and select the option to create `$Branch` and `$CommitHash` variables on build.
 
```
Set-BuildVariable Branch
(
    Value: $Branch
);
Set-BuildVariable CommitHash
(
    Value: $CommitHash
);
Create-Build
(
    Application: $ApplicationName
);
```

This is mostly for reference; if you wanted this exact behavior, you should just use the built-in *create a new build* function.
