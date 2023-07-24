## Introduction to Terraform State

When we run terraform apply command, terraform creates a state file `terraform.tfstate`. The state file is a json data 
structure that maps real world infrastructure resources to the resource definitions in the configuration files. It contains 
every little detail pertaining to the infrastructure that was created by terraform, and it uses it as a single source of
 truth when using commands such as terraform plan and apply.

If we make a change in the configuration files now and re-run terraform plan or apply command, terraform by default 
refreshes the state again and compares it against the configuration file. As a result, terraform knows if a resource need 
to be recreated and then update the state file as well.

## Purpose of State

We already saw how TerraForm used state file to map the resource configuration to the real world infrastructure. This 
mapping allows TerraForm to create execution plans when a drift, as identified between the resource configuration files 
and the state. Hence, a state file can be considered to be a blueprint of all the resources that are terraform manages 
out there in the real world.

When TerraForm creates a resource, it records their identity and their state. Besides the mapping between resources and 
the configuration and the real world, the state file also tracks metadata details such as resource dependencies.

One other benefit of using state **performance**. When dealing with handful number of resources, it may be feasible for 
terraform to reconcile state with the real word infrastructure after every single terraform command like plan or apply. 
But in real word, terraform would manage hundreds and thousands of such resources. And when these resources are 
distributed to multiple providers, it's not possible to reconcile state for every terraform operation. This is because 
it will take several seconds to minutes in some cases from terraform to phase details about every single resource from 
all the providers.

This may prove to be too slow in such cases that their form state can be used as their record of 
truth without having to reconcile. This would improve the performance significantly. Terraform stores a cache of 
attribute values for all resources in the state, and we can specifically make terraform to refer to the state file alone 
while running commands and bypass having to refresh state every time. To do this, we can use command 
`terraform apply --refresh=false`.

The final benefit of state is **collaboration** when working as a team. When working as a team, as we have seen in the 
previous lectures, the TerraForm State file is stored in the same configuration directory in a file `terraform.tfsate`. In 
a normal scenario, this means that the state file resides in the folder or a directory in the end users laptop. This is 
all right when starting off at their full learning and implementing small projects individually.

However, this is far from ideal when working as a team. Every user in the team should always have the latest data before
running TerraForm and make sure that nobody else wants TerraForm at the same time. Failure to do so can result in 
unpredictable errors as a consequence. In such a scenario, it is highly recommended to save the state of home state file 
in a remote data store rather than to rely on a local copy. This allows the state to be shared between all members of 
the team securely. Examples of remote state stores are S3, HashiCops Console, and TerraForm Cloud.

## Terraform State Considerations

State is the single source of truth for TerraForm to understand what is deployed in the real world. However, there are a
few things to keep note of when working with state.

State is a non-optional feature in TerraForm. However, there are a few considerations.

First one is that the state file contains sensitive information within it, it contains every little detail about our 
infrastructure. For example, a state file for an EC2 instance, consists of all the attributes for which the machine that 
is provision such as the allocated CPU's, the memory operating system or the image used type and size of desks, etc. It 
also stores information such as the IP address allocated.

For resources such as database's, the state may also store initial passwords when using local state. The state is stored
in plain text JSON files. And as we can see, this information can be classified as sensitive information. And as a result,
 we need to make sure that the state file is always stored in a single storage.

So we have two kinds of paths in our configuration directory, data from state file that stores state of the infrastructure
and those from configuration files that we use to provision and manage infrastructure.

When working as a team, it is considered a best practice to store their form configuration files and distributed version 
control systems such as GitHub get or Bitbucket.

However, owing to the sensitive nature of the state file, it is not recommended to store them in git repositories. 
Instead, stored the state and remote backend systems such as NWS, S3, Google Cloud Storage, Azure Storage, TerraForm 
Cloud, etc.