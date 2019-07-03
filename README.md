# SK_BigDataEngEdu
 빅데이터 엔지니어링 교육

 Install CDH Using Cloudera Manager

 기타
 github desktop 설치


 https://desktop.github.com/


 # SSH 접속
 ```
 ssh -i xxx.pem id@ip
 ```

## SSH 접속 시 권한 문제 발생시 해결 방법
```
chomd 600 xxx.pem
```
* 참고 : https://naleejang.tistory.com/40


# System Pre-configuration Checks


 ## 1. Update yum
```
 sudo yum update -y
```

## 2. Change the run level to multi-user text mode
 ```
 # 현재 세팅 확인
 sudo systemctl get-default
 # 세팅 변경
 sudo systemctl set-default multi-user.target
 ```
## 3. Disable SE Linux
```
sudo vi/etc/sysconfig/selinux
# 아래 속성값 변경
SELINUX=enforce -> SELINUX=disabled
# 서버 재시작
reboot
```

## 4. Disable firewall
방화벽 설정이 되어있지 않다면 이 단계는 건너띄면
```
sudo yum install firewalld
sudo systemctl unmask firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo systemctl disable firewalld
```
* 참고 : systemctl stop firewalld


## 5. Check vm.swappiness and update permanently as necessary.

 Set the value to 1

```
sudo sysctl -w vm.swappiness=1
```
* 참고
 https://unix.stackexchange.com/questions/123074/does-changing-of-the-swappiness-need-a-reboot


## 6. Disable transparent hugepage support permanently

```
vi /etc/grub.conf

kernal 뒤에 ‘transparent_hugepage=never’추가

grub2-mkconfig -o /boot/grub2/grub.cfg

echo never > /sys/kernel/mm/transparent_hugepage/enabled

cat /sys/kernel/mm/transparent_hugepage/enabled 에서
[never]에 이렇게 닫혀있는지 확인하시면 잘된 것임
```

## 7. Check to see that nscd service is running
```
yum install nscd

systemctl start nscd

#확인
systemctl status nscd
```

## 8. Check to see that ntp service is running Disable chrony as necessary
```
yum install ntp
# 확인
rpm -qa ntp
```
## 9. Disable IPV6
```
vi /etc/default/grub
GRUB_CMDLINE_LINUX=“ipv6.disable=1 ’ 추가
grub2-mkconfig -o /boot/grub2/grub.cfg
shutdown -r now
```
* 참고 : https://www.thegeekdiary.com/centos-rhel-7-how-to-disable-ipv6/

## 10.During the installation process, Cloudera Manager Server will need to remotely access each of the remaining nodes. In order to facilitate this, you may either set up an admin user and password to be used by Cloudera Manager Server or setup a private/public key access. Whichever method you choose, make sure you test access with ssh before proceeding.

## 11.Show that forward and reverse host lookups are correctly resolved

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


# 12.Change the hostname of each of the nodes to match the FQDN that you entered in the /etc/hosts file.

```
sudo vi /private/etc/hosts

13.125.84.13    cm.bdai.com cm
13.125.92.82    m1.bdai.com m1
13.209.123.110  d1.bdai.com d1
13.209.220.119  d2.bdai.com d2
13.209.55.218   d3.bdai.com d3
```

* Reboot each of the nodes
