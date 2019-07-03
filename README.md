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


 # System Pre-configuration Checks


 1. Update yum
```
 sudo yum update -y
```

 2. Change the run level to multi-user text mode
 ```
 # 현재 세팅 확인
 sudo systemctl get-default
 # 세팅 변경
 sudo systemctl set-default multi-user.target
 ```




 3. Disable SE Linux

 4. Disable firewall

 5. Check vm.swappiness and update permanently as necessary.

 Set the value to 1


 sudo sysctl -w vm.swappiness=1
 참고
 https://unix.stackexchange.com/questions/123074/does-changing-of-the-swappiness-need-a-reboot


 6. Disable transparent hugepage support permanently

 7. Check to see that nscd service is running
 8. Check to see that ntp service is running Disable chrony as necessary
 9. Disable IPV6

 10.During the installation process, Cloudera Manager Server will need to remotely access each of the remaining nodes. In order to facilitate this, you may either set up an admin user and password to be used by Cloudera Manager Server or setup a private/public key access. Whichever method you choose, make sure you test access with ssh before proceeding.

 11.Show that forward and reverse host lookups are correctly resolved

 In this lab, we will use /etc/hosts Files setting to accomplish this

 Add the necessary information to the /etc/hosts files Check to make sure that File lookup has priority

 Use getent to make sure you are getting proper host name and ip address

 12.Change the hostname of each of the nodes to match the FQDN that you entered in the /etc/hosts file.

 Reboot each of the nodes