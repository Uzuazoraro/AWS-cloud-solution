## AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

1. Properly configure your AWS account and Organization Unit

### STEPS

Open AWS console > Click on AWS logo on top left corner > Click on Services > AWS Organizations (Management & Governance) > Create Organization

Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this):
Click on Add account > Select create AWS account > Fill in the details >

![creation of aws organization](/aws org created.png)

Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)

Click on Root > click Action under children > Click create new > Type "Dev" in the org unit name > click on create org unit.

![organization unit created](/Dev org unit.png)

Move the DevOps account into the Dev OU.
![Moving of one account to another](/DevOps moved to Dev.png)

Login to the newly created AWS account using the new email address.

2 - Create a free domain name for your fictitious company at Freenom domain registrar here.

![creation of a new domain name "zireuz"](/domain name.png)

3 - Create a hosted zone in AWS, and map it to your free domain from Freenom.

Click Route 53 > Create hosted zone > Fill the blank space > select public hosted zone > create hosted zone > copy the value \/route traffic to > go to freenom > click manage domain > management tools > nameservers > use custom nameserver (use to map the name servers) > paste the copied value/route traffic to in the nameserver in freenom > click on change nameserver.

![mapping of hosted zone](/map hosted zone to freenom.png)

Go back to AWS > Click on create record >

### SET UP A VIRTUAL PRIVATE NETWORK (VPC)

Virtual Private Cloud > Click on Your VPC > Click Create VPC > Fill in the details (name of my vpc is Micolo-VPC) > IPv4 CIDR (10.0.0.0/16) > Click Create VPC

![VPC created](/Create VPC.png)

Enable the Hostname: Click Action > Click Edit > Select Enable > Click ok.

### Create Internet Gateway

Click on Internet gateways > Create Internet gateway (name it Micolo-IGW)> Click create internet gateway > Click Attach to VPC

![Creation of internet gateway](/Internal gateway created.png)

### Create subnet

Click on subnets > Click on create subnet > select your vpc from the dropdown > create 2 public subnets and 4 private subnets
Example: Micolo-public-subnet-1
10.0.0.0/24
10.0.2.0/24

![creation of subnet](/subnet created.png)

### Create route table

Click Route table > Create route table > Fill in the details for both public and private route table.

![Creation of route tables](/public route table created.png)

Associate the route tables with the subnets.

Click the route table > select the public rtb > click subnet associations > click edit subnet associations > select the public subnets 1 & 2 > click save association.

Select the private rtb > click subnet association > click edit subnet associations > select the 4 private subnets > click save association

### Edit the route of the route tables

Click route table > select public rtb > click Actions > click edit routes > click Add route > Destination (0.0.0.0/0) > Target (internet gateway) > click save changes
This means that every subnet associate with this route table will talk to the internet via the gateway.

### Create Elastic IP

Click Elastic IP > click allocate elastic IP > Add new tag (key - Name; Value - Micolo -NAT)

### Create NAT-Gateway

Click Nat gateways > Create NAT gateway > Name (Micolo-Natgateway) > Subnet (should be public subnet) > Select the elastic IP > Click Create Natgateway.

![Creation of Natgateway](/Natgateway created.png)

### Go back to the route table

Click on route table > Click private route table > Actions > Edit routes > Add route > Destination (0.0.0.0/0) > Target (NATgateway) > Save changes.

### Security Group

Click Security Group > Create security group > Fill in the details
Inbound rules > Add rule > HTTPS & HTTP from anywhere, ie 0.0.0.0/0 > Save
Outbound rule > All traffic > Destination (anywhere), ie 0.0.0.0/0 > Save

### Tags

Key (Name) > Value (Micolo-ext-security) > Save

### Create another Security Group for Bastion

Click security group > Create security group (Fill the basic details) > click inbound rules > Type (SSH), Source (My IP) > Tags > Key (Name), Value (Micolo-Bastion > Create security group)

![Creation of Bastion security group](/Bastion security group.png)

### Create security group for Nginx

Follow same process like the previous ones that were created.

Inbound rules > Type (Http), Source (Select the created security source for load balancer)
Inbound rule > Type (https), Source (Select the created security source for load balancer)
Inbound rule > Type (SSH), Source (Select the created Bastion security from the drop down)

### Create security group for Internal Load Balancer

Security group name: Micolo-Internal-ALB
VPC: Select the created VPC

Inbound rules > Type (http), Source (Select Nginx security from the drop down)
Inbound rules > Type (WinRM-HTTPS), Source (Select Nginx Security from the drop down)

### Create security group for the webserver

Security group name: Micolo-webserver
VPC: Select the created VPC
Inbound rule > Type (SSH), Source (Bastion security)
Inbound rule > Type (Https), Source (Micolo-Internal-ALB)
Inbound rule > Type (Http), Source (Micolo-Internal-ALB)

### Create security group for datalayer

Security group name: Micolo-datalayer
VPC: Select the created VPC
Inbound rule > Type (Mysql/Aurora), Source (Bastion security)
Inbound rule > Type (NFS), Source (webserver security)
Inbound rule > Type (Mysql/Aurora), Source (webserver security)

### AWS Certificate Manager

Request Certificate > Public certificate > Create records in route 53

![Creation of certificates](/AWS public certificate.png)

### Amazon Elastic File System (EFS)

Create file system > Name it, "Micolo-filesystem > Select the created VPC > Click customize > Tags: key (Name), Value (Micolo-filesystem) > Click Next > Security group (Select Micolo-datalayer) > Next > Create

![Creation of file system](/file system created.png)

### Access Points

Access point for wordpress:

Click on the created file system > Click on Access points > click create access point > Name (wordpress) > Root directory path (/wordpress) > user id and group id (value is 0) > Permission (0775) > Tag key (Name) > Tag value (wordpress-ap) > create access point.

![Creation of access points](/access points created.png)

Access point for tooling:

Follow same procedure as in wordpress.

![Creation of access points](/access points created for wordpress and tooling.png)

### AWS Key Management Service

Click on create a key > key type (leave at default) > Alias (Micolo-RDS) > Description (For the RDS Instance) > Tag key (Name) > Tag value (Micolo-rds-key) > Next > Finish

![Creation of RDS key](/RDS key created.png)

### Create subnet group

Click on or type Amazon RDS > Click subnet groups > Click Create DB subnet group > name it "Micolo-rds-subnet > Description (For RDS subnets) > Choose the created VPC > Add subnets (select the availability zones you created) > subnets (Add the private subnets you created) > Click create.

![Creation of subnets group](/RDS subnet groups created.png)

### Create Database

Go back to your Amazon RDS

Click dashboard > Click create database > Engine options (Select MySql) > Templates (Select free tier because the other two are very expensive) > Settings (Use "Micolo-database" as idenfier) > Credential settings (Micolo-admin) > Master password (passWord.1) > Leave other settings at default > Connectivity (select your created VPC) > public access (select No) > Existing security group (change to your created "datalayer") > Availability zone (choose either of your created zones) > Leave it at "password authentication" > Click additional configuration > initial database name (test) > Leave every other thing at default > create database

![database successfully created](/database created.png)

### Proceed With Compute Resources

Click on Launch Instance > Select RedHat Instance > Number of Instances (3) > Configure and launch Instance

![Launching of EC2 Instance](/EC2 Instance launched.png)

### Connect and configure bastion instance

Change to super super user with this command:
`sudo su`

`yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`
![Installation of epel-release](/Install epel-release.png)

`yum install -y dnf-utils https://rpms.remirepo.net/enterprise/remi-release-8.rpm`
![Installation of remi-repo](/remi-repo installed.png)

`yum install wget vim python3 telnet htop git mysql net-tools chrony -y`
![softwares installation successful](/installation of various softwares.png)

`systemctl start chronyd`
`systemctl enable chronyd`
![chronyd suuccessful](/chronyd started and enabled.png)

### Connect and configure nginx instance

Connect EC2 Instance and change to super user with this command:
`sudo su`
`yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`
`yum install -y dnf-utils https://rpms.remirepo.net/enterprise/remi-release-8.rpm`
`yum install wget vim python3 telnet htop git mysql net-tools chrony -y`
`systemctl start chronyd`
`systemctl enable chronyd`

### configure selinux policies for the webservers and nginx

`setsebool -P httpd_can_network_connect=1`
`setsebool -P httpd_can_network_connect_db=1`
`setsebool -P httpd_execmem=1`
`setsebool -P httpd_use_nfs 1`

![Installation of security policies](/Policies installed.png)

### This section will install amazon efs utils for mounting the target on the Elastic File System

`git clone https://github.com/aws/efs-utils`
`cd efs-utils`
`yum install -y make`
`yum install -y rpm-build`
`make rpm`
`yum install -y ./build/amazon-efs-utils*rpm`

![part installation of amazon efs utils](/amazon efs utils installation.png)
![continued installation of amazon efs utils](/amazon efs utils being installed.png)

### setting up self-signed certificate for the apache webserver instance

`yum install -y mod_ssl`
`openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -out /etc/ssl/certs/ACS.crt`
`vi /etc/httpd/conf.d/ssl.conf`

### setting up self-signed certificate for the nginx instance

`sudo mkdir /etc/ssl/private`
`sudo chmod 700 /etc/ssl/private`
`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt`
`sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048`

![generation of nginx self-signed certificate](/nginx certificate generated.png)
![self-signed nginx certficate confirmed](/confirmation of self-signed nginx certificate.png)

### Login into the database and databases for wordpress and tooling wordpress and tooling database

`mysql -h acs-database.cdqpbjkethv0.us-east-1.rds..amazonaws.com -u ACSadmin -p

`CREATE DATABASE toolingdb;`
`CREATE DATABASE wordpressdb;`

### webserver

`yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`
`yum install -y dnf-utils https://rpms.remirepo.net/enterprise/remi-release-8.rpm`
`yum install wget vim python3 telnet htop git mysql net-tools chrony -y`
`systemctl start chronyd`
`systemctl enable chronyd`

### configure selinux policies for the webserver

`setsebool -P httpd_can_network_connect=1`
`setsebool -P httpd_can_network_connect_db=1`
`setsebool -P httpd_execmem=1`
`setsebool -P httpd_use_nfs 1`

### This section will install amazon efs utils for mounting the target on the Elastic File System

`git clone https://github.com/aws/efs-utils`
`cd efs-utils`
`yum install -y make`
`yum install -y rpm-build`
`make rpm`
`yum install -y ./build/amazon-efs-utils*rpm`

### setting up self-signed certificate for the apache webserver instance

`yum install -y mod_ssl`
`openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -out /etc/ssl/certs/ACS.crt`
`vi /etc/httpd/conf.d/ssl.conf`

Edit this "SSLCertificateFile /etc/pki/tls/certs/localhost.crt" in the file to:
"SSLCertificateFile /etc/pki/tls/certs/ACS.crt"

This is done by changing the localhost to ACS in the file in multiple places.

### Create an AMI for each of the Instance

Go to Action > Images & Templates > Create image > Image name (ACS-webserver-ami) > Image description (for webserver) > Tag key (Name) > Tag value (ACS-webserver-ami)

![ami images created for instances](/creating ami for instances.png)

![ami images created](/ami created for instances.png)

### Create target groups

Select target groups under load balancing > create target groups > target group name (ACS-nginx-target) > Protoco (https 443) > VPC (Selected the created VPC - IE Micolo-vpc) > health check path (/healthstatus) > tag key (name) > tag value (ACS-nginx-target) > create target group

![target groups being created for instances](/creating target groups.png)

Follow same process and create target group for wordpress and tooling.

### Create loadbalancers

Create the external loadbalancers:

Click on loadbalancers > Create load balancers > select Application Load Balancer (ALB) and click create > Name (ACS-Ext-ALB) > Network mapping (Select the created VPC) > Select the 2 availability zones > Select public subnets (Bcos our external LB has to be in a public subnets) > Security group (select the external created security group) > Listener & routing (Select https 443) > Select the nginx target group > Create loadbalancer.

Create the internal loadbalancer:

Create loadbalancers > Create > Name (ACS-int-ALB) > Scheme (Internal) > Select the created VPC > Select the 2 availability zone > Select 2 private subnets (Because the internet LB resides in a private subnets) > Security group (Select the internal load balancer) > Listeners & routing (Select https 443) > target group (Select the ACS-wordpress as the default target) > Create

![Creation of loadbalancers](/Internal & External LB created.png)

Let's check for host headers

Select the ACS-int-ALB > Click on Listeners > click on view/edit rules > click on the + sign > click on insert rule > click add condition > host header >

![view/edit rules](/ACS-int-ALB.png)

### Launch Templates

=== For Bastion

Click launch template > Name (ACS-bastion-template) > template version description (for bastion) > Amazon Machine Image (select the bastion ami you created) > Instance type (select t2.micro) > key pair name (Micolo) > Resource tags (Key: Name, Value: ACS-bastion-template) > Network settings (subnet: select public subnet 1 or 2 for bastion, Security group: Micolo-bastion) > Advance network configuration (Auto-assign public IP: Enable) > Advance details (Bastion user data: #!/bin/bas, yum install -y mysql, yum install -y git tmux, yum install -y ansible) > Create launch template

![successfully created launch template](/launch template created.png)

=== For Nginx

Click Launch Template > Name (ACS-nginx-template) Description (for nginx) > AMI (Select the ami created for nginx) > Instance type (t2.micro) > key pair (Micolo) > Resource tags (key: Name, Value: ACS-nginx-template) > Network settings (subnet: slect public subnet 1 or 2, Security group: Micolo-nginx) > Adv network config (Auto-assign public IP: Enable) > Adv details (Nginx user data:

# !/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone <https://github.com/Livingstone95/ACS-project-config.git>
mv /ACS-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /ACS-project-config

> Create launch template

![creation of templates](/nginx launch template created.png)

=== For wordpress

Same procedure as in bastion and nginx.
The ami should be our webserver
The subnet should be a private subnet because it's a webserver.
Auto-assign public IP should be disabled.

wordpress userdata

#! /bin/bash/
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-0b8830fec3ed3e860 fs-03f85bf06fe118121:/ /var/www/
yum install -y httpd
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start fpm
systemctl enable fpm
wget <http://wordpress.org/latest.tar.gz>
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/micolo-database.ccztgcogh9jc.us-east-1.rds.amazonaws.com/g" wp-config.php
sed -i "s/username_here/MicoloAdmin/g" wp-config.php
sed -i "s/password_here/passWord.1/g" wp-config.php
sed -i "s/database_name_here/wordpressdb/g" wp-config.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

=== For tooling

ACS-tooling-template
Same procedure as in wordpress
Subnet should be private because it is a webserver

Tooling user data

#! /bin/bash/
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-0238153a7ce7b5c02 fs-03f85bf06fe118121:/ /var/www/
yum install -y httpd
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start fpm
systemctl enable fpm
git clone <https://github.com/Livingstone95/tooling-1.git>
mkdir /var/www/html/
cp -R /tooling-1/html/* /var/www/html/
cd /tooling-1
mysql -h acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com -u MicoloAdmin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com', 'MicoloAdmin', 'passWord.1', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

![creation of templates](/ACS-tooling-template.png)

### Create Auto Scaling Group

=== For Bastion

Click Auto scaling group > Click create auto scaling group > Name (ACS-bastion) > Launch template (ACS-bastion-template) > Click Next > VPC (Select the created VPC) > AZ & Subnets (Select public subnet 1 & 2) > Click Next > Check ELB > Click Next > Check target tracking scaling policy > Target value (increase it to 90) > Click Next > Click Add notifications > click create topic > Click next > Add tags (Key: Name, Value: ACS bastion) > Click Next > Click Create Auto Scaling Group

For Nginx:

Follow same process from biginning > Load balancer option (check attach to an existing load balancer) > Select nginx target > check ELB > Click next > select target tracking scaling policy > Target vakue (Increase to 90) > click Next > click Add notifications (Name: ACS-notification, Email: micaho2000@outlook.com) > Add tags (Key: Name, Value: ACS-nginx) > Click create auto scaling group.

### Login to AWS bastion

Go to RDS database > lick on Databases > Under DB identifier, click on the created database > Click on Connectivity & security > Copy the command at endpoint and paste it in your terminal after typing mysql -h

Use the complete command below:

`mysql -h micolo-database.ccztgcogh9jc.us-east-1.rds.amazonaws.com -u MicoloAdmin -p`

![Bastion database updated](/Bastion database.png)

`create database wordpressdb;`
`create database toolingdb;`

![creation of databases](/wordpress & tooling databases created.png)

### Create Auto Scaling Group

=== For wordpress

Click Auto scaling group > click create auto scaling group > Name (ACS-wordpress) > Launch template (select the wordpress template) > Click Next > VPC under Network (select the created VPC) > subnets (select private subnet 1 & 2) > Click Next > Load Balancing (Select attach to an existing LB and choose the wordpress target) > select ELB > click Next > Scaling policies (choose tracking scaling) > Target value (increase size to 90) > click Next > click Add notification > Click Next > Add tag (Key: Name, Value: ACS-wordpress) > click create auto scaling group.

![creation of wordpress autoscaling group](/wordpress autoscaling group created.png)

=== For Tooling

Follow same procedure.

![creation of tooling autoscaling group](/tooling autoscaling group created.png)

### Go to route 53

Create records:

=== For tooling:

Click hosted zone > click domain name > click create record > record name (tooling) > select Alias > select alias to App & Classic LB > Region (select US East North Virginia) > Select external load balancer.

Follow same procedure by adding records:

===  For www.tooling
=== For wordpress
=== www.wordpress

![creation of records in route 53](/creation of records.png)
![various records created](/records created in route 53.png)

### Health Status

![unhealthy target](/wordpress target unhealthy.png)
![tooling target showing unused](/tooling target unused.png)
![nginx target is draining](/nginx target draining.png)

### Troubleshoot unhealthy wordpress target

Run the Instance in your terminal using the private IP.
Login with the bastion public IP
ssh ec2-user@"wordpress private IP"

