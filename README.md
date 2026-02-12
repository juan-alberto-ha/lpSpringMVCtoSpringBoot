# Spring MVC to Spring Boot

This is the repository to upload the artifacts of the Spring MVC to Spring Boot liveProject

## Running the Legacy Application
### Software needed and setup
- Java JDK (version 8 or above): I have installed OpenJDK Runtime Environment Corretto-11.0.29.7.1 (build 11.0.29+7-LTS)
- Maven: I have Apache Maven 3.9.6
- Apache Tomcat: version 9.0.115 (so that it will work with Java 1.8, as required by the code)

To install Apache Tomcat I have downloaded the TAR.GZ file from https://tomcat.apache.org/download-10.cgi .
Then I run:
```bash
sudo mv ~/Downloads/apache-tomcat-9.0.115.tar.gz /opt/.
cd /opt
sudo tar -xf apache-tomcat-9.0.115.tar.gz
sudo rm apache-tomcat-9.0.115.tar.gz
```

Starting application dowloaded from: https://github.com/grafpoo/SpringMVCtoSpringBoot/tree/main/webreport

I use IntelliJ IDEA 2025.2.4 Community Edition.

To create the WAR file for Tomcat I go to the location where the downloaded pom.xml file is stored and I run:

```bash
mvn package
```

I have not defined Apache Tomcat as a service. I define __CATALINA_HOME__ and then start Tomcat.
```
export CATALINA_HOME=/opt/apache-tomcat-9.0.115

sudo $CATALINA_HOME/bin/catalina.sh start

```

To deploy the WAR file we can use curl. Make sure to have user and password set in file tomcat-users.xml. I have renamed the WAR file to webreport.war:
```
curl --user robot:robot --upload-file target/webreport.war http://localhost:8080/manager/text/deploy?path=/webreport
```

### Determining the endpoint

In class Matchcontroller we find the __/reports/season-report/<season>__ endpoint. <season> should have this format: yyyy-yyyy, for example '2019-2020'
```java
@RequestMapping("/reports")
public class MatchController 
.
.
.
@GetMapping("season-report/{season}")
@ResponseStatus(value = HttpStatus.OK)
```

