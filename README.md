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

## The WebUI proxy settings 

![Smart Proxys](foreman_infra_smart_proxies.png)

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

## Install smart_proxy_realm_ad_plugin 0.1 from rubygems.org

The rubygem is a native extension, is a ruby library that uses a C library to talk to active directory.

```bash
[centos@ip-172-31-0-200 ~]$ gem install smart_proxy_realm_ad_plugin
Fetching: rake-compiler-1.0.7.gem (100%)
Successfully installed rake-compiler-1.0.7
Fetching: radcli-1.0.0.gem (100%)
Building native extensions.  This could take a while...
ERROR:  Error installing smart_proxy_realm_ad_plugin:
	ERROR: Failed to build gem native extension.

    /usr/bin/ruby extconf.rb
mkmf.rb can't find header files for ruby at /usr/share/include/ruby.h


Gem files will remain installed in /home/centos/.gem/ruby/gems/radcli-1.0.0 for inspection.
Results logged to /home/centos/.gem/ruby/gems/radcli-1.0.0/ext/radcli/gem_make.out
[centos@ip-172-31-0-200 ~]$ 
```

## Ruby Developer Tools to use the C extension
The rubygem seems to be an C extension, we need GCC and ruby-devel package.
Lets see what happens.

```
sudo yum install -y gcc ruby-devel 
zlib-devel
```
Lets see what happens now:

```bash
[centos@ip-172-31-0-200 ~]$ gem install smart_proxy_realm_ad_plugin
Building native extensions.  This could take a while...
ERROR:  Error installing smart_proxy_realm_ad_plugin:
	ERROR: Failed to build gem native extension.

    /usr/bin/ruby extconf.rb
creating Makefile

make "DESTDIR="
gcc -I. -I/usr/include -I/usr/include/ruby/backward -I/usr/include -I. -I/usr/include -I/home/centos/.gem/ruby/gems/radcli-1.0.0/ext/radcli/ext/radcli    -fPIC -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -mtune=generic -fPIC -m64 -o radcli.o -c radcli.c
In file included from ./radcli.h:5:0,
                 from radcli.c:2:
./adconn.h:29:23: fatal error: krb5/krb5.h: No such file or directory
 #include <krb5/krb5.h>
                       ^
compilation terminated.
make: *** [radcli.o] Error 1


Gem files will remain installed in /home/centos/.gem/ruby/gems/radcli-1.0.0 for inspection.
Results logged to /home/centos/.gem/ruby/gems/radcli-1.0.0/ext/radcli/gem_make.out
[centos@ip-172-31-0-200 ~]$ 
```

The gem seems to include some header krb5/krb5.h

The smart_proxy_realm_ad_plugin has a dependency on this package "radcli":

https://github.com/theforeman/smart_proxy_realm_ad_plugin/blob/master/lib/smart_proxy_realm_ad/provider.rb#L2

It also published here: https://rubygems.org/gems/radcli

We run the gem install we get below:

```bash
> gem install radcli
[centos@ip-172-31-0-200 ~]$ gem install radcli
Building native extensions.  This could take a while...
ERROR:  Error installing radcli:
	ERROR: Failed to build gem native extension.

    /usr/bin/ruby extconf.rb
creating Makefile

make "DESTDIR="
gcc -I. -I/usr/include -I/usr/include/ruby/backward -I/usr/include -I. -I/usr/include -I/home/centos/.gem/ruby/gems/radcli-1.0.0/ext/radcli/ext/radcli    -fPIC -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -mtune=generic -fPIC -m64 -o radcli.o -c radcli.c
In file included from ./radcli.h:5:0,
                 from radcli.c:2:
./adconn.h:29:23: fatal error: krb5/krb5.h: No such file or directory
 #include <krb5/krb5.h>
                       ^
compilation terminated.
make: *** [radcli.o] Error 1


Gem files will remain installed in /home/centos/.gem/ruby/gems/radcli-1.0.0 for inspection.
Results logged to /home/centos/.gem/ruby/gems/radcli-1.0.0/ext/radcli/gem_make.out
[centos@ip-172-31-0-200 ~]$ 
```
This C library need this header file. It should be available among these packages:

```
sudo yum -y install git make gcc automake autoconf krb5-devel openldap-devel cyrus-sasl-devel cyrus-sasl-gssapi
```

We try one at a time:
```
sudo yum -y install krb5-devel
...
[centos@ip-172-31-0-200 ~]$ sudo find / -name krb5.h
/usr/include/krb5/krb5.h
/usr/include/krb5.h
[centos@ip-172-31-0-200 ~]$ 
```

We try again:
```bash
> gem install radcli
```

We get: 
```bash
[centos@ip-172-31-0-200 ~]$ gem install radcli
Building native extensions.  This could take a while...
ERROR:  Error installing radcli:
	ERROR: Failed to build gem native extension.

    /usr/bin/ruby extconf.rb
creating Makefile

make "DESTDIR="
gcc -I. -I/usr/include -I/usr/include/ruby/backward -I/usr/include -I. -I/usr/include -I/home/centos/.gem/ruby/gems/radcli-1.0.0/ext/radcli/ext/radcli    -fPIC -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -mtune=generic -fPIC -m64 -o radcli.o -c radcli.c
In file included from ./radcli.h:5:0,
                 from radcli.c:2:
./adconn.h:30:18: fatal error: ldap.h: No such file or directory
 #include <ldap.h>
                  ^
compilation terminated.
make: *** [radcli.o] Error 1


Gem files will remain installed in /home/centos/.gem/ruby/gems/radcli-1.0.0 for inspection.
Results logged to /home/centos/.gem/ruby/gems/radcli-1.0.0/ext/radcli/gem_make.out
[centos@ip-172-31-0-200 ~]$ u
```

We install
```bash
sudo yum -y install openldap-devel
```

Then it works:
```bash

Installed:
  openldap-devel.x86_64 0:2.4.44-21.el7_6                                                                                            

Dependency Installed:
  cyrus-sasl.x86_64 0:2.1.26-23.el7                              cyrus-sasl-devel.x86_64 0:2.1.26-23.el7                             

Complete!
[centos@ip-172-31-0-200 ~]$ gem install radcli
Building native extensions.  This could take a while...
Successfully installed radcli-1.0.0
Parsing documentation for radcli-1.0.0
unable to convert "\x81" from ASCII-8BIT to UTF-8 for lib/radcli.so, skipping
Installing ri documentation for radcli-1.0.0
1 gem installed
```

This is our installed RPM history:
```bash
openldap-devel-2.4.44-21.el7_6.x86_64         tor 11 apr 2019 09:16:59
cyrus-sasl-devel-2.1.26-23.el7.x86_64         tor 11 apr 2019 09:16:59
cyrus-sasl-2.1.26-23.el7.x86_64               tor 11 apr 2019 09:16:58
pcre-devel-8.32-17.el7.x86_64                 tor 11 apr 2019 09:15:29
libverto-devel-0.2.5-4.el7.x86_64             tor 11 apr 2019 09:15:29
libsepol-devel-2.5-10.el7.x86_64              tor 11 apr 2019 09:15:29
libselinux-devel-2.5-14.1.el7.x86_64          tor 11 apr 2019 09:15:29
libcom_err-devel-1.42.9-13.el7.x86_64         tor 11 apr 2019 09:15:29
krb5-devel-1.15.1-37.el7_6.x86_64             tor 11 apr 2019 09:15:29
keyutils-libs-devel-1.5.8-3.el7.x86_64        tor 11 apr 2019 09:15:29
ruby-devel-2.0.0.648-34.el7_6.x86_64          tor 11 apr 2019 09:08:27
gcc-4.8.5-36.el7_6.1.x86_64                   tor 11 apr 2019 09:08:27
```

# References

## Foreman 1.20
https://www.theforeman.org/manuals/1.20/quickstart_guide.html
## Katello 3.10
https://theforeman.org/plugins/katello/3.10/installation/index.html