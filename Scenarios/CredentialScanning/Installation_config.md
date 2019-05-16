# Installation & Configuration

Once you have installed the extension into the org, you can start to add components into the pipeline.  You need to add the following four components in order to have an effective pipeline.

- CredScan
  -  You can use a suppression file if you have created one.  See [Suppressions](./Suppressions.md) for guidance.

- Create Security Analysis Report
  - Ensure CredScan is checked under tools parameter

- Publish Security Analysis Logs
  - Ensure CredScan is checked under tools parameter

- Post Analysis
  - Ensure CredScan is checked under tools parameter

![CredScan basic pipeline](./images/CredScan_Pipeline_Components.png)

## External Links

- [CredScan Install](https://secdevtools.azurewebsites.net/helpcredscan.html)

- [Post Analysis Setup](https://secdevtools.azurewebsites.net/helpPostAnalysis.html)

- [1eswiki](https://www.1eswiki.com/wiki/CredScan_Azure_DevOps_Build_Task)

-[CESec Engineering](https://microsoft.sharepoint.com/teams/CESecEngineering/CredScan/CredScan%20Wiki/Home.aspx)

## NOTES

CredScan by itself in a pipeline will not be useful.  You need to add in analysis and publish tasks to make the tool output meaningful.

![Example CredScan Pipeline](./images/2_BuildDefinition.png)
