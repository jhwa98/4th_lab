
# 교육 시작전 확인 사항 
## 강의 자료 및 참고 
```
bit.ly/2Hrvegs \
github :star7357 \
https://github.com/ep-code-box/bigdata_study/tree/master/total_excute
https://github.com/kate0912/final_lab_2019/blob/master/20190521_%ED%86%B5%ED%95%A9%EC%8B%A4%EC%8A%B52%EC%9D%BC%EC%B0%A8.md
```

## local 에서 IP 접속하기.
```
ssh -i d:/skcc.pem centos@[퍼블릭_IP]

ssh -i skcc.pem centos@13.124.135.81
ssh -i skcc.pem centos@15.164.27.64
ssh -i skcc.pem centos@52.78.236.138
ssh -i skcc.pem centos@54.180.186.127
ssh -i skcc.pem centos@54.180.38.41
```

## local의  /etc/hosts 파일 변경후 접속 하기 
```
/etc/hosts 파일 변경후 ssh -i d:/skcc.pem centos@mn1

13.124.135.81    master01.cdhcluster.com mn1
15.164.27.64    util01.cdhcluster.com   util01
52.78.236.138   data01.cdhcluster.com   dn1
54.180.186.127  data02.cdhcluster.com   dn2
54.180.38.41    data03.cdhcluster.com   dn3
```

## update yum
```
sudo yum update
sudo yum install -y wget
```
# 1. hosts 파일 수정 및 cent os passwd 변경

## passwd 수정
sudo passwd centos

## /etc/ssh/sshd_config 파일 수정
```
$sudo vi sshd_config
>> PasswordAuthentication yes

$sudo systemctl restart sshd.service
```
# 2. host 파일 수정 및 keygen

## /etc/hosts 파일 수정 (priviate)
```
sudo -vi /etc/hosts 

172.31.7.227    master01.cdhcluster.com mn1
172.31.12.148   util01.cdhcluster.com   util01
172.31.8.146   data01.cdhcluster.com   dn1
172.31.10.234   data02.cdhcluster.com   dn2
172.31.3.22     data03.cdhcluster.com   dn3

```
## key gen
```
-- keygen 설정
cd .ssh

ssh-keygen -t rsa

ssh-copy-id -i ~/.ssh/id_rsa.pub mn1
ssh-copy-id -i ~/.ssh/id_rsa.pub dn1
ssh-copy-id -i ~/.ssh/id_rsa.pub dn2
ssh-copy-id -i ~/.ssh/id_rsa.pub dn3
```
# 3. hostname변경

```
hostname -f
sudo hostnamectl set-hostname util01.cdhcluster.com

sudo hostnamectl set-hostname data03.cdhcluster.com
hostname -f

init 6
```
# 4. install CM
## 4-1 Configure the repository for CM 5.15.2

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

# 5. jdk 설치 
```
AS-IS: sudo yum install -y openjdk java-1.8.0-openjdk-devel.x86_64 

TO-BE: sudo yum install oracle-j2sdk1.7

java -version

```


## 4-2.클라우데라서버설치
```
yum install cloudera-manager-daemons cloudera-manager-server

```
## 4-3. 마리아db설치
```
sudo yum install -y mariadb-server
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo /usr/bin/mysql_secure_installation
>> 나머지 Y (disable root login: n) 
>> toor

```
## 4-4. mysql-JDBC Connector 설치 
```
sudo yum install -y wget
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz

tar zxvf mysql-connector-java-5.1.47.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.47
sudo cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar

cd /usr/share/java/
sudo yum install mysql-connector-java
>> Y

```

## 4-5. 마리아 db DB,USER 생성

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

## Setup the CM database
```
sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm-user somepassword
sudo rm /etc/cloudera-scm-server/db.mgmt.properties
sudo systemctl start cloudera-scm-server
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
 ```

# Cluster 설정
```
Go to the CM GUI installer and install CDH 5.15.2 
1. Specify hosts for your CDH cluster installation
2. You may choose to use a common user / password 
3. Allow CM to install CM Agent on each of the nodes 
4. Allow CM to download, distribute and activate CDH on each of the nodes 
5. Choose the type of installation you want 
a. We will require Flume, Hive, Impala later on 
6. Assign roles to each of the hosts: Please refer here for hints.

### 클러스터 설치
http://15.164.27.64:7180/cmf/login
>> admin / admin

-- 선택
enterprise 체험판, trial 

--  호스토 지정
mn1, util01, dn1, dn2, dn3
>> 호스트 IP 확인하기

-- 리포지토리 설치
디폴트 답

-- JDK 설치 
체크(util만 설치 했을 경우)

-- ssh 로그인 인증 
유저/ 패스워드 (centos/ centos)

-- 클러스터 설치

### 기본 서비스 설치
>> https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm_ig_host_allocations.html#host_role_assignments 
>> 3 - 10 Worker Hosts without High Availability

#### HDFS
#### YARN
#### ▪ ZOOKEEPER 3개로 설정하기
-- 호스트 별 역할
. server: util01, mn1, dn1

172.31.7.227    master01.cdhcluster.com mn1    >> util1 CDH, Maria DB : SNN, Balancer,   
172.31.12.148   util01.cdhcluster.com   util01 >> mn1   : NN, CM
172.31.8.146    data01.cdhcluster.com   dn1     :
172.31.10.234   data02.cdhcluster.com   dn2     :
172.31.3.22     data03.cdhcluster.com   dn3     :

### 데이터베이스 설정
active monior: amon/amon-user/somepassword
report monior:rmon/rmon-user/somepassword

```
# Install Sqoop, Spark and Kafka
## • Create user “training” with password “training” and add to group wheel for sudo access.
```
-- 모든 host에 아래 계정 생성
cat /etc/passwd | grep training

sudo useradd training

sudo passwd training

sudo usermod -aG wheel training

-- 계정 그룹 설정 확인
getent group wheel

```

## • Log into Hue as user “training” with password “training”
### > This will create a HDFS directory for the user
```
> http://15.164.27.64:8889/
```
## Now, try ssh’ing into the hosts as user “training”
```
su training
```

## From the bit.ly folder, download all.zip and unzip it.
```
-- all.zip이 있는 윈도우 경로에서 아래 실행: 결과 값 호스트 홈 디렉토리에 업로드 됨 (util01, data01)
scp -i skcc.pem all.zip training@15.164.27.64:.&
```

### > Do this in both your CM host and one of the datanode hosts.
### > Go to training_material/devsh/scripts and review the setup.sh
### ▪ See how it works. We are going to use it to create some data for ourselves but it won’t work right away. See if you can figure out what needs to be done to make it work.
###  ▪ You will need to add user “training” with password “training” to your MySql installation and Grant the necessary rights
```
-- unzip(util01, data01에서 실행)
sudo yum install -y unzip
unzip all.zip
cd /home/training/training_materials/devsh/scripts

-- util01, data01에서 실행
 ./setup.sh
> accounts 테이블 HUE에서 확인 가능 (invalidate all metadata and rebuild index를 선택해야 impala에서도 hive metastore에 테이블을 볼 수 있음)

-- util01에서 진행
cat setup.sh
> 내용 확인  아래 sql 실행 필요
-- %: 모든 호스트에 대해서 권한을 줌
GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'training';
show grants for 'training'@'%';
```

## • Now, attempt some of the exercises that we did during our previous lessons in our new Hadoop cluster.
### > You should practice the following as a minimum
### ▪ Flume
```
-- 호스트 별 역할
. agent: util01
```

### ▪ Sqoop 1
```
-- 호스트 별 역할
. gateway: util01
```

### ▪ Hive
``` 
-- 호스트 별 역할
. gateway: 모두 (hive를 사용할 곳)
. metastoreserver, webhcat, hiveserver2 : util01
```

### ▪ Impala
```
-- 호스트 별 역할
. Impala Catalog Sever: util01
. Impala State Store: util01
. Impala Daemon: dn1~dn3
```
### Spark
### ▪ HUE
```
-- 호스트 별 역할
. hue server: util01
. load balancer: util01
```
### ▪ OOZIE
```
-- 호스트 별 역할
. oozie server: util01
```

# 해볼것들
## sqoop import + hive table
```
mysql -u training -p

show databases;

use loudacre;

show tables;

sqoop import \
--connect jdbc:mysql://util01/loudacre \
--username training \
--password training \
--table device \
--hive-import \
--hive-table device_dv1
```
## sqoop external import + hive loading
```
sqoop import \
--connect jdbc:mysql://util01/loudacre \
--table device \
--username training  \
--password training \
--as-parquetfile \
--target-dir /user/training/device_dv2

sqoop import \
--connect jdbc:mysql://util01/loudacre \
--table device \
--username training  \
--password training \
--as-parquetfile \
--fields-terminated-by '\t' \
--target-dir /loudacre/device_dv2



CREATE EXTERNAL TABLE ` `(
  `device_num` int,
  `release_dt` string,
  `device_name` string,
  `device_type` string)
COMMENT 'device_dv2'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS PARQUET 
LOCATION
  '/loudacre/device_dv2'
;


CREATE EXTERNAL TABLE `device_dv2`(
  `device_num` int,
  `release_dt` string,
  `device_name` string,
  `device_type` string)
  ROW FORMAT SERDE 'parquet.hive.serde.ParquetHiveSerDe'
  STORED AS 
    INPUTFORMAT "parquet.hive.DeprecatedParquetInputFormat"
    OUTPUTFORMAT "parquet.hive.DeprecatedParquetOutputFormat"
    LOCATION '/loudacre/device_dv2'




sudo -u hdfs hdfs dfs -mkdir /user/training/device_dv2

sudo -u hdfs hdfs dfs -chown training /user/training/device_dv2

```

# 참고사항
```
------------------------------------
yum install unzip

hue: training / training >> 처음 들어간 유저가 admin

hdfs 디렉토리 자동으로 만듦
sudo training traning

user modidify -add G wheel group ??? >> training 그룹 wheel 그룹에 넣기

all.zip 파일 unzip >> Mysql, 작업 노드(Data 노드)

Mysql, 작업노드 안에서 돌리기
setup.sh 안 돌거, 권한 문제 등으로 인해서
(accounts 테이블, 환경 설정 해줌)
-------------------------------------
-- raw data 디렉토리 바꿀 때: superuser 권한으로 접속해서 해야함 (user: hdfs)
sudo -u hdfs 
hdfs dfs -chmod 777


sudo -u hdfs hdfs dfs -mkdir /loudacre 

sudo -u hdfs hdfs dfs -chown training /loudacre


-------------------------------------
--- sqoop 2(얘는 서버 사용, 클라이언트-서버)가 아닌 sqoop1 설치하기
-------------------------------------
영향도 있음
>> ZOOKEEPER > HIVE > HUE
```



