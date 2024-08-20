# puppet
Puppet Server and Agent Configurations 

To install and configure Puppet Server and agents on ubuntu without DNS settings, follow these steps:

### 1. **Set Up Puppet Server on ubuntu**

#### Step 1: Launch an ubuntu Instance for Puppet Server
- Launch an ubuntu instance from the AWS Management Console.
- Choose an instance type like `t2.medium` for better performance.
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

### 1. **Set Up Puppet Server on Ubuntu**

#### Step 1: Update the System
- SSH into your Ubuntu instance and update the system:
  ```bash
  sudo apt-get update -y
  sudo apt-get upgrade -y
  ```

#### Step 2: Set Up Hostname and /etc/hosts
- Set the hostname of your server:
  ```bash
  sudo hostnamectl set-hostname puppet.local
  ```

- Update the `/etc/hosts` file to ensure the hostname resolves locally:
  ```bash
  sudo nano /etc/hosts
  ```
  Add the following line (assuming the server’s IP is `192.168.1.10`):
  ```bash
  127.0.0.1 puppet.local puppet
  192.168.1.10 puppet.local puppet
  ```

#### Step 3: Install Puppet Server
- Add the Puppet repository:
  ```bash
  wget https://apt.puppet.com/puppet7-release-focal.deb
  sudo dpkg -i puppet7-release-focal.deb
  sudo apt-get update
  ```

- Install Puppet Server:
  ```bash
  sudo apt-get install -y puppetserver
  ```

#### Step 4: Configure Puppet Server
- Edit the Puppet configuration file:
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
  sudo nano /etc/default/puppetserver
  ```
  Modify the `JAVA_ARGS` parameter:
  ```bash
  JAVA_ARGS="-Xms512m -Xmx512m"
  ```

#### Step 5: Start Puppet Server
- Start and enable the Puppet Server service:
  ```bash
  sudo systemctl start puppetserver
  sudo systemctl enable puppetserver
  ```

### 2. **Set Up Puppet Agents on Client Machines (Ubuntu)**

#### Step 1: Update and Set Up Hostname
- SSH into the agent machine and set the hostname:
  ```bash
  sudo hostnamectl set-hostname agent1.local
  ```

- Update the `/etc/hosts` file to resolve the Puppet server’s hostname:
  ```bash
  sudo nano /etc/hosts
  ```
  Add the following line:
  ```bash
  192.168.1.10 puppet.local puppet
  127.0.0.1 agent1.local
  ```

#### Step 2: Install Puppet Agent
- Add the Puppet repository:
  ```bash
  wget https://apt.puppet.com/puppet7-release-focal.deb
  sudo dpkg -i puppet7-release-focal.deb
  sudo apt-get update
  ```

- Install the Puppet agent:
  ```bash
  sudo apt-get install -y puppet-agent
  ```

#### Step 3: Configure Puppet Agent
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

#### Step 4: Start Puppet Agent
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

This setup provides a working Puppet Server and agent environment on Ubuntu, using `/etc/hosts` for name resolution instead of DNS.
