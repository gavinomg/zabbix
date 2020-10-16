My server info :

Server IP : 10.66.11.4
Disk : 25 GB
RAM : 1GB
vCPU : 2
Service : ZABBIX 5.0
Web : Apache
Database : Mariadb
Php : 7.2 must
User Name : Admin [‘A’ must be capital]
Password : zabbix

Step #1: Check your server upto date and ip address using below command
[root@Zabbix-Srv ~]# cat /etc/redhat-release
[root@Zabbix-Srv ~]# ip r
[root@Zabbix-Srv ~]# yum install epel-release -y
[root@Zabbix-Srv ~]# yum -y update
Step #02: Now download zabbix 5.0 repo from official web site.
[root@Zabbix-Srv ~]# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
[root@Zabbix-Srv ~]# yum clean all
[root@Zabbix-Srv ~]# yum install zabbix-server-mysql zabbix-agent
[root@Zabbix-Srv ~]# yum install centos-release-scl
Edit [zabbix+frontend] this section from below location. This option by default not enable.

[root@Zabbix-Srv ~]# vi /etc/yum.repos.d/zabbix.repo
Change enable=0 to enable=1

[zabbix+frontend]
enabled=1
and then save this file.

Step #03: Install Zabbix web mysql and mariadb database.
[root@Zabbix-Srv ~]# yum install zabbix-web-mysql-scl zabbix-apache-conf-scl
[root@Zabbix-Srv ~]# yum -y install mariadb-server
[root@Zabbix-Srv ~]# systemctl start mariadb
[root@Zabbix-Srv ~]# systemctl enable mariadb
Create zabbix database and import default database. By default mysql root password is blank if you not setup.

[root@Zabbix-Srv ~]# mysql -u root -p
After mysql terminal then run below command.

MariaDB [(none)]> create database zabbix_db character set utf8 collate utf8_bin;
MariaDB [(none)]> create user zabbix_user@localhost identified by 'passw0rd123';
MariaDB [(none)]> grant all privileges on zabbix_db.* to zabbix_user@localhost;
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> \q
Import default Zabbix 5.0 database using below command. Run this command then type password for zabbix_user

[root@Zabbix-Srv ~]# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix_user -p zabbix_db
Step #04: Edit Zabbix server configuration file this is important
[root@Zabbix-Srv ~]# vi /etc/zabbix/zabbix_server.conf
Edit below details into conf file. If this is missing so you can’t start Zabbix service.

DBName=zabbix_db
DBUser=zabbix_user
DBPassword=passw0rd123
Select your location time zone from below file.

[root@Zabbix-Srv ~]# vi /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
See this file is last line. comment out last line and enter your local time zone.

php_value[date.timezone] = Asia/Dhaka
Step #05: Start all service and setup firewall allow web port and zabbix service port.
[root@Zabbix-Srv ~]# systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
[root@Zabbix-Srv ~]# systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
[root@Zabbix-Srv ~]# firewall-cmd --add-service={http,https} --permanent
[root@Zabbix-Srv ~]# firewall-cmd --add-port={10051/tcp,10050/tcp} --permanent
[root@Zabbix-Srv ~]# firewall-cmd --reload
You see Zabbix service not start. So don’t worry. This is issue is selinux service. Must be disable selinux service and reboot your server then solved it.

Check selinux status using command

[root@Zabbix-Srv ~]# sestatus
Open selinux config file

[root@Zabbix-Srv ~]# vi /etc/selinux/config
set SELINUX=disabled

then save file and reboot your server.

[root@Zabbix-Srv ~]# reboot
After complete reboot and browse your server ip from any browser. http://10.66.11.4/zabbix/

Step #06: Start web installation follow step by step for complete this process.
This is first look Zabbix 5.0. click Next step

zabbix-server-5-install

Step #07: This is check of pre-requisites. If all is ok.
so click Next step

zabbix-server-install-check-pre-requisites

Step #08: Configure database connection step. I am created database, user and password from step #3.
So now enter this details. Click Next Step

zabbix-server-install-configure-db-connection

Step #09: Zabbix server details like host, port and if you want to any name here.
Click Next step

zabbix-server-details

Step #10: Pre-installation summary. Already you enter details so just see this is ok.
Click Next step

pre-installation-summary

Step #11: Installation completed and see Congratulation! message.
Click Finish.

successfully-installed-zabbix-frontend

Step #12: Then you see come login page and enter default user name and password.
User Name : Admin [‘A’ must be capital]
Password : zabbix

Finally you see Zabbix 5.0 new dashboard as like below.
