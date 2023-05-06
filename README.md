### Creating a 3 tier networking architecture for a dynamic website deployment ###
# STEP 1, CREATE A VPC WITH.

    Navigate to the VPC service in the management console and select create VPC
    Cidr_block            = 10.0.0.0/16
    Enable_dns_hostnames  = true
    Enable_dns_support    = true
    Tags                  = practice-vpc

# STEP 2, CREATE 6 SUBNETS, 2 PUBLIC AND 4 PRIVATE.

    In the VPC console click on subnets and select create subnets
    Public subnets for web server
    VPC_ID                  = practice-vpc
    Cidr_blocks             = [10.0.0.0/24, 10.0.1.0/24]
    Availability_zones      = [us-east-1a ,us-east-1b]
    Map_public_ip_on_launch = true
    Tags                    = [public-web-subnet-AZ1, public-web-subnet-AZ2]

    Private subnets for APP server
    VPC_id                  = practice-vpc
    Cidr_blocks             = [10.0.3.0/24, 10.0.4.0/28]
    Availability_zones      = [us-east-1a ,us-east-1b]
    Tags                    = [private-app-subnet-AZ1, private-app-subnet-AZ2]
 
    Private subnets for database server
    VPC_id                  = practice-vpc
    Cidr_blocks             = [10.0.2.0/24, 10.0.5.0/28]
    Availability_zones      = [us-east-1a ,us-east-1b]
    Tags                    = [private-db-subnet-AZ1, private-db-subnet-AZ2]

# STEP 3, CREATE AN INTERNET GATEWAY AND ATTACH TO THE ABOVE VPC.

    Navigate to the VPC console and click on IGW and select create IGW
    Vpc_id  = practice-vpc
    Tags    = practice-igw

# STEP 4, CREATE 2 NAT GATEWAY IN THE PUBLIC SUBNETS [AZI & AZ2].

     Navigate to the VPC console and click on NAT gateway and then create NAT gateway
     Select allocate elastic ip address and add it to the private 
     route tables

# STEP 5, CREATE 3 ROUTE TABLES, 1 PUBLIC AND 2 PRIVATE ROUTE TABLES.

  Navigate to the VPC console and click on route table and then create route tables
  1. Public route table for web subnets.
      - Vpc_id        = practice-vpc
      - Route         =
       Destination = 0.0.0.0/0
       Target      = practice-igw
     - Tags          = public-RT

  NOTE: Associate the 2 public-web-subnets to the public route table 

 2. Private route table  AZ1
   - Vpc_id        = practice-vpc
   - Route         =
      Destination = 0.0.0.0/0
      Target      = NAT Gateway-AZ1
   - Tags          = pri-RT-AZ1

  NOTE:Associate the pri-DB-sn1 and the pri-app-sn1 to the private route
  table AZ1

  3. Private route table AZ2
   - Vpc_id        = practice-vpc
   - Route         =
      Destination = 0.0.0.0/0
      Target      = NAT Gateway-AZ2
   - Tags          = pri-RT-AZ2

  NOTE:Associate the pri-DB-sn2 and the pri-app-sn2 to the private route table AZ2

# STEP 6, CREATE THE SECURITY GROUPS.

    Navigate to the VPC console and click on security group and then create SG
    1. ALB security group
        - port: 80 & 443 / source: internet gateway

    2. SSH security group
        - port: 22 / source: MyIp

    2. Webserver security group
        - port: 80 & 443 / source: ALB security group
        - port: 22 / source: SSH security group

    3. Database security group
        - port: 3306 / source: Webserver secrity group

# STEP 7, CREATE AN S3 BUCKET.
    Navigate to the s3 console and create a bucket
     Name: ikeboy-1234
     Region: us-east-1
     All other setting is default and then create bucket
     Inside the bucket create two folder
        - Fleetcart folder
        - Dummy-file folder
    then upload the Fleetcart code in the fleetcart folder and the dummy data in to the dummy-file folder

# STEP 8, CREATE AN IAM ROLE THAT EC2 WILL USE TO ACCESS S3
    Navigate to the s3 console and click role
    select create role 
    trusted entity: AWS service
    Use caes: EC2
    Add permissions: AmazonS3ReadOnly
    create the role
    

# STEP 9, CREATE A KEYPAIR.
    1. Navigate to the EC2 console an keypair
     - select create key pair
     - Name - FleetCart-keypair
     - check RSA AND .pem
     - click create key pair and make sure you save the private key.

# STEP 10, CREATE AN RDS DATABASE.
    1. Navigate to RDS in your console and select subnet group
        - click on create DB subnet group
        - Name: practice-DB-SG 
        - Description: practice-DB-SG
        - VPC: practice-vpc
        - Availability Zones: select [a & b]
        - Subnets: select the 2 database subnets.

    2. Navigate to RDS service in the console and select database
       - click on create database
       - select: standard create
       - Engine: MYSQL
       - Version: lastest 
       - Template: Dev/test
       - Deployment option: multiAZ
       - DB instance identify: practice-rds-db
       - master username: Anthony
       - Password: myfirstname
       - DB instance type: Burstable classes
       - Storage type: General purpose
       - Allocated storage: 30gb
       - Storage Autoscaling: Enable
       - VPC: practice-vpc
       - DB subnet group: practice-DB-SG
       - Security group: database-SG
       - Availability zone: select 1b
       - Database authentication: password authentication
       - click on additional configuration
           initial database name:Application-db
       - Click create.


# STEP 11, CREATE A SETUP SERVER (EC2 INSTANCE)  
    1.  Navigate to the EC2 service in the console
       - click on launch an instance
       - Name:setup-server
       - AMI: Amazon linux 2
       - instance type: t2.micro
       - select a keypair
       - VPC: practice-vpc
       - subnet: pub-web-sn1
       - select the IAM role that was created in step 8.
       - security group
          FrondEnd ALB-SG
          Webserver-SG
          SSH-SG
       - click create

# STEP 12, SSH INTO THE SETUP SERVER AND DOWNLOAD THE FLEETCART FILE FROM S3
    1. Run all the commands in the setup-fleetcart txt file 
      copy the public IP address of the setup server and access it through a browser
      then connect the EC2 instance with the RDS in the web page.
      by provider all the imformation of your RDS in the web page

# STEP 13, IMPORT THE DUMMY DATA FOR THE WEB-SITE
    1. Navigate to the EC2 instance console 
      create a keypair
        Name: dummy-keypair

   2. create an EC2 instance
       - Name:dummy-server
       - AMI: Amazon linux 2
       - instance type: t2.micro
       - select a keypair: dummy-keypair
       - VPC: practice-vpc
       - subnet: pub-web-sn2
       - security group: SSH-SG
       - create the dummy server
       
    3. create a security group
         Name: dummy-SG
         VPC: practice-vpc
         port: 3306 / source: private IP address of the dummy server

    4.  Modify the RDS database that was created in step 10.
        Navigate to RDS and select modify
        scrol to security group and add the dummy-SG security group

    5. Download mysql workbench in your computer and set it up
       open it and select database 
       click connect to database 
       select the first drop down and seclect
         standard TCP/IP over SSH
         SSH Hostname: EC2 public DNS
         SSh Username: EC2-User
         SSH Key File: dummy-keypair
         MySQL Hostname: RDS endpoint
         Username: RDS username
         Password: RDS password
         and then connect to the mysql workbench

    6. Download this file into your computer 
       https://drive.google.com/file/d/1e1tK0phHUK4Tnb4hpgBtOY0zkSGLO0lh/view?usp=sharing
       then import the sql file into your database using mysql workbench
       close mysql workbench, delete the dummy server the dummy keypair
       modify the RDS database and delete the dummy security group.

# STEP 14, SSH BACK INTO THE SETUP SERVER AND DOWNLOAD THE DUMMY DATA FROM S3
    make sure that you are on the ec2 user home directory
    Run all the commands in the setup-dummy-data txt file
    copy the public IP address of your setup server and access the website from the browser.

# STEP 15, CREATE AND AMI FROM THE SETUP SERVER
    Navigate to the EC2 console select your setup server
    click action and scrol down
    select security and AMI
    Selete create AMI

# STEP 16 CREATE TWO WEBSERVERS IN THE PRIVATE APP AZ1 & AZ2 SUBNET WITH THE AMI IN STEP 15
    Navigate to the EC2 service in the console and create 2 instances
       - click on launch an instance
       - Name:setup-server
       - AMI: Use the AMI from STEP 15
       - instance type: t2.micro
       - select a keypair
       - VPC: practice-vpc
       - subnet: private-app-subnetAZ1 and private-app-subnetAZ2
       - select the IAM role that was created in step 8.
       - security group
          FrondEnd ALB-SG
          Webserver-SG
          SSH-SG
       - click create

# STEP 17, CREATE AN APPLICATION LOAD BALANCER

    1. Navigate to the EC2 console and select target group
      - click create target group
      - type : instance
      - target group name : fleetcart-tg
      - VPC : practice-vpc
      - protocol / port : HTTP/80
      - health check : HTTP
      - health check path : /
      - click on th advance drop down and scrol down
      - success code: 200,301,302
      - click next and select the two servers that was created in step 16 and select include pending below 
      - click create target group
      
    2.  Navigate to the EC2 console and select load balancers
     - click create load balancer
     - select apllication load balancer
     - name = fleetcart-ALB
     - internet-facing
     - IPv4
     - VPC = practice-vpc
     - select AZ1(public-web-subnetAZ1) & AZ2(public-web-subnetAZ2) 
     - select ALB security group
     - listener protocol/port = HTTP/80
     - default action = target group(fleetcart-tg)
     - then click create load balancer,
     - use the load balancer dns to access the website

# STEP 18, REGISTER FOR A NEW DOMAIN NAME AND CREATE A RECORD SET IN ROUTE 53

    1. Navigate to the Amazon Route 53 console
      - under register domain, type your propose domain name and click check
      - click continue and fill out the form and then click register

    2. Create a record in Route 35
     - Navigate to the Amazom route 53 console and select hosted zone
     - click on the domain name we just registered  
     - select create record
     - subdomain = www
     - record type = A record and check the Alias box
     - route traffic = Appliocation and classic load balancer
     - region = same region we created our ALB
     - routing policy = simple routing
     - click create records

# STEP 19, REGISTER FOR SSL CERTIFICATE IN AWS CERTIFICATE MANAGER

    Navigate to the ACM console 
    click request a certificate
    reqest a public certificate
    domain name: eneter the domain name in step 18
    in the next tab type, (*.anthony.net) my domain name
    validation method: DNS
    click request certificate and then click create record in route 53 in the ACM console.

# STEP 20, CREATE AN HTTPS LISTENER 

    1. Navigate to the EC2 console
       select load balancer and select the ALB we created in step 17
       click listener and select add listener
       protocol: HTTPS
       port: 443
       default action : forward
       target group: 
       SSL certificate : select the SSL that was created in step 19
       click add listener

    2. modify the HTTP listener to redirect traffic to HTTPS
       select the HTTP listener and click edit
       under default actions remove the rule 
       click the drop down and select redirect
       protocol: HTTPS
       port: 443
       click save changes
       now check your webside to see if it is secure by the ssl certificate
       https://www.anthony.net
       if you website is not loading properly, then we need to update the ENV file 

# STEP 21, CREATE A JUMP SERVER TO UPDATE THE DOMAIN NAME IN THE WEBSITE CONFIG FILE
  
    Navigate to the EC2 service in the console and create an ec2 instance
       - click on launch an instance
       - Name:setup-server
       - AMI: Amazon Linux 2
       - instance type: t2.micro
       - select a keypair
       - VPC: practice-vpc
       - subnet: public-web-subnetAZ1
       - security group
          SSH-SG
       - click create
       SSH into the server and then jump into the server in the private-app-subnetAZ1

# STEP 22, UPDATE THE ENV FILE IN THE WEBSITE
    Navigate to the EC2 console
    terminate the setup serverin the buplic subnet
    terminate the webserver in the private app subnet AZ2
    go to the terminal in the private app subnet and run
      - sudo su
     - cd /var/www/html
     - vim .env
    under APP_URL, paste this URL 
    https://www.anthony.net
    save the and restart the webserver.
    systemctl restart httpd.

# STEP 23, CREATE A NEW AMI
    Navigate to the EC2 console and select the webserver in the private app subnet
    select actions
    selecte image and template
    name: fleetcart v2
    create image
    Now delete the first AMI and the snapshot

# STEP 24, CREATE AN AUTO SCALING GROUP
    1. Navigate to the EC2 console and select launch templates
      - click create a launch template
      - name :  fleetcart-LT
      - version : fleetcart-LT-v2
      - auto scaling guidance, check
      - TAG : key(Name), value(fleetcart-LT)
      - AMI : AMI that was created in step 23 (fleetcart v2)
      - instance type : t2.micro
      - key pair: fleetcart-keypair
      - security group : webserver security group
      - resource tag : ASG-Webserver 
      - click create launch template

    CREATE AN AUTO SCALING GROUPS
      1. Navigate to the EC2 console and select auto scaling
      - click create auto scaling 
      - name : fleetcart-ASG
      - select a launch template : fleetcart-LT
      - VPC : practice-vpc
      - select the 2 pravite app subnets
      -  then click next
      - select attach to an exiting load balancer
       - choose from your load balancer target group, check
       - click the drop down, select taget groups and select(fleetcart-tg)
       - health check type : ELB
       - monitoring : Enable, then click next
       - group size 
         - Desire capacity : 2
         - Minimum capacity : 1
         - Maximum capacity : 4
      - scaling policy : None
      - clcik add notification
        - click create a Topic 
          - name : fleetcart-topic
          - input your email and check all the box
          - click next
          - tag : key(name), value(webserver)
          - review and launch
