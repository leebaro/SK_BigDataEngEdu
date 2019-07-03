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
sudo yum install wget

wget https://downloads.mariadb.com/Connectors/java/latest/mariadb-java-client-2.3.0.jar

cp -a mariadb-java-client-2.3.0.jar /usr/lib/jvm/java/jre/lib/ext/
```
참고  
https://xinet.kr/?p=1600
* On the host that you will install CM:
  * Configure the [repository](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/configure_cm_repo.html) for CM 5.15.2
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
