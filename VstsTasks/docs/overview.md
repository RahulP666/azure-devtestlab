# Overview
This extension provides several build / release tasks to allow you to integrate with [Azure DevTest Labs](https://azure.microsoft.com/en-us/services/devtest-lab/). 
 
1. **Create Custom Image**
2. **Create VM**
3. **Delete VM**
4. **Create Environment**
5. **Delete Environment**
6. **Populate Environment**

You can find more details about Azure DevTest Labs [here](https://azure.microsoft.com/en-us/services/devtest-lab/).
# Details
## Create Custom Image
![Create Custom Image](screenshots/azure-dtl-createcustomimage.png)
The task allows you  to create a Custom Image in your lab based on an existing lab VM. You can install your latest build on a Lab VM and then create a custom image based on the VM. You can quickly create a lab VM using the custom image with the latest build already baked in. Moreover, the custom image is shared with all the users in the lab. Any lab user can use the custom image to create a lab VM and get started with the testing of the latest build in minutes.   
### Input Parameters
The task requires the following inputs: 

**Azure RM Subscription** - Azure Resource Manager subscription to configure before running. 

**Lab Name** - Name of an existing lab in which the custom image will be created. This is a pick list generated as a result of selecting an **Azure RM Subscription**.

**Custom Image Name** - Name of the custom image that will be created. You can use any variable such as `$(Build.BuildNumber)` or `$(Release.ReleaseName)` when in a build or a release, respectively. 

**Description** (optional) - Description of the custom image that will be created. If left blank, an auto-generated string will be used as the description.

**Source Lab VM ID** - Resource ID of the source lab VM. The source lab VM must be in the selected lab, as the custom image will be created using its VHD file. You can use any variable such as `$(labVMId)`, the output of calling **Create Azure DevTest Labs VM**, that contains a value in the form `/subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.DevTestLab/labs/{labName}/virtualMachines/{vmName}`.

**OS Type** - Type of operating system of the source lab VM. This is a pick list whose allowed values are _Linux_ or _Windows_.

**Linux OS State** (when **OS Type** = _Linux_) - Value indicating how to prepare the source lab VM for custom image creation. This is a pick list whose allowed values are _NonDeprovisioned_, _DeprovisionRequested_ or _DeprovisionRequested_. See [Deprovisioning](http://aka.ms/Deprovisioning) for more information.

**Windows OS State** (when **OS Type** = _Windows_) - Value indicating how to prepare the the source lab VM for custom image creation. This is a pick list whose allowed values are _NonSysprepped_, _SysprepRequested_ or _SysprepRequested_. See [Sysprep](http://aka.ms/Sysprep) for more information.

### Output Variables
The task can produce the following outputs into corresponding variables:

**Custom Image ID** - Variable to capture the created custom image ID. Default is `customImageId`. The variable can be referred as `$(customImageId)` in subsequent tasks. 


## Create VM
![Create VM](screenshots/azure-dtl-createvm.png)
The task allows you to create a lab VM using an ARM template generated in your Lab. You can generate the ARM template by selecting all the configurations required to create a lab VM and also adding [artifacts](https://azure.microsoft.com/en-us/documentation/articles/devtest-lab-artifact-author/) which you want to apply after the VM is created.  
### Input Parameters
The task requires the following inputs: 

**Azure RM Subscription** - Azure Resource Manager subscription to configure before running. 

**Lab Name** - Name of an existing lab in which the lab VM will be created. This is a pick list generated as a result of selecting an **Azure RM Subscription**.

**Template Name** - Path to the ARM template. You can [generate the ARM template](https://azure.microsoft.com/en-us/documentation/articles/devtest-lab-add-vm-with-artifacts/#save-arm-template) from the **View ARM template** section when creating a Lab VM. Select the ARM template by browsing to the file that you have saved in your VSTS source control. It can be either a relative path in a build output or a relative path inside an artifacts package.

**Template Parameters** (optional) - ARM template parameters to use. You can use any variable such as `$(Build.BuildNumber)` or `$(Release.ReleaseName)` when in a build or a release, respectively, for the `newVMName`. Similarly, you can create variables such as `User.Name` and `User.Password`, where the latter is marked as a _secret_.

### Advanced Options
The following advanced options can be specified on the task to help control the behavior of deployment:

**Fail on artifact error** (optional) - Fail the task if any artifact fails to apply successfully.

**Retry the deployment following any failure** (optional) - Retry the deployment when failing to create the lab VM or if any artifact fails to apply successfully.

**Number of times to retry the deployment** (optional) - Number of times to retry the deployment when an error occurs, either while creating the lab VM or if any artifact fails to apply successfully.

### Output Variables
The task can produce the following outputs into corresponding variables:

**Lab VM ID** - Variable to capture the created lab VM ID. Default is `labVMId`. The variable can be referred as `$(labVMId)` in subsequent tasks. You can use a name other than the default.

## Delete VM
![Delete VM](screenshots/azure-dtl-deletevm.png)
The task allows you to delete a lab VM.
### Input Parameters
The task requires the following inputs: 

**Azure RM Subscription** - Azure Resource Manager subscription to configure before running. 

**Lab VM ID** - Resource ID of the lab VM to delete. Default is `$(labVMId)`. You can use any variable such as `$(labVMId)`, which is the output of calling **Create Azure DevTest Labs VM**, that contains a value in the form `/subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.DevTestLab/labs/{labName}/virtualMachines/{vmName}`.

## Create Environment
![Create Environment](screenshots/azure-dtl-createenvironment.png)
The task allows you to create an Environment.
### Input Parameters
The task requires the following inputs:

**Azure RM Subscription** - Azure Resource Manager subscription to configure before running.

**Lab Name** - Azure DevTest Lab name.

**Repository Name** - Repository friendly name where the environment ARM templates are stored.

**Template Name** - Name of the environment ARM template.

**Environment Name** - Name of the deployed template.

**Parameters File (optional)** - The input parameters file. Use either a parameters file or the parameters.

**Parameters (optional)** - The individual parameters.

**Create output variables based on the environment template output.** - Create output variables resulting from the creation of the environment. See Output Variables for details. 

### Output Variables
If enabled, the task can produce the following output variables.  Note that you will need to define a Reference Name (i.e. `<refName>` such as `dtl`) under the Output Variables section to correctly reference the variables in the list.

**EnvironmentResourceId** - Resource ID of the environment created. Usage: `$(<refName>.environmentResourceId)`.

**EnvironmentResourceGroupId** - Resource ID of the resource group for the environment created. Usage: `$(<refName>.environmentResourceGroupId)`.

## Delete Environment
![Delete Environment](screenshots/azure-dtl-deleteenvironment.png)
The task allows you to delete an Environment.
### Input Parameters
The task requires the following inputs.

**Azure RM Subscription** - Azure Resource Manager subscription to configure before running.

**Lab Name** - Azure DevTest Lab name.

**Environment Name or ID** - Either the name of the environment to be deleted, selected from the dropdown list; or, the fully qualified resource ID of the environment to be deleted. This can be a reference to the output variable from a call to the Create Environment task. For example, `$(<refName>.environmentResourceId)`.

For more information about creating and deleting environments, refer to [Integrate Azure DevTest Labs Environments into your VSTS continuous integration and delivery pipeline.](https://blogs.msdn.microsoft.com/devtestlab/2018/07/11/integrate-azure-devtest-labs-environments-into-your-vsts-continuous-integration-and-delivery-pipeline/)

## Populate Environment
The task allows you to deploy an ARM template to an existing DevTest Lab environment.
### Input Parameters
The task requires the following inputs.

**Azure RM Subscription** - Azure Resource Manager subscription to configure before running.

**Lab Name** - Azure DevTest Lab name.

**Repository Name** - Repository friendly name where the environment ARM templates are stored.

**Environment Name** - Name of the deployed environment.

**Source ARM template** - Name of the template to be deployed.

**Source ARM Parameter File (optional)** - The input parameters file. Use either a parameters file or the parameters.

**Source ARM Template Parameters (optional)** - The individual parameters.

**Create output variables based on the local template output.** - Create output variables resulting from the creation of the environment. See Output Variables for details.

### Output Variables
If enabled, the task can produce the following output variables.  Note that you will need to define a Reference Name (i.e. `<refName>` such as `dtl`) under the Output Variables section to correctly reference the variables in the list.