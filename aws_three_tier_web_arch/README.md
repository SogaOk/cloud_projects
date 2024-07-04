# Project: AWS Three Tier Web Architecture

This is a project from the AWS Workshop Studio - [find here](https://catalog.us-east-1.prod.workshops.aws/workshops/85cd2bb2-7f79-4e96-bdee-8078e469752a/en-US). The objective of the project is to deploy a three-tier web architecture consisting of external and internal load balancers, a web tier, an app tier and a database tier.

The project is split into seven parts and will give builders hands-on experience with VPCs, subnets, security groups, database deployment, load balancing and autoscaling.

The reference architecture can be seen below. I recreated this using the Lucidchart App - [lucidchart](https://www.lucidchart.com/pages/?);

![jpeg of three tier web architecture.](./3twba.jpeg)

The architecture is laid out as follows;

- An internet-facing load balancer which forwards traffic to our web tier, consisting of Nginx servers deployed on ec2 instances in two public subnets.
- The Nginx servers serve a React.js website and directs API calls to the app tier's internal load balancer. The internal load balancer then forwards the traffic to the application tier.
- The application tier manipulates data in an Amazon Aurora MySQL database and returns it to our web tier.
- VPC, Subnets, Autoscaling, Security groups, Launch Template and Target groups are all configured as part of a successful deployment of this architecture.

## Initial Setup

The application code for this project has been provided and will be downloaded from Github. The next step is to create an s3 bucket where we will upload our code later on.

![S3 bucket creation screenshot](./imgs/create_bucket.JPG)

I provide my bucket with a unique name and leave the default entries unchanged.

The next piece of the initial set up is creating an EC2 Instance role in IAM. The purpose of this role is to give our instances permissions to download our code from S3 and to also use Systems Manager Session Manager to securely connect to our instances without using SSH keys.

I navigate to the IAM dashboard and create an EC2 role under the Roles menu option.

![Create EC2 role](./imgs/ec2_role1.JPG)

![Select trusted entity](./imgs/ec2_role2.JPG)

The AmazonSSMManagedInstanceCore and AmazonS3ReadOnlyAccess permissions policies will be added to the EC2 role to provide the required permissions.

![Add permissions](./imgs/ec2_role3.JPG)

## Networking & Security

In this section of the project we will create a custom VPC and configure our subnets, route tables, internet gateway, NAT gateway and security groups.

I navigate to the VPC dashboard and create VPC.

![VPC dashboard](./imgs/create_vpc1.JPG)

I provide a name tag for the VPC and choose a CIDR range. I click the create VPC button to create my VPC. Choose a CIDR range that allows you to create at least 6 subnets.

![create VPC](./imgs/create_vpc2.JPG)

Next, I move on to create the subnets. For this project we will need 6 subnets across two availability zones. Each availability zone will consist of a 1 public subnet and 2 private subnets as shown in the architecture diagram.
I navigate to subnets in my VPC dashboard and use the create subnet button.

![create subnet](./imgs/create_subnet1.JPG)

It is important here to use a naming convention that will allow you to easily identify your subnets and all the other resources that we will be configuring. This will save you some head-scratching as you progress with the project (I learnt the hard way 😖)

![input subnet details](./imgs/create_subnet2.JPG)

So now I have 3 subnets each across two availability zones.

![subnets created](./imgs/create_subnet3.JPG)

Now we move on to configure internet access for ur public subnets by creating an internet gateway and attaching it to our VPC. I move to Internet gateways on my VPC Dashboard and click on the Create internet gateway button.

![create IGW](./imgs/create_igw1.JPG)

I provide a suitable name for my internet gateway and create it.

![name and create IGW](./imgs/create_igw2.JPG)

The next step is to attach the internet gateway to our VPC. This can be done immediately after the internet gateway is created using the Attach to VPC button or from the Actions drop down selection.

![attach IGW to VPC](./imgs/igw_attach_vpc.JPG)

I select my VPC from the dialog box and complete the attachment.

![complete IGW attach to VPC](./imgs/igw_attach_vpc2.JPG)

Now we move on to configure internet access for our private subnets, to allow our app tier to access the internet. To make our architecture highly available, we will deploy one NAT gateway in each public subnet. To get this done I navigate to NAT gateways on my VPC dashboard.

![create NAT gateway](./imgs/create_natgw1.JPG)

I click on the create NAT gateway button and provide a suitable name for the NAT gateway. I select one of the public subnets created earlier and use the Allocate Elastic IP button to allocate a public IP address to the NAT gateway. 

![provide NAT gw details](./imgs/create_natgw2.JPG)

I repeat this process for the other public subnet. I now have two NAT gateways provisioned. 

![NAT gateways provisioned](./imgs/create_natgw3.JPG)

Public IP addresses are billed by AWS so keep this in mind to avoid any surprise costs. You can estimate the cost of your public IPs using the aws pricing calculator [here](https://calculator.aws/#/createCalculator/VPC)