# Introduction of Infrastructure as Code - IaC

## Challenges with traditional IT infrastructure

Let's consider an organization that wants to configure a new application. The business comes up with the requirements
for the new application. The business analyst then gathers the needs from the business, analyses them, and convert them
into a set of high level technical requirements. These are then pass to a solution architect/technical lead that designs
the architecture to be followed for the deployment of the application. This would typically include the infrastructure
considerations such as the type, spec, and count of servers that are needed, such as those for frontend web servers, 
backend servers, databases, load balancers, etc. Following the traditional infrastructure model, this would have to be
deployed in the organization's on premise environment, which would mean making use of the assets in the data center. If 
additional hardware is needed, they would have to be procured by the procurement team. This team will put in a new 
request with the vendors. It can then take anywhere between a few days to weeks or even months for the hardware to be 
purchase and delivered to the data center. Once received at the data center, the field engineers would be in charge of 
the rack and stack of the equipments. The system admin performs initial configurations and the network admin make these 
systems available on the network. The storage admin assigns storage to servers and the backup admin configure backups. 
And finally, once the system has been set up as per the standards, they can be handed over to the application teams to 
deploy their applications. 

This deployment model, which is still quite commonly used today, has quite a few disadvantages. The turnover time can 
range between weeks to months, and that's just to get the system in ready state to begin the application deployment. 
This includes the time it takes for this system to be initially procured and handed over between teams. Also, scaling up
 or down the infra on demand cannot be achieved quickly. The overall cost to deploy and maintain this model is generally
 quite high. While some aspects of the infra provisioning process can be automated, several steps such as the rack and 
stack cabling and other deployment procedures are manual and slow. With so many teams working on so many different tasks
, chances of human error are high and this results in inconsistent environments. Another major disadvantage of using 
this model is the under utilization of the compute resources. The infra sizing activity is generally carried out well in
 advance, and the severs are sized considering the peak utilization. The inability to scale up or down easily means that
 most of these resources would not be used during off-peak hours.

In the past decade or so, organizations have been moving to virtualization and cloud platforms to take advantages of 
services provided by major cloud providers such as Amazon Web Services, Microsoft Azure, Google Cloud Platform, etc. By 
moving to cloud, the time to spin up the infra and the time to market for applications are significantly reduced. This 
is because in the cloud, we don't have to invest in or manage the actual hardware assets that we would in case of a 
traditional infra model. The data center, the hardware, assets and services are managed by the cloud provider. A virtual
 machine can be spin up in a cloud environment in a matter of minutes and the time to market is reduced from several 
months to weeks. And infra costs are reduced when compared with the additional data center management and human resource
cost.  Cloud infra comes with support for apis, and that opens up a whole new world of opportunities for automation. 
Finally,  the built-in auto-scaling and elastic functionality of cloud infra reduces resource wastage.

With virtualization and cloud, we can now provision infra with a few clicks. While this approach is certainly faster and
 more efficient, using the management console for resource provisioning is not always the ideal solution. It is ok to 
have this approach when we are dealing with limited number of resources. But in a large organization with elastic and 
highly scalable cloud environment with immutable infra, this approach is not feasible. Once provision, the system still 
have to go through different teams with a lot of process overhead. That increases the delivery time, and the chances of 
human error are still at large, resulting in inconsistent environments. 

So different organizations started solving these challenges within themselves by developing their own scripts and tools 
(Shell, Python, Ruby, Perl, Powershell). Everyone was solving the same problems, trying to automate infra provisioning, 
to deploy environments faster and in a consistent fashion by leveraging the api functionalities of the various cloud 
environments. These evolved into a set of tools that came to be known as **Infrastructure as Code - IaC**.

## Infrastructure as Code

The better way to provision cloud infra is to codify the entire provisioning process. This way, we can write and execute
 code to define, provision, configure updates, and eventually destroy infra resources. This called **IaC**. With iac, we
 can manage nearly any infra component as code such as databases, networks, storages, or even application configuration.

```shell
# ec2.sh

#!/bin/bash
IP_ADDRESS="10.2.2.1"

EC2_INSTANCE=$(ec2-run-instances --instance-type t2.micro ami-0edab43b6fa892279)
INSTANCE=$(echo ${EC2_INSTANCE} | sed 's/*INSTANCE //' | sed 's/ .*//')

# Wait for instance to be ready
while ! ec2-describe-instances $INSTANCE | grep -q
"running"
do
  echo Waiting for $INSTANCE is to be ready...
done

# Check if instance is not provisioned and exit
if [ ! $(ec2-describe-instances $INSTANCE | grep -q "running") ]; then
  echo Instance $INSTANCE is stopped.
  exit
fi

ec2-associate-address $IP_ADDRESS -i $INSTANCE

echo Instance $INSTANCE was created successfully!!!
```

The code above is a shell script. However, it is not easy to manage it. It requires programming or development skills to
 build and maintain. There's a lot of logic that needed to be coded, and it is not easily reusable. And that's where 
tools like **Terraform** and **Ansible** help that code that is easy to learn, human-readable and maintain a large 
script can now be converted into a simple terraform configuration.

```terraform
# main.tf - Terraform
resource "aws_instance" "webserver" {
 ami = "ami-0edab43b6fa892279"
 instance_type = "t2.micro"
}
```

```yaml
# ec2.yaml - Ansible
- amazon.aws.ec2:
  key_name: my_key
  instance_type: t2.micro
  image: ami-123456
  wait: yes
  group: webserver
  count: 3
  vpc_subnet_id: subnet-29e63245
  assign_public_ip: yes
```

Although terraform and ansible are both iac tools, they have some key differences in what they're trying to achieve. And
 as result, they have some very different use cases.

There are several different tools part of iac family: Ansible, Terraform, Puppet, CloudFormation, Packer, SaltStack, 
Vagrant, Docker, etc. Although we can possibly use any of these tools to design similar solutions, they are all 
been created to address a very specific goal. With that in mind, these can be broadly classified into three types:

- **Configuration Management:** Ansible, Puppet & SaltStack.
  - Install and manage software on existing infra resources
  - Maintain standard structure
  - Version control
  - Idempotent.
- **Server Templating:** Docker, Parker, Vagrant
  - Pre-installed software & dependencies
  - Virtual machine or docker images
  - immutable infra
- **Infrastructure Provisioning:** Terraform, CloudFormation
  - Deploy immutable infra resources
  - Servers, databases, networks components, storages, etc.
  - Multiple providers