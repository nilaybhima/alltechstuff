---
title: "Apply DSC Configuration to Azure Virtual Machines with Azure DevOps"
categories:
  - Azure DevOps
tags:
  - Azure DevOps
  - Azure
  - Azure Virtual Machine
  - Windows
last_modified_at: 2020-04-13
read_time: true

---

There are a few ways to apply PowerShell DSC Configuration to Azure Virtual Machines running Windows. This article shows how to apply DSC Configuration to Azure VMs through Azure DevOps and encourages the concept of GitOps for a DevOps Team.

Applying DSC Configuration through a local machine usually involves establishing a CIM Session with your VM and installing certificates from the Cloud Service where necessary. You can also use the Set-AzRmVMDscExtension to apply configuration to the Azure VM. Another way to apply DSC configuration would be to use an Azure Automation Account. This is a fantastic way to manage configuration for your Azure Virtual Machines but could be a longer process especially if you are trying to push DSC Configuration through tools like Azure DevOps. This article introduces the ARM PowerShell DSC Extension for Windows VMs and shows how to build a CD Pipeline for pushing your DSC Configuration to Azure Virtual Machines in the fastest yet efficient way possible.

## The ARM PowerShell Extension

This is an endpoint available on the Azure Resource Manager that pushes a DSC Configuration onto your Azure Virtual Machine. Here is a quick link that gives you all the information regarding the endpoint [here](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/dsc-windows).

## How Its Done

The initial step is to package your DSC Configuration and to make it available on a Storage Account. The package link is then referenced in the ARM template with a SAS Token acquired from the Storage Account. The rest is taken care by the ARM Template!

## GitOps

The GIT Repository will have all the infrastructure templates source controlled. This includes the ARM Templates, DSC Configuration files and any other scripts that would be involved in setting the desired state of your VMs. This implements the core concept of GitOps where the declarative description of your Infrastructure is source-controlled, hence versioned and can be deployed through a CD Pipeline. This introduces several benefits like high velocity of deployment, faster recovery and more security over the deployed infrastructure. The model that we consider is the Push-Based Deployment for GitOps in this scenario.

## The CD Pipeline

The first step is to reference your existing GIT Repository as an artifact in your CD Pipeline. In this case, you need to reference the GIT Repository that has all the DSC Configuration files source controlled.

You can then provision an Azure Storage Account with a container that can hold the DSC Packages. The snippet below is a resource for an ARM Template that provisions a storage account:

```yaml
{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.Storage/storageAccounts",
  "name": "[parameters('storageAccountName')]",
  "location": "[parameters('location')]",
  "sku": { "name": "Standard_LRS" },
  "kind": "StorageV2",
  "resources":
    [
      {
        "name": "[concat('default/', parameters('containerName'))]",
        "type": "blobServices/containers",
        "apiVersion": "2018-02-01",
        "dependsOn":
          [
            "[concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName'))]",
          ],
        "properties": { "publicAccess": "Blob" },
      },
    ],
}
```

The next step is to publish your DSC Configuration using the Publish-AzRmVMDscConfiguration command. This command can be used with the Azure PowerShell task of Azure DevOps so that you can use your existing service connection for Azure.

```yaml
$parameters =@{
'configurationpath' = '$(System.DefaultWorkingDirectory)/scripts/configuration.ps1'
'resourcegroupname' = '$(resourceGroupName)'
'StorageAccountName' = '$(storageAccountName)'
}

Publish-AzRmVMDscConfiguration @parameters -force - Verbose
```

You can then deploy the ARM Template with the DSC PowerShell Extension but first, it will require a SAS Token to access the storage account. There are a few ways to achieve this: One is to obtain a SAS Token within the ARM Template. However, there is a bug currently which can be seen [here](https://github.com/MicrosoftDocs/azure-docs/issues/15061). Another way would be to use PowerShell to get a SAS Token as in the snippet below:

```yaml
$context = (Get-AzStorageAccount -ResourceGroupName '$(resourceGroupName)' -AccountName '$(storageAccountName)').context

$token = New-AzStorageAccountSASToken -Context $context -Service "Blob" -ResourceType "Service,Container,Object" -Permission "rwl"

# This will save the token into a variable secret in the pipeline called SAS Token.
Write-Host "##vso[task.setvariable variable=sasToken;issecret=true]$token"
```

The sasToken variable secret can then be referenced into your ARM Template via a parameter. The snippet below shows how you can use the Micorosft.PowerShell DSC Property with the "Microsoft.Compute/virtualMachines/extensions" endpoint. The snippet registers a VM to a Host Pool for Azure WVD:

```yaml
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "apiVersion": "2018-10-01",
  "name": "[concat(parameters('rdshName'),'/', 'dscextension')]",
  "location": "[parameters('location')]",
  "dependsOn": [],
  "properties":
    {
      "publisher": "Microsoft.Powershell",
      "type": "DSC",
      "typeHandlerVersion": "2.73",
      "autoUpgradeMinorVersion": true,
      "settings":
        {
          "modulesUrl": "[uri(parameters('_artifactsLocation'), concat('Configuration.zip', parameters('sasToken')))]",
          "configurationFunction": "Configuration.ps1\\RegisterSessionHost",
          "properties":
            {
              "TenantAdminCredentials":
                {
                  "userName": "[parameters('tenantAdminUpnOrApplicationId')]",
                  "password": "PrivateSettingsRef:tenantAdminPassword",
                },
              "RDBrokerURL": "[parameters('rdBrokerURL')]",
              "DefinedTenantGroupName": "[parameters('existingTenantGroupName')]",
              "TenantName": "[variables('existingTenantName')]",
              "HostPoolName": "[variables('hostPoolName')]",
              "Hours": "[variables('registrationExpirationHours')]",
              "isServicePrincipal": "[parameters('isServicePrincipal')]",
              "AadTenantId": "[parameters('aadTenantId')]",
              "RDPSModSource": "[variables('RDPSModSource')]",
            },
        },
      "ProtectedSettings":
        {
          "items":
            { "tenantAdminPassword": "[parameters('tenantAdminPassword')]" },
        },
    },
}
```

Once the above job is done, the storage account can then be removed:

```yaml
Remove-AzStorageAccount -ResourceGroupName "$(resourceGroupName)" -AccountName "$(storageAccountName)"
```

Here is how the pipeline steps look like:

<figure style="width: 500px">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/2020_04_13_cd_pipeline_dsc_config.png" alt="">
</figure>

## Comments

This article focused on showing how to apply DSC Configuration through an Azure DevOps CD Pipeline to an Azure Virtual Machine. Whether it is traditional VMs or Kubernetes infrastructure, you can build, deploy and update your entire Infrastructure by implemeting GitOps in your Team and this is all possible through Azure DevOps!
