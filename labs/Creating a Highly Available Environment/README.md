# Guided Lab: Creating a Highly Available Environment

## Lab Overview and Objectives

Critical business systems should be deployed as highly available applications. This means applications remain operational even when some components fail.

To achieve high availability in Amazon Web Services (AWS):

- Run services across multiple Availability Zones
- Use services that are inherently highly available (e.g., load balancers)
- Configure services like Amazon EC2 for redundancy

### Objectives

After completing this lab, you will be able to:

1. Inspect a provided virtual private cloud (VPC)
2. Create an Application Load Balancer
3. Create an Auto Scaling group
4. Test the application for high availability

At the end of this lab, your architecture will resemble a highly available setup distributed across multiple Availability Zones.

---

## Task 1: Inspecting Your VPC

### Overview

This lab begins with an environment already deployed using AWS CloudFormation, which includes:

- A VPC
- Public and private subnets in two Availability Zones
- An Internet Gateway associated with the public subnets
- A NAT Gateway in one of the public subnets
- An Amazon RDS instance in one of the private subnets

---

### Steps to Inspect the VPC

1. Open VPC Console  
   - Go to AWS Management Console  
   - In the search bar, type VPC  
   - Select VPC  

2. Filter by Lab VPC  
   - In the left navigation pane, select "Filter by VPC"  
   - Choose "Lab VPC"  

3. View VPC Details  
   - Click "Your VPCs"  
   - Select "Lab VPC"  
   - In the Details tab:  
     - IPv4 CIDR: 10.0.0.0/16  
     - This means all IPs range from 10.0.x.x  

4. Inspect Subnets  
   - Go to Subnets  
   - Select Public Subnet 1  

   Key details:  
   - VPC: Lab VPC  
   - CIDR: 10.0.0.0/24  
   - 256 IP addresses (5 reserved)  
   - Located in an Availability Zone  

5. Review Route Table  
   - Select Route table tab  

   Routes:  
   - 10.0.0.0/16 → local (internal traffic)  
   - 0.0.0.0/0 → Internet Gateway (igw-) (public access)  

6. Check Network ACL  
   - Go to Network ACL tab  

   Default configuration:  
   - Allows all inbound traffic  
   - Allows all outbound traffic  

7. Inspect Internet Gateway  
   - Go to Internet gateways  
   - Confirm Lab IG is attached to Lab VPC  

8. Review Security Groups  
   - Go to Security groups  
   - Select Inventory-DB  

   Inbound rules:  
   - MySQL/Aurora (port 3306)  
   - Source: 10.0.0.0/16  

   Outbound rules:  
   - Allows all traffic (default)

## Task 2: Creating an Application Load Balancer

To build a highly available application, it is a best practice to launch resources in multiple Availability Zones. Availability Zones are physically separate data centers (or groups of data centers) in the same Region. If you run your applications across multiple Availability Zones, you can provide greater availability if a data center experiences a failure.

Because the application runs on multiple application servers, you need a way to distribute traffic among those servers. You can accomplish this goal by using a load balancer. This load balancer also performs health checks on instances and sends requests to only healthy instances.

---

### Steps

1. Open EC2 Console  
   - Go to AWS Management Console  
   - Search for EC2 and open it  

2. In the left navigation pane, choose Load Balancers  

3. Click Create load balancer  

4. Select Application Load Balancer → Create  

<img width="1179" height="828" alt="1" src="https://github.com/user-attachments/assets/cfd16181-cdad-4cd1-81c0-7e2a0efa7d80" />

---

### Basic Configuration

5. Load balancer name: Inventory-LB  

---

### Network Mapping

6. Select VPC: Lab VPC  

   Important: Ensure Lab VPC is selected  

7. Select subnets:

   - First Availability Zone → Public Subnet  
   - Second Availability Zone → Public Subnet  

   You should have:
   - Public Subnet 1  
   - Public Subnet 2  

---

<img width="1458" height="585" alt="2" src="https://github.com/user-attachments/assets/5912fd36-abcb-4433-82c3-448cfdac6cc9" />


### Security Group

8. Click Create a new security group  

9. Configure:

   - Security group name: Inventory-LB  
   - Description: Enable web access to load balancer  
   - VPC: Lab VPC  

10. Add inbound rules:

   - Rule 1:
     - Type: HTTP  
     - Source: Anywhere-IPv4  

   - Rule 2:
     - Type: HTTPS  
     - Source: Anywhere-IPv4  

11. Click Create security group  

---

### Assign Security Group

12. Return to load balancer tab  

13. Click refresh  

14. Select Inventory-LB security group  

15. Remove default security group  

---

<img width="1019" height="561" alt="3" src="https://github.com/user-attachments/assets/cf1d0488-8e58-4d1a-9b2f-ee745aae6c1e" />


### Create Target Group

16. Click Create target group  

---

#### Step 1: Group Details

17. Configure:

   - Target type: Instances  
   - Target group name: Inventory-App  
   - VPC: Lab VPC  

---
<img width="902" height="759" alt="4" src="https://github.com/user-attachments/assets/b8398882-3102-499c-a0d2-2577aec54f65" />


#### Health Checks

18. Configure advanced settings:

   - Healthy threshold: 2  
   - Interval: 10 seconds  

   The system checks every 10 seconds.  
   Two successful responses = healthy instance  

---

19. Click Next  


<img width="869" height="724" alt="5" src="https://github.com/user-attachments/assets/a6e1f0b9-8ec7-4ca1-9479-e05d74f40fa7" />

---

#### Step 2: Register Targets

20. Skip this step (no instances yet)  

---

21. Click Create target group  

22. Return to load balancer tab  

23. Click refresh  

24. Select Inventory-App as default action  

<img width="1627" height="336" alt="6" src="https://github.com/user-attachments/assets/c89b2c5a-92e4-4a3f-8bd9-dac2e05017ad" />

---

### Create Load Balancer

25. Scroll down  

26. Click Create load balancer  

27. Click View load balancer  

## Task 3: Creating an Auto Scaling Group

Amazon EC2 Auto Scaling is a service designed to automatically launch or terminate EC2 instances based on user-defined policies, schedules, and health checks. It also distributes instances across multiple Availability Zones to ensure high availability.

In this task, you create an Auto Scaling group that deploys EC2 instances across private subnets. This is a security best practice because instances in private subnets cannot be accessed directly from the internet. Instead, traffic is routed through the load balancer.


<img width="1188" height="511" alt="7" src="https://github.com/user-attachments/assets/90172566-ab32-4426-94b3-08bd25e18430" />

---

### Task 3.1: Creating an AMI for Auto Scaling

You will create an Amazon Machine Image (AMI) from the existing Web Server 1 instance.

1. Open EC2 Console  
   - Go to AWS Management Console  
   - Search for EC2 and open it  

2. Go to Instances  
   - Select Web Server 1  
   - Wait until status checks show **2/2 checks passed**  

<img width="1627" height="336" alt="6" src="https://github.com/user-attachments/assets/10f3ead1-da94-443c-8cf5-40ab7c17e553" />


3. Create AMI  
   - Actions → Image and templates → Create image  

4. Configure AMI  
   - Image name: Web Server AMI  
   - Description: Lab AMI for Web Server  

5. Create image  

A new AMI ID will be generated. This will be used later in the launch template.

<img width="1188" height="511" alt="7" src="https://github.com/user-attachments/assets/c2a69681-7080-4fdc-a6a5-51cfd003860a" />



---

### Task 3.2: Creating a Launch Template and Auto Scaling Group



#### Create Launch Template

1. Go to Launch Templates  
2. Click Create launch template  

3. Configure:

- Launch template name: Inventory-LT  
- Enable Auto Scaling guidance  


<img width="1266" height="493" alt="8" src="https://github.com/user-attachments/assets/e1096ca3-e647-48a7-9849-b40f77f28bbf" />

4. AMI  
   - Select My AMIs  
   - Choose Web Server AMI  
   
<img width="917" height="609" alt="9" src="https://github.com/user-attachments/assets/f97804c5-5ae0-40aa-95b3-3469ad999861" />




5. Instance type  
   - Select t2.micro  

6. Key pair  
   - Select vockey  

7. Network settings  
   - Select existing security group  
   - Choose Inventory-App  

<img width="844" height="722" alt="10" src="https://github.com/user-attachments/assets/aeac3d26-d1e1-4049-acda-15ab28b48fae" />


8. Advanced details  

- IAM instance profile: Inventory-App-Role  
- Enable CloudWatch monitoring  


<img width="1107" height="155" alt="11" src="https://github.com/user-attachments/assets/e5fe26d2-e067-4461-847d-7c27682021db" />



9. User data script:

```bash
#!/bin/bash

yum install -y httpd mysql
amazon-linux-extras install -y php7.2

wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/12-lab-mod10-guided-Scaling/s3/scripts/inventory-app.zip
unzip inventory-app.zip -d /var/www/html/

wget https://github.com/aws/aws-sdk-php/releases/download/3.62.3/aws.zip
unzip aws -d /var/www/html

chkconfig httpd on
service httpd start
```

<img width="914" height="430" alt="13" src="https://github.com/user-attachments/assets/d7bdc832-bb51-49f4-8a9f-fdd331cf13fc" />

## Task 3: Creating an Auto Scaling Group

Amazon EC2 Auto Scaling automatically launches or terminates EC2 instances based on defined policies and distributes them across multiple Availability Zones.

---

### Task 3.1: Creating an AMI for Auto Scaling

1. Open EC2 Console  
2. Go to Instances  
3. Select Web Server 1  
4. Wait for **2/2 checks passed**  
5. Actions → Image and templates → Create image  

6. Configure:
   - Image name: Web Server AMI  
   - Description: Lab AMI for Web Server  

7. Click Create image  

---

### Task 3.2: Creating a Launch Template and Auto Scaling Group

#### Create Launch Template

1. Go to Launch Templates  
2. Click Create launch template  

3. Configure:
   - Name: Inventory-LT  
   - AMI: Web Server AMI  
   - Instance type: t2.micro  
   - Key pair: vockey  
   - Security group: Inventory-App  



4. Advanced:
   - IAM role: Inventory-App-Role  
   - Enable CloudWatch monitoring  

5. Add user data script  

6. Click Create launch template  

---

#### Create Auto Scaling Group

1. Open launch template Inventory-LT  
2. Actions → Create Auto Scaling group  

---

### Step 1: Choose Template

- Name: Inventory-ASG  
- Launch template: Inventory-LT  

<img width="1512" height="820" alt="14" src="https://github.com/user-attachments/assets/e4076e17-3c0c-4da8-81d2-ab0640aff3ce" />


---

### Step 2: Instance Launch Options

- VPC: Lab VPC  

- Subnets:
  - Private Subnet 1  
  - Private Subnet 2  

<img width="960" height="456" alt="15" src="https://github.com/user-attachments/assets/638f3c63-4190-4883-bcf6-5c8062aed0a8" />


---

### Step 3: Advanced Options

1. Load balancing  
   - Attach to existing load balancer  
   - Target group: Inventory-App  

<img width="1322" height="789" alt="16" src="https://github.com/user-attachments/assets/d12ee864-ab2b-4f76-96db-1e8e14a2b477" />

2. Health checks  
   - Enable ELB health checks  
   - Grace period: 90 seconds  

3. Additional settings  
   - Enable CloudWatch metrics  
<img width="596" height="486" alt="17" src="https://github.com/user-attachments/assets/678435e6-505b-405e-a37e-5c9fbc00b58a" />

---

### Step 4: Group Size and Scaling

- Desired capacity: 2  
- Minimum capacity: 2  
- Maximum capacity: 2  

No scaling policies required for this lab.

<img width="1067" height="333" alt="18" src="https://github.com/user-attachments/assets/e03f5725-1171-4292-ada6-27173df71dc1" />

<img width="1292" height="344" alt="19" src="https://github.com/user-attachments/assets/f8a45d4b-dee6-48e5-9634-72b39aa08af8" />


---

### Step 5: Notifications

- No configuration needed  

---

### Step 6: Tags

- Key: Name  
- Value: Inventory-App  

---

### Step 7: Review and Create

1. Review settings  
2. Click Create Auto Scaling group  

The group will launch 2 instances across multiple Availability Zones.

---

## Task 4: Updating Security Groups

The application uses a three-tier architecture.

---

### Task 4.1: Load Balancer Security Group

Already configured:

- HTTP (80)  
- HTTPS (443)  

---
<img width="1427" height="175" alt="21" src="https://github.com/user-attachments/assets/60d4d305-473e-4bfa-af78-db62fcedf9be" />

### Task 4.2: Application Security Group

1. Go to Security Groups  
2. Select Inventory-App  
3. Open Inbound rules  
4. Click Edit inbound rules  

5. Add rule:
   - Type: HTTP  
   - Port: 80  
   - Source: Inventory-LB  
   - Description: Traffic from load balancer  

6. Click Save rules  

---

### Task 4.3: Database Security Group

1. Select Inventory-DB  
2. Go to Inbound rules  
3. Click Edit inbound rules  

4. Delete existing rule  

5. Add new rule:
   - Type: MySQL/Aurora  
   - Port: 3306  
   - Source: Inventory-App  
   - Description: Traffic from application servers  

6. Click Save rules  

---

Now the architecture enforces three-tier security:

1. Load balancer → Application  
2. Application → Database  
3. Database only accepts internal traffic  

## Task 5: Testing the application

Your application is now ready for testing.

In this task, you confirm that your web application is running. You also test that it is highly available.

1. In the left navigation pane, choose Target Groups.  

2. Select Inventory-App.  

3. Choose the Targets tab.  

   This tab should show two registered targets. The Health status column shows the results of the load balancer health check.

4. In the Registered targets area, click refresh until both instances show as healthy.  

5. If the status does not change to healthy, check the configuration.  

---

### Access the Application

6. In the left navigation pane, choose Load Balancers.  

7. Select Inventory-LB.  

8. In the Details tab, copy the DNS name.  

   It should look like:  
   Inventory-LB-xxxx.elb.amazonaws.com  

9. Open a new browser tab, paste the DNS name, and press Enter.  

10. Reload the page multiple times.  

   You will notice:
   - The instance ID changes  
   - The Availability Zone changes  

---

### How the Application Works

1. The user sends a request to the load balancer (public subnet).  

2. The load balancer forwards the request to an EC2 instance (private subnet).  

3. The EC2 instance processes the request and returns the response.  

4. The load balancer sends the response back to the browser.  

---

## Task 6: Testing high availability

Your application is configured to be highly available.

You will now simulate a failure.

1. Go back to the EC2 console.  

2. In the left navigation pane, choose Instances.  

3. Select one Inventory-App instance.  

4. Choose Instance state → Terminate instance.  

5. Confirm termination.  

---

### Observe Behavior

6. The load balancer detects the failure.  

7. All traffic is routed to the remaining instance.  

8. Return to the web browser and reload the page several times.  

   You will notice:
   - The Availability Zone no longer changes  
   - The application is still available  

---

### Auto Scaling Recovery

9. After a few minutes, Auto Scaling detects the failure.  

10. It launches a new instance automatically.  

11. Return to the EC2 console and refresh every 30 seconds.  

12. Wait until a new instance appears.  

13. After a few minutes, the new instance becomes healthy.  

14. The load balancer starts distributing traffic again.  

---

This confirms that the application is highly available.


## Optional Task 1: Making the Database Highly Available

This task is optional. You can complete it if you have remaining lab time.

The application architecture is now highly available. However, the Amazon RDS database currently runs on a single instance.

In this task, you configure the database for Multi-AZ deployment.

---

### Steps

1. Open RDS Console  
   - Go to AWS Management Console  
   - Search for RDS  

2. In the left navigation pane, choose Databases  

3. Select the inventory-db instance  

4. Review the database details  

5. Click Modify  

---

### Enable Multi-AZ

6. In the Availability & durability section:  
   - Select Create a standby instance  

This enables Multi-AZ deployment.

---

### Explanation

- One instance = Primary (handles traffic)  
- One instance = Standby (failover)  
- Same DNS name is used  
- Automatic failover if primary fails  

---

### Scale the Database

7. In Instance configuration:  
   - DB instance class: db.t3.small  

8. In Storage:  
   - Allocated storage: 20  

---

### Apply Changes

9. Click Continue  

10. Select Apply immediately  

11. Click Modify DB instance  

The database status will show **Modifying**. No need to wait.

---

## Optional Task 2: Configuring a Highly Available NAT Gateway

This task is optional. You can complete it if you have remaining lab time.

Currently, only one NAT gateway exists. This creates a single point of failure.

You will create a second NAT gateway in another Availability Zone.

---

### Steps

1. Open VPC Console  
   - Go to AWS Management Console  
   - Search for VPC  

2. In the left navigation pane, choose NAT Gateways  

3. Click Create NAT gateway  

---

### Create NAT Gateway

4. Configure:

   - Name: NatGateway2  
   - Subnet: Public Subnet 2  
   - Click Allocate Elastic IP  

5. Click Create NAT gateway  

---

### Create Route Table

6. In the left navigation pane, choose Route tables  

7. Click Create route table  

8. Configure:

   - Name: Private Route Table 2  
   - VPC: Lab VPC  

9. Click Create route table  

---

### Configure Routes

10. Open the Routes tab  

11. Click Edit routes  

12. Add route:

   - Destination: 0.0.0.0/0  
   - Target: NAT Gateway → NatGateway2  

13. Click Save changes  

---

### Associate Subnet

14. Open Subnet associations tab  

15. Click Edit subnet associations  

16. Select Private Subnet 2  

17. Click Save associations  

---

### Result

- Each private subnet uses a NAT gateway in its own Availability Zone  
- No single point of failure  
- High availability is achieved
