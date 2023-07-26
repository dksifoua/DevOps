Here are some benefits of terraform state:
- Mapping configuration to real world
- Tracking metadata
- Performance
- Collaboration

So far, we have been working with the terraform state file that is stored locally. Let's store terraform state to S3.

```terraform
terraform {
 backend "s3" {
   bucket = "bucket-name"
   key = "terraform/terraform.tfstate"
   region = "us-east-1"
   dynamodb_table = "" # for state locking
 }
}
```

**Terraform State Commands**

- `terraform state show [resource-address]`: List the attributes of a resource
- `terraform state list`: list all resources managed by terraform. It prints only the resource address.
- `terraform state mv [source] [destination]`
- `terraform state pull`: download and display the remote state.
- `terraform state rm  [resource-address]`: Remove a resource from terraform management (not from real world infra).
