
bit.ly/2Hrvegs \
github :star7357 \
https://github.com/ep-code-box/bigdata_study/tree/master/total_excute
https://github.com/kate0912/final_lab_2019/blob/master/20190521_%ED%86%B5%ED%95%A9%EC%8B%A4%EC%8A%B52%EC%9D%BC%EC%B0%A8.md

# 각 IP 접속하기.
ssh -i d:/skcc.pem centos@[퍼블릭_IP]

ssh -i skcc.pem centos@13.124.135.81
ssh -i skcc.pem centos@15.164.27.64
ssh -i skcc.pem centos@52.78.236.138
ssh -i skcc.pem centos@54.180.186.127
ssh -i skcc.pem centos@54.180.38.41

# local /etc/hosts
```
13.124.135.81    master01.cdhcluster.com mn1
15.164.27.64    util01.cdhcluster.com   util01
52.78.236.138   data01.cdhcluster.com   dn1
54.180.186.127  data02.cdhcluster.com   dn2
54.180.38.41    data03.cdhcluster.com   dn3
```

# update yum
```
sudo yum update
sudo yum install -y wget
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
>> PasswordAuthentication yes

$sudo systemctl restart sshd.service
```

# key gen
```
-- keygen 설정
cd .ssh

ssh-keygen -t rsa

ssh-copy-id -i ~/.ssh/id_rsa.pub mn1
ssh-copy-id -i ~/.ssh/id_rsa.pub dn1
ssh-copy-id -i ~/.ssh/id_rsa.pub dn2
ssh-copy-id -i ~/.ssh/id_rsa.pub dn3
```
# reboot a host
```
hostname -f
sudo hostnamectl set-hostname util01.cdhcluster.com

sudo hostnamectl set-hostname data03.cdhcluster.com
hostname -f

init 6
```

# repository
```
-- Configure repository
sudo yum install -y wget
sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo \
-P /etc/yum.repos.d/

sudo vi /etc/yum.repos.d/cloudera-manager.repo
>> baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.15.2/

-- change the baseurl within cloudera-manager.repo to fit the version you want to install
baseurl=https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.7.4/
for example: https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.14.4/

sudo rpm --import \
https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
java -version

```

# jdk 설치 
```
AS-IS: sudo yum install -y openjdk java-1.8.0-openjdk-devel.x86_64 

TO-BE: sudo yum install oracle-j2sdk1.7

java -version

```


# mysql-JDBC Connector 설치 
```
**configure repository 
sudo yum install -y wget
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz


$ tar zxvf  mysql-connector-java-5.1.47.tar.gz
$sudo mkdir -p /usr/share/java/
$cd mysql-connector-java-5.1.47
$sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
$cd  /usr/share/java
$ sudo yum install mysql-connector-java
```

#클라우데라서버설치
```
yum install cloudera-manager-daemons cloudera-manager-server
yum install cloudera-manager-daemons cloudera-manager-agent
cd /etc/cloudera-scm-agent/
ls
vi config.ini  --수정없음.
service cloudera-scm-agent start
```
#마리아db설치
```
yum install mariadb-server
systemctl stop mariadb
cd /etc
ls
vi my.conf
systemctl enable mariadb
systemctl start mariadb
/usr/bin/mysql_secure_installation
https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_mariadb.html#install_cm_mariadb
```

#마리아 db DB,USER 생성

```
mysql -u root -p

CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE rmon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rmon.* TO 'rmon-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'metastore-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry-user'@'%' IDENTIFIED BY 'somepassword';

CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie-user'@'%' IDENTIFIED BY 'somepassword';

FLUSH PRIVILEGES;
SHOW DATABASES;
EXIT;
```

# Setup the CM database
```
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user somepassword
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
sudo systemctl start cloudera-scm-server
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
 ```

# Cluster 지정 
'''
1. enterprise 체험판선택
2. host 지정 
 mn1, util01, dn1 , dn2, dn3
3. active monitor : amon 
   Report Monitor ; rmon
