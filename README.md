<!-- BEGIN_TF_DOCS -->
# it-ae-tfmod-azure-state

This is a terraform module for initializing a terraform state backend in Azure.
By default, it creates a resource group named `terraform-state`, a storage account with a unique name, and a container named "terraform-state".

## Generation

This file was generated with the following command:

```bash
terraform-docs .
```

## Example usage

A common pattern for using this is to create a folder within your terraform IaC project for setting up your environment, such as `/environments/{env_name}/setup`, containing a `main.tf` like:

```terraform
module "state_backend" {
  source = "github.com/tamu-edu/it-ae-tfmod-azure-state?ref=v0.1.2"

  container_name = "tfstate"
  location = "southcentralus"
  resource_group_name = "your-project-tfstate-dev"
  # storage_account_name = "LeaveBlankToAutoGen"
  subscription_id = "f5358b4a-0a02-4485-8157-367fc107a27d"
  tenant_id = "68f381e3-46da-47b9-ba57-6f322b8f0da1"

  remove_secrets_from_state = false
}


# If you need to grant access to someone or something else (such as a service principal) you will need to change this code to allow this.
# This block is optional if you need to grant additional access
resource "azurerm_role_assignment" "tfstate_role_assignment" {
  scope               = module.state_backend.storage_account.tfstate.id
  role_definition_name  = "Storage Blob Data Contributor"
  principal_id        = "<object id of entity you need to have access>" 
}

output "container_name" {
  value = module.state_backend.container_name
}
output "resource_group_name" {
  value = module.state_backend.resource_group_name
}
output "storage_account_name" {
  value = module.state_backend.storage_account_name
}
```

To execute, first `az login` with an appropriately permissioned Azure account using the Azure CLI. Once logged in, run command `terraform init` within the new `terraform-state` folder. Then, run `terraform plan` to see what will be created. If satisfied with the results, run command `terraform apply`. This will create the appropriate Azure Blob Storage for holding state files for the main project. Azure Blobs are semaphore-locked from concurrent writes automatically. The state file for this remote state terraform script will be stored on the file system. Be sure to capture the results of the output (run `terraform output` to see it again) and copy it into your main Terraform stack variables. It is recommended to alter the name of the key to fit the granularity of separation of concerns that you require. By default, this will assign the role "Storage Blob Data Contributor" on the identity that is used to create the module.

> [!CAUTION]
> The terraform statefile with an Azure storage account resource will contain the initial storage account access keys. It is best practice to _disable_ access key access in favor of Entra ID authentication for your storage accounts. Do not commit the statefile until either the access keys are removed, rotated, or access keys disabled.

Consider adding the following to your parent terraform IaC project `.gitignore` file:

```sh
# .tfstate files
*.tfstate
*.tfstate.*
!environments/*/setup/*.tfstate
!environments/*/setup/*.tfstate.*
```

This will ignore the `.tfstate` file in your project, which will use remote storage, but retain the `.tfstate` for the remote tfstate infrastructure.

> [!NOTE]
> If you are using an identity that has limited access to the Azure subscription, be sure to set the value of `resource_provider_registrations` to `none`. If this is the case, you will also need to work with someone who has owner access to the subscription to enable provider registration for all the API's that you may need to use. You may also want to set the value of `create_resource_group` to `false` if the resource group has already been created for you. Additionally, if you have never done any resource provider registrations in a subscription, note that the first run of `terraform plan` or `terraform apply` may take quite some time as the provider will attempt to automatically do the provider registration.

## Example consumption of created backend storage

This module will create a file named `copy-me-and-rename-to-backend.tf` in the current directory in which it is run. You should copy this file into the directory where your Terraform is located that will consume this backend storage. Be sure to rename the key as needed. It will also create an output with this same content. Note that this is configured to use the `Azure Active Directory with Azure CLI` method for authentication as documented [here](https://developer.hashicorp.com/terraform/language/backend/azurerm#azure-active-directory-with-azure-cli). If you need a different method, you will need to edit it per that documentation.

## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.13 |
| <a name="requirement_azurerm"></a> [azurerm](#requirement\_azurerm) | =4.26.0 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_azurerm"></a> [azurerm](#provider\_azurerm) | 4.26.0 |
| <a name="provider_local"></a> [local](#provider\_local) | n/a |
| <a name="provider_null"></a> [null](#provider\_null) | 3.2.3 |
| <a name="provider_random"></a> [random](#provider\_random) | 3.7.1 |
| <a name="provider_terraform"></a> [terraform](#provider\_terraform) | n/a |

## Resources

| Name | Type |
|------|------|
| [azurerm_resource_group.tfstate](https://registry.terraform.io/providers/hashicorp/azurerm/4.26.0/docs/resources/resource_group) | resource |
| [azurerm_role_assignment.tfstate_role_assignment](https://registry.terraform.io/providers/hashicorp/azurerm/4.26.0/docs/resources/role_assignment) | resource |
| [azurerm_storage_account.tfstate](https://registry.terraform.io/providers/hashicorp/azurerm/4.26.0/docs/resources/storage_account) | resource |
| [azurerm_storage_container.tfstate](https://registry.terraform.io/providers/hashicorp/azurerm/4.26.0/docs/resources/storage_container) | resource |
| [local_file.backend_config](https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/file) | resource |
| [null_resource.sanitize_state](https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource) | resource |
| [random_string.storage_account_name](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string) | resource |
| [terraform_data.always_run](https://registry.terraform.io/providers/hashicorp/terraform/latest/docs/resources/data) | resource |
| [azurerm_client_config.current](https://registry.terraform.io/providers/hashicorp/azurerm/4.26.0/docs/data-sources/client_config) | data source |
| [azurerm_resource_group.tfstate](https://registry.terraform.io/providers/hashicorp/azurerm/4.26.0/docs/data-sources/resource_group) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_client_id"></a> [client\_id](#input\_client\_id) | The client ID to use for authenticating to Azure. Terraform authentication will overwrite this. | `string` | `null` | no |
| <a name="input_container_name"></a> [container\_name](#input\_container\_name) | The name of the storage container to use for the Terraform state | `string` | `"terraform-state"` | no |
| <a name="input_create_resource_group"></a> [create\_resource\_group](#input\_create\_resource\_group) | Set to 'false' if using a limited user without permission to create a resource group. If false, assumes the resource group already exists and will read data instead of creating. | `bool` | `true` | no |
| <a name="input_location"></a> [location](#input\_location) | The location to use for the Terraform state | `string` | `"centralus"` | no |
| <a name="input_remove_secrets_from_state"></a> [remove\_secrets\_from\_state](#input\_remove\_secrets\_from\_state) | Whether to sanitize tfstate of access keys automatically created on created resources. Actual, assigned keys remain untouched on created assets. | `bool` | `true` | no |
| <a name="input_resource_group_name"></a> [resource\_group\_name](#input\_resource\_group\_name) | The name of the resource group to use for the Terraform state | `string` | `"terraform-state"` | no |
| <a name="input_resource_provider_registrations"></a> [resource\_provider\_registrations](#input\_resource\_provider\_registrations) | Set to 'none' if using a limited user without permission to do provider registrations | `string` | `null` | no |
| <a name="input_storage_account_name"></a> [storage\_account\_name](#input\_storage\_account\_name) | The name of the storage account to use for the Terraform state. Leave blank to let Terraform manage a globally unique name to fit Azure constraints. | `string` | `null` | no |
| <a name="input_subscription_id"></a> [subscription\_id](#input\_subscription\_id) | The subscription ID to use for the Terraform state | `string` | `null` | no |
| <a name="input_tenant_id"></a> [tenant\_id](#input\_tenant\_id) | The tenant ID to use for the Terraform state | `string` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_backend_config"></a> [backend\_config](#output\_backend\_config) | Will contain a block of Terraform code that can be used to consume the created backend config. |
| <a name="output_container_name"></a> [container\_name](#output\_container\_name) | Will output the storage container name |
| <a name="output_resource_group_name"></a> [resource\_group\_name](#output\_resource\_group\_name) | Will output the name of the resource group |
| <a name="output_storage_account_name"></a> [storage\_account\_name](#output\_storage\_account\_name) | Will output the storage account name |

These output values will serve as your terraform IaC project inputs.
<!-- END_TF_DOCS -->