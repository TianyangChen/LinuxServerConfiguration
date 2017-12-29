# LinuxServerConfiguration
Udacity - Full Stack - Project 6

## 1 IP address & SSH port

## 2 Web Application

## 3 Software installed

`$ sudo apt-get update`
`$ sudo apt-get upgrade`

## 4 configuration changes

### 4.1 Add new user grader and give sudo permission

Connect as `ubuntu`:

`$ ssh -i LightsailDefaultPrivateKey-us-west-2.pem ubuntu@34.214.154.162 `	

Add user `grader`:

`$ sudo adduser grader sudo`

Open `/etc/ssh/sshd_config` and modify:

```
PermitRootLogin no
PasswordAuthentication yes
```

Add new line:

`AllowUsers grader ubuntu`

Restart ssh service:

`$ sudo service ssh restart`

### 4.2 Configure key based authentication for grader

Generate key pairs on local machine:

`$ ssh-keygen`

Get two key files `grader`, `grader.pub`

Login as grader using password:

`$ ssh grader@34.214.154.162`

Run following command lines:

```
$ mkdir .ssh
$ touch .ssh/authorized_keys
```

Copy the content of `grader.pub` to `authorized_keys`

Setup key file permissions:

```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```

Force key based authentications

Open `/etc/ssh/sshd_config` and modify:

`PasswordAuthentication no`

Restart ssh service:

`$ sudo service ssh restart`

### 4.1 Configure Firewall

Open `/etc/ssh/sshd_config` and modify line `Port 22` to `Port 2200`

Run following command lines to configure UFW:

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw enable
```

## 5 Third-party resources

<a href="https://www.cyberciti.biz/tips/setup-ssh-to-run-on-a-non-standard-port.html">Setup SSH to run on a non-standard port</a>

<a href="https://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server">Create a new SSH user on Ubuntu Server</a>

<a href="https://unix.stackexchange.com/a/179956">Add user to sudo group</a>