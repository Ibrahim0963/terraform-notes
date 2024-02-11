# Resource:
https://www.youtube.com/playlist?list=PLLc2nQDXYMHowSZ4Lkq2jnZ0gsJL3ArAw
# 1. What and Why Terraform
open source tool for provisioning and managing cloud infrastructure

### Use-Case-Senario-1:
let say, you have to create a azure virtual network, subnet within the network and a setup for azure virtual machine and azure sql-database and azure storage account. Normal you can to each one on the portal and start deploying these resources. But we we use a terraform we can just define a terraform configuration file and then submit the configuration file to the terraform executable and then terraform will automatically deploy these resources based on our create terraform configuration file. so it a quick way to automate the deployment of the resources.

### Use-Case-Senario-2:
We have an application and the code of that application is modifie, so we need to have a test enviroment to test.

Application modified its code by developer -> build test enviroment -> Deploy the app to test enviroment -> test the app -> destroy the test enviroment.

Normally you have to build the resources of the test enviroment one by one and then delete these resources one by one. This will cost time and work each time I want to build the test enviroment. Here we can use terraform to do that for us and deploy the defined resources in that terraform configuration file.


# 2. Concepts when it comes to Terraform
- Terraform configuration file: This tells Terraform how to manage the infrastructure
- Block in the configuration file: These are used to represent the configuration of an object
- The resource block is used to represent the infrastructure you want to deploy. The resource group will contain the resource type and the name that is assign to that resource block.
The resource type and the name will then become the resource identifier in the format of "resource_type.resource_name"


# 3. Terraform workflow to deploy a terraform configuration file
Build the configuration file and define the resources to build

- Step 1: terraform init - This is used to initialize the working directory that contains the Terraform configuration file.

- Step 2: terraform plan - Here terraform will create an execution plan. Here we can see what changes Terraform is going to make to our infrastructure based on the configuration file.

- Step 3: terraform apply - This will execute the actions in the Terraform plan.

- Step 4: terraform destory - This will destory your infrastructure objects based on the Terraform configuration.


# 4. Installing Terraform
- Download terraform executable file.
- add the directory to the path to run terraform from anywhere:
Advanced System Setting -> Enviroment Variables -> System Variables -> Path -> Here you can add the directory :)
- Now open cmd and type: `terraform version`

Add to vs code:
Download it and install those extension: HashiCorp Terraform, Azure Terraform.
Now create a file in vs code and save it and anyname.tf, then enjoy :)

# 5. Creating a resource group
Firstly Manuel to understand the process, then Automatically with terraform :)

Manuelly:
- Login to your Azure account from Here (url)
- See the video/photos


Automatically:
Firstly we need to authenticate on azure account then we can create our resources.
To authenticate, we will create an app object.
Remember: Azure Active Directory is our Identity Provider (Here we can define users, groups and applications)
So in order to make our user account authenticate with the azure active directory, we need to create a user identities in active directory. That user will login to our user account and get access to resource on azure active directory. You can create also an application instead of a user for authentication. This form of identity can then be embedded in our terraform configuration file. 
The Terraform configuration file will run on our local machine. You need to submit that file onto the terraform executable. Here terraform will user the embedded application/user object credentials in the configuration file to authenticate to the azure active directory. After that Terraform executable will take the command in the configuration file and issue an API call to execute these commands and create our resources based on the configuration file.

Summary:
1. Create an Application Object or a user Identity.

2. Ensure to give the permission on the application/user object, so that can create the resources
search for subscription and open it -> access contain (IAM) -> add -> add role assignment -> choose the contributor role -> select members -> add your create application/user -> Review and asign.

3. Start build the terraform configuration file
	- Setup the provider -> https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs
	- We strongly recommend using the required_providers block to set the
	- Azure Provider source and version being used
	```
		terraform {
		  required_providers {
			azurerm = {
			  source  = "hashicorp/azurerm"
			  version = "2.93.0"
			}
		  }
		}
	```
	
	```
	- Configure the Microsoft Azure Provider
		provider "azurerm" {
		  subscription_id	= ""
		  client_id			= ""
		  client_scret		= ""
		  tenant_id			= ""
		  features {}
		}
	```

But from where to get these credentials? follow step 3

4. embedde the credentials into the terraform configuration file
In the overview of the create application, you will find:
	- client_id, which is the application ID
	- tenant_id, which is the directory (tenant) ID
	- client_secret, which is the client credentials (Certificates & Secrets) -> New Client Secret -> add the values -> copy the value ID (the client_secret)
	- subscription_id for this search for subscriptions and copy the subscription ID :)

5. Start write terraform to create resources
```
resource "$resource_type" "block_name" { // the name of this block. Known in this file only.
	name="app-grp" // the name of the created resource group
	location="West Europe"
}
```
##example##
```
resource "azurerm_resource_group" "app_grp" { // the name of this block. Known in this file only.
	name="app-grp" // the name of the created resource group
	location="West Europe"
}
```

6. save the file and start executing the following commands
```
terraform init // will download the terraform provider and install the azurerm provider in our case.
// this will create .terraform and .terraform.lock.hcl

terraform plan -out main.tfplan // terraform here will contact the azure and go to our azure subscription using the application/user object credentails to create a plan and check if our resource already exists

terraform apply main.tfplan // terraform here will perform the actions in the configuration file.
```
Well done! 



# 6. Creating an Azure Storage Account 
Manully:
search for storage account -> create -> fill the values (Performance:Standard, Redundancy:LRS) -> review + create
Remember: Our inputs to create the storage account -> Resource group, location, name, performance, redundancy

Automatically:
```
resource "azurerm_storage_account" "storage_account" {
  name                     			 = "storage-account-ibrahim"
  resource_group_name      = azurerm_resource_group.example.name
  location                 		    = azurerm_resource_group.example.location
  account_tier            		= "Standard"
  account_replication_type = "LRS"

  tags = {
	environment = "Enviroment"
  }
}
```
then execute the terraform commands.




# 7. Terraform state
How to create a container and upload an object onto an azure storage account by terraform
if you want to upload objects on the storage account, you need to create a container service
Note: by default the allow blob public access is disable, so you need to enable it.
storage account -> configuration -> allow blob public access

storage account -> container -> +container 


# 8.  Creating a container and a blob
Storage Account -> Container -> created container -> upload 

```
resource "azurerm_storage_container" "data" {
	name = "data" // name of the container you want to create
	storage_account_name = "terraformstorage" // name of the storage account
	container_access_type = "private" // private or blob or container
}
```

``` // add files to the container in the storage account
resource "azurerm_storage_blob" "files" {
	name = "file.txt"
	storage_account_name = "terraformstorage"
	storage_container_name = "container_name"
	type = "Block"
	source = "file.txt"
}
```

Note: In the container, we can change access level


# 9. Dependencies between resources
Let say, we want to create a blob. We need for that a container, then add the blob and to create a container we need to create a storage account. so blob is depended on the container and the container depended on the storage account. Terraform will not execute the command in a particular order, so we have to tell the terraform configuration file to create firstly the storage account, then the container, then the blob.
To do that we can add a depends reference to tell the terraform, only if the depend exists or created, then start create this also
```
	depends_on = [
		$resource_type.$resource_name
	]
```

``` // add files to the container in the storage account
resource "azurerm_storage_blob" "files" {
	name = "file.txt"
	storage_account_name = "terraformstorage"
	storage_container_name = "container_name"
	type = "Block"
	source = "file.txt"+
	depends_on = [ 
		azurerm_storage_container.resourceName
	]
}
```
Here we are telling the terraform, firstly check if the azure storage container exists or created, then create the storage blob. if not, do not create the storage blob :)


# 10. Destroying the resources
We apply this command to delete any defined resource in the terraform configuration file
```
terraform destroy
```
Additional, if we only delete the resource group, azure will delete all other resource in this resource group.


# 11. Variables
```
variable "storage_account_name" {
	type = "string"
	description = "This is a storage account variables"
	default = "Any Value"
}
```
Here, when you run the terraform plan command, you need to enter the values for the variables.

to reference the variables in the terraform code:
```
name = var.storage_account_name
```

##locals##
locals will be only used within the terraform configuration file:
```
locals {
	resouce_group="anyname"
	location="locations"
}
```

To reference the local variables
```
name = locals.resouce_group
```

# 12. Creating an Azure virtual network
Manuelly: (from the portal)
- search for virtual network in the search bar in azure portal and open it 
- click on the +create button 
- fill the values


Autoamtically: 
```
resource "azurerm_virtual_network" "example" {
  name                = "sandbox1_network"
  location            = locals.location // or you can reference any resources by: azurerm_resource_group.sandbox_rg.location or write diretly the location as a string :)
  resource_group_name = azurerm_resource_group.sandbox_rg.name
  address_space       = ["10.0.0.0/16"]
  dns_servers         = ["10.0.0.4", "10.0.0.5"]

  subnet {
    name           = "subnet1"
    address_prefix = "10.0.1.0/24"
  }
}
```


# 13. Creating an Azure virtual machine









