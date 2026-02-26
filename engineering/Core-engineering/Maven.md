
---

### Maven plugins:

A **Maven plugin** is a collection of **goals** that perform specific tasks during the **build lifecycle** of a project. These tasks can include:

- **Compiling** source code
- **Packaging** the project into a JAR or WAR
- **Running tests**
- **Deploying** to a remote repository
- **Generating documentation**
- And more

Each maven plugin can have multiple tasks each of which is called goals, which can be executed by following syntax.

```bash
mvn [plugin-name]:[goal-name]
mvn compile:compile
```

There are two types of plugins:

1. Build plugins: Executed during build process and should be configured in `<build/>` element.
2. They execute during the site generation process and they should be configured in the <reporting/> element of the pom.xml.

Structure of maven plugin:

- **GroupId**: Maven group (usually `org.apache.maven.plugins`)
- **ArtifactId**: Plugin name (e.g., `maven-compiler-plugin`)
- **Version**: Specific version of the plugin
- **Configuration**: Optional settings for the plugin
- **Executions**: Define when the plugin runs and which goals it should execute

example:
```xml
<build>
  <plugins>
    <plugin>
      <!-- GroupId: Identifies the plugin's organization -->
      <groupId>org.apache.maven.plugins</groupId>

      <!-- ArtifactId: Name of the plugin -->
      <artifactId>maven-compiler-plugin</artifactId>

      <!-- Version: Specific version of the plugin -->
      <version>3.10.1</version>

      <!-- Configuration: Optional plugin-specific settings -->
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
        <compilerArgs>
          <arg>-Xlint:all</arg>
        </compilerArgs>
      </configuration>

      <!-- Executions: Define plugin execution phases and goals -->
      <executions>
        <execution>
          <id>default-compile</id> <!-- Optional ID for the execution -->
          <phase>compile</phase> <!-- When to run -->
          <goals>
            <goal>compile</goal> <!-- What goal to run -->
          </goals>
        </execution>
        <execution>
          <id>default-test-compile</id>
          <phase>test-compile</phase>
          <goals>
            <goal>testCompile</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>

```

The `<executions>` block inside a plugin tells Maven:

1. **WHEN** to run the plugin — using the `<phase>` element (e.g., `compile`, `package`, `install`, etc.)
2. **WHAT** task(s) to run — using the `<goals>` element (e.g., `compile`, `testCompile`, `run`, etc.)

```xml
<executions>
  <execution>
    <id>custom-id</id>       <!-- Optional identifier -->
    <phase>compile</phase>   <!-- When to run -->
    <goals>
      <goal>compile</goal>   <!-- What to run -->
    </goals>
    <configuration>          <!-- Optional: overrides plugin-wide config -->
      <!-- goal-specific settings -->
    </configuration>
  </execution>
</executions>
```

Goals are just the set of executions in a given lifecycle. Phases are important to understand.

There are three kinds of lifecycle in any java build lifecycle.

1. default: Handles your project deployment
2. clean: Handles project cleaning
3. site: Handles project site creation

In default life cycle the following are important phases:

1. validate: verifies pom is correctly configured.
2. compile: compiles all the files in `src/main/java` output goes to `target/classes`.
3. test: Run unit tests in a framework
4. package: Packages the compiled code into distributable format like .jar,.war
5. verify: In verify we run integration tests, code analysis or custom verification. Like sonar
6. install: Install the package into local repo. - Copies the built JAR/WAR and POM to your local repo.
7. deploy: Copy the final package to a remote repository (for sharing with others).

We can run any phase by 

```sh
mvn <phase-name>
```

There are always **pre** and **post** phases to register **goals**, which must run prior to, or after a particular phase.

eg:

```
mvn compile
```

And Maven will automatically run all preceding phases in the correct order.

A **goal** represents a specific task which contributes to the building and managing of a project.

When we execute _mvn post-clean_ command, Maven invokes the clean lifecycle consisting of the following phases.

- pre-clean
- clean
- post-clean

Maven clean goal (clean:clean) is bound to the _clean_ phase in the clean lifecycle. Its **clean:cleangoal** deletes the output of a build by deleting the build directory. Thus, when _mvn clean_ command executes, Maven deletes the build directory.

We can customize this behavior by mentioning goals in any of the above phases of clean life cycle.

- When a phase is called via Maven command, for example **mvn compile**, only phases up to and including that phase will execute.

### POM:

Project object model is a fundamental unit of work in Maven. The POM contains information about the project and various configuration detail used by Maven to build the project(s).

POM also contains the goals and plugins. While executing a task or goal, Maven looks for the POM in the current directory. It reads the POM, gets the needed configuration information, and then executes the goal.

```xml
<project xmlns = "http://maven.apache.org/POM/4.0.0" xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"> 
	<modelVersion>4.0.0</modelVersion> 
	<groupId>com.companyname.project-group</groupId>
	<artifactId>project</artifactId> 
	<version>1.0</version>
</project>
```

- All POM files require the **project** element and three mandatory fields: **groupId, artifactId, version**.

Group Id: 
	This is an Id of project's group. This is generally unique amongst an organization or a project. For example, a banking group com.company.bank has all bank related projects.
artifact Id:
	This is an Id of the project. This is generally name of the project. For example, consumer-banking. Along with the groupId, the artifactId defines the artifact's location within the repository.
version:
	This is the version of the project. Along with the groupId, It is used within an artifact's repository to separate versions from each other. 

Important plugins:

1. compiler plugin

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.10.1</version>
  <configuration>
    <source>1.8</source>
    <target>1.8</target>
  </configuration>
</plugin>
```

2. Jar plugin:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <version>3.3.0</version>
  <configuration>
    <archive>
      <manifest>
        <mainClass>com.example.Main</mainClass>
      </manifest>
    </archive>
  </configuration>
</plugin>

```

### Sping-boot plugin:

Spring boot plugin is used to simplyfy building, running and deploying spring-boot applications.
- Run your app (`spring-boot:run`)
- Package your app as an executable JAR/WAR (`spring-boot:repackage`)
- Build Docker images (`spring-boot:build-image`)

We can add plugin like this--

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

repackage: is bound to package phase




