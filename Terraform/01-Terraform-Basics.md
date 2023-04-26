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