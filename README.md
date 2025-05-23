# Spring App Deployment on WebSphere Liberty Using liberty-maven-plugin

This guide explains how to set up a Spring application with the Liberty Maven Plugin for streamlined development and deployment on WebSphere Liberty.

## Table of Contents

1. [Liberty Maven Plugin Overview](#1-liberty-maven-plugin-overview)
2. [Project Setup in IntelliJ](#2-project-setup-in-intellij)
3. [Configure POM for Liberty Maven Plugin](#3-configure-pom-for-liberty-maven-plugin)
4. [Spring Application Configuration](#4-spring-application-configuration)
5. [Development Workflow](#5-development-workflow)
6. [Customizing Liberty Configuration](#6-customizing-liberty-configuration)
7. [Production Deployment](#7-production-deployment)
8. [Troubleshooting](#8-troubleshooting)

## 1. Liberty Maven Plugin Overview

The Liberty Maven Plugin provides a suite of tools to integrate Liberty server management into your Maven build. Key benefits include:

- Start/stop the Liberty server from Maven
- Deploy applications automatically
- Run integration tests against a running server
- Package the server and application together
- Dev mode for automatic application redeployment during development

## 2. Project Setup in IntelliJ

### Create a New Maven Project

1. Open IntelliJ IDEA
2. Go to File > New > Project
3. Select "Maven" from the project types
4. Ensure JDK is set to Java 8
5. Click "Next"
6. Enter project details:
    - GroupId: `com.example`
    - ArtifactId: `spring-liberty-app`
    - Version: `1.0-SNAPSHOT`
7. Click "Finish"

## 3. Configure POM for Liberty Maven Plugin

Replace your `pom.xml` content with the following:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-liberty-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <liberty.version>3.7.1</liberty.version>
        <spring.version>5.3.27</spring.version>
    </properties>

    <dependencies>
        <!-- Spring Core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        
        <!-- Spring Web -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        
        <!-- Servlet API -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        
        <!-- JSTL -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        
        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.36</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.11</version>
        </dependency>
    </dependencies>

    <build>
        <finalName>spring-liberty-app</finalName>
        <plugins>
            <!-- Configure the Maven WAR Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.2</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>

            <!-- Liberty Maven Plugin -->
            <plugin>
                <groupId>io.openliberty.tools</groupId>
                <artifactId>liberty-maven-plugin</artifactId>
                <version>${liberty.version}</version>
                <configuration>
                    <serverName>springServer</serverName>
                    <include>runnable</include>
                    <bootstrapProperties>
                        <app.context.root>/${project.build.finalName}</app.context.root>
                        <default.http.port>9080</default.http.port>
                        <default.https.port>9443</default.https.port>
                    </bootstrapProperties>
                    <features>
                        <acceptLicense>true</acceptLicense>
                    </features>
                    <looseApplication>true</looseApplication>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 4. Spring Application Configuration

### Create the Directory Structure

Create the following directory structure:

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── example/
│   │           ├── config/
│   │           └── controller/
│   ├── liberty/
│   │   └── config/
│   │       └── server.xml
│   └── webapp/
│       └── WEB-INF/
│           ├── resources/
│           └── views/
```

### Create the Spring Configuration

Create `src/main/java/com/example/config/WebConfig.java`:

```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.example")
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        return resolver;
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/WEB-INF/resources/");
    }
}
```

Create `src/main/java/com/example/config/WebAppInitializer.java`:

```java
package com.example.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { WebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

### Create a Controller

Create `src/main/java/com/example/controller/HomeController.java`:

```java
package com.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {
    
    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("message", "Hello from Spring MVC on Liberty!");
        return "home";
    }
}
```

### Create JSP View

Create `src/main/webapp/WEB-INF/views/home.jsp`:

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Spring MVC on Liberty</title>
</head>
<body>
    <h1>Welcome to Spring MVC on Liberty</h1>
    <p>${message}</p>
</body>
</html>
```

### Configure Liberty Server

Create `src/main/liberty/config/server.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<server description="Liberty Server for Spring Applications">
    <featureManager>
        <feature>servlet-4.0</feature>
        <feature>jsp-2.3</feature>
        <feature>localConnector-1.0</feature>
        <feature>jndi-1.0</feature>
        <feature>ssl-1.0</feature>
        <feature>jaxrs-2.1</feature>
        <feature>cdi-2.0</feature>
    </featureManager>

    <httpEndpoint id="defaultHttpEndpoint" 
                  host="*" 
                  httpPort="${default.http.port}" 
                  httpsPort="${default.https.port}" />

    <webApplication id="spring-liberty-app" 
                   location="spring-liberty-app.war" 
                   contextRoot="${app.context.root}" />
</server>
```

## 5. Development Workflow

### Running the Application

Liberty Maven Plugin provides several goals to manage the server:

1. **Start the server in dev mode** (recommended for development):
   ```bash
   mvn liberty:dev
   ```
   This starts the server and automatically reloads the application when changes are detected.

2. **Start the server**:
   ```bash
   mvn liberty:start
   ```

3. **Stop the server**:
   ```bash
   mvn liberty:stop
   ```

4. **Run a full build and deploy**:
   ```bash
   mvn clean package liberty:run
   ```

5. **Create a runnable JAR** (containing both the app and server):
   ```bash
   mvn clean package
   ```

### Accessing the Application

Once the server is running, access your application at:
- http://localhost:9080/spring-liberty-app/

### Hot Deployment in Dev Mode

When running in dev mode:
1. Change your Java code or resources
2. Save the files
3. The server automatically recompiles and deploys the changes
4. Refresh your browser to see the changes

## 6. Customizing Liberty Configuration

### JVM Options

Create `src/main/liberty/config/jvm.options`:

```
-Xms256m
-Xmx512m
```

### Server Environment Variables

Create `src/main/liberty/config/server.env`:

```
JAVA_HOME=/path/to/your/java8/jdk
```

### Bootstrap Properties

Bootstrap properties can be specified in the `pom.xml`:

```xml
<bootstrapProperties>
    <app.context.root>/${project.build.finalName}</app.context.root>
    <default.http.port>9080</default.http.port>
    <default.https.port>9443</default.https.port>
    <app.logging.level>INFO</app.logging.level>
</bootstrapProperties>
```

## 7. Deploying to an Existing Liberty Server

1. Build the WAR file:
   ```bash
   mvn clean package
   ```

2. Copy the WAR to the Liberty server's dropins directory:
   ```bash
   cp target/spring-liberty-app.war /path/to/liberty/usr/servers/serverName/dropins/
   ```

## 8. Troubleshooting

### Common Issues

1. **Plugin Not Found**
    - Check that the Liberty Maven Plugin is correctly configured in your pom.xml
    - Verify your Maven settings.xml has access to the required repositories

2. **Application Not Starting**
    - Check logs in `target/liberty/wlp/usr/servers/springServer/logs/`
    - Verify that all required features are enabled in your server.xml
    - Ensure that your application has all the necessary dependencies

3. **ClassNotFoundException**
    - Make sure your pom.xml has all required dependencies
    - Verify scope is set correctly (provided vs. compile)
    - Check for conflicts between Liberty provided libraries and your dependencies

4. **Port Conflicts**
    - Change the default ports in your pom.xml bootstrapProperties or server.xml if they are already in use

### Logs Location

Logs are available at:
```
target/liberty/wlp/usr/servers/springServer/logs/
```

Key log files:
- `console.log` - Liberty server console output
- `messages.log` - Liberty server messages
- `ffdc/` - First Failure Data Capture logs for errors

