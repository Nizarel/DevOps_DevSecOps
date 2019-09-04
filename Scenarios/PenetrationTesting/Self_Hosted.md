## Self-hosted Agents and Pen-Testing

- Create or select an existing VM to host the Linux agent.
- Install a [self-hosted agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-linux?view=azure-devops). 

**NOTES:** 
 - Installing Docker via snap may cause permissions and file errors with the scanner. Please avoid using the extension with docker install via this manner. 
 - When using a Self-Hosted VM we need to make sure that the '-rm' argument is given to docker run command so that the container is taken down after completing scans. 
 - If a self-hosted agent is used with a dockerized application the container running the target applicaiton should be cleand up after the scan. If the agent host is only running docker containers in relation to the scans you can clean up by appending a script to your pipeline with simple docker stop and rm commands.

 ``` YAML
    docker stop $(docker ps -a -q)
    docker rm $(docker ps -a -q)
 ```
 
Return to chosen [pipeline setup instructions](./README.md)

