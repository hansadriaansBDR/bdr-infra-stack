# Overview

In this guide you will learn:

* How to automatically provision [the datalake you manually provisioned here](../1-manual-emr-deployment/README.md) using CloudFormation and/or Ansible.


**We assume you know the Ansible basics ([playbooks](http://docs.ansible.com/ansible/playbooks_intro.html), [roles](http://docs.ansible.com/ansible/playbooks_roles.html), and [inventory](http://docs.ansible.com/ansible/intro_inventory.html) concepts) and are able to quickly pick up on the [AWS CloudFormation](https://aws.amazon.com/documentation/cloudformation/) principles.**

# First steps with Ansible and CloudFormation
Ansible can be used for automating various things: installation of software on VMs, deploying/configuring network infrastructure components, automating Docker container deployments, etc. Ansible is cloud provider and OS agnostic, meaning you can use it to provision various network infrastructure components on either AWS, Azure, Google Cloud or on-premise, or on any Windows or Linux based OS.

Amazon CloudFormation is used to automate deployment of AWS infrastructure components. It is specific to AWS and cannot be used on any other cloud provider. Other cloud providers have their own mechanism to attain the same goal. For example, Azure has Azure Resource Templates, and Google Cloud has Google Cloud Deployment Manager.

For the first assignment we will give some pointers and suggestions to automate [the datalake you manually provisioned here](../1-manual-emr-deployment/README.md). No step-by-step instructions are provided, but [sample solutions](../../README.md) are included in this repository for you to use as a reference.

1. As a first step, try to provision a VPC using Ansible's cloud formation module. To do this follow [this guide](http://docs.ansible.com/ansible/cloudformation_module.html), a sample solution can be found in the [network_aws module](../../common/network/network_aws). Reference to VPC CloudFormation template [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html).
2. Next, try the same with provisioning routing tables (the private and public one for the VPC), a sample solution can be found in the [routing_aws module](../../common/network/routing_aws). Reference to Routing table CloudFormation template [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-route-table.html).
3. To finish the networking infrastructure, also automate the provisioning of the subnets of the data lake solution (the private and public subnets). A sample solution can be found in the [subnets_aws module](../../common/network/subnets_aws). Reference to subnet CloudFormation components [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html), and to add them to the routing tables created earlier [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet-route-table-assoc.html).
4. Try to come up with other roles for the remaining components (OpenVPN, NAT instance, AWS EMR cluster etc.).

# Designing for reusability

The previous exercises focused on deploying specifically on AWS. Try to come up with an Ansible role structure that mimics the [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) principles used for software engineering (e.g. always depend on abstract roles/interfaces instead of implementation so that you can swap out different cloud provider implementations easily, and even make it possible to do a hybrid on-premise/multi-cloud approach in one system deployment).

The ideas and conventions that were introduced to make this possible with the BDR Infra Stack are described in the main [README](../../README.md)

# Open Exercises

#### Implement a JupyterHub notebook service in the cloud
Try to automate the provisioning of JupyterHub (preferably on RHEL) so that it runs inside the private subnet of the data lake, instead of on your laptop. Make sure it is generic enough to be cloud provider agnostic (how would you split this up in Ansible roles?).

#### Implement (part of) the same data lake infrastructure using another cloud provider

You can create a [Microsoft Azure](https://azure.microsoft.com/en-us/free/) or [Google Cloud Platform](https://cloud.google.com/) account with free starting credit.

Many components of different cloud providers can be mapped one-to-one. Try to find out which components you can use on Azure or Google Cloud to get the same data lake features in place. 

Tip: [Azure vs AWS mapping chart](http://www.tomsitpro.com/articles/azure-vs-aws-cloud-comparison,2-870-2.html) or [Google Cloud vs AWS mapping chart](https://cloud.google.com/free-trial/docs/map-aws-google-cloud-platform)


# Wrap-up
When you are done with all activities we would like you to shutdown any resources you might have created.
To prevent unnecessary costs, please make sure that you removed any resource you created, this might include:

1. You terminate the EMR cluster.
2. You terminate the OpenVPN instance.
3. You terminate the NAT instance.
4. Release the elastic IP associated to the NAT instance (you can check it when terminating the NAT instance or go to "EC2"->Elastic IPs").
4. You remove the VPC.
