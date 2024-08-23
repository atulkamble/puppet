Examples of Puppet manifests that demonstrate various functionalities, such as managing files, services, packages, users, and more.

### 1. **Basic File Management**
Create or manage a file on the system:

```puppet
file { '/tmp/hello.txt':
  ensure  => 'present',
  content => "Hello, world!\n",
  mode    => '0644',
  owner   => 'root',
  group   => 'root',
}
```
This manifest ensures that the file `/tmp/hello.txt` is present with the specified content, permissions, and ownership.

### 2. **Package Installation**
Ensure a package is installed:

```puppet
package { 'nginx':
  ensure => 'installed',
}
```
This will install the `nginx` package on the system if it’s not already installed.

### 3. **Service Management**
Ensure a service is running and enabled at boot:

```puppet
service { 'nginx':
  ensure     => 'running',
  enable     => true,
  require    => Package['nginx'],
}
```
This manifest makes sure the `nginx` service is running and enabled to start on boot. It also ensures that the service management depends on the `nginx` package being installed.

### 4. **User Management**
Create a user and manage their home directory:

```puppet
user { 'john':
  ensure     => 'present',
  uid        => '1001',
  gid        => '1001',
  home       => '/home/john',
  managehome => true,
  shell      => '/bin/bash',
}
```
This manifest ensures that the user `john` is present on the system with the specified UID, GID, home directory, and shell.

### 5. **Group Management**
Ensure a group is present:

```puppet
group { 'developers':
  ensure => 'present',
  gid    => '1001',
}
```
This manifest creates a group called `developers` with the specified GID.

### 6. **Managing Cron Jobs**
Create a cron job to run a script every day at midnight:

```puppet
cron { 'daily_backup':
  ensure  => 'present',
  command => '/usr/local/bin/backup.sh',
  user    => 'root',
  hour    => '0',
  minute  => '0',
}
```
This manifest sets up a cron job named `daily_backup` to run a backup script daily at midnight as the `root` user.

### 7. **File Download and Extraction**
Download and extract a tar.gz file:

```puppet
archive { '/tmp/sample.tar.gz':
  source       => 'https://example.com/sample.tar.gz',
  ensure       => 'present',
  extract      => true,
  extract_path => '/opt',
  creates      => '/opt/sample',
}
```
This manifest downloads a tar.gz file from a URL and extracts it to `/opt`.

### 8. **Simple Notify**
Notify is a way to send a message during the Puppet run:

```puppet
notify { 'This is a test notification from Puppet!':
}
```
This will simply output the message during the Puppet run.

### 9. **Ensuring a Directory Exists**
Ensure a directory is present with specific permissions:

```puppet
file { '/var/log/myapp':
  ensure  => 'directory',
  owner   => 'myappuser',
  group   => 'myappgroup',
  mode    => '0755',
}
```
This manifest ensures the `/var/log/myapp` directory exists with the specified ownership and permissions.

### 10. **Managing a Service with a Configuration File**
Here’s a more complex example that installs a package, manages its configuration file, and ensures the service is running:

```puppet
package { 'httpd':
  ensure => 'installed',
}

file { '/etc/httpd/conf/httpd.conf':
  ensure  => 'file',
  content => template('my_module/httpd.conf.erb'),
  require => Package['httpd'],
  notify  => Service['httpd'],
}

service { 'httpd':
  ensure     => 'running',
  enable     => true,
  require    => File['/etc/httpd/conf/httpd.conf'],
}
```
This example installs the `httpd` package, manages the `httpd.conf` file using a template, and ensures the `httpd` service is running and enabled. The service is restarted whenever the configuration file changes.

These examples should give you a solid foundation for writing Puppet manifests to manage your infrastructure.
