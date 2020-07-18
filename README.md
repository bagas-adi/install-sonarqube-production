# Install SonarQube for Production
## Description
This repo is for reference to install SonarQube for Production environment using PostgreSQL

<table>
	<thead>
		<tr>
			<th></th>
			<th>URL</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Install Docs</td>
			<td>https://docs.sonarqube.org/latest/setup/install-server/</td>
		</tr>
		<tr>
			<td>Hardware Recommendations</td>
			<td>https://docs.sonarqube.org/latest/setup/install-server/</td>
		</tr>
		<tr>
			<td>Requirements</td>
			<td>https://docs.sonarqube.org/latest/requirements/requirements/</td>
		</tr>
		<tr>
			<td>Download</td>
			<td>https://www.sonarqube.org/downloads/</td>
		</tr>
	</tbody>
</table> 

## Download Sonarqube 
Download SonarQube from https://www.sonarqube.org/downloads/
```
cd /opt
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.4.1.35646.zip
```
Use this user to run ./sonar.sh, because you can't run SonarQube with root user
```
useradd sonarqube -p testing
```
Change the owner of the file to sonarqube
```
chown sonarqube. -R /opt/sonarqube-8.4.1.35646
```
## PostgreSQL
1. Install postgreSQL 12
2. Create user & schema
```
CREATE USER sonarqube WITH PASSWORD 'sonarqube';
CREATE DATABASE sonarqube;
CREATE SCHEMA IF NOT EXISTS sonarqubeschema AUTHORIZATION sonarqube;

GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonarqube; 
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA sonarqubeschema TO sonarqube;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA sonarqubeschema TO sonarqube;

# alter default privilege for newly created table
ALTER DEFAULT PRIVILEGES
FOR USER sonarqube IN SCHEMA sonarqubeschema GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO sonarqube;

ALTER USER sonarqube SET search_path to sonarqubeschema;
```
3. Change pg_hba.conf at /var/lib/pgsql/12/data/pg_hba.conf
```
...
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     ident
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
...
```
4. Config SonarQube properties at conf/sonar.properties
```
...
#----- PostgreSQL 9.3 or greater
# By default the schema named "public" is used. It can be overridden with the parameter "currentSchema".
#sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube?currentSchema=my_schema
#sonar.jdbc.username=postgres
#sonar.jdbc.password=postgres
#sonar.jdbc.url=jdbc:postgresql://localhost/postgres?currentSchema=sonarqubeschema 

sonar.jdbc.username=sonarqube
sonar.jdbc.password=sonarqube
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube?currentSchema=sonarqubeschema 

...
```

## Web Server

Update conf/sonar.properties
```
...
# Binding IP address. For servers with more than one IP address, this property specifies which
# address will be used for listening on the specified ports.
# By default, ports will be used on all IP addresses associated with the server.
sonar.web.host=0.0.0.0
#sonar.web.host=192.0.0.1
#sonar.web.port=80
#sonar.web.context=/sonarqube

...
```
Update conf/wrapper.conf
```
# Path to JVM executable. By default it must be available in PATH.
# Can be an absolute path, for example:
#wrapper.java.command=/path/to/my/jdk/bin/java
#wrapper.java.command=java
wrapper.java.command=/usr/bin/java
```

## OpenJDK 11
```
yum install java-11-openjdk-devel
```
## ElasticSearch config
Edit max virtual memory at /etc/sysctl.conf
```
vi /etc/sysctl.conf
vm.max_map_count=262144
```

Edit File Descriptor Limits at /etc/security/limits.conf
```
vi /etc/security/limits.conf
...
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

sonarqube      hard    nofile          65535 
...
```


# Install SonarScanner

## Download Sonar Scanner here
```
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.4.0.2170-linux.zip
```

## Add PATH env var to bin/sonar-scanner 
```
echo $PATH
PATH=$PATH:/opt/sonar-scanner-4.4.0.2170-linux/bin
echo $PATH
```

