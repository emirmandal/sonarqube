# SonarQube Installation on Centos 7

# Install CentOS 7 minimal
set hostname: ```sonarqube.local```

```sh
yum clean all ; yum -y update
yum -y install net-tools mc wget open-vm-tools
yum -y install epel-release
yum -y install htop
```
Install PostgreSQL 9.6 repository  
```sh
rpm -U https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
```
Install following packages:
```sh
yum install postgresql96 postgresql96-server postgresql96-libs postgresql96-contrib
```
Initialize the database (only once):
```sh
/usr/pgsql-9.6/bin/postgresql96-setup initdb
```
Enable remote access to PostgreSQL database server:
```vi /var/lib/pgsql/9.6/data/postgresql.conf```

uncoment following line and set:
```sh
listen_addresses = '*'
```

```vi /var/lib/pgsql/9.6/data/pg_hba.conf```
```sh
# IPv4 local connections:
host    all             all             10.0.34.0/24            md5
host    all             all             10.0.3.0/24             md5
host    all             all             127.0.0.1/32            md5
```
Start and enable automatic start of PostgreSQL
```sh
service postgresql-9.6 start
chkconfig postgresql-9.6 on
systemctl list-unit-files | grep enabled
```

As postgres user create superuser
```sh
su postgres
psql
CREATE role adminpg LOGIN PASSWORD 'PGadmin' SUPERUSER;
```
```sh
CREATE EXTENSION adminpack;
```
Create user and database for SonarQube
```sh
create user sonar with password 'sonar';
\du
create database sonar with owner sonar encoding 'UTF8';
\l
\q
exit
```
Open ports for PostgreSQL and SonarQube
```sh
firewall-cmd --add-port 5432/tcp --permanent
firewall-cmd --add-port 9000/tcp --permanent
```

Test connection (127.0.0.1 or real IP)
```sh
psql -h 127.0.0.1 -d sonar -U sonar
\q
```
# Install Oracle JAVA
SonarQube Server require versions 8 of the JVM
Download rpm package
```sh
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u152-b16/aa0333dd3019491ca4f6ddbe78cdb6d0/jdk-8u152-linux-x64.rpm"
```
Install rpm
```sh
yum localinstall jdk-8u152-linux-x64.rpm
```
Select default Java:
```sh
alternatives --config java
```
Check the version of default JAVA:
```sh
java -version
```

# Install SonarQube 6.6
As `root` set following values 
```sh
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
```
You can verify the values
```sh
sysctl vm.max_map_count
sysctl fs.file-max
ulimit -n
```
Download sonar repository:
```sh
wget -O /etc/yum.repos.d/sonar.repo http://downloads.sourceforge.net/project/sonar-pkg/rpm/sonar.repo
```
Install sonar with yum
```sh
yum install sonar
```
SonarQube will be installed in `/opt/sonar/`

Confgure database parameters for SonarQube 
```sh
vi /home/sonar/sonarqube-6.6/conf/sonar.properties
```
In `# User credentials` section
```sh
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
```
In `#----- PostgreSQL 8.x/9.x` section
```sh
sonar.jdbc.url=jdbc:postgresql://127.0.0.1/sonar
```
In `# WEB SERVER` section add `-server` in `sonar.web.javaOpts`
```sh
sonar.web.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -server
```
Start SonarQube server:
```sh
/etc/init.d/sonar start
```
Log file location for troubleshooting
```sh
/opt/sonar/logs/
```
