# realmad-notes

This document is my notes on how to set up foreman, katello with Active Directory and the
smart_proxy_realm_ad_plugin .

# Provision servers

* Provision a Centos 7 x86 instance on AWS with a fixed Elastic IP Address
* Provision a Windows Server 2019 server. Configure as a Domain Controller DC01.

# Login
```
ssh -i "~/.ssh/ansible.pem" centos@xxxx.compute.amazonaws.com
```

```bash
[centos@ip-172-31-0-200 ~]$ cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core) 
[centos@ip-172-31-0-200 ~]$ 
```

# Disable SELinux
Set SELINUX=permissive in /etc/sysconfig/config, then reboot, with sudo reboot.

# Update the system
```bash
sudo yum -y update
```

# Install Foreman 1.20

```bash
sudo yum -y update
```

## Enable Puppet repo
```bash
sudo yum -y install https://yum.puppetlabs.com/puppet5/puppet5-release-el-7.noarch.rpm
```

## Enable the EPEL and the Foreman Repos
```bash
sudo yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y install https://yum.theforeman.org/releases/1.20/el7/x86_64/foreman-release.rpm
```

## Check whats installed
```bash
# rpm -q --all --last 
foreman-installer-1.20.2-1.el7.noarch         tor 11 apr 2019 08:43:53
puppet-agent-5.5.12-1.el7.x86_64              tor 11 apr 2019 08:43:51
rubygem-powerbar-2.0.1-1.el7.noarch           tor 11 apr 2019 08:43:29
rubygem-multi_json-1.12.2-3.el7.noarch        tor 11 apr 2019 08:43:29
rubygem-logging-2.2.2-3.el7.noarch            tor 11 apr 2019 08:43:29
rubygem-little-plugger-1.1.3-23.el7.noarch    tor 11 apr 2019 08:43:29
rubygem-kafo_wizards-0.0.1-2.el7.noarch       tor 11 apr 2019 08:43:29
rubygem-kafo_parsers-0.1.6-1.el7.noarch       tor 11 apr 2019 08:43:29
rubygem-kafo-2.1.0-1.el7.noarch               tor 11 apr 2019 08:43:29
rubygem-hashie-3.6.0-1.el7.noarch             tor 11 apr 2019 08:43:29
rubygem-clamp-1.1.2-4.el7.noarch              tor 11 apr 2019 08:43:29
rubygem-ansi-1.4.3-2.el7.noarch               tor 11 apr 2019 08:43:29
foreman-selinux-1.20.2-1.el7.noarch           tor 11 apr 2019 08:43:29
ruby-libs-2.0.0.648-34.el7_6.x86_64           tor 11 apr 2019 08:43:28
ruby-irb-2.0.0.648-34.el7_6.noarch            tor 11 apr 2019 08:43:28
rubygems-2.0.14.1-34.el7_6.noarch             tor 11 apr 2019 08:43:28
rubygem-rdoc-4.0.0-34.el7_6.noarch            tor 11 apr 2019 08:43:28
rubygem-psych-2.0.0-34.el7_6.x86_64           tor 11 apr 2019 08:43:28
rubygem-json-1.7.7-34.el7_6.x86_64            tor 11 apr 2019 08:43:28
rubygem-io-console-0.4.2-34.el7_6.x86_64      tor 11 apr 2019 08:43:28
rubygem-highline-1.7.8-4.el7.noarch           tor 11 apr 2019 08:43:28
rubygem-bigdecimal-1.2.0-34.el7_6.x86_64      tor 11 apr 2019 08:43:28
ruby-2.0.0.648-34.el7_6.x86_64                tor 11 apr 2019 08:43:28
gpg-pubkey-ef8d349f-57b6233e                  tor 11 apr 2019 08:43:25
gpg-pubkey-565ea533-5bc49db5                  tor 11 apr 2019 08:43:25
gpg-pubkey-352c64e5-52ae6884                  tor 11 apr 2019 08:43:25
foreman-release-1.20.2-1.el7.noarch           tor 11 apr 2019 08:41:18
epel-release-7-11.noarch                      tor 11 apr 2019 08:41:12
puppet5-release-5.0.0-4.el7.noarch            tor 11 apr 2019 08:40:19
kernel-3.10.0-957.10.1.el7.x86_64             tor 11 apr 2019 08:39:12

...
```

## Download the installer
```bash
sudo yum -y install foreman-installer
```

## Run the installer
```bash
sudo foreman-installer
```

## The installation result
```bash
[centos@ip-172-31-0-200 ~]$ sudo foreman-installer

Installing             Package[foreman-postgresql]                        [0%]
Installing             Package[foreman-cli]                               [28%]
Installing             Exec[foreman-rake-apipie:cache:index]              [85%]

Installing             Done                                               [100%]
  Success!
  * Foreman is running at https://ip-172-31-0-200.eu-central-1.compute.internal
      Initial credentials are admin / yzZhFrgEnAQCiPTv
  * Foreman Proxy is running at https://ip-172-31-0-200.eu-central-1.compute.internal:8443
  * Puppetmaster is running at port 8140
  The full log is at /var/log/foreman-installer/foreman.log
```

## The installer will install foreman-proxy 

```bash
[centos@ip-172-31-0-200 ~]$ sudo systemctl status foreman-proxy
● foreman-proxy.service - Foreman Proxy
   Loaded: loaded (/usr/lib/systemd/system/foreman-proxy.service; enabled; vendor preset: disabled)
   Active: active (running) since tor 2019-04-11 08:52:22 UTC; 4min 47s ago
 Main PID: 17172 (ruby)
   CGroup: /system.slice/foreman-proxy.service
           └─17172 ruby /usr/share/foreman-proxy/bin/smart-proxy --no-daemonize

apr 11 08:52:22 ip-172-31-0-200.eu-central-1.compute.internal systemd[1]: Starting Foreman Proxy...
apr 11 08:52:22 ip-172-31-0-200.eu-central-1.compute.internal systemd[1]: Started Foreman Proxy.
apr 11 08:52:24 ip-172-31-0-200.eu-central-1.compute.internal smart-proxy[17172]: ip-172-31-0-200.eu-central-1.compute.internal... 35
apr 11 08:52:24 ip-172-31-0-200.eu-central-1.compute.internal smart-proxy[17172]: - -> /features
apr 11 08:52:24 ip-172-31-0-200.eu-central-1.compute.internal smart-proxy[17172]: ip-172-31-0-200.eu-central-1.compute.internal... 35
apr 11 08:52:24 ip-172-31-0-200.eu-central-1.compute.internal smart-proxy[17172]: - -> /features
apr 11 08:52:24 ip-172-31-0-200.eu-central-1.compute.internal smart-proxy[17172]: ip-172-31-0-200.eu-central-1.compute.internal... 35
apr 11 08:52:24 ip-172-31-0-200.eu-central-1.compute.internal smart-proxy[17172]: - -> /features
Hint: Some lines were ellipsized, use -l to show in full.
[centos@ip-172-31-0-200 ~]$ 
```

## Here are the settings files for foreman
```bash
[centos@ip-172-31-0-200 ~]$ sudo ls -R /etc/foreman/
/etc/foreman/:
database.yml  encryption_key.rb  foreman-debug.conf  plugins  settings.yaml

/etc/foreman/plugins:
[centos@ip-172-31-0-200 ~]$ 

The plugins directory is empty.
```

## Logs for foreman-proxy
The foreman-proxy is a webserver that write its logs to /var/log/foreman-proxy/proxy.log
```
[centos@ip-172-31-0-200 ~]$ sudo less /var/log/foreman-proxy/proxy.log
2019-04-11T08:52:22  [I] Successfully initialized 'foreman_proxy'
2019-04-11T08:52:22  [I] Successfully initialized 'tftp'
2019-04-11T08:52:22  [I] Successfully initialized 'puppetca_hostname_whitelisting'
2019-04-11T08:52:22  [I] Successfully initialized 'puppetca'
2019-04-11T08:52:22  [I] Started puppet class cache initialization
2019-04-11T08:52:22  [I] Successfully initialized 'puppet_proxy_puppet_api'
2019-04-11T08:52:22  [I] Successfully initialized 'puppet'
2019-04-11T08:52:22  [I] Successfully initialized 'logs'
2019-04-11T08:52:22  [I] WEBrick 1.3.1
2019-04-11T08:52:22  [I] ruby 2.0.0 (2015-12-16) [x86_64-linux]
...
```

# References

## Foreman 1.20
https://www.theforeman.org/manuals/1.20/quickstart_guide.html
## Katello 3.10
https://theforeman.org/plugins/katello/3.10/installation/index.html