week 1
Connect Ubuntu EC2 Using PuTTY (.ppk)
Step 1: Check Security Group
Go to EC2
Open Security Groups
Edit inbound rules
Allow:
SSH → Port 22 → My IP
Step 2: Open PuTTY
Launch PuTTY
In Host Name enter:
ubuntu@<PUBLIC-IP>
Step 3: Attach .ppk Key
Go to:
Connection → SSH → Auth → Credentials
Click Browse
Select .ppk file
Step 4: Connect
Click Open
Click Yes on security alert
Login successful
Step 5: Verify Connection

Run:

whoami
Convert .pem to .ppk Using PuTTYgen
Step 1: Open PuTTYgen
Launch PuTTYgen
Click:
Load
Select .pem file
Change file type to:
All Files
Step 2: Save .ppk
Click:
Save private key
Save as:
keyname.ppk
Connect EC2 Using AWS CloudShell
Step 1: Launch EC2
Create EC2 instance
Download .pem key
Step 2: Open CloudShell
Login to AWS Console
Click CloudShell icon
Step 3: Open EC2 Connect Command
Go to EC2
Select instance
Click:
Connect → SSH Client
Copy SSH command
Step 4: Upload .pem File
In CloudShell click:
Actions (+)
Click:
Upload File
Upload .pem file
Step 5: Connect to EC2
Paste SSH command
Run command
Create Windows EC2 and Connect Using RDP
Step 1: Launch Windows EC2
Go to EC2
Click Launch Instance
Select:
Windows Server AMI
Select instance type
Configure Security Group:
RDP → Port 3389 → My IP
Create/download key pair
Launch instance
Step 2: Get Public IP
Open EC2 Instances
Copy:
Public IPv4 / Public DNS
Step 3: Get Administrator Password
Select instance
Click:
Connect → RDP Client
Click:
Get Password
Upload .pem file
Click:
Decrypt Password
Copy password
Step 4: Connect Using RDP
Open:
Remote Desktop Connection (mstsc)
Enter Public IP
Click Connect
Enter:
Username → Administrator
Password → Decrypted Password
Login successfully


week 2
1(a). Create and Attach EBS Volume to EC2 Instance
Open AWS Console
Go to EC2 Dashboard
Click Volumes under Elastic Block Store
Click Create Volume
Select:
Volume Type → gp3/gp2
Size
Same Availability Zone as EC2
Click Create Volume
Select created volume
Click Actions → Attach Volume
Select EC2 instance
Give device name:
/dev/xvdf
Click Attach
Connect to EC2 using SSH
Verify volume:
lsblk
1(b). Increase CPU and RAM by Changing Instance Type
Go to EC2 Dashboard
Select EC2 instance
Click Instance State → Stop Instance
After stopping:
Click Actions
Instance Settings
Change Instance Type
Select higher instance type
Click Apply
Start the instance
Verify increased CPU/RAM
2. Attach and Permanently Mount EBS Volume in Linux EC2
Step 1: Create Volume
EC2 → Volumes
Click Create Volume
Select:
gp3
20 GB
Same AZ
Click Create Volume
Step 2: Attach Volume
Select volume
Actions → Attach Volume
Select EC2 instance
Device name:
/dev/xvdf
Click Attach
Step 3: Connect to EC2
ssh -i key.pem ec2-user@public-ip
Step 4: Verify Disk
lsblk
lsblk -fs
Step 5: Create Partition
sudo fdisk /dev/nvme1n1

Inside fdisk:

n
p
Enter
Enter
Enter
w

Then:

sudo partprobe
Step 6: Verify Partition
lsblk -fs
Step 7: Format Partition
sudo mkfs.xfs /dev/nvme1n1p1
Step 8: Create Mount Directory
sudo mkdir /mnt/archana
Step 9: Mount Volume
sudo mount /dev/nvme1n1p1 /mnt/archana

Verify:

df -h
Step 10: Permanent Mount Using fstab

Get UUID:

sudo blkid /dev/nvme1n1p1

Edit fstab:

sudo nano /etc/fstab

Add:

UUID=your-uuid /mnt/archana xfs defaults,nofail 0 0

Test:

sudo mount -a

Create file:

cd /mnt/archana
touch f1
Step 11: Verify Persistence

Reboot:

sudo reboot

After reconnect:

df -h
3. Create Snapshot and Attach Volume in Another Region
Step 1: Create Snapshot
EC2 Dashboard
Volumes
Select volume
Actions → Create Snapshot
Enter name/description
Click Create Snapshot
Step 2: Copy Snapshot to Another Region
Go to Snapshots
Select snapshot
Actions → Copy Snapshot
Select destination region
Click Copy Snapshot
Step 3: Monitor Snapshot
Check snapshot status
Wait until status becomes:
completed
Step 4: Switch Region
Change AWS region from top-right corner
Step 5: Create Volume from Snapshot
Open Snapshots
Select copied snapshot
Actions → Create Volume
Select:
Volume Type
Size
Availability Zone
Click Create Volume
Step 6: Attach Volume to EC2
Go to Volumes
Select new volume
Actions → Attach Volume
Select EC2 instance
Provide device name
Click Attach


week 3
Elastic File Storage (EFS) Lab Steps
Step 1: Launch EC2 Instances
Create EFS-1
Open AWS Console
Go to EC2
Click Launch Instance
Enter name:
EFS-1
Select Amazon Linux AMI
Select instance type:
t2.micro
Create/select key pair
Select subnet:
Subnet-1a
Create Security Group
Allow:
NFS
SSH
Launch instance
Create EFS-2
Launch another instance
Name:
EFS-2
Select:
Subnet-1b
Use same Security Group
Launch instance
Step 2: Create EFS File System
Open EFS service
Click:
Create file system
Select:
Default VPC
Enable region option
Click Next
Remove default security group
Add same NFS security group used for EC2
Click Next
Click Create File System
Step 3: Connect to Both EC2 Instances
Connect to EFS-1
ssh -i key.pem ec2-user@public-ip

Switch root:

sudo su

Create directory:

mkdir efs

Install EFS utilities:

yum install -y amazon-efs-utils

Check files:

ls

Check version:

efs-utils --version
Repeat Same Steps for EFS-2
Step 4: Attach EFS to EC2
Open EFS service
Select created EFS
Click:
Attach
Select:
Mount via DNS
Copy mount command
Run command in EFS-1 terminal
Run same command in EFS-2 terminal
Step 5: Verify EFS
On EFS-1

Go to mounted directory:

cd efs

Create file:

touch file1
On EFS-2

Go to mounted directory:

cd efs

Check files:

ls

Verify:

file1

appears automatically.

week 4 
1. Create S3 Bucket and Enable Public Access
Step 1: Create Bucket
Open AWS Console
Go to S3
Click:
Create bucket
Enter bucket name
Select region
Uncheck:
Block all public access
Acknowledge warning
Click:
Create bucket
Step 2: Enable Bucket Public Access
Open bucket
Go to:
Permissions
Open:
Access Control List (ACL)
Click:
Edit
Enable:
Everyone → Read access
Save changes
Step 3: Upload Object
Open:
Objects
Click:
Upload
Add file
Click Upload
Step 4: Enable Object Public Access
Open uploaded object
Go to:
Permissions
Open ACL
Click Edit
Enable:
Everyone → Read access
Save changes
Step 5: Verify Public Access
Copy object URL
Open in browser/incognito
2. Enable Versioning in S3
Open S3
Select bucket
Go to:
Properties
Scroll to:
Bucket Versioning
Click:
Edit
Select:
Enable
Save changes
Test Versioning
Upload file
Upload same file again
Observe different version IDs
Delete file
Observe delete marker
3. Cross-Region Replication (CRR)
Step 1: Create Destination Bucket
Create another S3 bucket
Select different region
Enable versioning
Step 2: Create Replication Rule
Open source bucket
Go to:
Management
Open:
Replication rules
Click:
Create replication rule
Step 3: Configure Rule
Enter rule name
Enable rule
Select:
Apply to all objects
Step 4: Select Destination Bucket
Choose:
Another bucket
Select destination bucket
Use AWS managed role/default role
Save rule
Step 5: Verify Replication
Upload file in source bucket
Open destination bucket
Verify replicated object
4. Static Website Hosting in S3
Step 1: Create Bucket
Open S3
Click Create bucket
Enter bucket name
Select region
Uncheck:
Block all public access
Create bucket
Step 2: Upload Website Files
Open bucket
Click Upload
Upload:
index.html
error.html
Step 3: Enable Static Website Hosting
Go to:
Properties
Scroll to:
Static website hosting
Click Edit
Enable hosting
Enter:
Index document → index.html
Error document → error.html
Save changes
Step 4: Enable Bucket-Level Public Access
Go to Permissions
Open ACL
Click Edit
Enable:
Everyone → Read
Save changes
Step 5: Enable Object-Level Public Access
For index.html
Open object
Permissions
ACL → Edit
Enable:
Everyone → Read
Save
Repeat same for:
error.html
Step 6: Access Website
Go to Properties
Copy:
Bucket website endpoint
Open in browser
Step 7: Verify Error Page
Open invalid URL:
/invalid.html
Verify error.html loads


week 5
VPC Lab Steps
Step 1: Create VPC
Open AWS Console
Go to:
VPC
Click:
Create VPC
Select:
VPC only
Enter:
Name → Custom-VPC
IPv4 CIDR → 10.0.0.0/16
Click Create VPC
Step 2: Create Subnets
Create Public Subnet
Go to:
Subnets
Click:
Create subnet
Select VPC
Enter subnet details
Enter CIDR block
Create subnet
Create Private Subnet
Click Create subnet
Select same VPC
Enter subnet details
Enter CIDR block
Create subnet
Step 3: Create Internet Gateway
Go to:
Internet Gateways
Click:
Create internet gateway
Enter:
Custom-IGW
Create IGW
Click:
Attach to VPC
Select VPC
Attach
Step 4: Create Route Table for Public Subnet
Go to:
Route Tables
Click:
Create route table
Select VPC
Create route table
Add Internet Route
Open Route Table
Go to:
Routes
Click:
Edit routes
Add:
Destination → 0.0.0.0/0
Target → Internet Gateway
Save routes
Associate Public Subnet
Go to:
Subnet Associations
Click Edit
Select public subnet
Save
Step 5: Enable Auto Public IP
Select Public Subnet
Click:
Edit subnet settings
Enable:
Auto-assign public IPv4
Save
Step 6: Create Security Group
Go to EC2
Open:
Security Groups
Click:
Create security group
Select VPC
Add inbound rules:
HTTP → 80
HTTPS → 443
SSH → 22
Save security group
Step 7: Launch EC2 Instances
Launch Public EC2
Launch EC2
Select:
Public Subnet
Enable public IP
Attach security group
Launch instance
Launch Private EC2
Launch another EC2
Select:
Private Subnet
Disable public IP
Attach security group
Launch instance
Step 8: Verify Setup
Public EC2
SSH into instance
Test internet:
ping google.com
Private EC2
Verify:
No direct internet access

week 6
VPC Using Bastion Server Steps
Step 1: Create VPC
Open AWS Console
Go to:
VPC
Click:
Create VPC
Enter:
CIDR → 10.0.0.0/16
Create VPC
Step 2: Create Subnets
Public Subnet
Go to Subnets
Click Create Subnet
Enter:
10.0.1.0/24
Create subnet
Private Subnet
Click Create Subnet
Enter:
10.0.2.0/24
Create subnet
Step 3: Create Internet Gateway
Go to:
Internet Gateways
Click Create
Attach IGW to VPC
Step 4: Configure Route Table for Public Subnet
Create Route Table
Add route:
0.0.0.0/0 → Internet Gateway
Associate Public Subnet
Step 5: Enable Auto Public IP
Select Public Subnet
Edit subnet settings
Enable:
Auto Assign Public IP
Step 6: Create Security Groups
Bastion Security Group

Allow:

SSH (22) → My IP
DB Server Security Group

Allow:

MySQL (3306) → 10.0.1.0/24
SSH (22) → Bastion Private IP
Step 7: Launch Bastion Server
Launch EC2
Select:
Ubuntu
Select Public Subnet
Attach Bastion SG
Create key:
bastion.pem
Launch instance
Step 8: Launch Database Server
Launch EC2
Select:
Ubuntu
Select Private Subnet
Attach DB Security Group
Create key:
dbserver.pem
Launch instance
Step 9: Copy DB Key to Bastion Server

Run from local system:

scp -i bastion.pem dbserver.pem ubuntu@<BASTION_PUBLIC_IP>:/home/ubuntu/
Step 10: Connect to Bastion Server
ssh -i bastion.pem ubuntu@<BASTION_PUBLIC_IP>

Check files:

ls
Step 11: Connect from Bastion to DB Server
ssh -i dbserver.pem ubuntu@10.0.2.x
NAT Gateway Steps
Step 12: Create NAT Gateway
Go to:
NAT Gateways
Click:
Create NAT Gateway
Select:
Public Subnet
Allocate Elastic IP
Click Create
Step 13: Create Route Table for Private Subnet
Go to Route Tables
Click Create Route Table
Select VPC
Create route table
Step 14: Associate Private Subnet
Open Route Table
Edit Subnet Associations
Select Private Subnet
Save
Step 15: Add NAT Route
Open Routes
Click Edit Routes
Add:
0.0.0.0/0 → NAT Gateway
Save changes
Step 16: Verify Internet in Private EC2

Inside DB Server:

sudo apt update

or

ping google.com
