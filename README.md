
<!-- BEGIN_TF_DOCS -->

# AWS Organization Module

## Providers

| Name | Version |
|------|---------|
| <a name="provider_aws"></a> [aws](#provider\_aws) | >= 6.0 |


## Reference Example

```hcl
  #provider "aws" {}

module "aws_organization" {
  source = "../.."

  # aws organization
  feature_set                   = "ALL"
  aws_service_access_principals = ["sso.amazonaws.com"]
  enabled_policy_types          = ["SERVICE_CONTROL_POLICY"]

  # root policies
  root_unit_policies = ["region_control"]

  # policies
  policies = [
    {
      name : "region_control",
      template_file : "./policies/scps/scp_region_lock.json",
    },
    {
      name : "deny_all",
      template_file : "./policies/scps/deny_all.json",
    }
  ]

  # organizational units
  organizational_units = [
    {
      name : "CoreOU",
      policies : [],
      children : [
        {
          name : "DevelopmentOU",
          policies : ["dev_control_access"],
          children : []
        },
        {
          name : "StageOU",
          policies : [],
          children : []
        },
        {
          name : "ProductionOU",
          policies : [],
          children : []
        }
      ]
    },
    {
      name : "SandboxOU",
      policies : ["deny_all"],
      children : []
    }
  ]

  # accounts
  // we can define parent unit on two ways
  // the first one is by exactly parent_id which is unit ID or Root ID for the account
  // the second way is to define parent_path, this means that we will define a node in the hierarchical structure through path
  // example1: parent_path="CoreOU/DevelopmentOU" to put account in DevelopmentOU which is placed in CoreOU
  // example2: parent_path="" or parent_id="" that means in root unit
  accounts = [
    {
      name : "ExampleOfAccountInRootOnit",
      email : "test+root@test.com",
      parent_id : ""
    },
    # {
    #   name : "AccountBYID",
    #   email : "test+dev@test.com",
    #   parent_id : "B1234323" # specified by concrete id, you need to know the ID of your OU or root account, this value is example only
    # },
    {
      name : "Development",
      email : "test+dev@test.com",
      parent_path : "CoreOU/DevelopmentOU"
    },
    {
      name : "Stage",
      email : "test+stage@test.com",
      parent_path : "CoreOU/StageOU",
      policies : ["deny_all"]
    },
    {
      name : "Pruduction",
      email : "test+shared@test.com",
      parent_path : "CoreOU/ProductionOU"
    },
    {
      name : "Shared",
      email : "test+shared@test.com",
      parent_path : "CoreOU"
    },
    {
      name : "Sandbox01",
      email : "test+sandbox01@test.com",
      parent_path : "SandboxOU"
    },
    {
      name : "Sandbox02",
      email : "test+sandbox01@test.com",
      parent_path : "SandboxOU"
    },
    {
      name : "Sandbox03",
      email : "test+sandbox01@test.com",
      parent_path : "SandboxOU"
    }
  ]
}
```

  ## Example with yaml
```hcl
  #provider "aws" {}

module "aws_organization" {
  source = "../.."

  # variables are configured via yaml files inside "conf" folder
}
```


## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_accounts"></a> [accounts](#input\_accounts) | The list of accounts | <pre>list(object({<br/>    name                       = string,<br/>    email                      = string,<br/>    parent_id                  = optional(string)<br/>    parent_path                = optional(string)<br/>    role_name                  = optional(string)<br/>    close_on_deletion          = optional(string)<br/>    create_govcloud            = optional(string)<br/>    iam_user_access_to_billing = optional(string)<br/>    policies                   = optional(list(string))<br/>  }))</pre> | `[]` | no |
| <a name="input_aws_service_access_principals"></a> [aws\_service\_access\_principals](#input\_aws\_service\_access\_principals) | A list of AWS service principals for which you want to enable integration with your organization. | `list(string)` | `[]` | no |
| <a name="input_enabled_policy_types"></a> [enabled\_policy\_types](#input\_enabled\_policy\_types) | List of organization policy types to enable in the organization. Organization must have feature\_set set to ALL. Valid policy types: AISERVICES\_OPT\_OUT\_POLICY, BACKUP\_POLICY, SERVICE\_CONTROL\_POLICY, and TAG\_POLICY | `list(string)` | `[]` | no |
| <a name="input_feature_set"></a> [feature\_set](#input\_feature\_set) | The feature set of the organization. One of 'ALL' or 'CONSOLIDATED\_BILLING'. (default: ALL) | `string` | `"ALL"` | no |
| <a name="input_import_mode"></a> [import\_mode](#input\_import\_mode) | Whether import mode is active, if true, resources can be imported smoothly (In that case, it is not possible to create resources safely, because outputs won't have valid outputs and all resources will be created in the root unit) WARNING: use import\_mode only in case when you want to import resources, after importing, set import\_mode to false or remove it | `bool` | `false` | no |
| <a name="input_organizational_units"></a> [organizational\_units](#input\_organizational\_units) | The tree of organizational units to construct. Defaults to an empty tree. You must take care of the list format, which is explained in the Readme | `any` | `[]` | no |
| <a name="input_policies"></a> [policies](#input\_policies) | The list of policies | <pre>list(object({<br/>    name          = string,<br/>    template_file = string,<br/>    type          = optional(string)<br/>    skip_destroy  = optional(bool)<br/>    description   = optional(string)<br/>  }))</pre> | `[]` | no |
| <a name="input_root_unit_policies"></a> [root\_unit\_policies](#input\_root\_unit\_policies) | The list of policies for root unit | `list(string)` | `[]` | no |


## Outputs

| Name | Description |
|------|-------------|
| <a name="output_accounts"></a> [accounts](#output\_accounts) | List of accounts |
| <a name="output_organization_arn"></a> [organization\_arn](#output\_organization\_arn) | ARN of the organization |
| <a name="output_organization_id"></a> [organization\_id](#output\_organization\_id) | Identifier of the organization |
| <a name="output_organizational_units"></a> [organizational\_units](#output\_organizational\_units) | List of organization units which contain the root unit |
| <a name="output_policies"></a> [policies](#output\_policies) | List of policies |



<!-- END_TF_DOCS -->