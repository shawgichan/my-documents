# AWS EC2 Deployment Guide for Docker Applications

## Prerequisites
- AWS Account with Free Tier access
- AWS CLI installed
- AWS Access Key and Secret Key
- A Docker application with docker-compose.yaml

## Step 1: Configure AWS CLI
Configure your AWS CLI with your credentials:
```bash
aws configure
```
When prompted, enter:
- AWS Access Key ID
- AWS Secret Access Key
- Default region name: ap-southeast-1
- Default output format: json

## Step 2: Create Security Group
Create a security group and add necessary inbound rules:
```bash
# Create the security group
aws ec2 create-security-group \
    --group-name exam-dashboard-sg \
    --description "Security group for exam dashboard"

# Add inbound rule for SSH
aws ec2 authorize-security-group-ingress \
    --group-name exam-dashboard-sg \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# Add inbound rule for application port
aws ec2 authorize-security-group-ingress \
    --group-name exam-dashboard-sg \
    --protocol tcp \
    --port 8080 \
    --cidr 0.0.0.0/0

# Add inbound rule for database port
aws ec2 authorize-security-group-ingress \
    --group-name exam-dashboard-sg \
    --protocol tcp \
    --port 5432 \
    --cidr 0.0.0.0/0
```

## Step 3: Create Key Pair
Create and secure your EC2 key pair:
```bash
# Create key pair
aws ec2 create-key-pair \
    --key-name exam-dashboard-key \
    --query 'KeyMaterial' \
    --output text > exam-dashboard-key.pem

# Set correct permissions
chmod 400 exam-dashboard-key.pem
```

## Step 4: Launch EC2 Instance
Launch a t2.micro instance (Free Tier eligible):
```bash
aws ec2 run-instances \
    --image-id ami-0df7a207adb9748c7 \
    --count 1 \
    --instance-type t2.micro \
    --key-name exam-dashboard-key \
    --security-group-ids $(aws ec2 describe-security-groups --group-names exam-dashboard-sg --query 'SecurityGroups[0].GroupId' --output text)
```

## Step 5: Get Instance IP
Retrieve your instance's public IP:
```bash
aws ec2 describe-instances \
    --query 'Reservations[].Instances[].PublicIpAddress' \
    --output text
```

## Step 6: Connect to Instance
SSH into your instance:
```bash
ssh -i exam-dashboard-key.pem ubuntu@<your-instance-ip>
```

## Step 7: Install Dependencies
Install Docker and other required packages:
```bash
# Update package list
sudo apt-get update

# Install required packages
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    git

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Configure Docker permissions
sudo usermod -aG docker ubuntu
sudo chmod 666 /var/run/docker.sock
```

## Step 8: Deploy Application
Clone and start your application:
```bash
# Clone repository
git clone https://github.com/shawgichan/exam-dashboard.git
cd exam-dashboard

# Start application
sudo docker-compose up -d
```

## Monitoring and Management

### View Application Logs
```bash
sudo docker-compose logs -f
```

### Stop Application
```bash
sudo docker-compose down
```

### Monitor via AWS Console
1. Log into AWS Console (https://console.aws.amazon.com)
2. Navigate to EC2 Dashboard
3. Click "Instances (running)"
4. Select your instance to view:
   - Instance summary
   - Monitoring metrics
   - Status checks
   - Security settings

### Monitor Costs
1. Access Billing Dashboard from account dropdown
2. View:
   - Monthly costs
   - Free tier usage
   - Service cost breakdowns
   - Set up billing alerts

## Free Tier Limits
- 750 hours of t2.micro instance usage per month
- 30GB of EBS storage
- 1 million AWS Lambda requests
- 750 hours of RDS usage

## Security Notes
- The security group configuration allows access from any IP (0.0.0.0/0)
- For production, restrict IP ranges to known sources
- Regularly update security patches
- Monitor AWS Security Hub for security recommendations
