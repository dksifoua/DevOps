## Terraform Commands

- `terraform validate`: validates the configuration files
- `terraform format`: formats the code in the configuration files to improve its readability.
- `terraform show`: displays the current state of the resources managed by terraform, including all the attributes for 
those resources.
- `terraform providers`: gets all providers used in the configuration files.
- `terraform output <variable-name|optional>`: prints all the output variables.
- `terraform refresh`: Reconcile infra with state file. Update the state file from the real world infrastructure.
- `terraform graph`: creates a visual representation of resources dependencies. `terraform graph | dot -Tsvg > graph.svg`

## Mutable vs Immutable Infrastructure

When terraform updates a resource, such as updating a permission of a local file, it first destroys it and re-creates it
 with the new permission (immutable). Terraform doesn't update resource in place (mutable).

## Lifecycle Rules

By default, terraform deletes the resource first and create it back when updating a resource. However, it would be the 
desire approach in all cases. And sometimes, we may want the update version of the resource to be created first before 
the older one is deleted, or we may not want the resource to be deleted at all even if there's change made in its 
configuration. This can be achieved in terraform by making use of lifecycle rules.

```terraform
resource [resource-type] resource-name {
  # ...
  lifecycle {
    [lifecycle-rule]: [true|false|[]]
  } 
}
```

Here are the lifecycle rules:
- `create_before_destroy`
- `prevent_destroy`: Use to prevent resources to be accidentally destroyed. Resources can still be destroyed if we use 
the terraform destroy command. This rule may only prevent deletions from changes that are made to the configurations and
 its subsequent apply.
- `ignore_changes`: Prevent the resource from being updated based on a list of attributes defined in the lifecycle 
block.

## Data Sources

Data sources allow terraform to read attributes from resources that are provisioned outside it control.

```terraform
resource local_file pet {
 filename = "/root/pets.txt" # The file we want terraform to create (managed resource)
 content = data.local_file.dog.content
}

data local_file dog {
 filename = "/root/dog.txt" # The file created outside terraform (data resource)
}
```

## Meta-Arguments

They are used to change the behavior od resources. 

- **depend_on**
- **lifecycle**
- **count**

```terraform
variable filename {
 default = [
  "/root/pets.txt",
  "/root/dogs.txt",
  "/root/cats.txt",
  "/root/cows.txt",
  "/root/ducks.txt"
 ]
}

resource local_file pet {
 filename = var.filename[count.index]
 content = "Some content"
 
 count = length(var.filename)
}
```

- **foreach** Only works with maps or sets.

```terraform
variable filename {
 default = [
  "/root/pets.txt",
  "/root/dogs.txt",
  "/root/cats.txt",
  "/root/cows.txt",
  "/root/ducks.txt"
 ]
}

resource local_file pet {
 filename = each.value
 content = "Some content"
 
 for_each = toset(var.filename)
}
```

## Versions Constraints

```terraform
terraform {
 required_providers {
  local = {
   source = "hashicorp/local"
   version = "1.4.0"
  }
 }
}
```

- `version ≈ "!= 1.4.0"`
- `version = "< 1.4.0"`
- `version = "> 1.4.0"`
- `version = "> 1.4.0"`
- `version = "> 1.2.0 < 2.0.0 != 1.4.0"`
- `version = "~> 1.2.0"` Anything from `1.2.0` to `1.2.9`