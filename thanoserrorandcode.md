##[debug]System.HostType=build
##[debug]agent.proxyurl=undefined
##[debug]VSTS_ARM_REST_IGNORE_SSL_ERRORS=undefined
##[debug]AZURE_HTTP_USER_AGENT=VSTS_b31c8309-3109-4a58-b50c-97c3337a6fef_build_29_0
##[debug]which '/usr/bin/docker'
##[debug]found: '/usr/bin/docker'
##[debug]/usr/bin/docker arg: images
##[debug]DOCKER_HOST=undefined
##[debug]exec tool: /usr/bin/docker
##[debug]arguments:
##[debug]   images
/usr/bin/docker images
##[debug]arguments=undefined
##[debug]tags=myimage:latest
##[debug]repository=***/thanoslistener
##[debug]containerRegistry=undefined
##[debug]DOCKER_CONFIG=/home/vsts/work/_temp/DockerConfig_1726758280678
##[debug]agent.tempDirectory=/home/vsts/work/_temp
##[debug]Found the Docker Config stored in the temp path. Docker config path: /home/vsts/work/_temp/DockerConfig_1726758280678/config.json, Docker config: {"auths": { "***": {"auth": "***", "email": "***" } }, "HttpHeaders":{"X-Meta-Source-Client":"VSTS"} }
##[debug]dockerFile=/home/vsts/work/1/s/**/Dockerfile
##[debug]Pushing ImageNameWithTag: ***/thanoslistener:myimage:latest
##[debug]which '/usr/bin/docker'
##[debug]found: '/usr/bin/docker'
##[debug]/usr/bin/docker arg: push
---------------
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [1/1] FROM quay.io/thanos/thanos:v0.30.2
#4 DONE 0.0s

#5 exporting to image
#5 exporting layers done
#5 writing image sha256:ae902dbabdd532825e000705dab0cd832a4900c4ce3eb581768cbb3546536f92 done
#5 DONE 0.0s
##[debug]Exit code 0 received from tool '/usr/bin/docker'
##[debug]STDIO streams have closed for tool '/usr/bin/docker'
##[debug]agent.tempDirectory=/home/vsts/work/_temp
##[debug]Synced the file content to the disk. The content is .
##[warning]No data was written into the file /home/vsts/work/_temp/task_outputs/build_1726758285665.txt
##[debug]Processed: ##vso[task.issue type=warning;source=TaskInternal;correlationId=aa01ae8a-9481-410f-a654-b5432ddea1f5;]No data was written into the file /home/vsts/work/_temp/task_outputs/build_1726758285665.txt
##[debug]set DockerOutput=/home/vsts/work/_temp/task_outputs/build_1726758285665.txt
##[debug]Processed: ##vso[task.setvariable variable=DockerOutput;isOutput=false;issecret=false;]/home/vsts/work/_temp/task_outputs/build_1726758285665.txt
##[debug]task result: Succeeded
##[debug]Processed: ##vso[task.complete result=Succeeded;]
Finishing: Build Docker Image
-------------
trigger:
- main

resources:
- repo: self

variables:
  system.debug: true
  tag: '$(Build.BuildId)'   # Use Build ID as a tag
  imageName: 'myimage'   # Full Docker Hub image name with username
  repoName: 'pexabo/thanoslistener'

stages:
- stage: Build
  displayName: Build and Push Docker Image
  jobs:
  - job: Build
    displayName: Build Docker Image
    pool:
      vmImage: ubuntu-latest   # Ubuntu as the agent
    steps:
    # Login to Docker Hub
    - task: Docker@2
      displayName: 'Login to Docker Hub'
      inputs:
        command: login
        containerRegistry: 'DockerHub'   # Use the Docker Hub service connection

    # Build the Docker image
    - task: Docker@2
      displayName: 'Build Docker Image'
      inputs:
        command: build
        dockerfile: '$(Build.SourcesDirectory)/Dockerfile'   # Ensure Dockerfile path is correct
        tags: |
          $(imageName):latest

    # Push the Docker image to Docker Hub
    - task: Docker@2
      displayName: 'Push Docker Image'
      inputs:
        command: push
        repository: $(repoName)   # Push the built image to Docker Hub
        tags: |
          $(imageName):latest

      -------------

      >>> FIX 
