
---

Definitions:

- Build
	- Process of converting the source code to an executable code that can be run on a computer. Build also refers to the executable created.

- Publish
	- It refers to Build getting published to the location where they are stored.

- Test
	- It refers to a process which ensures software quality so that it’s bug free.

- Deploy
	- It refers to performing the process of application deployment to servers in different environments of the project so that it becomes available to the end user.

#### Java Types of files:

1. Source code files: (.java) contain actual java code.
2. Compiled class files: (.class) These are the bytecode files generated after compilation by `javac`. These are used by the JVM.They are often placed in a `/bin` or `/out` directory or a `target/classes` folder (in Maven).
3. Package files(.jar): Java ARchive files bundle `.class` files and resources into a distributable format.

#### Java ClassPath

Java Class Path is one way to tell applications **where to look for classes** and other files. This is specified as an Environment variable or configuration on the system.

- JVM uses the Java Classpath to find classes during runtime.
- Java commands and tools also use the classpath to locate classes.
- When building and running a Java project there are two classpaths involved:

	- Compile classpath
		- List of dependencies that are required for the JDK to be able to compile Java code into .class files

	- Runtime classpath
		- List of dependencies that are required to actually run the compiled Java code

#### Compiling using commands:

```bash
javac -d bin -cp @classpath.txt  @main_sources.txt
```

- -d specifies the directory to generate the compiled classes in (.class files)
- -cp specifies a list of directories, JAR archives and ZIP archives to search for class files which might be required for other classes to compile. (Here these files are present in classpath.txt and main_sources.txt)

Running the file

```bash
java -cp @classpath.txt com.crio.session5.activity1.WeatherAdvisorApplication 12.9762 77.6033 "2020-05-29 08:15" "2020-05-29 09:15" v1
```

Observe that we also provided the compile classpath along with runtime classpath.
We need java to know where to find the compiled classes which will be needed by some other classes which are dependent on it for execution (runtime).

Running Tests:

```bash
java -jar ./lib/junit-platform-console-standalone-1.8.1.jar -cp @classpath.txt --scan-class-path
```

-jar tells which executable to run. This is standalone jar that can be executed.
-cp tells classpath which is list of directories.
--scan-class-path scans all the directories of the classpath to find compiled classes that have JUnit test and runs the unit tests

Packaging java project:

A Jar(Java Archive) is standard package file format in java.
Used to **aggregate many Java class files** and associated metadata and resources (text, images, etc.) **into one jar file**. This jar file can be used to **distribute application software or libraries** on the Java platform. Others can include this jar when they compile their program that depends on these class files.

```bash
jar cvfm WeatherAdvisorApplication.jar ./jar_manifest.txt -C ./bin/ .
```

cvfm -> c - create new jar file , v - generate verbose output to standard output , f - specifies jarfile to be created. m - indicated manifest file we will be using. The manifest file includes name-value pairs.

C: indicates temporary changes to the directory. Classes are added from this directory to the JAR file. The dot indicates all classes (files).

**Manifest** is a special file that can contain information about the files packaged in a JAR file. By tailoring this "meta" information that the manifest contains, you enable the JAR file to serve a variety of purposes.

Running a jar file:

```bash
java  -jar ./WeatherAdvisorApplication.jar 12.9762 77.6033 "2020-05-29 08:15" "2020-05-29 09:15" v1
```

### Build Tools:

![[Pasted image 20250624195723.png]]

Gradle is a build tool which meets all the requirements of building Java applications. Once it’s set up, building your application is as simple as running a single command on the command line.

```bash
./gradlew build
```

That **includes compiling, testing, and packaging your application**, all with one command. 

We can tell gradle to do lot of things using `build.gradle` file.

A **library** provides a set of helper functions/objects/modules which your application code calls for specific functionality. Libraries typically focus on a narrow scope (e.g., strings, IO, sockets), so their methods also tend to be smaller and require fewer dependencies. It is just a collection of class definitions which you can include and use in your code.

These **libraries on which the application is dependent are called the dependencies** of the application.**Dependency management** allows teams to manage dependencies for multi-module projects and applications.

We can declare dependency in `build.gradle` file
```groovy
dependencies {
    implementation 'group:name:version'       // Main library
    testImplementation 'group:name:version'   // Only for tests
    compileOnly 'group:name:version'          // Compile-time only
    runtimeOnly 'group:name:version'          // Runtime only
}
```

| Configuration        | When It’s Used                                |
| -------------------- | --------------------------------------------- |
| `implementation`     | Used at compile + runtime (most common)       |
| `api`                | Like `implementation`, but exported to others |
| `compileOnly`        | Available at compile time, not bundled        |
| `runtimeOnly`        | Available only at runtime                     |
| `testImplementation` | Used only in test code                        |
What is a repository - It's a directory where all the **standard project jars, library jar,** plugins or any other project specific artifacts are stored and can be used by build tools like Gradle easily. Usually on the internet where it is accessible for everyone.

**Maven central repository** ([https://mvnrepository.com/](https://mvnrepository.com/)) is the standard go-to repository provided by the Maven community for Java developers.

Gradle requires the files to be organized in a specific folder structure for them to operate. If the code doesn’t follow this structure, it will not be able to function. Sample folder structure can be seen in the diagram below.

![[Pasted image 20250624200447.png]]

The **Gradle wrapper** also known as `gradlew`  is a script you add to your Gradle project and use to execute your build. The advantages are:
- you don’t need to have Gradle installed on your machine to build the project
- the wrapper guarantees you’ll be using the version of Gradle required by the project (and not necessarily the one installed locally)
- you can easily update the project to a newer version of Gradle, and push those changes to version control so other team members also use the newer version.

How do I see what version of Gradle the wrapper is using in a project?
Either run `./gradlew --version` or you can also inspect the contents of the `gradle/wrapper/gradle-wrapper.properties` file.

The gradlew script i**s tied to a specific Gradle version and not the locally installed Gradle version**. That’s very useful, because it means whoever manages the project can enforce what version of Gradle should be used to build it.

Yes, if you’re setting up a new Gradle project in an empty directory you can use `gradle init` to do that. This command also generates the gradlew script for you too.

![[Pasted image 20250624200948.png]]

#### Gradle Tasks:

The work that Gradle can do on a project is defined by one or more tasks. A **task** represents some atomic piece of work which a build performs. Gradle tasks are individual build actions you can run from the command line.
You might have one task to compile your Java code, another task to test the code, and yet another task to package the compiled classes into a jar file.

Developers have built a bunch of common tasks for Gradle wrapped under `plugins`.

**Plugin** is nothing but a set of useful tasks, such as compiling, setting domain objects, setting up source files, etc. grouped under it. **Applying a plugin** to a project means that the project allows the plugin to extend its capabilities.

The standard and much used **[Gradle Java plugin](https://docs.gradle.org/current/userguide/java_plugin.html)**, automatically adds tasks to compile, test, and package your application (and much more).

```groovy
plugin{
	id 'java'
}
```

##### Gradle `build`
	Build task includes compiling, testing, and packaging your application, all with one command. The `build` task depends on a lot of other tasks which will be executed before the `build` task.

The `build` task will compile all your Java classes and create a JAR file with them. Gradle outputs classes and JAR files to the directory named “build” inside your project root directory. If the “build” directory does not exist, Gradle will create it. The compiled classes are written to “build/classes” and the JAR file to “build/libs”.

Gradle remembers which tasks have been executed, and if no file required by a task has been changed, Gradle will not execute that task again. 

To do so you must first remove all previous build output. You do so using the clean task, like this:
```bash
./gradlew clean
```

##### Gradle custom tasks:

Sometimes, we want to execute various programs from Gradle that require input parameters (like our Weather Application). When we want to pass input arguments using Gradle tasks, we have two choices:

- setting **system properties** with the -D flag
-  System properties define the environment in which you run your Java programs
- setting **project properties** with the -P flag
- Most Java Projects need to use some simple parameters as key-value pairs, and are stored outside of compiled code. Project properties are used to store configuration data or settings for a specific Java project.

In the weatherApplication, we have defined a custom task in `build.gradle` which will enable us to pass command line arguments using -P flag to the main application which expects them.

```groovy
task runWeather(type: JavaExec, dependsOn: classes) {
	if ( project.hasProperty("args") ){
	        args project.args.split(',')    
	} 
	description = "Weather Advisor"   
	main = "com.crio.session5.activity1.WeatherAdvisorApplication"
	classpath = sourceSets.main.runtimeClasspath 
}
```

Running custom task

```bash
./gradlew runWeather -q -Pargs=12.9762,77.6033,"2019-05-29 08:15","2019-05-29 09:15",v1
```


#### Anatomy of build.gradle file

```groovy
// 1. Plugins
plugins {
    id 'java'                                 // Java plugin
    id 'application'                          // For running Java apps
}

// 2. Project Metadata
group = 'com.example'
version = '1.0.0'
description = 'My Java Project'

// 3. Java Compatibility
java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

// 4. Repositories (Where to fetch dependencies)
repositories {
    mavenCentral()                            // Standard public repo
    // jcenter(), google(), maven { url "..." }
}

// 5. Dependencies (Libraries you use)
dependencies {
    implementation 'com.google.code.gson:gson:2.10.1'
    testImplementation 'junit:junit:4.13.2'
}

// 6. Application Plugin Config (Optional)
application {
    mainClass = 'com.example.Main'            // Full class name with main()
}

// 7. Custom Tasks
task hello {
    doLast {
        println 'Hello, Gradle!'
    }
}
```