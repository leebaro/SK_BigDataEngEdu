# SK_BigDataEngEdu
 빅데이터 엔지니어링 교육

 Install CDH Using Cloudera Manager

# System Pre-configuration Checks

 기타
 github desktop 설치


 https://desktop.github.com/


 ## SSH 접속
 ```
 ssh -i xxx.pem id@ip
 ```

### SSH 접속 시 권한 문제 발생시 해결 방법
```
chomd 600 xxx.pem
```
* 참고 : https://naleejang.tistory.com/40


## System Pre-configuration Checks
```
# 고려사항
6단계 이전에 reboot 할 경우 변경한 설정 값이 초기화 됨
```

 ### 1. Update yum
```
 sudo yum update -y
```

### 2. Change the run level to multi-user text mode
 ```
 # 현재 세팅 확인
 sudo systemctl get-default
 # 세팅 변경
 sudo systemctl set-default multi-user.target
 ```
### 3. Disable SE Linux
```
sudo vi /etc/sysconfig/selinux
# 아래 속성값 변경
SELINUX=enforce -> SELINUX=disabled
```

### 4. Disable firewall
방화벽 설정이 되어있지 않다면 이 단계는 건너띄면 됨
```
sudo yum install firewalld
sudo systemctl unmask firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo systemctl disable firewalld
```
* 참고 : systemctl stop firewalld


### 5. Check vm.swappiness and update permanently as necessary.

 Set the value to 1

```
sudo sysctl -w vm.swappiness=1
```
* 참고
 https://unix.stackexchange.com/questions/123074/does-changing-of-the-swappiness-need-a-reboot


### 6. Disable transparent hugepage support permanently

1. Add the “transparent_hugepage=never” kernel parameter option to the grub2 configuration file. Append or change the “transparent_hugepage=never” kernel parameter on the GRUB_CMDLINE_LINUX option in /etc/default/grub file.
아래 코드에 "transparent_hugepage=never" 부분만 추가하면 됨
```
# vi /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="nomodeset crashkernel=auto rd.lvm.lv=vg_os/lv_root rd.lvm.lv=vg_os/lv_swap rhgb quiet transparent_hugepage=never"
GRUB_DISABLE_RECOVERY="true"
```

2. Rebuild the /boot/grub2/grub.cfg file by running the grub2-mkconfig -o command. Before rebuilding the GRUB2 configuration file, ensure to take a backup of the existing /boot/grub2/grub.cfg.
On BIOS-based machines
```
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

2. On UEFI-based machines(redhat일 경우만, 아니면 3번으로 넘어감)
```
 grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

3. Reboot the system and verify option are in effect.
```
sudo shutdown -r now
```

4. Verify the parameter is set correctly
```
# cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-514.10.2.el7.x86_64 root=/dev/mapper/vg_os-lv_root ro nomodeset crashkernel=auto rd.lvm.lv=vg_os/lv_root rd.lvm.lv=vg_os/lv_swap rhgb quiet transparent_hugepage=never LANG=en_US.UTF-8
```

5. 정상 변경 여부 확인
```
sudo su
sudo echo never > /sys/kernel/mm/transparent_hugepage/enabled
sudo reboot
#다시 ssh 접속 후
cat /sys/kernel/mm/transparent_hugepage/enabled
```
* 참고  
https://www.thegeekdiary.com/centos-rhel-7-how-to-disable-transparent-huge-pages-thp/

### 7. Check to see that nscd service is running
```
yum install nscd

systemctl start nscd

#확인
systemctl status nscd
```

### 8. Check to see that ntp service is running Disable chrony as necessary
```
yum install ntp
# 확인
rpm -qa ntp
```
### 9. Disable IPV6
```
vi /etc/default/grub
GRUB_CMDLINE_LINUX=“ipv6.disable=1 ’ 추가
grub2-mkconfig -o /boot/grub2/grub.cfg
shutdown -r now
```
* 참고 : https://www.thegeekdiary.com/centos-rhel-7-how-to-disable-ipv6/

### 10.During the installation process, Cloudera Manager Server will need to remotely access each of the remaining nodes. In order to facilitate this, you may either set up an admin user and password to be used by Cloudera Manager Server or setup a private/public key access. Whichever method you choose, make sure you test access with ssh before proceeding.

* 아이디/패스워드를 모든 서버에 동일하게 설정(실습에선 centos 계정을 사용
* key를 이용하는 경우 잘 안되는 경우가 있음. 모든 서버에 한 번씩 미리 접속해보면 접속 기록이 남아서 문제가 해결될 경우도 있음

```
# 비밀번호 변경
passwd centos

sudo vi /etc/ssh/sshd_config
PasswordAuthentication yes
reboot
```
ssh 시 key없애고 패스워드로 로그인 확인  
참고 : https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-password-login/  
/etc/ssh/sshd_config


### 11.Show that forward and reverse host lookups are correctly resolved

*  In this lab, we will use /etc/hosts Files setting to accomplish this
* Add the necessary information to the /etc/hosts files Check to make sure that File lookup has priority
* Use getent to make sure you are getting proper host name and ip address

```
# 로컬 PC에서 아래와 같이 설정함(아래는 macOS의 경우)
sudo vi /private/etc/hosts

아래 정보 입력
13.125.84.13    cm.bdai.com cm
13.125.92.82    m1.bdai.com m1
13.209.123.110  d1.bdai.com d1
13.209.220.119  d2.bdai.com d2
13.209.55.218   d3.bdai.com d3
```


### 12.Change the hostname of each of the nodes to match the FQDN that you entered in the /etc/hosts file.
모든 서버의 hosts 파일 업데이트
```
# 서버에서는 private ip를 사용해야함
sudo vi /etc/hosts

172.31.1.187      cm.bdai.com cm
172.31.2.44       m1.bdai.com m1
172.31.8.54       d1.bdai.com d1
172.31.0.244      d2.bdai.com d2
172.31.4.72       d3.bdai.com d3
```

* Reboot each of the nodes

# Path B install using CM 5.15.x

[The full rundown is here.](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_cdh.html) You will have to modify your package repo to get the right release. The default repo download always points to the latest version.

Use the documentation to complete the following objectives:

## [Install JDK](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cdh_ig_jdk_installation.html#topic_29_1) on all the nodes.
```
sudo yum install java-1.8.0-openjdk-devel
```
참고 : https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora


## Install a supported JDBC [mysql connector](https://dev.mysql.com/downloads/connector/j/5.1.html) or [mariadb connector](https://downloads.mariadb.org/connector-java/) on all nodes
### MariaDB 설치
mariadb는 cm에만 설치하면 됨
```
# sudo vi /etc/yum.repos.d/MariaDB.repo

[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

sudo yum install MariaDB or sudo yum clean all
```
참조 : https://zetawiki.com/wiki/CentOS7_MariaDB_%EC%84%A4%EC%B9%98

### connector 설치
```
# https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_mariadb.html
# https://dev.mysql.com/downloads/connector/j/5.1.html에서 파일 다운로드 후 각 노드에 복사, 아래와 같이 wget으로 하면 됨

# connector 다운로드
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz

# 압축 풀기
tar zxvf mysql-connector-java-5.1.47.tar.gz

# 파일 복사
sudo cp mysql-connector-java-5.1.47/mysql-connector-java-5.1.47.jar /usr/share/java/mysql-connector-java.jar

# 파일 확인
sudo ls /usr/share/java/

```
참고  
https://xinet.kr/?p=1600


### On the host that you will install CM:
#### Step 1: Configure a Repository for Cloudera Manager

  * Configure the [repository](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/configure_cm_repo.html) for CM 5.15.2

```
# sudo wget <repo_file_url> -P /etc/yum.repos.d/
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/

sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
```

#### Step 2: Install Java Development Kit
https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cdh_ig_jdk_installation.html

https://download.oracle.com/otn/java/jdk/8u202-b08/1961070e4c9b4e26a04e7f5a083f551e/jdk-8u202-linux-x64.tar.gz


#### Step 3: Install Cloudera Manager Server
```
sudo yum install cloudera-manager-daemons cloudera-manager-server
```

#### Step 4: Install and Configure MariaDB for Cloudera Software
https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_mariadb.html#install_cm_mariadb

##### Configuring and Starting the MariaDB Server
```
sudo systemctl stop mariadb
```

아래 내용 추가
```
# sudo vi /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

#In later versions of MariaDB, if you enable the binary log and do not set
#a server_id, MariaDB will not start. The server_id must be unique within
#the replicating group.
server_id=1

binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid
```

Ensure the MariaDB server starts at boot:
```
sudo systemctl enable mariadb
```

Start the MariaDB server:
```
sudo systemctl start mariadb
```

Run /usr/bin/mysql_secure_installation to set the MariaDB root password and other security-related settings. In a new installation, the root password is blank. Press the Enter key when you're prompted for the root password. For the rest of the prompts, enter the responses listed below in bold:
```
sudo /usr/bin/mysql_secure_installation
```

```
[...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] Y
New password:
Re-enter new password:
[...]
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
[...]
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

#### Creating Databases for Cloudera Software

1. Log in as the root user, or another user with privileges to create database and grant privileges:
```
mysql -u root -p
```

```
-- services
-- Cloudera Manager Server	scm	scm
-- Activity Monitor	amon	amon
-- Reports Manager	rman	rman
-- Hue	hue	hue
-- Hive Metastore Server	metastore	hive
-- Sentry Server	sentry	sentry
-- Cloudera Navigator Audit Server	nav	nav
-- Cloudera Navigator Metadata Server	navms	navms
-- Oozie	oozie	oozie

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'rman';
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'hue';
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive';
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry';
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'nav';
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'navms';
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';

SHOW DATABASES;
SHOW GRANTS FOR 'hive'@'%';
```

Start CM
preparing the Cloudera Manager Server Database

Step 6: Install CDH and Other Software
https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_software_cm_wizard.html
```
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
```
  * Install CM
  * Install and enable Maria DB (or a DB of your choice)
    * Don’t forget to secure your DB installation
    * You can refer to instructions below for more
details on installing Maria DB

Install the mysql connector or mariadb connector Create the necessary users and databases
▪ Grant them the necessary rights Setup the CM database
    •
•
Start the CM server and prepare to install the cluster through the CM GUI installation process
Do not continue until you can browse your CM instance at port 7180




Install JDK 1.8 (All nodes)
install jdk
# Installing the JDK Manually
# https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cdh_ig_jdk_installation.html#topic_29_1
# jdk를 local 다운로드 받아서 각 노드에 복사

```
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" https://download.oracle.com/otn/java/jdk/8u202-b08/1961070e4c9b4e26a04e7f5a083f551e/jdk-8u202-linux-x64.tar.gz
```

sudo mkdir -p /usr/java
sudo tar xvfz /home/centos/jdk-8u202-linux-x64.tar.gz -C /usr/java/
setup java path
# JAVA 경로 지정
sudo vi /etc/profile # export JAVA_HOME=/usr/java/jdk1.8.0_202 추가
source /etc/profile
env | grep JAVA_HOME
# JAVA_HOME 출력되면 성공


20190704 시작
#
## SSH 접속
```
ssh -i xxx.pem id@ip
```

### SSH 접속 시 권한 문제 발생시 해결 방법
```
chomd 600 xxx.pem
```
* 참고 : https://naleejang.tistory.com/40



## check hostname resolution
 first put something in /etc/hosts and then

```
# 호스##에 등록된 정보 확인
getent hosts cm.bdai.com
```

** setup a password for centos
```
sudo passwd centos
sudo vi /etc/ssh/sshd_config
	change ->
PasswordAuthentication yes
sudo systemctl restart sshd.service
```

## Update /etc/host

```
sudo vi /etc/hosts
#	add ->
#******* THIS IS PRIVATE IP
172.31.10.5     cm.bdai.com     cm
172.31.1.196    d1.bdai.com     d1
172.31.5.204    d2.bdai.com     d2
172.31.7.82     d3.bdai.com     d3
172.31.1.17     d4.bdai.com     d4
```


## Change the hostname
해당 서버에 맞는 호스트명으로 변경
```
# cm server
sudo hostnamectl set-hostname cm.bdai.com
hostname -f

# d1 server
sudo hostnamectl set-hostname d1.bdai.com
hostname -f

# d2 server
sudo hostnamectl set-hostname d2.bdai.com
hostname -f

# d3 server
sudo hostnamectl set-hostname d3.bdai.com
hostname -f

# d4 server
sudo hostnamectl set-hostname d4.bdai.com
hostname -f

```

## REBOOT THE SERVER AND CHECK
```
reboot
```


## Install JDK on all machines
// From the Mac
```
cd ~/Downloads
scp -i ~/KeyPair/SKT.pem ~/Downloads/jdk-8u201-linux-x64.rpm ec2-user@ad3:. &
// From each of the nodes
sudo -i rpm -ivh /home/centos/jdk-8u201-linux-x64.rpm
```
OR
```
yum list java*jdk-devel
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
#sudo yum install -y oracle-j2sdk1.8
```



**아래 부분은 cm에서만 진행**

## Configure repository
```
sudo yum install -y wget
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo \
-P /etc/yum.repos.d/
```

## change the baseurl within cloudera-manager.repo to fit the version you want to install
```
#맞는 버전 확인하기
https://www.cloudera.com/documentation/enterprise/release-notes/topics/cm_vd.html

sudo vi /etc/yum.repos.d/cloudera-manager.repo
-> 아래 내용 변경
baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.2/

sudo rpm --import \
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
java -version
```

## Install Cloudera Manager
```
sudo yum install -y cloudera-manager-daemons cloudera-manager-server
```


## Install MariaDB
```
sudo yum install -y mariadb-server
```

{ // use this repo in case yum install does not work
sudo vi /etc/yum.repos.d/MariaDB.repo
	add ->
sudo rpm --import https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
}

```
** 실습 시에는 아래 구성을 변경할 필요 없다.
sudo vi /etc/my.cnf
	add ->
******************************************** Don't add this line

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
transaction-isolation = READ-COMMITTED
# Disabling symbolic-links is recommended to prevent assorted security risks;
# to do so, uncomment this line:
symbolic-links = 0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550
#expire_logs_days = 10
#max_binlog_size = 100M

#log_bin should be on a disk with enough free space.
#Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
#system and chown the specified folder to the mysql user.
log_bin=/var/lib/mysql/mysql_binary_log

#In later versions of MariaDB, if you enable the binary log and do not set
#a server_id, MariaDB will not start. The server_id must be unique within
#the replicating group.
server_id=1

binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
******************************************** Don't add this line
```

MariaDB 설정
```
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo /usr/bin/mysql_secure_installation

# 아래와 같이 세팅하면 됨

Set root password? [Y/n] Y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] N
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```


## Install mysql connector
// From the mac
```
scp -i ~/KeyPair/SEBC_HP.pem ~/Downloads/mysql-connector-java-5.1.47.tar.gz ec2-user@acm:.
```
OR
```
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
```
```
// From cm node
cd /home/centos

tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/

cd mysql-connector-java-5.1.47

sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
```

## Create the databases and users in MariaDB
```
mysql -u root -p
```


## Create the databases and users in MariaDB
```
mysql -u root -p

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY '1q2w3e4r';

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY '1q2w3e4r';

CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY '1q2w3e4r';

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY '1q2w3e4r';

CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'metastore'@'%' IDENTIFIED BY '1q2w3e4r';

CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY '1q2w3e4r';

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY '1q2w3e4r';

FLUSH PRIVILEGES;
SHOW DATABASES;
EXIT;
```


## Setup the CM database
```
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm 1q2w3e4r

sudo rm /etc/cloudera-scm-server/db.mgmt.properties

sudo systemctl start cloudera-scm-server

sudo tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log

# INFO - 아래 로그가 보이면 성공 WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.

```



```
# 아래 로그가 보인다면 성공
# INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server.
```

## cloudera manager wizard

* http://cm:7180 접속

* 계정 : admin / admin

* Cloudera Enterprise 체험판 선택

* CDH 클러스터 설치에 대한 호스트를 지정합니다.
 -> cm d1 d2 d3 d4
* 패키지: 모든 패키지는 추후 세팅함
* 동일한 아이디 비밀번호로 지정
* 역할 할당 사용자 지정하기 ([참고문서](http://daum.net][https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm_ig_host_allocations.html#host_role_assignments))
* 데이터베이스 설정 - 호스트, 데이터베이스, 유저, 패스워드 입력 - 테스트 연결: CM에서 각 서비스의 패키지 설치를 위한 설정


## Install Sqoop
1. Add Service로 이동
2. Sqoop1 패키지 선택 (Sqoop2는 deprecated 될 가능성 있음)
3. Gateway: 모든 노드

## Install Impala
최초에 Parcel에서 Impala를 선택했으면 또 할 필요 없음
1. Add Service로 이동
2. statestore: CM 노드
3. metastore: CM 노드
4. impalad: 모든 데이터 노드

## Install Kafka
1. parcel로 이동 (우상단 선물 그림 아이콘)
2. KAFKA Download 버튼 클릭
3. Distribute 버튼 클릭
4. Activate 버튼 클릭
5. Add Service로 이동
6. Kafka Broker 설치: 모든 데이터 노드
7. 옵션 설정은 전부 default로 진행


# Let’s test out our cluster


## create user “training” in linux (All nodes)
```
sudo su
adduser training
usermod -aG wheel training
exit
sudo su training
groups
# training wheel -> wheel 추가 확인
```

## create user “training” in hdfs
```
su hdfs
hdfs dfs -mkdir /user/training
hdfs dfs -chown training:training /user/training
hdfs dfs -ls /user
```

## create the sample tables that will be used for the rest of the test
```
# zip 파일들을 미리 1번 노드에 업로드 함
scp ~/Desktop/dlp/authors.zip centos@cm:.

# 실습 파일을 이용
sudo mv *.zip /training
sudo chmod training:training *.zip
su - training
unzip -Z authors.sql.zip
unzip -Z posts23.sql.zip
mysql -u root -p
```
아라 sql 파일에 테이블 생성부터 데이터 입력 스크립트가 있음
```
CREATE DATABASE test DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
use test;
source /home/training/authors.sql
source /home/training/posts23.sql

create user 'training'@'localhost' identified by 'training';
create user 'training'@'%' identified by 'training';
grant all on test.* to 'training'@'%' identified by 'test';
GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';
select user,host from mysql.user where user='training' ;
exit;
```

```
create user 'centos'@'localhost' identified by '1q2w3e4r';
create user 'centos'@'%' identified by '1q2w3e4r';
GRANT ALL ON *.* TO 'centos'@'%' IDENTIFIED BY '1q2w3e4r';
```


## extract tables posts from the database and create Hive tables (managed)
sqooping posts table
```
# target_dir 옵션을 넣으면 경로 변경 가능

sqoop import --connect jdbc:mysql://cm:3306/test \
--username centos \
--password 1q2w3e4r \
--split-by id \
--columns id,author_id,title,description,content,date \
--table posts \
--fields-terminated-by "\t" \
--hive-table default.posts \
--create-hive-table \
--hive-import

# 모든 서버에 training 계정 생성
sqoop import --connect jdbc:mysql://cm:3306/test \
--username training \
--password 1q2w3e4r \
--split-by id \
--columns id,author_id,title,description,content,date \
--table posts \
--fields-terminated-by "\t" \
--hive-table default.posts \
--create-hive-table \
--hive-import

```

```
sudo su training

hdsf dfs -ls
```

## extract tables authors from the database and create Hive tables (external)
sqooping authors table
```
sqoop import --connect jdbc:mysql://cm:3306/test \
--username centos \
--password 1q2w3e4r \
--split-by id \
--columns id,first_name,last_name,email,birthdate,added \
--table authors \
--fields-terminated-by "\t" \
--hive-table default.authors \
--create-hive-table \
--hive-import

sqoop import --connect jdbc:mysql://cm:3306/test \
--username training \
--password 1q2w3e4r \
--split-by id \
--columns id,first_name,last_name,email,birthdate,added \
--table authors \
--fields-terminated-by "\t" \
--hive-table default.authors \
--create-hive-table \
--hive-import
```

```
#
hive

ALTER TABLE default.authors SET TBLPROPERTIES('EXTERNAL'='TRUE');
```

```
hive
desc foramtted default.authros;
```

## copy
```
hdfs dfs -cp /user/hive/warehouse/authors /user/training/
```
```
-- change hdfs path on schema
USE default;
DESC extended default.authors;
ALTER TABLE default.authors SET LOCATION "/user/training/authors";
```
## test hive query
```
drop table results purge;
create table results
stored as textfile
as select a.id as id, a.first_name as fname, a.last_name as lname, count(*) num_posts
 from authors a
 join posts p on (a.id = p.author_id)
 group by a.id, a.first_name, a.last_name
 ;
 ```

mysql에 테이블 생성 후 데이터 삭제
 ```
 create table results
 as select a.id as id, a.first_name as fname, a.last_name as lname, count(*) num_posts
  from authors a
  join posts p on (a.id = p.author_id)
  group by a.id, a.first_name, a.last_name;

delete from results;
```


```
sqoop export \
--connect jdbc:mysql://cm:3306/test \
--username training \
--password 1q2w3e4r \
--table results \
--export-dir /user/hive/warehouse/results \
--input-fields-terminated-by "\001"
```
OR   
```
sqoop export --connect jdbc:mysql://cm:3306/test --username centos --password 1q2w3e4r --table results --hcatalog-table results
```
