# puppet
Puppet Server and Agent Configurations 

To install and configure Puppet Server and agents on ubuntu DNS settings, follow these steps:

### 1. **Set Up Puppet Server on ubuntu**

#### Step 1: Launch an ubuntu Instance for Puppet Server
- Launch an ubuntu instance from the AWS Management Console.
- Choose an instance `t2.medium` (recommended).
- Configure the security group to allow:
  - SSH (Port 22)
  - Puppet (Port 8140)
  - HTTP/HTTPS if needed (Ports 80/443)

Here’s how to install and configure Puppet Server and agents on Ubuntu (while also addressing Amazon Linux without DNS settings):

Here are the installation commands for Ruby 3.1 or later, Facter 2.0 or later, and optional gems such as `hiera-eyaml`, `hocon`, `msgpack`, `ruby-shadow`, and others.

### 1. **Install Ruby 3.1 or Later**

#### Step 1: Install Prerequisites
```bash
sudo apt-get update -y
sudo apt-get install -y gnupg2 build-essential libssl-dev libreadline-dev zlib1g-dev
```

#### Step 2: Add Brightbox Repository (for Ubuntu)
```bash
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
```

#### Step 3: Install Ruby 3.1
```bash
sudo apt-get install -y ruby3.1
```

#### Step 4: Verify Ruby Installation
```bash
ruby -v
```

### 2. **Install Facter 2.0 or Later**

Facter is usually installed as part of the Puppet installation. However, you can install or upgrade Facter manually as follows:

```bash
sudo gem install facter
```

### 3. **Install Optional Gems**

#### 3.1 Install `hiera-eyaml`
`hiera-eyaml` is used for encrypted data in Hiera.

```bash
sudo gem install hiera-eyaml
```

#### 3.2 Install `hocon`
`hocon` is used to parse HOCON (Human-Optimized Config Object Notation) files.

```bash
sudo gem install hocon
```

#### 3.3 Install `msgpack`
`msgpack` is a binary serialization format.

```bash
sudo gem install msgpack
```

#### 3.4 Install `ruby-shadow`
`ruby-shadow` is used for managing Unix user accounts' shadow password entries.

```bash
sudo gem install ruby-shadow
```

#### 3.5 Install Other Gems
If there are additional gems you need, you can install them similarly:

```bash
sudo gem install <gem_name>
```

### 4. **Verify Gem Installations**

You can check the installed versions of the gems with:

```bash
gem list | grep <gem_name>
```

This setup should cover installing Ruby, Facter, and the specified optional gems on your system.

### 1. **Set Up Puppet Agent on Ubuntu**

// update
```
sudo apt update -y
```
// upgrade
```
sudo apt --upgradable list
sudo apt upgrade -y
```
// install puppet-agent pkg
```
sudo apt-get install puppet-agent
```
// config
```
sudo nano /etc/puppet/puppet.conf
```
// starting puppet service
```
sudo systemctl start puppet
```
// check status
```
sudo systemctl status puppet
```
// enable service boot
```
sudo systemctl enable puppet
```
// update of hosts file
// copy srerver ip and client ip update it 
```
sudo nano /etc/hosts
```
```
[server-ip] puppetmaster
[client-ip] puppetclient
example:
172.31.45.253 puppetclient ec2-54-173-224-4.compute-1.amazonaws.com
172.31.46.68 puppetserver ec2-34-229-253-76.compute-1.amazonaws.com
```
// status
```
sudo systemctl status puppet
```
### 1. **Set Up Puppet Server on Ubuntu**

Launch 2 instances on same network | us-east-1c

// update public ip details of client and server 

sudo nano /etc/hosts

// Public ip
```
puppet-server 54.162.99.19
puppet-client 54.227.168.66
```

// Private ip
```
puppet-server 172.31.37.94
puppet-client 172.31.45.253
```

// Note Down instance DNS
Client DNS: ec2-54-227-168-66.compute-1.amazonaws.com
Server DNS: ec2-54-162-99-19.compute-1.amazonaws.com

// on puppet agent
sudo nano /etc/puppet/puppet.conf
```
[main]
certname = puppetclient
server = ec2-54-162-99-19.compute-1.amazonaws.com
environment = production
```

// Connections

To connect the Puppet Agent and Puppet Server hosted on EC2 instances using their public or private IP addresses, follow these steps:

### 1. **Set Up Hostname Resolution**

You can choose to connect via the private IPs or public DNS. The private IPs are typically preferred for internal communication due to lower latency and no internet exposure.

#### Option 1: **Using Private IPs**
Update the `/etc/hosts` file on both the Puppet Server and Puppet Agent to resolve the hostname using private IPs.

On **Puppet Server (54.162.99.19):**
```bash
sudo nano /etc/hosts
```
Add the agent’s private IP and hostname:
```bash
172.31.45.253 puppet-client ec2-54-227-168-66.compute-1.amazonaws.com
```

On **Puppet Agent (54.227.168.66):**
```bash
sudo nano /etc/hosts
```
Add the server’s private IP and hostname:
```bash
172.31.37.94 puppet-server ec2-54-162-99-19.compute-1.amazonaws.com
```

#### Option 2: **Using Public DNS**
If you prefer to use public DNS names, ensure your security groups allow traffic on port 8140 from the public IPs of both instances.

On **Puppet Server:**
```bash
sudo nano /etc/hosts
```
Add:
```bash
54.227.168.66 puppet-client ec2-54-227-168-66.compute-1.amazonaws.com
```

On **Puppet Agent:**
```bash
sudo nano /etc/hosts
```
Add:
```bash
54.162.99.19 puppet-server ec2-54-162-99-19.compute-1.amazonaws.com
```

### 2. **Configure Puppet Server**
On the Puppet Server, edit the `puppet.conf` file to ensure it uses the correct server name.

```bash

sudo nano /etc/puppetlabs/puppet/puppet.conf
```
Set the `[master]` and `[main]` sections to match the chosen hostname:
```bash
[main]
certname = ec2-54-162-99-19.compute-1.amazonaws.com
server = ec2-54-162-99-19.compute-1.amazonaws.com
dns_alt_names = puppet,ec2-54-162-99-19.compute-1.amazonaws.com
```

Restart the Puppet Server:
```bash
sudo systemctl restart puppetserver
```

### 3. **Configure Puppet Agent**
On the Puppet Agent, configure it to communicate with the Puppet Server using the DNS name.

Edit the `puppet.conf` file:
```bash
sudo nano /etc/puppetlabs/puppet/puppet.conf
```
Set:
```bash
[main]
server = ec2-54-162-99-19.compute-1.amazonaws.com
certname = ec2-54-227-168-66.compute-1.amazonaws.com
```

### 4. **Sign the Agent’s Certificate on the Server**
On the Puppet Server, sign the Puppet Agent’s certificate request:
```bash
sudo puppetserver ca list --all
sudo puppetserver ca sign --certname ec2-54-227-168-66.compute-1.amazonaws.com
```

### 5. **Test the Connection**
On the Puppet Agent, run the test to ensure it can connect to the server:
```bash
sudo puppet agent --test
```

### 6. **Verify Firewall and Security Group Settings**
Ensure that your security groups allow inbound traffic on port `8140` for Puppet communication between the server and the agent.

With these steps, your Puppet Agent should successfully connect to the Puppet Server. Let me know if you encounter any issues during this setup!


// on puppet server 
```
/etc/puppetlabs/puppet/puppet.conf
```
// open source puppet
```
https://www.puppet.com/
https://www.puppet.com/downloads/puppet-enterprise
https://www.puppet.com/success/community/open-source/free-trial
https://www.puppet.com/docs/puppet/8/installing_and_upgrading.html
https://www.puppet.com/docs/puppet/8/install_puppet
```
```
sudo rpm -Uvh https://yum.puppet.com/puppet7-release-el-8.noarch.rpm
sudo yum-config-manager --enable puppet7
```
```
sudo yum install java-21-amazon-corretto-headless
```
## puppet server installation - ubuntu ec2 | ubuntu 22.04 | t2.medium
```
sudo apt update -y
sudo apt list --upgradable
sudo apt upgrade -y
```
```
sudo -i passwd 
sudo apt install wget
lsb_release -a
```

// prerequisite 
Links: 
```
https://www.puppet.com/docs/puppet/8/server/install_from_packages.html#java-support
https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html
```
```
sudo wget https://download.oracle.com/java/17/archive/jdk-17.0.11_linux-x64_bin.deb
sudo dpkg -i jdk-17.0.11_linux-x64_bin.deb
sudo apt update -y
java --version
```
// enable puppet platform on apt
Link: https://www.puppet.com/docs/puppet/7/install_puppet#enable_the_puppet_platform_apt
```
wget https://apt.puppet.com/puppet7-release-focal.deb
sudo dpkg -i puppet7-release-focal.deb
sudo apt-get update
```
```
sudo apt-get install puppetserver
sudo systemctl start puppetserver
sudo systemctl status puppetserver
```
// ERROR
```
Job for puppetserver.service failed because the control process exited with error code.
See "systemctl status puppetserver.service" and "journalctl -xeu puppetserver.service" for details.
```
// Solution for ERROR | Update RAM details
```
sudo nano /etc/default/puppetserver
```
// update 2g

// ERROR
```
Fatal error when running action 'list'
  Error: Failed connecting to https://54.162.99.19:8140/puppet-ca/v1/certificate_statuses/any_key
  Root cause: SSL_connect returned=1 errno=0 state=error: certificate verify failed (Hostname mismatch)
```

// Test Connection
```
sudo /opt/puppetlabs/bin/puppet agent --test
```
// from puppet server - sign certificate
```
puppetserver ca list --all
puppetserver ca sign --certname ec2-54-227-168-66.compute-1.amazonaws.com
```
// from puppet agent 
```
sudo puppet agent --test
```

sudo puppetserver ca generate --certname ec2-54-162-99-19.compute-1.amazonaws.com

// Test Connection
To test the connection between the Puppet Server and Puppet Agent, you can use the following commands on both the Puppet Server and Puppet Agent:

### 1. **Test Puppet Agent Connection to Puppet Server:**

   Run the following command on the Puppet Agent to check if it can communicate with the Puppet Server:
   ```bash
   sudo puppet agent --test --server=<puppet_server_fqdn_or_ip>
   // working command
   sudo puppet agent --test --server=ec2-3-87-117-147.compute-1.amazonaws.com
   ```
   Replace `<puppet_server_fqdn_or_ip>` with the actual hostname or IP address of your Puppet Server.

### 2. **Check for SSL Certificate Signing Request (CSR) on the Puppet Server:**

   If the Puppet Agent connects successfully, it will send an SSL certificate signing request (CSR) to the Puppet Server. To list pending certificate requests on the Puppet Server, run:
   ```bash
   sudo puppetserver ca list --all
   // working command
   sudo /opt/puppetlabs/server/bin/puppetserver ca list --all
   ```

### 3. **Sign the Agent’s Certificate on the Puppet Server:**

   If the CSR is listed, you can sign the agent's certificate on the Puppet Server using:
   ```bash
   sudo puppetserver ca sign --certname <agent_hostname>
   ```
   Replace `<agent_hostname>` with the actual hostname of the Puppet Agent.

### 4. **Verify the Puppet Agent Connection:**

   After signing the certificate, run the following command again on the Puppet Agent to confirm that it can now pull configurations from the Puppet Server:
   ```bash
   sudo puppet agent --test
   ```

### 5. **Check Agent Logs for Errors:**

   If there are any issues, check the logs on the Puppet Agent:
   ```bash
   sudo tail -f /var/log/puppetlabs/puppet/puppet.log
   ```

These steps will help ensure that the Puppet Agent can successfully communicate with the Puppet Server.


// Check Connectivity between two instances 
```
ping private-ip
ssh -i /path/to/your-key.pem ec2-user@<target-instance-public-or-private-ip>
telnet <target-instance-private-ip> <port>
nc -zv <target-instance-private-ip> <port>
traceroute <target-instance-private-ip>
curl http://<target-instance-private-ip>:<port>
```

// test connectivity | open client
```
chmod 400 puppet.pem 
ssh -i "puppet.pem" ubuntu@172.31.46.68
```
// Note Down details 
// Both Agent and server | update hosts file 
```
sudo nano /etc/hosts
```
```
172.31.45.253 puppetclient ec2-54-80-194-251.compute-1.amazonaws.com
172.31.46.68 puppetserver ec2-54-224-115-5.compute-1.amazonaws.com
```
// On Puppet Agent
```
sudo nano /etc/puppet/puppet.conf
```
```
[main]
certname = ec2-54-224-115-5.compute-1.amazonaws.com
server = ec2-54-224-115-5.compute-1.amazonaws.com
```
// On Puppet Server 
```
sudo nano /etc/puppetlabs/puppet/puppet.conf 
```
```
[main]
certname = ec2-54-224-115-5.compute-1.amazonaws.com
server = ec2-54-224-115-5.compute-1.amazonaws.com
dns_alt_names = puppet,ec2-54-224-115-5.compute-1.amazonaws.com
environment = production
```

// on agent machine
```
sudo puppet agent --test --server=ec2-54-224-115-5.compute-1.amazonaws.com
```
// On Server | list all certificate
```
sudo /opt/puppetlabs/server/bin/puppetserver ca list --all
```
// sign specific certificate
```
sudo /opt/puppetlabs/server/bin/puppetserver ca sign --certname ec2-54-224-115-5.compute-1.amazonaws.com
```

// revoke specific certificate
```
sudo /opt/puppetlabs/server/bin/puppetserver ca clean --certname ec2-3-87-117-147.compute-1.amazonaws.com
```
```
sudo openssl crl -in /etc/puppetlabs/puppetserver/ca/ca_crl.pem -text -noout
```

// on puppet- agent | generate new csr
```
sudo rm -rf /etc/puppetlabs/puppet/ssl/*
sudo puppet agent --test --server=ec2-3-87-117-147.compute-1.amazonaws.com
```
// Practice Manifests
```
sudo touch test.pp
sudo nano test.pp
```
// copy code
```
file { '/tmp/testfile':
  ensure  => 'present',
  content => "This is a test file created by Puppet.\n",
}
```
// run
```
puppet apply test.pp
```
// test output
```
cat /tmp/testfile
```
// 
```
sudo touch site.pp
sudo nano site.pp
```
```
node default {
  file { '/tmp/puppet_test_file':
    ensure  => 'present',
    content => 'This file is managed by Puppet.',
  }
}
```
```
sudo cat /tmp/puppet_test_file 
```
// Run multiple files 
```
mkdir manifests
mv site.pp manifests
mv test.pp manifests
```
// run 
```
sudo puppet apply manifests/site.pp  
sudo puppet apply manifests
```
// simulate without apply
```
puppet apply test.pp --noop
```
// detailed output
```
puppet apply test.pp --debug
```
