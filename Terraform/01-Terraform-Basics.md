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

After we write a terraform configuration file, the first thing to do is to initialize the directory with the `terraform init` command. When we run the initialization command, terraform downloads and install plugins for the providers used within the configuration. This can be plugins for cloud providers such as aws, gcp, azure, or something as simple as the local provider that we use to create a local file type resource. Terraform uses a plugin based architecture to work with hundreds of such infrastructure platforms.

Terraform providers are distributed by Hashicorp and are publicly available in the terraform registry at (registry.terraform.io)[registry.terraform.io]. There are three tiers of providers:
- **Official providers:** these are owned and maintained by hashicorp and include major cloud provider like aws, gcp or azure.
- **Partner providers:** these are owned and maintained by a third party technology company that has gon through a partner provider process with hashicorp. Some examples are bigip, heroku, or digital ocean.
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

The `terraform init`command can be run as many times as needed without impacting the actual infra that is deployed. The plugins are downloaded into a hidden directory called `.terraform/plugins` in the working directory containing the configuration files.

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

