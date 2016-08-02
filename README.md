# BigData Republic Infra Stack

The goal of this repository is two-fold:

1. Provide hands-on tutorials for deploying Big Data infrastructure components on some specific cloud provider and/or on-premise (see [docs](docs/README.md)).
2. Provide automated deployment scripts (Ansible) to deploy Big Data infrastructure components in either the cloud or on-premise (see below).
 




The Big Data infrastructure we use as a reference is shown below.


![BigData Republic Infra Stack](docs/bdr-infra-stack.png "BigData Republic Infra Stack") 

We automated the deployment of various Big Data infrastructure components by using Ansible. 
We apply the [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) software engineering practice on our Ansible roles such that we can use different cloud provider managed components interchangeably. This allows us even to deploy a stack that spans over multiple clouds and/or on-premise.


## Current state
The current code consists of only the BATCH component, with an AWS implementation using S3 and EMR; applying security best-practices. This resembles the end result of the Hands-on [Manual EMR Deployment Guide](docs/1-manual-emr-deployment/README.md).
