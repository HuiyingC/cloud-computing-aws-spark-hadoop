# Cloud Computing Programming Assignment 2 - Build Web App on AWS

An e-commerce Django web-application and a backend relational database (MySQL), which stores the data regarding the application. The application will need an EC2 instance running a webserver and an RDS instance running a database the webserver can access.

### Architecture
AWS EC2 (Ubuntu) + Django + MySQL

### Execution
```
1. Download key pair aws-ec2.pem
2. chmod 400 aws-ec2.pem
3. ssh -i aws-ec2.pem ubuntu@ec2-3-212-42-252.compute-1.amazonaws.com
4. cd ecommerce
5. python3 manage.py runserver 0.0.0.0:8000
6. login as admin
	- username: admin
	- password: password
7. login as existing customer
	- username: testuser_004
	- password: testuser_004
8. signup as a new customer
9. Have fun!
```

==============================================================
### Create EC2 Instance and Access via SSH
```
# Download .pem key
chmod 400 ~/.ssh/labsuser.pem
```
```
# Associate an Elastic IP Address to Instancse
# Find Public IPv4 address in EC2 instance details
# Access via SSH
(Ubuntu) ssh -i aws-ec2.pem ubuntu@ec2-3-212-42-252.compute-1.amazonaws.com
(AWS Linux) ssh -i estore.pem ec2-user@54.237.226.170
```
```
# scp local project to EC2 Instance via pem key
scp -i ../../Downloads/aws-ec2.pem -r ecommerce ubuntu@ec2-3-212-42-252.compute-1.amazonaws.com:/home/ubuntu/
```

### Deployment

#### Env
```
sudo apt-get install -y python3-pip
sudo apt-get install -y virtualenv
virtualenv -p python3 env
source env/bin/activate
pip3 install -r requirement.txt
```

#### MySQL
```
# AWS Linux
sudo yum install -y https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
sudo yum install -y mysql-community-client

# Failing GPG Keys, run
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
(ref: 
https://gist.github.com/sshymko/63ee4e9bc685c59a6ff548f1573b9c74
https://dev.mysql.com/doc/mysql-repo-excerpt/5.7/en/linux-installation-yum-repo.html)
sudo yum install mysql-community-server

sudo grep 'temporary password' /var/log/mysqld.log
mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';

mysql> CREATE USER 'admin'@'%' IDENTIFIED BY 'MyNewPass4!';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%';

mysql> SELECT USER,plugin FROM mysql.user;
mysql> SHOW GRANTS FOR 'admin'@'%';
mysql> FLUSH PRIVILEGES;


# Ubuntu
https://stackoverflow.com/questions/11657829/error-2002-hy000-cant-connect-to-local-mysql-server-through-socket-var-run

# Install packages
sudo apt update & sudo apt upgrade
sudo apt install python3-pip
python3 -m pip install --upgrade pip
sudo apt-get install mysql-server
sudo apt install mysql-client-core-8.0
sudo apt-get install python-mysqldb
sudo apt-get install python3-dev default-libmysqlclient-dev build-essential
sudo apt-get install mysql-client
pip3 install mysqlclient

sudo service mysqld start
sudo service mysqld status

admin
lVWwHFYF415Zg4LCkSPy

ecom_customer.html
UPDATE ecom_customer SET profile_pic = 'profile_pic/CustomerProfilePic/smile.jpeg' WHERE id = 1;

# force delete
SET foreign_key_checks = 0;
DELETE FROM auth_user WHERE id=2;

# Access via endpoint
mysql -u admin -h ubuntu-estore-db.cn9k8wzus1yk.us-east-1.rds.amazonaws.com -p

```

#### Start Django Server
```
# Init if doesn't exist
django-admin startproject ecommerce

# Modify settings.py file
DATABASES = { 
    'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'estoredb',
    'USER': 'admin',
    'PASSWORD': 'lVWwHFYF415Zg4LCkSPy',
    'HOST': 'ubuntu-estore-db.cn9k8wzus1yk.us-east-1.rds.amazonaws.com',
    'PORT': '3306',
    }
}

ALLOWED_HOSTS = ['3.212.42.252']


python3 manage.py makemigrations
python3 manage.py migrate

python3 manage.py createsuperuser
username: admin
password: password

python3 manage.py runserver 0.0.0.0:8000
# Running at http://3.212.42.252:8000/ 
# BINGO !!!!!!!!

# Signup as customer
username: testuser_001
password: testuser_001

```


=============================================================
---
### Troubleshooting
#### Invalid http_host header
In settings.py file, add url in 
```python
ALLOWED_HOSTS = ['198.168.202.0', 'localhost', '127.0.0.1']
```

#### django.core.exceptions.ImproperlyConfigured: mysqlclient 1.4.3 or newer is required; you have 1.0.3
https://stackoverflow.com/questions/55657752/django-installing-mysqlclient-error-mysqlclient-1-3-13-or-newer-is-required  

In \__init\__.py
```python
import pymysql
pymysql.version_info = (1, 4, 3, "final", 0)  # add this line 
pymysql.install_as_MySQLdb()
```

#### Permission denied (publickey).
https://stackoverflow.com/questions/18551556/permission-denied-publickey-when-ssh-access-to-amazon-ec2-instance
[Connect to your Linux instance using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
[Common causes for connection issues](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html#TroubleshootingInstancesConnectingSSH)

#### Access denied for user 'root@localhost' (using password:NO)
https://stackoverflow.com/questions/2995054/access-denied-for-user-rootlocalhost-using-passwordno
```
# Configuration in django project setting.py
https://stackoverflow.com/questions/64523533/environment-properties-are-not-passed-to-application-in-elastic-beanstalk

 if 'RDS_HOSTNAME' in os.environ:
     DATABASES = {
         'default': {
         '    ENGINE': 'django.db.backends.mysql',
              'NAME': os.environ['RDS_DB_NAME'],
              'USER': os.environ['RDS_USERNAME'],
              'PASSWORD': os.environ['RDS_PASSWORD'],
              'HOST': os.environ['RDS_HOSTNAME'],
              'PORT': os.environ['RDS_PORT'],
     }
 }
 else:
     env_vars = get_environ_vars()
     DATABASES = {
         'default': {
         'ENGINE': 'django.db.backends.mysql',
         'NAME': env_vars['RDS_DB_NAME'],
         'USER': env_vars['RDS_USERNAME'],
         'PASSWORD': env_vars['RDS_PASSWORD'],
         'HOST': env_vars['RDS_HOSTNAME'],
         'PORT': env_vars['RDS_PORT'],
     }
 }
```

### Citation
Django web-application development based on this [github repository](https://github.com/sumitkumar1503/ecommerce)

