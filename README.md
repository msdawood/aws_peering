# AWS VPC peering
A virtual private cloud (VPC) is a virtual network dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS Cloud. Sometimes you may need to connect to other AWS resources hosted in different VPCs. One way to access the resource is routing the traffic through internet and the other option is VPC Peering.

A VPC peering connection is a networking connection between two VPCs that enables you to route traffic between them using private IPv4/IPv6 addresses. Instances in either VPC can communicate with each other as if they are within the same network. You can create a VPC peering connection between your own VPCs, or with a VPC in another AWS account. The VPCs can be in different Regions also known as an inter-Region VPC peering connection.

![image](https://github.com/user-attachments/assets/54f55ebd-089a-4273-a647-a9d5135b2e8d)

In this demo we will go through steps to set up VPC peering to communicate between two EC2 instances.

# Setup

## Create VPCs
**<ins>1. Create VPC1</ins>**

Create a VPC with CIDR range 10.0.0.0/16
![image](https://github.com/user-attachments/assets/2b5acf4b-7e15-4bb3-bbc8-482398ca05a0)

<ins>**2. Create VPC2**</ins>

Create a VPC with CIDR range 20.0.0.0/16
![image](https://github.com/user-attachments/assets/9809429e-1cf9-4417-9aec-95604792be6e)


## Create Internet Gateway
<ins>**1. Create Internet Gateway1**</ins>

Create internet gateway (vpc2vpc_ig1).
![image](https://github.com/user-attachments/assets/cd2c25e0-7b90-43d0-bc7e-bccaf3f27820)

<ins>**2. Create Internet Gateway2**</ins>

Create internet gateway (vpc2vpc_ig2).
![image](https://github.com/user-attachments/assets/e7780df4-a82d-47e9-bee5-3f468b62abc0)

<ins>**3. Attach the internet gateway to VPC**</ins>

![image](https://github.com/user-attachments/assets/5ab8e0bc-c7a6-4373-adaa-bd7fe96ed1d6)

![image](https://github.com/user-attachments/assets/3713abe2-0855-40a1-9f74-b7f19418dafe)

Do the same for Internet Gateway2.

## Create Subnet
Create a public subnet 1 (vpc2vpc_s1).

![image](https://github.com/user-attachments/assets/04133350-79a8-4f5b-9878-6ead77803a21)

Create a public subnet 2 (vpc2vpc_s2).

![image](https://github.com/user-attachments/assets/1418004a-74f3-44dd-b56b-5b0d31df6e3d)

## Create Route Table

<ins>**1. Route table 1**</ins>

Create a route table 1 (vpc2vpc_rt1)
![image](https://github.com/user-attachments/assets/0d21ae00-235a-42bd-8940-945b50db0ec7)

<ins>**2. Route table 2**</ins>

Create a route table 2 (vpc2vpc_rt2)

<ins>**3. Update Route table**</ins>
Update Route table to add internet routing through internet gateway.

Select route tables from LHS under VPC 
![image](https://github.com/user-attachments/assets/8807467c-72ef-4c62-b761-b7b18eb4d391)

Select edit routes button. Add route and select "Internet Gateway" from the dropdown
![image](https://github.com/user-attachments/assets/da6cc78d-a55e-4a88-bd3b-881f2fd4c145)

Search for the internet gateway we created in the previous step and select it.
![image](https://github.com/user-attachments/assets/5305be70-9217-483a-9402-48aa810dca2e)

Add 0.0.0.0/0 as destination.
![image](https://github.com/user-attachments/assets/b63af426-a21e-4a7d-a575-3b133b44cd06)

Once the route table is updated. You would see the newly added route to internet gateway.
![image](https://github.com/user-attachments/assets/c275c030-f17a-4634-8663-10107db0d58c)


## Create Key Value Pair
Create a key value pair (vpc2vpc2_kp1.pem) to be used with EC2 instance. This pem file will be downloaded to your computer.

## Create EC2 instances
1) Create a new EC2 instance (vpc2vpc_ecc1)
2) Leave the default values for Application and OS images (AMI)
3) Select the key pair 
![image](https://github.com/user-attachments/assets/0ca3c34e-2f0e-41c2-9e79-75f02240e172)

Edit Network Settings
1) Select the VPC created before (vpc2vpc_1).
2) Select the public subnet (vpc2vpc_s1).
3) Auto-assign public IP
4) Select "Create security group" radio button.
   Give a name "vpc2vpc_sg1".
   Add rules for "ssh" and "http"

![image](https://github.com/user-attachments/assets/7597b2b7-abe3-4ef4-b455-73abbc264e5e)

Create second EC2 instance in VPC vpc2vpc_2 in subnet vpc2vpc_s2.

## Download and install httpd web server on both the EC2 instances
1) Connect to the EC2 instance in the AWS Console.
2) Run the below commands to install httpd web server
   - yum update -y
   - yum install -y httpd
   - systemctl start httpd
   - systemctl enable httpd
3) Run the below command to add a default index html file. This will create an index.html file.
   - echo "&lt;h1&gt;Hello World!  This is $(hostname -f)&lt;/h1&gt;" > /var/www/html/index.html

# Testing Without VPC Peering

Now we have 2 VPCs with associated resources as seen below. Both the webservers can be accessed independently.

![image](https://github.com/user-attachments/assets/8658ebed-c41a-4d34-838f-373bd52111d0)

Use the EC2 public IP address to access the website. The scheme should be http. You should see the default index.html page with message "Hello World! This is <<host name>>".

Connect the EC2 instance 1 say vpc2vpc_ecc1 and try to ping EC2 instance 2 with its private IP address
![image](https://github.com/user-attachments/assets/aebcc6e5-97e9-497a-b9ff-55036f484b15)

EC2 instance 1 can not access EC2 instance 2 as both the EC2 instances are launched in their respective VPCs and both the VPCs are isolated to each other.

# Testing With VPC Peering
AWS VPC peering connection enables communication between 2 services across VPCs.

## Create VPC Peering
Go to VPC dashboard and select "Peering connections"
![image](https://github.com/user-attachments/assets/dc7e8937-afc9-4c5b-b861-c3168f721141)

In the Create peering connection screen

![image](https://github.com/user-attachments/assets/c6cc7fa1-2a5a-4e8f-9527-82386481e7c8)

- Provide a name for peering connection Ex: "vpc2vpc_pc1".
- Select VPC1 "vpc2vpc_1" from the VPC ID (Requester) drop down.
- Select VPC2 "vpc2vpc_2" from the VPC ID (Accepter) drop down.
- Leave the "Account" and "Region" radio buttons as is. This is because we are creating a VPC peering with in same account and same region.
- Create the peering connection
- 
## Accept the peering connection
Once the VPC peering connection gets created, by default the status is "pending acceptance". We need to accept the connection manually. 

![image](https://github.com/user-attachments/assets/c5d28f19-c3d9-44ca-8a58-bf694a6cbeb6)

Select "Actions" dropdown and "Accept request" option.
![image](https://github.com/user-attachments/assets/09bff629-df72-4841-9918-651cbe37c3fe)

Once accepted, the VPC peering connection status becomes "Active". Now the connection is established between VPC1 and VPC2.

## Route Table Changes
To send and receive traffic across this VPC connection, you must add a route to the peered VPC in one or more of  your VPC route tables.

If we want to connect from VPC1 to VPC2, then go to route table of VPC1 and select "Edit routes" button

- Select "Add route" button.
- Select the CIDR range of VPC2 which is "20.0.0.0/16" as Destination.
- Select "Peering Connection" as Target.
- Select the peering connection we created in the previous step.
- Save changes.

![image](https://github.com/user-attachments/assets/3631ff64-23b6-4747-831e-5fd550ab36fb)

## Testing
Now go back to the EC1 instance connection and run the curl command again. You will see the index.html response from the EC2 instance.

![image](https://github.com/user-attachments/assets/7f536901-42c1-4efe-ba46-838711644609)

Note: If you want to access VPC1 from VPC2, then you need to add the route in the VPC3 route table.

# Clean up
- Delete VPC peering connection
- Delete EC2 instances
- Delete security groups
- Delete route tables
- Delete subnets
- Delete internet gateways
- Delete VPCs


