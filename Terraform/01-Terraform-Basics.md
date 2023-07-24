# Terraform Basics

## HCL - HashiCorp Configuration Language

```
[block-name] "[resource-type]" "[resource-name]" {
    [argument-name] = "[argument-value]"
    ...
}
```

```terraform
# main.tf
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content = "We love pets!"
}
```

```shell
$ terraform init
$ terraform plan
$ terraform apply
$ terraform show
```

## Using Terraform Providers

After we write a terraform configuration file, the first thing to do is to initialize the directory with the 
`terraform init` command. When we run the initialization command, terraform downloads and install plugins for the 
providers used within the configuration. This can be plugins for cloud providers such as aws, gcp, azure, or something 
as simple as the local provider that we use to create a local file type resource. Terraform uses a plugin based 
architecture to work with hundreds of such infrastructure platforms.

Terraform providers are distributed by Hashicorp and are publicly available in the terraform registry at 
(registry.terraform.io)[registry.terraform.io]. There are three tiers of providers:
- **Official providers:** these are owned and maintained by hashicorp and include major cloud provider like aws, gcp or 
azure.
- **Partner providers:** these are owned and maintained by a third party technology company that has gon through a 
- partner provider process with hashicorp. Some examples are bigip, heroku, or digital ocean.
- **Community providers:** these are published and maintained by individual contributors of the hashicorp community.

When the `terraform init` command is ran, it shows the version of the plugins that have been installed.

```shell
$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding latest version of hashicorp/local...
- Installing hashicorp/local v2.0.0...
- Installed hashicorp/local v2.0.0 (signed by HashiCorp)
The following providers do not have any version constraints in
configuration,
so the latest version was installed.
To prevent automatic upgrades to new major versions that may
contain breaking
changes, we recommend adding version constraints in a
required_providers block
in your configuration, with the constraint strings suggested
below.
* hashicorp/local: version = "~> 2.0.0"
Terraform has been successfully initialized!
```

The `terraform init`command can be run as many times as needed without impacting the actual infra that is deployed. 
The plugins are downloaded into a hidden directory called `.terraform/plugins` in the working directory containing the 
configuration files.

## Configuration Directory

- `main.tf`: main configuration file containing resource definition
- `variables.tf`: contains variable declaration
- `outputs.tf`: contains outputs from resources
- `providers.tf`: contains provider definition

## Multiple Providers

Terraform supports the use of multiple providers within the same configuration.

```terraform
# main.tf
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "We love pets!"
}

resource "random_pet" "my-pet" {
    prefix = "Mrs"
    separator = "."
    length = "1"
}
```

## Define Input Variables

```terraform
# variables.tf
variable "filename" {
    default = "/root/pets.txt"
    type = string
    description = "the path of local file"
}

variable "content" {
    default = "We love pets!"
    type = string
    description = "Lorem ipsum"
}

variable "prefix" {
    default = "Mrs"
    type = string
    description = "Lorem ipsum"
}

variable "separator" {
    default = "."
    type = string
    description = "Lorem ipsum"
}

variable "length" {
    default = "1"
    type = number
    description = "Lorem ipsum"
}
```

```terraform
# main.tf
resource "local_file" "pet" {
    filename = "/root/pets.txt"
    content = "We love pets!"
}
resource "random_pet" "my-pet" {
    prefix = "Mrs"
    separator = "."
    length = "1"
}
```

## Understanding the Variable Block

```
variable "[variable-name]" {
    default = "[default-value]" # Optional. If not defined, it is set to "any" by default.
    type = "[variable-type]"
    description = "[variable-description]"
}
```

Terraform supports multiple variable types:

- `String`: "/root/pets.txt"
- `number`: 1
- `bool`: true/false
- `any`: Default value
- `list/set`:
```terraform
# variable.tf
variable "prefix" {
  default = ["Mr", "Mrs", "Sir"]
  type = list(string) # We can also use set. But we have to make sure there's no duplicate values in the list.
}
```

```terraform
# main.tf
resource "random_pet" "my-pet" {
  prefix = var.prefix[0]
}
```
- `map`:
```terraform
# variable.tf
variable "file-content" {
  default = { 
         "statement1": "We love pet!",
         "statement2": "We love animals!"
  }
  type = map(string)
}
```

```terraform
# main.tf
resource local_file my-pet {
  filename = "/root/pets.txt"
  content = var.file-content["statement1"]
}
```
- `object`: Complex data structure
```terraform
# variable.tf
variable "file" {
  type = object({
    name = string
    location = string
  })
  default = {
    name = "pet.txt"
    location = "/root/pets.txt"
  }
}
```
- `tuple`:
```terraform
# variable.tf
variable kitty {
  type = tuple([string, number, bool])
  default = tuple(["cat", 7, false])
}
```

## Using Terraform Variables

So far, we've created input variables in terraform and assigned default values to them. This is just one of the ways to 
pass values to variables.The default argument in variable block is optional. Therefore, we can the variable block looks 
like this:

```terraform
# variable.tf
variable variable-name {
  
}
```

But what would happen if we run terraform commands now? When we run terraform apply command, we will be prompted to 
enter values for each variable in an interactive mode. If we don't to supply values in an interactive mode, we can also 
make use of command line flag like this: `terraform apply -var "key1=value1" -var "key2=value2"`. We can also make use of
 environment variables formatted like this: `TF_VAR_[variable-name]`. Finally, we can also make use of variable 
definition file like this:

```terraform
# terraform.tfvars
filename = "/root/pets.txt"
content = "We love pets!"
```

The variable definition files are automatically loaded by terraform. They can be named `terraform.tfvars`, 
`terraform.tfvars.json`, `*.auto.tfvars` or `*.auto.tfvars.json`.

If we use any other filename, we will have to pass it with command line flag like this: 
`terraform apply -var-file variable.tfvars`.

**Variable Definition Precedence**

1) Environment variables
2) `terraform.tfvars`
3) `*.auto.tfvars` (alphabetical order)
4) `-var` or `-var-file`(command line flag)

## Resources Attributes

Let's say we have two resources in our terraform configuration and we want to use the output of one resource as the input
 of the other one.

```terraform
# main.tf
resource random_pet my-pet {
 prefix = var.prefix
 separator = var.separator
 length = var.length
}
# If we look at the attribute reference, the random_pet resource outputs an attribute named id.

resource local_file pet {
 filename = var.filename
 content = "My favorite pet is ${random_pet.my-pet.id}"
}
```

# Resource Dependencies

We used implicit dependencies the above example. However, we can also use explicit dependencies in terraform like this:

```terraform
# main.tf
resource random_pet my-pet {
 prefix = var.prefix
 separator = var.separator
 length = var.length
}

resource local_file pet {
 filename = var.filename
 content = "My favorite pet is ${random_pet.my-pet.id}"
 depends_on = [random_pet.my-pet]
}
```

This ensure that the `local_file` resource is created only after the `random_pet` resource is created.

## Output Variables

```terraform
output [variable-name] {
  value = [variable-value]
  [arguments]
}
```

```terraform
# main.tf
resource random_pet my-pet {
 prefix = var.prefix
 separator = var.separator
 length = var.length
}

resource local_file pet {
 filename = var.filename
 content = "My favorite pet is ${random_pet.my-pet.id}"
 depends_on = [random_pet.my-pet]
}

output pet-name {
 value = random_pet.my-pet.id
 description = "Record the value of pet ID generated by the random_pet resource"
}
```

When run terraform apply, the output variable will be printed on the screen. We can also use the command 
`terraform output <variable-name|optional>` to print the value of the output variables.