
bit.ly/2Hrvegs \
github :star7357 \
https://github.com/ep-code-box/bigdata_study/tree/master/total_excute

# 각 IP 접속하기.
ssh -i d:/skcc.pem centos@[퍼블릭_IP]

ssh -i d:/skcc.pem centos@15.164.146.95
ssh -i d:/skcc.pem centos@15.164.151.13
ssh -i d:/skcc.pem centos@15.164.86.198
ssh -i d:/skcc.pem centos@15.164.5.68
ssh -i d:/skcc.pem centos@15.164.63.69

# update yum
```
sudo yum update
sudo yum install -y wget
```
#public 
```
15.164.146.95    master01.cdhcluster.com mn1
15.164.151.13   util01.cdhcluster.com   util01
15.164.86.198   data01.cdhcluster.com   dn1
15.164.5.68     data02.cdhcluster.com   dn2
15.164.63.69    data03.cdhcluster.com   dn3
```
# /etc/hosts 파일 수정 (priviate)
```
sudo -vi /etc/hosts 

172.31.7.227    master01.cdhcluster.com mn1
172.31.12.148   util01.cdhcluster.com   util01
172.31.8.146   data01.cdhcluster.com   dn1
172.31.10.234   data02.cdhcluster.com   dn2
172.31.3.22     data03.cdhcluster.com   dn3

```

# passwd 수정
sudo passwd centos

# /etc/ssh/sshd_config 파일 수정
```
$sudo vi sshd_config

PasswordAuthentication yes
$sudo systemctl restart sshd.service
```

# key gen
```
ssh-keygen
ls ~/.ssh/
ssh-copy-id -i ~/.ssh/id_rsa.pub t4h2
ssh-copy-id -i ~/.ssh/id_rsa.pub t4h1
ssh-copy-id -i ~/.ssh/id_rsa.pub t4h3
ssh-copy-id -i ~/.ssh/id_rsa.pub t4h4
ssh-copy-id -i ~/.ssh/id_rsa.pub t4h5
ssh t4h1
ssh t4h3
ssh t4h2
```


# jdk 설치 
```
yum list java*jdk-devel
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```

# repository
```
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
vi /etc/yum.repos.d/cloudera-manager.repo
cat /etc/yum.repos.d/cloudera-manager.repo

java -version

```
# mysql-JDBC Connector 설치 
```
**configure repository 
sudo yum install -y wget
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz


$ tar -xvf  mysql-connector-java-5.1.47.tar.gz
$sudo mkdir -p /usr/share/java/
$cd mysql-connector-java-5.1.47
$sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
$cd  /usr/share/java
$ sudo yum install mysql-connector-java
```

# 방화벽해제

vi /etc/sysconfig/selinux
SELINUX=enforcing => disabled
init 6

iptables -L
sestatus

#클라우데라서버설치
yum install cloudera-manager-daemons cloudera-manager-server
yum install cloudera-manager-daemons cloudera-manager-agent
cd /etc/cloudera-scm-agent/
ls
vi config.ini  --수정없음.
service cloudera-scm-agent start

#마리아db설치
yum install mariadb-server
systemctl stop mariadb
cd /etc
ls
vi my.conf
systemctl enable mariadb
systemctl start mariadb
/usr/bin/mysql_secure_installation
#마리아 db DB,USER 생성
mysql -u root -p
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
cd /usr/share/cmf/schema
ls
cat scm_prepare_database.sh
./scm_prepare_database.sh mysql scm scm toor

# scm server start
systemctl start cloudera-scm-server
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
 