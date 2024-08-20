# puppet
Puppet Server and Agent Configurations 

To install and configure Puppet Server and agents on Amazon Linux without DNS settings, follow these steps:

### 1. **Set Up Puppet Server on Amazon Linux**

#### Step 1: Launch an Amazon Linux Instance for Puppet Server
- Launch an Amazon Linux 2 instance from the AWS Management Console.
- Choose an instance type like `t2.medium` for better performance.
- Configure the security group to allow:
  - SSH (Port 22)
  - Puppet (Port 8140)
  - HTTP/HTTPS if needed (Ports 80/443)

#### Step 2: Update the System and Install Required Tools
- SSH into your instance and run the following commands:
  ```bash
  sudo yum update -y
  sudo yum install -y wget nano
  ```

#### Step 3: Set Up Hostname and /etc/hosts
- Set the hostname of your server:
  ```bash
  sudo hostnamectl set-hostname puppet.local
  ```

- Update the `/etc/hosts` file to resolve the hostname locally:
  ```bash
  sudo nano /etc/hosts
  ```
  Add the following line (assuming the server's private IP is `192.168.1.10`):
  ```bash
  127.0.0.1 puppet.local puppet
  192.168.1.10 puppet.local puppet
  ```

#### Step 4: Install Puppet Server
- Enable Puppet repositories and install Puppet Server:
  ```bash
  sudo rpm -Uvh https://yum.puppet.com/puppet7-release-el-7.noarch.rpm
  sudo yum-config-manager --enable puppet7
  sudo yum install -y puppetserver
  ```

#### Step 5: Configure Puppet Server
- Edit the Puppet Server configuration file:
  ```bash
  sudo nano /etc/puppetlabs/puppet/puppet.conf
  ```
  Add the following under the `[main]` section:
  ```ini
  [main]
  certname = puppet.local
  server = puppet.local
  ```

- Adjust memory allocation if needed:
  ```bash
  sudo nano /etc/sysconfig/puppetserver
  ```
  Modify the `JAVA_ARGS` parameter:
  ```bash
  JAVA_ARGS="-Xms512m -Xmx512m"
  ```

#### Step 6: Start Puppet Server
- Start and enable the Puppet Server service:
  ```bash
  sudo systemctl start puppetserver
  sudo systemctl enable puppetserver
  ```

### 2. **Set Up Puppet Agents on Client Machines**

#### Step 1: Launch Amazon Linux Instances for Agents
- Launch one or more Amazon Linux instances that will act as Puppet agents.

#### Step 2: Set Up Hostname and /etc/hosts for Each Agent
- Set the hostname for each agent:
  ```bash
  sudo hostnamectl set-hostname agent1.local
  ```

- Update the `/etc/hosts` file to resolve the Puppet server’s hostname:
  ```bash
  sudo nano /etc/hosts
  ```
  Add the following line (assuming the Puppet server's private IP is `192.168.1.10`):
  ```bash
  192.168.1.10 puppet.local puppet
  127.0.0.1 agent1.local
  ```

#### Step 3: Install Puppet Agent
- Enable Puppet repositories and install the Puppet agent:
  ```bash
  sudo rpm -Uvh https://yum.puppet.com/puppet7-release-el-7.noarch.rpm
  sudo yum-config-manager --enable puppet7
  sudo yum install -y puppet
  ```

#### Step 4: Configure Puppet Agent
- Update the Puppet agent configuration file:
  ```bash
  sudo nano /etc/puppetlabs/puppet/puppet.conf
  ```
  Add the following under the `[main]` section:
  ```ini
  [main]
  certname = agent1.local
  server = puppet.local
  environment = production
  ```

#### Step 5: Start Puppet Agent
- Start and enable the Puppet agent service:
  ```bash
  sudo systemctl start puppet
  sudo systemctl enable puppet
  ```

### 3. **Sign Agent Certificates on Puppet Server**

#### Step 1: Check for Agent Certificates on Puppet Server
- On the Puppet server, list pending certificate requests:
  ```bash
  sudo /opt/puppetlabs/bin/puppetserver ca list --all
  ```

#### Step 2: Sign the Certificates
- Sign the agent certificates:
  ```bash
  sudo /opt/puppetlabs/bin/puppetserver ca sign --certname agent1.local
  ```

### 4. **Verify the Configuration**

#### Step 1: Test Puppet Agent Communication
- On each agent machine, run:
  ```bash
  sudo /opt/puppetlabs/bin/puppet agent --test
  ```
- Ensure that the agent successfully connects to the Puppet server and applies the configuration.

This setup should provide a working Puppet Server and agent environment on Amazon Linux without needing DNS settings. The configuration relies on the `/etc/hosts` file for name resolution.
