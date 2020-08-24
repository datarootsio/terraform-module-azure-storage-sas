# Terraform module for Azure Storage SAS tokens

Facilitates the use of rotating SAS tokens in Terraform modules.

When you simply use `azurerm_storage_account_sas` or `azurerm_storage_account_blob_container_sas` with the `timestamp()` and `timeadd()` functions, you'll notice that the tokens are updated on each call to `terraform apply`.
This module avoids that and still allows you to rotate the SAS token. You simply tell the module how often the SAS token should rotate.

Note that you should run `terraform apply` to actually rotate the token when needed.

[![maintained by dataroots](https://img.shields.io/badge/maintained%20by-dataroots-%2300b189)](https://dataroots.io)
[![Terraform 0.13](https://img.shields.io/badge/terraform-0.13-%23623CE4)](https://www.terraform.io)
[![Terraform Registry](https://img.shields.io/badge/terraform-registry-%23623CE4)](https://registry.terraform.io/modules/datarootsio/azure-storage-sas/module)
[![build](https://github.com/datarootsio/terraform-module-azure-storage-sas/workflows/build/badge.svg)](https://github.com/datarootsio/terraform-module-azure-storage-sas/actions?query=workflow%3Abuild)

## Usage

The example below creates a resource group, a storage account, a blob container and a SAS token. The token rotates yearly and is valid for 72h after the next rotation point. The token has all permissions in the storage container.

```hcl-terraform
resource "azurerm_resource_group" "rg" {
  location = "eastus2"
  name     = "rg"
}

resource "azurerm_storage_account" "sa" {
  account_replication_type = "LRS"
  account_tier             = "Standard"
  location                 = "eastus2"
  name                     = "sa"
  resource_group_name      = azurerm_resource_group.rg.name
}

resource "azurerm_storage_container" "container" {
  name                 = "container"
  storage_account_name = azurerm_storage_account.sa.name
}

module "storage-sas" {
  source                 = "datarootsio/azure-storage-sas/module"
  rotation_years         = 1
  rotation_margin        = "72h"
  storage_account_name   = azurerm_storage_account.sa.name
  storage_container_name = azurerm_storage_container.container.name
  resource_group_name    = azurerm_resource_group.rg.name
}

output "sas" {
  value       = module.storage-sas.sas
}
```

All possible options are documented in the Terraform Registry.

## License

MIT license. Please see [LICENSE](LICENSE.md) for details.