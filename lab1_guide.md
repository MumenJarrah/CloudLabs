# Lab 03 - Deploy Azure Container Instances

## Lab overview

Azure Container Instances enables exposing your container groups directly to the internet with an IP address and a fully qualified domain name (FQDN). When you create a container instance, you can specify a custom DNS name label so your application is reachable. Azure Container Instances offers the fastest and simplest way to run a container in Azure, without having to manage any virtual machines and without having to adopt a higher-level service.

In this walkthrough, we create, configure, and deploy a Docker container by using Azure Container Instances (ACI) in the Azure Portal. The container is a Welcome to ACI web application that displays a static HTML page.

## Lab objectives

In this lab, you will complete the following tasks:

+ Task 1: Create a container instance
+ Task 2: Verify deployment of the container instance

## Estimated timing: 15 minutes

## Architecture diagram

![](../images/az900lab03.PNG) 

### Task 1: Create a container instance

In this task, we will create a new container instance for the web application. 

1. On Azure Portal page, in **Search resources, services and docs (G+/)** box at the top of the portal, enter **Container instances (1)**, and then select **Container instances (2)** under services.

   ![](../images/lab3-image1.png)
   
1. On **Container instances** blade, click **+ Create**. 

1. On the **basics** tab. Provide the following basic details for creating a new container instance then click **Next : Networking >**.

	| Setting| Value|
	|----|----|
	| Subscription | **Choose your subscription** |
	| Resource group | **myRGContainer-<inject key="DeploymentID" enableCopy="false" />** |
	| Container name| **mycontainer**|
	| Region | **<inject key="Region" enableCopy="false"/>** |
	| Image source| **Other registry**|
	| Image type| **Public**|
	| Image| **mcr.microsoft.com/azuredocs/aci-helloworld**|
	| OS type| **Linux** |
	| Size| ***Leave at the default***|

	
1. On **Networking** tab . Specify the following and leave all other settings at their default values and click **Review + create (2)**.

    | Setting| Value|
    |--|--|
    | DNS name label| **mycontainerdns<inject key="DeploymentID" enableCopy="false" /> (1)** |
    |||

    ![](../images/lab3-image2.png)
   
	**Note**: Your container will be publicly reachable at dns-name-label.region.azurecontainer.io. If you receive a **DNS name label not available** error message following the deployment.

1. Click **Create** to create the container instance. 

1. Monitor the deployment page and the **Notifications** page. 

1. While you wait you may be interested in viewing the [sample code behind this simple application](https://github.com/Azure-Samples/aci-helloworld). Browse the \app folder. 

### Task 2: Verify deployment of the container instance

In this task, we verify that the container instance is running by ensuring that the welcome page displays.

1. After the deployment is complete, click the **Go to resource** button.

   ![](../images/lab3-image3.png)

1. On the **Overview** blade of **mycontainer**, ensure your container **Status** is **Running**.

    ![](../images/lab3-image6.png)

1. Locate and copy the **Fully Qualified Domain Name (FQDN)**.

    ![](../images/lab3-image4.png)

1. Paste the container's FQDN into the new browser tab and press **Enter**. The Welcome page should display.

   **Note**: It might take 3 - 5 minutes to load the page.
 
   ![](../images/lab3-image5.png)
	
   **Note**: You could also use the container IP address in your browser.

   <validation step="f87beab3-3bb6-467e-839f-c821f172a4b8" />
   
    > **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
    > - Click Lab Validation tab located at the upper right corner of the lab guide section and navigate to the Lab Validation tab.
    > - Hit the Validate button for the corresponding task.
    > - If you receive a success message, you can proceed to the next task. If not, carefully read the error message and retry the step, following the instructions in the lab guide.
    > - If you need any assistance, please contact us at labs-support@spektrasystems.com. We are available 24/7 to help you out.
    
### Review
In this lab, you have completed:
- Create a container instance
- Verify deployment of the container instance

## Reference links

- https://learn.microsoft.com/en-us/azure/container-instances/container-instances-overview

- https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-portal
  
## You have successfully completed this lab.

