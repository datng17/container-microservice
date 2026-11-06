---
title: "Initialize Spring Boot Project"
date: "2026-09-30"
weight: 1
chapter: false
pre: "<b>2.3.1. </b>"
---

### 1. Create project directory

Example: Create project in the `IaC-ECS-CDK` directory.

* **Create Spring Boot project using the Extension in VS Code (in the `fcaj-spring` folder)**

Open VS Code, select the `IaC-ECS-CDK` directory, and click **Select folder**.

![Architect](/images/UPDATE/02-environment/3.infraSpring/1.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/2.png)

Press `Ctrl + Shift + P` (or `Cmd + Shift + P` on macOS) to open the Command Palette -> type **java** in the search box -> select **Java: Create Java Project**

![Architect](/images/UPDATE/02-environment/3.infraSpring/3.png)

Select the following configurations in order:
  * **Project type:** Spring Boot -> Maven Project
  * **Version:** 4.2.0 (M2)
  * **Project language:** Java
  * **Spring Initializr (input group ID):** `com.example`
  * **Spring Initializr (input artifact ID):** `fcaj-spring`
  * **Spring Initializr (input package name):** `com.example.fcaj_spring`
  * **Spring Initializr (specify packaging type):** `Jar`
  * **Spring Initializr (Java version):** `25` (Corresponding to JDK 25)
  * **Spring Initializr (choose dependencies):** `Spring Boot Devtools`, `Thymeleaf`, `Spring Web`, `Spring Boot Actuator`

![Architect](/images/UPDATE/02-environment/3.infraSpring/4.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/5.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/6.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/7.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/15.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/16.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/17.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/18.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/19.png)

![Architect](/images/UPDATE/02-environment/3.infraSpring/20.png)

After completing the steps in creating the Spring Boot project, a window will appear to select the directory to save the Spring Boot project. Select the **IaC-ECS-CDK** directory and click **Generate into folder**.

![Architect](/images/UPDATE/02-environment/3.infraSpring/21.png)

After successful creation, we will have the `fcaj-spring` folder inside the **IaC-ECS-CDK** directory.

Next, click **Run java** to check that the Spring Boot project has downloaded all dependencies and works normally.

- Run the following command in the terminal to start the Spring Boot project:

~~~terminal
.\mvnw spring-boot:run
~~~

As you can see in the image, the project is running on port `8080`.

![Architect](/images/UPDATE/02-environment/3.infraSpring/8.png)

---

### 2. Add required libraries to `pom.xml`

#### Add Log4j2 Library

The Log4j2 library supports Asynchronous Loggers based on the LMAX Disruptor library, minimizing latency for the main application thread when writing high volumes of logs, outperforming traditional logging libraries.

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/10.png)

#### Add Centralized AWS SDK v2 Dependency Management

~~~xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>bom</artifactId>
            <version>2.25.16</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/22.png)

#### Add AWS SDK v2 DynamoDB & Enhanced Client

~~~xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>dynamodb</artifactId>
</dependency>
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>dynamodb-enhanced</artifactId>
</dependency>
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/23.png)

#### Add AWS X-Ray Core SDK & Trace Interceptors and Spring Integration

~~~xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-xray-recorder-sdk-core</artifactId>
    <version>2.15.3</version>
</dependency>
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-xray-recorder-sdk-aws-sdk-v2</artifactId>
    <version>2.15.3</version>
</dependency>
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-xray-recorder-sdk-spring</artifactId>
    <version>2.15.3</version>
</dependency>
~~~

![Architect](/images/UPDATE/02-environment/3.infraSpring/24.png)

---

### 3. Test project on Postman

Create an HTTP GET request to `http://localhost:8080/actuator/health` to test.

The response returns `"status": "UP"`, indicating that the project is operating normally.

![Architect](/images/UPDATE/02-environment/3.infraSpring/9.png)

{{% notice info %}}
If you encounter an issue where you cannot test HTTP GET with `http://localhost:8080/actuator/health`, please check the configuration in **pom.xml** to ensure the **spring-boot-starter-actuator** dependency has been added. If not, add the library:

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
~~~
{{% /notice %}}
