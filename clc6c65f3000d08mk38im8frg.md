# Azure Terrafy: Import and Manage Existing Azure Resources with Terraform

As an Azure user, you may be looking for a way to manage your infrastructure with the power of Terraform. However, getting started with Terraform can be challenging, especially if you have existing resources that you want to include in your configuration. That's where Azure Terrafy comes in. Azure Terrafy is a tool that makes it easy to import your existing Azure resources into Terraform modules. In this blog post, I will introduce you to Azure Terrafy and show you how it can streamline your Azure resource management process. By the end, you will have a better understanding of how Terrafy can help you use Terraform to manage your infrastructure with ease.

[Azure Terrafy](https://github.com/Azure/aztfy) is a tool that makes it easy to import your existing Azure resources into Terraform modules. Suppose you're an Azure user looking to manage your infrastructure with the power of Terraform. In that case, Azure Terrafy can save you time and effort by automating the process of incorporating your existing resources into your Terraform configuration. This is especially useful for those who have a "brownfield" environment, where their infrastructure already has a number of existing resources that need to be brought under the management of Terraform. It can save you a lot of time and effort. Without Terrafy, you would need to manually create a Terraform configuration file for each resource you want to manage. This can be tedious and error-prone, especially if you have many resources.

In addition to saving time and effort, using Azure Terrafy can also help you ensure that your Terraform configuration is accurate and up-to-date. By importing your existing resources into Terraform, you can ensure that your configuration reflects the current state of your resources rather than relying on potentially outdated documentation or manual configuration. This can help you avoid errors and ensure your infrastructure is managed effectively.

%[https://github.com/Azure/aztfy] 

### Features and benefits of using Terrafy:

* Automates the process of importing existing Azure resources into Terraform modules
    
* Saves time and effort compared to manual resource management
    
* Helps ensure that your Terraform configuration is accurate and up-to-date
    
* Works with both Azure Resource Manager and Azure Classic resources
    
* Can be easily integrated into existing Terraform workflows
    

### How Azure Terrafy fits into the overall Azure and Terraform landscape:

* Azure Terrafy is a tool specifically designed to help manage Azure resources in Terraform
    
* It works alongside Azure Resource Manager to discover and import resources into Terraform modules
    
* Azure Terrafy is just one piece of the puzzle regarding managing Azure infrastructure with Terraform. Other tools and technologies, such as Azure DevOps and Azure Functions, can also be used in conjunction with Terrafy to create a comprehensive infrastructure management solution.
    

### Prerequisites

* An Azure account and subscription
    
* Terraform installed on your machine
    
* The Azure CLI installed on your machine
    
* A configured Azure CLI profile with the necessary permissions to read resources in your Azure subscription
    
* Install `aztfy` from [GitHub Releases](https://github.com/Azure/aztfy/releases) - pre-compiled binaries on macOS, Windows & Linux platforms
    

### Importing Existing Resources

1. Run the `aztfy` command to create a new Terraform configuration file on a single resource, resource group or query for the list of resources. You can pass the —non-interactive option if you want to run in non-interactive mode. Here dapr-talk is the resource group name.
    
    ```bash
    **aztfy rg --non-interactive dapr-talk
    // or
    aztfy res <resource_id>**
    ```
    
2. Review the generated Terraform configuration file to ensure that it accurately reflects the state of your Azure resources
    
    1. You may need to modify or expose certain configurations through terraform variables/sensitive variables.
        
3. Run the `terraform plan` command to preview the changes that will be made to your resources when you apply the Terraform configuration
    
    ```bash
    ➜  terraform plan
    
    azurerm_resource_group.res-0: Refreshing state... [id=/subscriptions/xxxx/resourceGroups/dapr-talk]
    azurerm_service_plan.res-1: Refreshing state... [id=/subscriptions/xxxx/resourceGroups/dapr-talk/providers/Microsoft.Web/serverfarms/ASP-daprtalk-814c]
    azurerm_application_insights.res-8: Refreshing state... [id=/subscriptions/xxxx/resourceGroups/dapr-talk/providers/Microsoft.Insights/components/webappdapr]
    azurerm_windows_web_app.res-2: Refreshing state... [id=/subscriptions/xxxx/resourceGroups/dapr-talk/providers/Microsoft.Web/sites/webappdapr]
    azurerm_app_service_custom_hostname_binding.res-6: Refreshing state... [id=/subscriptions/xxxx/resourceGroups/dapr-talk/providers/Microsoft.Web/sites/webappdapr/hostNameBindings/webappdapr.azurewebsites.net]
    
    No changes. Your infrastructure matches the configuration.
    ```
    
4. If the plan looks correct, run the `terraform apply` the command to apply the Terraform configuration and bring your Azure resources under the management of Terraform.
    
    ```bash
    terraform apply
    ```
    

Here are a few tips and best practices to keep in mind when using Terrafy to import your existing resources into Terraform:

* Be sure to review the generated Terraform configuration file carefully before applying it. This will help you ensure that the configuration accurately reflects the state of your resources and avoid any errors or unintended consequences.
    
* If you have many resources, it may take some time for Azure Terrafy to scan and generate the configuration file. Be patient and allow the process to complete before applying the configuration.
    
* If you have resources that you don't want to include in your Terraform configuration, you can use the `aztfy res or aztfy query` command to import specific resources individually. This can be helpful if you only want to manage a subset of your resources with Terraform.
    
* You must create terraform resources manually if there are not-supported / skipped resources.
    
* Don't forget to run the `terraform plan` command before applying your configuration. This will help you identify potential issues or conflicts before they become a problem.
    

By following these steps and best practices, you can use Azure Terrafy to easily import your existing Azure resources into Terraform modules and start managing them with ease.

### Limitations

Here are some limitations of Azure Terrafy to consider:

1. Only works with Azure resources: Terrafy cannot import resources from other cloud providers.
    
2. It may not be suitable for highly complex or customized infrastructure: Azure Terrafy does not support all resources. The resources which are not supported have to be done manually.
    
3. Limited context: The review is necessary to convert the sensitive configurations to terraform-managed variables.
    

### Conclusion

In conclusion, I have found Azure Terrafy to be a valuable tool for managing Azure resources in Terraform. It has saved me time and effort compared to manual resource management, and its integration with Azure and Terraform has made it easy to use in various infrastructure management scenarios. However, like any tool, it has its limitations, such as only working with Azure resources and potentially complex for more customized infrastructure.

Overall, I recommend giving Azure Terrafy a try if you are looking for a tool to help manage your Azure resources with Terraform. Just be sure to consider its limitations and whether it fits the needs of your specific infrastructure. By following the steps outlined in this blog post and keeping in mind best practices like reviewing the generated code before applying it, you can use Azure Terrafy to manage your Azure resources with Terraform effectively.

I'm Siva - Director, DevOps & Principal Architect at [**Computer Enterprises Inc**](https://www.ceiamerica.com/) from Orlando. I'm an AWS Community builder. I write blogs and tutorials about Cloud, Containers, IoT, and DevOps. If you are interested, please follow me @[@ksivamuthu](@ksivamuthu) on Twitter or check out my blogs at [**sivamuthukumar.com**](http://sivamuthukumar.com)