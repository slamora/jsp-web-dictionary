# Installing jsp-web-dictionary
Following instructions asume that you are using a Debian based GNU/Linux.

## Requirements
- JDK >= 6
- Tomcat >= 5.5
- jdbc-mysql-connector.jar (>=5) /usr/share/java
	- In Ubuntu (or Debian): sudo apt-get install libmysql-java

### Install Tomcat 8
To get the basic package, we can update our package source list and then install the main package
```sudo apt-get install tomcat8```

Take a look at the default Tomcat page by going to your IP address or domain followed by `:8080` in your web browser:
```your_domain_or_ip:8080```

Optional: install Tomcat admin package, these will allow us to use a web interface to control Tomcat.
```sudo apt-get install tomcat8-admin```

### Install JDK
```sudo apt-get install default-jdk```

### Optional: configure Tomcat Web Interface
Edit the file called `/etc/tomcat8/tomcat-users.xml` and define a user:
```
<tomcat-users>
    <user username="admin" password="password" roles="manager-gui,admin-gui"/>
</tomcat-users>
```
NOTE: Choose whatever username and password you would like.

Save your changes and then restart Tomcat to apply the changes.
```sudo service tomcat8 restart```


### Intall JDBC MySQL connector
```sudo apt-get install libmysql-java```


## Database preparation
1. Import SQL in directory `web/_var/DB_creation_v2.sql`. This will create a database called `dict` and their tables.

2. Create a user and grant privileges to access `dict` database.

## DBCP
[Download Apache Commons DBCP](https://commons.apache.org/proper/commons-dbcp/download_dbcp.cgi)
```
## Debian !
sudo apt-get install libcommons-dbcp-java
```

## Grant app permissions on Tomcat
+ /etc/tomcat5.5/policy.d/04webapps.policy   | $CATALINA_HOME
	grant codeBase "file:/usr/share/tomcat5.5-webapps/jspWebDict/-" {
	   permission java.net.SocketPermission "localhost:3306", "connect,resolve";
   };
   
	// WebApps permission to run DBCP 
	grant {
	  permission java.lang.RuntimePermission 
	  	"accessClassInPackage.org.apache.tomcat.dbcp.*";
	};


## Install Apache mod_jk connector
```
sudo apt-get install libapache2-mod-jk
```

Add URIEncoding="UTF-8" in /etc/tomcat6/server.xml --> Connector 8180 or 8080 (Debian)
NOTE: If you use the mod_jk connector (Tomcat<-->Apache) you should modify this connector too.


## Deploying jsp-web-dictionary application
1. Copy WAR file to `/var/lib/tomcat8/webapps/`
2. Configure user and password (to grant access to DB) in META-INF/context.xml


###################
# COMMON PROBLEMS #
###################

1) javax.naming.NamingException: Could not create resource factory instance
[Root exception is java.lang.ClassNotFoundException:
org.apache.tomcat.dbcp.dbcp.BasicDataSourceFactory]

 -> Solution: copy (download) naming-factory-dbcp.jar into 
 	/usr/share/tomcat5.5/common/lib/ directory  
	(some GNU/Linux distributions don't have it by default)


2) Datasource: Cannot load JDBC driver class 'com.mysql.jdbc.Driver'

 -> Solution: check that you have the "mysql-connector-java.jar". Frecuently 
	it can be found in the /usr/share/java path and will be referenced makin a simbolic 
	link 
    ```
    cd /usr/share/tomcat8/lib
    sudo ln -s ../../java/mysql-connector-java.jar mysql-connector-java.jar
    ```

3) java.security.AccessControlException: access denied (java.lang.RuntimePermission accessClassInPackage.org.apache.tomcat.dbcp.pool.impl)

 -> Solution: add to the policy file  /etc/tomcat6/policy.d/04webapps.policy 

	// WebApps permission to run DBCP 
	grant {
	  permission java.lang.RuntimePermission 
	  	"accessClassInPackage.org.apache.tomcat.dbcp.*";
	};

4) Cannot create PoolableConnectionFactory (Communications link failure Last 
	packet sent to the server was 0 ms ago.)

 -> Solution: Add to the policy file  /etc/tomcat6/policy.d/04webapps.policy:
  [$CATALINA_HOME_WEBAPPS = /var/lib/tomcat5.5/webapps/]
	 grant codeBase "file:/var/lib/tomcat5.5/webapps/jspWebDict/-" {
       permission java.net.SocketPermission "localhost:3306", "connect,resolve";
	 };

## Tomcat 8 cannot load website
Output messages in catalina.out
```
WARNING: Failed to scan [file:/usr/share/java/el-api-3.0.jar] from classloader hierarchy
java.io.FileNotFoundException: /usr/share/java/el-api-3.0.jar (No such file or directory)
    at java.util.zip.ZipFile.open(Native Method)
    at java.util.zip.ZipFile.<init>(ZipFile.java:219)
    at java.util.zip.ZipFile.<init>(ZipFile.java:149)
    at java.util.jar.JarFile.<init>(JarFile.java:166)
    at java.util.jar.JarFile.<init>(JarFile.java:130)
    at org.apache.tomcat.util.scan.JarFileUrlJar.<init>(JarFileUrlJar.java:60)
    ...
```

Solution described at [stackoverflow](https://stackoverflow.com/a/41138185/1538221) 
```
sudo apt-get install libservlet3.1-java
```

###############################################
+ Java libraries [SOLVED if you have installed "libmysql-java" package]
	/usr/share/tomcat6/lib/
		naming-factory.jar
		naming-factory-dbcp.jar
		naming-resources.jar

	/usr/share/java/
		mysql-connector-java.jar

NOTE: add a symbolic link in /usr/share/tomcat6/lib to 
	/usr/share/java/mysql-connector-java.jar
