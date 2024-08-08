#!/bin/bash
set -x

# Update the instance
yum update -y

# Install Docker
amazon-linux-extras install docker -y
service docker start
systemctl enable docker
usermod -a -G docker ec2-user
chmod 666 /var/run/docker.sock

# Install Docker Compose
DOCKER_CONFIG=${DOCKER_CONFIG:-/home/ec2-user/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.29.1/docker-compose-$(uname -s)-$(uname -m) -o $DOCKER_CONFIG/cli-plugins/docker-compose
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
chown -R ec2-user:ec2-user $DOCKER_CONFIG

# Install Git
yum install git -y

# Clone your GitHub repository (replace <your-repo-url> with your actual GitHub repo URL)
sudo -u ec2-user git clone https://github.com/gegeen123/grafanaDashboard.git /home/ec2-user/app

# Change to the directory containing the Docker Compose file
cd /home/ec2-user/app

# Run Docker Compose as ec2-user
sudo -u ec2-user docker compose up -d

# Create /etc/rc.local to ensure Docker Compose runs on reboot
cat <<EOF | sudo tee /etc/rc.local
#!/bin/bash
cd /home/ec2-user/app
sudo -u ec2-user docker compose up -d
EOF

# Make /etc/rc.local executable
sudo chmod +x /etc/rc.local
