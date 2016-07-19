# Overview

In this guide you will learn:

* How to use Amazon managed services to build a data lake.
* How to remotely access the data lake.
* How to run and monitor jobs in the data lake.

**We assume you will provision all resources in the EU Ireland data center. Shorthand for this is 'eu-west-1'**.

# Provisioning a data lake using EMR

We will go through all the steps of setting up a data lake in AWS. The managed service we will use is Amazon Elastic MapReduce (EMR). This service allows you to automatically provision a Hadoop/Spark Cluster, and run jobs on it through the Amazon console.
The benefit of this is that you do not have to configure all the components and applications in the Hadoop eco-system yourself. Amazon will do this for you. In addition, Amazon provides APIs and a web interface to access the cluster intead of you having to dig into the details of Hadoop.

### Create Key Pair
For accessing the AWS instances through SSH, an SSH key pair is required. A first step is to create one, which we will use later throughout the guide.

1. From the AWS Console select "EC2", and go to the key pairs section.

2. Create a Key Pair called "dev-<name>" where <name> is substituted with your name.

3. Save the .pem.txt file on your local file system. This is your private key, do not lose it, or you will not be able to access the instances you create.

![Create Key Pair](resources/create_key_pair.png "Create Key Pair")

### Create VPC
By default, an EMR cluster is created in a public subnet that is internet-facing. This means that only one firewall mistake may lead to giving out access to the public Internet.
In this guide we will configure the cluster in a private subnet, which is way more secure, and recommended when processing confidential data. Private subnets are not directly internet-facing, and can only be accessed from The Internet if the traffic is routed through an intermediate instance (e.g. load balancer/software vpn) in a public subnet. Instances in a private subnet cannot access any resources on The Internet, unless they are using a NAT or other outgoing proxy instance in a public subnet. This concept is similar to how your home router works (e.g. everybody behind the NAT will show as the same IP on The Internet).

In order to host an EMR cluster in a private subnet, we will first create a Virtual Private Cloud (VPC) environment which encapsulates all the resources you will provision in the remaining of the guide.
This is an isolated environment that allows you to run different Amazon services without interfering with other users.


1. From the AWS Console select "VPC", and start the VPC Wizard. ![Create VPC](resources/create_vpc.png "Create VPC")
2. Select to create a VPC with Public and Private subnets. ![Create Public/Private Subnet](resources/create_public_private_subnet.png "Create Public/Private Subnet")
3. Configure the VPC to use a NAT instance (not gateway, since instance is cheaper), an S3 endpoint, and name it "dev-<name>" where <name> is substituted with your name. Use the key pair you created with the NAT instance. ![Configure VPC](resources/configure_vpc.png "Configure VPC")
4. When the VPC is created, inspect various options. First filter by VPC, and select the VPC you just created.
    1. Inspect the "Subnets" tab, which should contain 2 subnets "Public subnet" and "Private subnet". By default each of the subnets does have the default Access Control List (ACL) firewall attached, which allows all inbound and outbound traffic. We are going to tighten the security later in the guide.
    2. Inspect the "Route Tables" tab, which should contain 2 route tables. Routing tables are used to route traffic between subnets, and from and/to The Internet. All AWS managed service APIs can only be accessed through The Internet, and not directly from within a private subnet. One exception is S3, which allows you to setup a dedicated VPC endpoint (we did this in previous steps). The public subnet is explicitly associated with the public routing table, and therefor can access the Internet, the S3 VPC endpoint, and local subnets in 10.0.0.0/16. The private subnet is associated with the private routing table, and therefor can access the S3 VPC endpoint, local subnets in 10.0.0.0/16, and all traffic to the internet goes through the NAT (eni-xxx).


#### Questions
Answers to the questions can be viewed on-click.

<details>
  <summary>1. Do instances in a private subnet have a public internet-routable IP ? </summary>
   No, they only have a private IP in the range associated with the subnet (in this case 10.0.1.0/24 , equaling to 10.0.1.4  to 10.0.1.254 ; .0 reserved for subnet network address , .1 reserved for VPC routing table / gateway, .2 reserved for DNS resolving, .3 reserved for AWS future use, .255 reserved for subnet broadcast).
</details>

<details>
  <summary>2. How many *private* IP addresses are available in the public subnet after taking the NAT instance into account?</summary>
   250, since the NAT takes up 1 private, and Amazon reserves 5 IPs by default (see answer to question 1).
</details>

<details>
  <summary>3. How many *public* IP addresses are available in the public subnet after taking the NAT instance into account?</summary>
   As many as needed. Public IP addresses are not reserved for any subnet. When assigning a public IP to an instance (in the public subnet), you will automatically get an IP address from the public IPv4 pool Amazon owns. If you want to assign a static public IP address, you can use Amazon's Elastic IP service, which allows you to lease public IPs even when you have not attached them to an instance.
</details>

<details>
  <summary>4. Which of these Amazon Managed Services require Internet access on instances in the private subnet, in order to be able to use them from these instances? S3, Route53 (managed DNS), SQS (simple queue service).</summary>
   It depends. If you have added an S3 region endpoint to your VPC routing tables, then S3 access to that specific region does not go over The Internet. Access to other S3 regions DOES go over The Internet (including the global endpoint!), so make sure to correctly configure the region your instances use. If you did not configure an S3 endpoint, then all traffic to S3 goes over The Internet. Route53 DNS management and SQS queue (and any other service besides S3) access always goes over The Internet. DNS lookup/resolving does not go over The Internet, because the third private IP address (.2) of each subnet is reserved for DNS lookup.
</details>


### Configure EMR

Now you have created your VPC with both a public and private subnet, we can start provisioning the Amazon EMR cluster.

1. From the AWS Console select "EMR", and press the "Create cluster" button. In addition, make sure to click the "Go to advanced options" link.
2. Configure the cluster with release "emr-4.7.2", and check at least "Hadoop 2.7.2" and "Spark 1.6.2". ![Create EMR Step 1](resources/create_emr_1.png "Create EMR Step 1")
3. In the second step, select the VPC you created in the previous section. Select the *private* subnet. Select m4.large as instance type for all nodes. Because we are using the cluster for testing purposes, we can save a lot of money by using spot instances. Spot instances are the instances that are available for a reduced cost as long as there is an excess capacity of the instance type. Tick the box and enter a maximum price of $0.264 (normal on-demand price in eu-west-1) for the Master and Core instances. **Note that for production systems it is not recommended to use spot instances, because if the market price exceeds your bid, the instances will be terminated.** You generally use on-demand instances for Master and Core to keep a stable cluster, and you can use spot instances for Task nodes that provide extra capacity when running a large job, but do not necessarily need to be up. ![Create EMR Step 2](resources/create_emr_2.png "Create EMR Step 2")
4. In the third step, keep the defaults (see picture below). ![Create EMR Step 3](resources/create_emr_3.png "Create EMR Step 3")
5. In the last step, select your EC2 key pair created at the beginning of this guide. This allows you to login with SSH to the cluster instances. Keep the permissions and security groups to the defaults for now, these set the AWS API access permissions for the EMR instances, and their instance specific firewalling respectively. Make sure to select "S3 server-side encryption", so all data that is being saved to S3 is encrypted on disk by default. ![Create EMR Step 4](resources/create_emr_4.png "Create EMR Step 4")

While the EMR cluster is getting provisioned, you can check the "EC2" page for "Running Instances". In a minute or two, 3 new instances should appear of type m4.large. You can inspect their private IPs and security groups to see in which subnet their are allocated, and what firewalling is configured by default.
In addition, you can find the details of your EMR cluster on the "EMR" page by selecting your dev-<name> cluster. When the status of the cluster becomes "Waiting", it is ready to receive jobs, and you can move on to the next section. In the mean time, you can answer the questions below.
#### Questions

<details>
  <summary>1. What security groups have been automatically created, and for what purpose? (hint: Check the "VPC"->"Security Groups" page after filtering on your VPC)</summary>
  In addition to the default security group that is assigned to instances which do not explicitly provide security groups, we have:<br>
  <b>ElasticMapReduce-Master-Private:</b> Security group for the Master instance of the cluster. Allows all outbound traffic, allows all inbound traffic from Core/Task instances, and allows TCP 8443 inbound from the EMR API controller for provisioning purposes.<br>
  <b>ElasticMapReduce-Slave-Private:</b> Security group for the Core/Task instances of the cluster. Allows all outbound traffic, allows all inbound traffic from Core/Task instances, and allows TCP 8443 inbound from the EMR API controller for provisioning purposes.<br>
  <b>ElasticMapReduce-ServiceAccess:</b> Security group for the Elastic Network Interface (ENI) of the EMR API controller that is provisioned out of user's control. Only allows outbound traffic to the Master/Core/Task nodes on port TCP 8443 for provisioning purposes.<br>
</details>

<details>
  <summary>2. Will we be able to access the EMR cluster through SSH with current configuration? What allows/prevents us from doing so?</summary>
  No. The EMR instances do not have public IP addresses attached. Even if they had (in theory), then we would still require an intermediate instance in the public subnet to route traffic to them, because instances in the private subnet cannot directly be reached from the Internet since no Internet Gateway is attached to the private routing table. If such a route would be in place (through a load balancer or software VPN), then it would still not be possible because although subnet ACLs do not impose any restrictions, the security groups of the instances do not allow port 22 TCP traffic from outside the cluster.
</details>

<details>
  <summary>3. Will we be able to access the EMR cluster through SSH with current configuration if we were already logged in on an instance in the public subnet (e.g. the NAT) ?</summary>
  No. Although public/private subnet ACLs do not impose any restrictions, the security groups of the instances do not allow port 22 TCP traffic from outside the cluster. In this case, even if you logged in into an instance in the private subnet other than one from the cluster, the security groups still prevent access.
</details>

<details>
  <summary>4. Check the EMR_DefaultRole and EMR_EC2_DefaultRole in "IAM"->"Roles". What purpose do these roles have in the EMR cluster? How would you limit access to a subset of S3 buckets to the EMR cluster instead of all buckets in the account (default) ?</summary>
    <b>EMR_DefaultRole:</b> Role used by the Elastic MapReduce managed service to provision EC2 instances, create log S3 buckets, and pass the EMR_EC2_DefaultRole's accompanying instance profile to the EC2 instances created.<br>
    <b>EMR_EC2_DefaultRole:</b> The policies attached to this role are used for giving the Master/Core/Task instances access to write their logs to cloudwatch, read and write to S3 (all buckets can be accessed by default), and access various other services. The Gotcha here is that Amazon uses Instance Profile and Role interchangeably in its documentation. An IAM Role cannot be attached to an instance directly, but requires an instance profile object that encapsulates the role. <br>
    You can only limit access to specific S3 buckets, by creating an alternative InstanceProfile+Role combination to the EMR_EC2_DefaultRole, and specify this in the EMR configuration. In the accompanying policies, you can specify a range of S3 buckets instead of all (default).
</details>


# Running an example job

There are two ways of running a job on an EMR cluster. Either you connect to the cluster REST API endpoint (YARN), which allows you to run interactive jobs using the Spark Shell from your own workstation for example. Another option is to use pre-compiled JAR files and submit them through the AWS EMR API and/or console.
In this step we explore the second option, since we do not have access to the cluster's REST API endpoint for YARN at this point. After the next section, we will also explore the YARN endpoint option.

1. Navigate to your EMR cluster's detail page.
2. Submit an example Spark job to compute PI using 10 parallel tasks, as shown in the picture below. Use "--class org.apache.spark.examples.SparkPi" as spark-submit options, and "/usr/lib/spark/lib/spark-examples.jar" as executable path. ![Add EMR Step](resources/add_emr_step.png "Add EMR Step")
3. After the EMR step has been submitted it will run on the cluster. This EMR step feature is typically used to schedule batch jobs in production environments periodically. It however provides little feedback on what is going on. In order to see this, we will also explore other options of running jobs later in the guide.
4. (Optional) If you have a pre-compiled Spark application ready, try to run it on EMR using the Steps interface. You can upload the jar to S3, and select it as JAR location.

# Remote Access to the Cluster

In order to get more insights into what is going on during the run time of a job, and to allow for running jobs from your workstation, we will require remote access to the cluster. In this section, we will configure an OpenVPN instance to provide you VPN access to the EC2 instances in the cluster.

## Configuring the OpenVPN instance

1. Navigate to "EC2" and click "Launch Instance".
2. Select the OpenVPN Acces Server **Bring Your Own License** Amazon Machine Image (AMI), which includes 2 free connections suitable for testing purposes. ![Select OpenVPN AMI](resources/create_openvpn_1.png "Select OpenVPN AMI")
3. Select t2.micro as instance type. ![Select t2.micro Instance Type](resources/create_openvpn_2.png "Select t2.micro Instance Type")
4. Select your VPC, the *public* subnet, enable auto-assign public IP, and check "Protect against accidental termination". ![Configure Instance Details](resources/create_openvpn_3a.png "Configure Instance Details")
5. On the same page, paste the following User Data. This automates the configuration of the OpenVPN access server on the instance. Make sure you include the last comment so the restart command also gets executed.
```bash
#!/bin/bash
admin_user=openvpn
reroute_gw=0
reroute_dns=1
apt-get update && apt-get install -y awscli,
/usr/local/openvpn_as/bin/ovpn-init --ec2 --batch
/bin/sleep 5 # Wait for OpenVPN Start
/usr/local/openvpn_as/scripts/sacli status
/usr/local/openvpn_as/scripts/sacli --key vpn.server.google_auth.enable --value true ConfigPut
/usr/local/openvpn_as/scripts/confdba -mk vpn.server.group_pool.0 -v 172.27.240.0/20 # Note: These are NAT-ed ip addresses (openvpnclient to openvpnserver communication), they are not claimed in our VPC
/usr/local/openvpn_as/scripts/confdba -mk vpn.daemon.0.client.netmask_bits -v 20
/usr/local/openvpn_as/scripts/confdba -mk vpn.daemon.0.client.network -v 172.27.224.0 # Note: These are NAT-ed ip addresses (openvpnclient to openvpnserver communication), they are not claimed in our VPC
/usr/local/openvpn_as/scripts/confdba -mk vpn.daemon.0.listen.ip_address -v all
/usr/local/openvpn_as/scripts/confdba -mk vpn.daemon.0.server.ip_address -v all
/usr/local/openvpn_as/scripts/confdba -mk cs.https.ip_address -v all
/usr/local/openvpn_as/scripts/confdba -mk admin_ui.https.ip_address -v all
/usr/local/openvpn_as/scripts/confdba -mk vpn.server.port_share.service -v client
echo "openvpn:*2016ManualEmrDeploymentRemoteAccess" | chpasswd
/etc/init.d/openvpnas restart
# A newline should be after the last line, thus this comment to enforce it.
```
![Insert User Data](resources/create_openvpn_3b.png "Insert User Data")
6. Skip the "Add Storage" settings / keep the defaults.
7. Tag the instance with "OpenVPN-dev-<name>" with your name substituted for <name>. This makes it easier to find the instance in the instance list later on. ![Tag Instance](resources/create_openvpn_5.png "Tag Instance")
8. Use the default security groups. ![Configure Security Groups](resources/create_openvpn_6.png "Configure Security Groups")
9. Finally, create the instance, and select your key pair so you will be able to access it in case automatic configuration fails. ![Launch Instance](resources/create_openvpn_7.png "Launch Instance")
10. When the instance is correctly provisioned you can browse to https://<your_public_dns> , which can be retrieved by selecting your instance in "EC2"->"Instances" , and pressing the "Connect" button.

#### Troubleshooting

In case automatic provisioning fails, login to the instance using SSH with (or use PuTTY): ssh -i "<your key pair locally stored>" openvpnas@<your_public_dns>. If the instance is not provisioned fully you should get a configuration prompt of OpenVPN immediately. You can use the configuration as defined at the top of the user data script in there manually, or exit the script CTRL+C and then manually execute all the steps from the user data script like shown below.

## Connecting to your VPC using OpenVPN





* OpenVPN
* ELB/proxy alternative


### EMR console



### Scaling up/down

### Troubleshooting

# Running your custom job

