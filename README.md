# LinuxServerConfiguration
Udacity - Full Stack - Project 6

## 1 IP address & SSH port

## 2 Web Application

## 3 Software installed

## 4 configuration changes

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

### 4.1 Add new user `grader` and give `sudo` permission

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

```
AllowUsers grader ubuntu
``` 
```

sudo service ssh restart
```

## 5 Third-party resources

<a href="https://www.cyberciti.biz/tips/setup-ssh-to-run-on-a-non-standard-port.html">Setup SSH to run on a non-standard port</a>

<a href="https://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server">Create a new SSH user on Ubuntu Server</a>

<a href="https://unix.stackexchange.com/a/179956">Add user to sudo group</a>