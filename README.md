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

In class Matchcontroller.java we find the __/reports/season-report/<season>__ endpoint. <season> should have this format: yyyy-yyyy, for example '2019-2020'
```java
@RequestMapping("/reports")
public class MatchController 
.
.
.
@GetMapping("season-report/{season}")
@ResponseStatus(value = HttpStatus.OK)
```

## Modernizing the Legacy Application with Spring Boot

I have used __spring initialzr__ (https://start.spring.io/) to create the skeleton of the bootified application. 
These are the settings I have used: 
- Project: Maven
- Language: Java
- Spring Boot: 4.0.2
- Project Metadata Group: liveproject
- Project Metadata Artifact: webreport
- Packaging: Jar
- Configuration: Properties
- Java: 17
- Dependencies: Lombok, Spring Web, Thymeleaf, H2 Database, Spring Data JPA, Spring Boot Actuator

I switch the Java alternative, in order to use Java 17:
```
sudo update-java-alternatives --set /usr/lib/jvm/java-1.19.0-openjdk-amd64

```
I checked that the dowloaded application from Spring Initializr can be packaged using Maven. JAR was created.

To check that the downloaded application can run we execute:
```
mvn spring-boot:run
```
### Migration of Spring MVC code to Spring Boot skeleton
The approach that I will follow is to migrate (basically copy) the code from the Spring MVC application to the Spring Boot.


#### Files in resources directory
I copied the files in the ``resources`` directory.
Remove all files from the ``resources`` folder, except for ``application.properties`` and the data file ``epl2019-2020.json`` .
In file ``application.properties`` we add the properties for the persistence layer:
```
spring.datasource.url=jdbc:h2:~/epl-test
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=create
```
Additionally, in order to keep the ``webreport`` context path, I have added:
```
server.servlet.context-path=/webreport
```
I have copied directories ``fragments`` and ``reports`` to the ``resources/templates`` directory. I have also copied the ``css`` directory to the ``resources/static`` directory of the project.

In file ``SeasonReport.html`` I modified the reference to the ``main.css`` file as follows: 
```html
<link rel="stylesheet" media="screen" th:href="@{/css/main.css}" />
```

#### Java source files
Package ``config``: copied files MatchLoader.java and MatchResultDeserializer.java 
- MatchLoader.java : Added Jackson dependency to pom.xml. Copied match/Match.java, match/MatchRepository.java and config/MatchResultDeserializer.java. Necessary to replace the javax.persistence imports with jakarta.persistence in the classes of the ``match`` package. Additionally, commented 
```java
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextRefreshedEvent;
```
and added import of ``org.springframework.boot.CommandLineRunner`` . In the class, I modified the method signature to 
```java
public void run(String... args) throws Exception 
```

Package ``match`` :
- Removed MatchResultDeserializer.java
- MatchService.java : copied season/Season.java
- MatchController.java 
- MatchRepository.java : Added the @Repository annotation.

Package ``liveproject.webreport``: 
- WebreportApplication.java : Class with @SpringBootApplication annotation

