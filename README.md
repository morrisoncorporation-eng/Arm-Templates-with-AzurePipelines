# Arm-Templates-with-AzurePipelines

```
trigger:
- master

stages:
- stage: Build
  jobs:
  - job: Build

    pool:
      vmImage: 'vs2017-win2016'

    steps:    
    - task: CopyFiles@2
      inputs:
        SourceFolder: '$(system.DefaultWorkingDirectory)/ArmTemplates'
        Contents: '**'
        TargetFolder: '$(system.DefaultWorkingDirectory)/ArmTemplates'
    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Pipeline.Workspace)'
        artifact: 'drop'
        publishLocation: 'pipeline'

- stage: Release
  jobs:
  - job: Release
  
    steps:
    - task: DownloadPipelineArtifact@2
      inputs:
        buildType: 'current'
        artifactName: 'drop'
        targetPath: '$(System.ArtifactsDirectory)'
    - task: AzureResourceManagerTemplateDeployment@3
      inputs:
        deploymentScope: 'Resource Group'
        azureResourceManagerConnection: 'ServicePrincipal-connectionName'
        subscriptionId: '000000-000-000f-82343-2353647'
        action: 'Create Or Update Resource Group'
        resourceGroupName: '$(resourcegroupname)'
        location: 'East US'
        templateLocation: 'Linked artifact'
        deploymentMode: 'Incremental'
        csmFile: '$(System.ArtifactsDirectory)/s/ArmTemplates/azurevm.json'
        overrideParameters: '-adminUsername "$(vmuser)" -adminPassword $(vmpassword) -customscript $(blobsas) -vmName "$(vmname)"'
        
 ```
