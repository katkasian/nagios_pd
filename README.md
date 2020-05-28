This is a rundown of basic setup for Nagios Core on Ubuntu, which also includes configuration of notifications on local host.
Links to instructions about integration with PagerDuty and comments are provided.


_modified from https://github.com/andrewpuch/nagios_setup _

__NAGIOS SETUP__
```
sudo su
apt-get update
apt-get upgrade -y
apt-get dist-upgrade -y
apt-get autoremove -y
apt-get install build-essential libgd-dev openssl libssl-dev apache2-utils apache2 php -y
dd if=/dev/zero of=/swap bs=1024 count=2097152
mkswap /swap && sudo chown root. /swap && sudo chmod 0600 /swap && sudo swapon /swap
sh -c "echo /swap swap swap defaults 0 0 >> /etc/fstab"
sh -c "echo vm.swappiness = 0 >> /etc/sysctl.conf && sysctl -p"
useradd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios
cd /root
curl -L -O http://prdownloads.sourceforge.net/sourceforge/nagios/nagios-4.0.8.tar.gz
tar xvf nagios-*.tar.gz
cd nagios-4.0.8
./configure --with-nagios-group=nagios --with-command-group=nagcmd
make all
make install
make install-commandmode
make install-init
make install-config
/usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-available/nagios.conf
cd ../
curl -L -O http://nagios-plugins.org/download/nagios-plugins-2.0.3.tar.gz
tar xvf nagios-plugins-*.tar.gz
cd nagios-plugins-2.0.3
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
make
make install
nano /usr/local/nagios/etc/nagios.cfg
```
Remove the hashtag in front of this line. 
```
Before --
#cfg_dir=/usr/local/nagios/etc/servers

After --
cfg_dir=/usr/local/nagios/etc/servers

mkdir /usr/local/nagios/etc/servers
```
Add Your Email
```
nano /usr/local/nagios/etc/objects/contacts.cfg

a2enmod rewrite cgi
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
ln -s /etc/apache2/sites-available/nagios.conf /etc/apache2/sites-enabled/
service nagios start
```
===!if this throws an error, follow instructions:
===!https://serverfault.com/questions/774498/failed-to-start-nagios-service-unit-nagios-service-failed-to-load-no-such-file/774501

```
service apache2 restart
ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios
```

__ADDING NOTIFICATIONS FOR LOCALHOST__
```
nano /usr/local/nagios/etc/objects/localhost.cfg
-----
```
Add the following to the host definition:

```

        check_command           check-host-alive
        check_interval          2
        retry_interval          1
        max_check_attempts      3
        check_period            24x7
        contact_groups          admins
        notification_interval   2
        notification_period     24x7
        notification_options    d,u,r
 
 ```
 -----
 Add the following to a service you would like to receive notifications about (in this example, root partition)
 
 ```
        check_command                   check_local_disk!20%!10%!/
        #check_command                  check_local_disk!95%90% / !=== change parameters by uncommenting to test notifications
        notifications_enabled           1
        contact_groups                  admins
        notification_period             24x7
        notification_options            c,w !==recieve notifications for "critical" and "warning states"
 ```     
        
 __INTEGRATION WITH PAGERDUTY__
 
 Instructions:
 https://www.pagerduty.com/docs/guides/nagios-integration-guide/
 - For a Google VM, follow steps for source installation/Amazon Linux & CentOS 6+
 - For one-way integration (PAGERDUTY can receive alerts but not modify anything on NagiosCore), follow up to the step 8.


