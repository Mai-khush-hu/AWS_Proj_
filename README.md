# ☁️ Cloud Arcade: Gaming Website on AWS

**AWS Capstone Project | Group 53 | B.Tech CS (AI/ML)**

A browser-based mini arcade (Target Tap + Tic Tac Toe) built with plain HTML, CSS and JavaScript, hosted on AWS using **VPC, EC2, S3, IAM and EBS**. The same game is served from an Apache web server on EC2 and from an S3 static website.

## 👥 Team

| Name | Roll Number |
| --- | --- |
| Khushal Kumar | 202410101240073 |
| Suyash Srivastava | 202410101240088 |
| Hemant Kumar | 202410101240067 |

## 🎯 Objective

- **Host**: serve the game from an S3 bucket and from an Apache server on EC2
- **Network**: run the server inside a VPC with subnets, routing and an internet gateway
- **Secure**: control access with security groups and an IAM role
- **Store**: add an EBS volume for extra persistent storage

## 🧰 AWS Services Used

Region: **Europe (Stockholm), `eu-north-1`**

| Service | Details |
| --- | --- |
| Amazon VPC | `capstone-vpc` with 3 subnets, route table, internet gateway |
| Amazon EC2 | `capstone-web-1`, t3.micro, Amazon Linux 2023, Apache (httpd) on port 80 |
| Amazon S3 | Bucket `capstone-s3-arcade` holding `index.html`, static website hosting enabled |
| AWS IAM | Role `EC2-S3-Role` with `AmazonS3FullAccess`, attached to EC2 as an instance profile |
| Amazon EBS | 2 GiB volume mounted at `/data` |

## 🏗️ Architecture

```
                    Users (browser)
                    /             \
                   /               \
        Internet Gateway        S3 static website
                |               (direct, outside VPC)
   capstone-vpc / Subnet (eu-north-1b)
                |
     EC2 capstone-web-1 (Apache, port 80)
        |                      |
   EBS 2 GiB (/data)     IAM role EC2-S3-Role
                               |
                     S3 bucket capstone-s3-arcade
```

The EC2 instance uses the IAM role to read `index.html` from S3, so no access keys are stored on the server.

## 🛠️ Implementation Steps

### 1. VPC and networking
- Created `capstone-vpc` (`vpc-09bc1039a11168c84`), state **Available**
- Created 3 subnets inside the VPC
- Created the `capstone` route table, associated with the 3 subnets
- Attached internet gateway `igw-075eaa2f8491b7557` to the VPC

| VPC | Subnets |
| --- | --- |
| ![VPC](images/02-vpc.png) | ![Subnets](images/03-subnets.png) |

![Route table](images/04-route-table.png)
![Internet gateway](images/05-internet-gateway.png)

### 2. EC2 instance and security group
- Launched `capstone-web-1` (t3.micro, Amazon Linux 2023) in `capstone-vpc`, public IP `51.20.183.97`, **3/3 status checks passed**
- Security group `capstone-web-sg`:
  - Inbound: HTTP (80) and SSH (22)
  - Outbound: all traffic allowed

![EC2 running](images/01-ec2-instance-running.png)
![Security group rules](images/06-security-group-rules.png)

### 3. Apache web server
```bash
sudo dnf install -y httpd
sudo systemctl enable --now httpd
systemctl status httpd
```
Apache is **enabled, active (running)** and listening on port 80.

### 4. IAM role for EC2
- Created role `EC2-S3-Role` (trusted entity: EC2) with the `AmazonS3FullAccess` managed policy
- Attached it to the instance so EC2 can access S3 without stored credentials
- IAM users in the account include `Teammate1` and `Teammate2`

![IAM role](images/10-iam-role-ec2-s3.png)
![IAM users](images/09-iam-users.png)

### 5. S3 bucket and file upload
- Created bucket `capstone-s3-arcade` in `eu-north-1`
- Uploaded `index.html` (3.0 KB)

![S3 buckets](images/13-s3-buckets.png)
![S3 objects](images/14-s3-objects.png)
![S3 object details](images/15-s3-object-details.png)

### 6. EC2 pulls the game from S3 using the role
```bash
aws s3 ls
sudo aws s3 cp s3://capstone-s3-arcade/index.html /var/www/html/index.html
```

![Apache and S3 CLI](images/07-apache-and-s3-cli.png)

### 7. EBS volume
- Created and attached a 2 GiB EBS volume, mounted at `/data`
- Wrote a test file and read it back to confirm persistence

```bash
lsblk
df -h /data
cat /data/test.txt   # EBS volume working
```

![lsblk](images/11-ebs-lsblk.png)
![EBS mounted](images/12-ebs-mounted-test.png)

### 8. S3 static website hosting
Static website hosting is enabled on `capstone-s3-arcade`.

Endpoint: `http://capstone-s3-arcade.s3-website.eu-north-1.amazonaws.com`

![S3 static hosting](images/16-s3-static-website-hosting.png)

## 🎮 The Website

A single `index.html` using HTML, CSS and JavaScript:

- **Target Tap**: hit as many moving targets as possible in 20 seconds
- **Tic Tac Toe**: two players take turns, with win and draw detection

![Game source](images/08-game-source-code.png)

### Live on EC2 (`http://51.20.183.97`)

| Target Tap | Tic Tac Toe |
| --- | --- |
| <img src="images/17-live-target-tap.jpeg" width="300"> | <img src="images/18-live-tic-tac-toe.jpeg" width="300"> |

## 🧩 Challenges and Fixes

| Problem | Cause and fix |
| --- | --- |
| `curl` could not connect | No web server was listening on port 80. Installed and started Apache. |
| Package install got stuck | Security group had no outbound rule. Added one. |
| EC2 showed the old page | Uploading to S3 does not update EC2. Copied `index.html` across with the AWS CLI. |

## 🔒 Possible Improvements

- Limit SSH (port 22) to trusted IPs instead of `0.0.0.0/0`
- Give the IAM role access to one bucket only (least privilege) instead of `AmazonS3FullAccess`
- Add HTTPS using CloudFront
- Use a load balancer and a second Availability Zone
- Release the public IP and stop unused resources to avoid charges

## 📚 What We Learned

How compute (EC2), storage (S3, EBS), networking (VPC) and permissions (IAM) work together to put a website online.

## 📁 Repository Structure

```
.
├── README.md
├── index.html            # the game (download it from your S3 bucket)
├── Cloud_Arcade_Group53.pptx
└── images/               # project screenshots
```

---
*Group 53 | B.Tech CS (AI/ML) | AWS Capstone Project*
