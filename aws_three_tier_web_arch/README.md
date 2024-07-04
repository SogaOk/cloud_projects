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
