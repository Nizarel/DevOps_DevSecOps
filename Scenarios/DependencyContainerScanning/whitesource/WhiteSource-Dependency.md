# Add WhiteSource Configuration to Source Code

With a WhiteSource account [setup](./WhiteSource-Setup.md) we need to add a configuration file to the source code that will be associated with the project. This will need to be done for each repo.

_If a configuration has already been added to your source skip downloading the template and review the configuration as indicated below._ A template configuration for WhiteSource can be found [here](https://s3.amazonaws.com/unified-agent/wss-unified-agent.config) and will need to have several settings altered before being committed to the repository.  To retrieve the api key, navigate to the **Integrate** tab in your whitesource organization.  Documentation for Whitesource config file settings can be found [here](https://whitesource.atlassian.net/wiki/spaces/WD/pages/489160834/Unified+Agent+Configuration+File+Parameters).

## Minimum configuration details

``` shell

    checkPolicies=true
    forceCheckAllDependencies=true
    #check policies at the end of the scan. The default is to complete the scan and only report in whitesource inventory.
    #with this option set true we can get feedback into the pipeline and fail the build on best-practice and custom policies.
    ...

    wss.url=https://saas.whitesourcesoftware.com/agent
    #ensure that the url matches your WhiteSource provided url with the /agent endpoint specified
    ...

    apiKey=
    #The API Key must be provided. It can be explicitly set at runtime, or provided in this config.
   #Privacy of source code is a major deciding factor on location.  We recommend setting the apiKey in your pipeline using secure variables.


    projectName=
    projectToken=
    #projectName or projectToken is required. Both are not accepted in a single configuration and will cause the scan to fail.

    productName=
    productVersion=
    productToken=
    #productName and productToken can both be given, but the requirement is for one or the other.

    includes=**/*.dll **/*.cs **/*.nupkg **/*.js
    #the config requires have 1 (and only 1) include line that defines the patterns of files to scan.

    nuget.resolvePackagesConfigFiles=false
    nuget.resolveCsProjFiles=true
    nuget.resolveDependencies=true
    nuget.restoreDependencies=true
    nuget.preferredEnvironment=dotnet
    nuget.packagesDirectory=
    nuget.ignoreSourceFiles=false
    nuget.runPreStep=true
    nuget.resolveNuspecFiles=false
    #These are recommended settings for .net core projects.

```

With the project's whitesourece config set as described above any policy violations will cause the pipeline to fail. We simply need to create a couple of task in the pipeline that run the scanner. We will start out by adding an appropriate task for making our dependencies availble to whitesource. Although the WhiteSource agent scans package management files we need to restore dependencies to make sure none are missed. For instance, if we are working with dotnet core we would want to preempt running the scan with **dotnet retore**, or in a TypeScript application we would start by running **npm install** on any directories that have package.json dependencies.

Next, we will need to download the scanner into our build agent. In a new or existing pipeline agent add 2 task of the type "CMD".

![add cmd task](images/add-cmd.png)

Configure the first task to download the WhiteSource agent into the build agent.

- Set the display name to something like "Download WS Agent"
- Configure the script as follows:

``` shell
    curl -LJO https://github.com/whitesource/fs-agent-distribution/raw/master/standAlone/whitesource-fs-agent.jar
```

![download WS agent](images/agent-download.png)

Finally, add the other mentioned "CMD" task and configure it to run the scanner

- Set the display name of the task to something like "Run WS Agent"
- Configure the script as follows, replacing `FILENAME` and `API_KEY`:

``` shell
    java -jar whitesource-fs-agent.jar -c FILENAME.config -apiKey API_KEY
    exit $?
```

*NOTE:* Although the API key can be added to the configuration file added to the source code repo it is recommended that keys of that type be secured. As a best practice adding the API Key to the pipeline/variable configuration offers better control.

![run the WS scanner](images/run-scanner.png)

## Run the pipeline

*NOTE:* If you have not already set up policies in you WhiteSource account you will want to do so before running the pipeline. Read back through the [setup doc](./WhiteSource-Setup.md/) for help with that.

At this point you sould be able to run the WhiteSource scanner and find vulnerable packages with exclusions & inclusions based on best practice (Defaults in whitesource) combined with any customer policies.

With the example configuration above the scanner will check for policies that apply to the project in whitesource. These policies are applied based on the whitesource policy prioritization (project -> product -> organization) with project level policies being the winner overall. This allows applicaiton of policies for a project that override an organization level policy.

## Important Notes

- Scan is prone to failure related to corrupted downloads of wss-agent possibly related to load on download site.
- If there are any issues with the whitesource api then pipeline is slowed significantly. A timeout should be set inline with allowable fault tolerance for the pipeline.
