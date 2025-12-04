# AWS Load-Balanced Web Application

A production-ready, highly available web application deployed on AWS using Infrastructure as Code principles.

![Project Banner](https://img.shields.io/badge/AWS-Cloud-orange) ![Ansible](https://img.shields.io/badge/Ansible-Automation-red) ![Status](https://img.shields.io/badge/Status-Complete-green)

## 🚀 Project Overview

This project demonstrates the deployment of a load-balanced web application across multiple AWS availability zones with automated configuration management using Ansible.

### Key Features

- ✅ Multi-AZ deployment for high availability
- ✅ Application Load Balancer for traffic distribution
- ✅ Automated deployment with Ansible
- ✅ Secure VPC configuration
- ✅ Comparison between EC2 and S3 hosting

## 🏗️ Architecture
```
┌─────────────────────────────────────────────────────────┐
│                    Internet Gateway                      │
└──────────────────────┬──────────────────────────────────┘
                       │
           ┌───────────▼───────────┐
           │  Application Load     │
           │     Balancer          │
           └───────┬───────┬───────┘
                   │       │
        ┏━━━━━━━━━━┷━━━━┓  ┃
        ┃   us-east-1a   ┃  ┃   us-east-1b
        ┃                ┃  ┃
    ┌───▼────┐      ┌────▼───┐
    │  EC2   │      │  EC2   │
    │ NGINX  │      │ NGINX  │
    └────────┘      └────────┘
    10.0.1.0/24     10.0.2.0/24
```

## 📋 Table of Contents

- [Technologies Used](#technologies-used)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Deployment Guide](#deployment-guide)
- [Configuration Files](#configuration-files)
- [Testing](#testing)
- [Comparison: EC2 vs S3](#comparison-ec2-vs-s3)
- [Screenshots](#screenshots)
- [Lessons Learned](#lessons-learned)
- [Future Enhancements](#future-enhancements)
- [Author](#author)

## 🛠️ Technologies Used

- **Cloud Provider:** Amazon Web Services (AWS)
  - EC2 (Elastic Compute Cloud)
  - VPC (Virtual Private Cloud)
  - ALB (Application Load Balancer)
  - S3 (Simple Storage Service)
  - Security Groups
  
- **Automation:** Ansible 2.x
- **Web Server:** NGINX 1.24
- **Operating System:** Ubuntu 24.04 LTS
- **Version Control:** Git & GitHub

## 📦 Prerequisites

- AWS Account (Free Tier eligible)
- Basic understanding of:
  - Linux command line
  - Networking concepts (VPC, subnets, security groups)
  - SSH and key pairs
  - YAML syntax

## 📁 Project Structure
```
cloud-devops-capstone-project/
├── ansible/
│   ├── ansible.cfg           # Ansible configuration
│   ├── inventory.ini         # Server inventory
│   └── playbook.yml          # Deployment playbook
├── documentation/
│   └── deployment-guide.md   # Detailed deployment steps
├── architecture/
│   └── architecture-diagram.png
├── screenshots/
│   ├── alb-web-server-1.png
│   ├── alb-web-server-2.png
│   ├── s3-static-website.png
│   └── aws-console-ec2.png
├── index.html                # Main application (EC2/ALB)
├── index-s3.html            # S3 static website version
└── README.md                # This file
```

## 🚀 Deployment Guide

### Phase 1: Network Infrastructure

1. **Create VPC**
   - CIDR: 10.0.0.0/16
   - Enable DNS hostnames

2. **Create Subnets**
   - Public Subnet 1: 10.0.1.0/24 (us-east-1a)
   - Public Subnet 2: 10.0.2.0/24 (us-east-1b)

3. **Configure Internet Gateway**
   - Attach to VPC
   - Update route tables

4. **Create Security Groups**
   - ALB Security Group: HTTP from 0.0.0.0/0
   - EC2 Security Group: HTTP from ALB, SSH from My IP

### Phase 2: EC2 Instances

1. **Launch Instances**
   - AMI: Ubuntu 24.04 LTS
   - Instance Type: t2.micro
   - Count: 2 (one per AZ)
   - User Data: Install Python3

2. **Generate SSH Key Pair**
   - Store securely in `~/.ssh/`

### Phase 3: Ansible Automation

1. **Install Ansible**
```bash
   sudo apt update
   sudo apt install ansible -y
```

2. **Configure Ansible**
   - Update `ansible/inventory.ini` with your instance IPs
   - Update `ansible/ansible.cfg` with your key path

3. **Run Playbook**
```bash
   cd ansible
   ansible-playbook playbook.yml
```

### Phase 4: Load Balancer

1. **Create Target Group**
   - Protocol: HTTP
   - Port: 80
   - Health check path: /

2. **Create ALB**
   - Internet-facing
   - Select both subnets
   - Attach target group

3. **Test**
   - Access ALB DNS name
   - Refresh to see load balancing

### Phase 5: S3 Static Website

1. **Create S3 Bucket**
   - Unique name
   - Disable block public access

2. **Enable Static Hosting**
   - Index document: index.html

3. **Upload Files**
   - Upload `index-s3.html` as `index.html`
   - Set public read permissions

## ⚙️ Configuration Files

### Ansible Inventory (`ansible/inventory.ini`)
```ini
[webservers]
web1 ansible_host=<PUBLIC-IP-1> private_ip=10.0.1.x
web2 ansible_host=<PUBLIC-IP-2> private_ip=10.0.2.x

[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/cloud-devops-key.pem
ansible_python_interpreter=/usr/bin/python3
```

### Ansible Configuration (`ansible/ansible.cfg`)
```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
remote_user = ubuntu
private_key_file = ~/.ssh/cloud-devops-key.pem
```

## 🧪 Testing

### Test Load Balancer
```bash
# Test ALB DNS
curl http://<ALB-DNS-NAME>

# Test multiple times to see different servers
for i in {1..10}; do curl -s http://<ALB-DNS-NAME> | grep "Private IP"; done
```

### Test Individual Servers (via SSH)
```bash
# SSH into server
ssh -i ~/.ssh/cloud-devops-key.pem ubuntu@<PUBLIC-IP>

# Check NGINX status
systemctl status nginx

# Test local response
curl http://localhost
```

## 📊 Comparison: EC2 vs S3

| Feature | EC2 + ALB | S3 Static Website |
|---------|-----------|-------------------|
| **Setup Complexity** | High | Low |
| **Cost** | ~$15-20/month | ~$0.50/month |
| **Scalability** | Manual (Auto Scaling) | Automatic |
| **Management** | Server maintenance required | Serverless |
| **Dynamic Content** | ✅ Yes | ❌ No |
| **Use Case** | Web apps, APIs | Static sites, docs |
| **Performance** | Good | Excellent |

### When to Use Each

**EC2:**
- Dynamic web applications
- Server-side processing (PHP, Python, Node.js)
- Database integration
- Custom software requirements

**S3:**
- Static websites
- Landing pages
- Documentation sites
- Portfolio sites
- Marketing pages

## 📚 Lessons Learned

### Technical Insights

1. **Networking is Fundamental:** Understanding VPCs, subnets, and security groups is crucial
2. **Automation Saves Time:** Ansible reduced deployment time from hours to minutes
3. **Security Best Practices:** Principle of least privilege for security groups
4. **High Availability:** Multi-AZ deployment is essential for production workloads

### Challenges Overcome

1. **SSH Connectivity:** Learned about public vs private IPs in AWS
2. **File Permissions:** NGINX requires specific directory/file permissions
3. **Load Balancer Health Checks:** Proper configuration is critical
4. **Instance Metadata:** IMDSv2 requires token-based authentication

## 🔮 Future Enhancements

- [ ] Add HTTPS with AWS Certificate Manager
- [ ] Implement Auto Scaling groups
- [ ] Add CloudWatch monitoring and alarms
- [ ] Use Terraform for infrastructure provisioning
- [ ] Set up CI/CD pipeline with GitHub Actions
- [ ] Add custom domain with Route 53
- [ ] Implement CloudFront CDN
- [ ] Add RDS database backend
- [ ] Container orchestration with ECS/EKS

## 🧹 Cleanup Instructions

**Important:** Delete all resources to avoid charges!
```bash
# Delete in this order:
1. Load Balancer
2. Target Group
3. EC2 Instances (Terminate)
4. S3 Bucket (Empty first, then delete)
5. Security Groups
6. VPC (deletes subnets, IGW, route tables)
```

## 👨‍💻 Author

**Ofonime Offong**

- GitHub: [@Ofony-85](https://github.com/Ofony-85)
- LinkedIn: [https://www.linkedin.com/in/ofonime-offong-139322a3/]
- Medium: [https://medium.com/@ofonimeoffong]
- Email: [ofonyme3@gmail.com]

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🙏 Acknowledgments

- AWS Documentation
- Ansible Documentation
- Ubuntu Community
- Cloud Engineering mentors and community

---

**⭐ If you found this project helpful, please give it a star!**

## 📖 Additional Resources

- [AWS Free Tier](https://aws.amazon.com/free/)
- [Ansible Documentation](https://docs.ansible.com/)
- [NGINX Documentation](https://nginx.org/en/docs/)
- [My Medium Article](link-to-your-article)

---

*Last Updated: December 2025*# cloud-devops-capstone-project
End-to-end cloud infrastructure deployment featuring AWS EC2, ALB, S3, and Ansible automation
