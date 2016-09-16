# Maven Toolchains Bamboo Plugin
## Introduction
Software projects evolve. We update dependencies to never versions. But we still have to maintain older release brances wich still use older dependencies. Maven makes the dependency manegement for each build easy. You reference an artifact and it's exact version. Maven cares for fetching the correct binary and linking to it.

But there are some dependencies which are not so easily maintained. One of those is the Java Runtime used to compile the project. Maven uses the same Java runtime for compilation under which it runs. So if your project has to support Java 7 runtime environments you not only have to set the `maven-compiler-plugin` configuration option to use 1.7 target runtime, you also have to start Maven using Java 7. Only this way you can make sure your code does not use any Java runtime classes only available in later releases.

Getting back to your project. When you need to build your project using Java 7, you configure your bulid plan to use a Java 7 runtime. Easy. But once you make the next step and use Java 8 in the next release you get problems if you still have to maintain (and bulid) the older realeases using Java 7. So one branch of your project has to be build using Java 8 and the other branch has to be build using Java 7. How do you do this in Bamboo? The easy but cumbersome solution is to duplicate your build plan. Let the one build plan checkout the older branch and use Java 7, and the other build plan check out the newer branch and use Java 8. This is not [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)! If there are changes to apply to the bulid plan, you have to do it twice.

This is where [Maven Toolchains](https://maven.apache.org/guides/mini/guide-using-toolchains.html) come in. Toolchains allow you to configure in your `pom.xml` which JDK you want to use for the build. Your `pom.xml` is commited along with your source. When you checkout any branch Maven knows which JDK to use.

```xml
<plugins>
  ...
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.5.1</version>
  </plugin>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-toolchains-plugin</artifactId>
    <version>1.1</version>
    <executions>
      <execution>
        <goals>
          <goal>toolchain</goal>
        </goals>
      </execution>
    </executions>
    <configuration>
      <toolchains>
        <jdk>
          <version>1.7</version>
          <vendor>oracle</vendor>
        </jdk>
      </toolchains>
    </configuration>
  </plugin>
  ...
</plugins>
```

But the reference in the `pom.xml` is only half part of the story. It only says which tool Maven has to use. It does not tell Maven where to find the tool. You have to create a `toolchains.xml` in the Maven home folder. It contains the mapping between vendor/version and the path of the tool.

The Maven Toolchains Plugin for Bamboo does this for you. It analyzes the JDKs supported by the current agent, and creates a `toolchains.xml` file referencing this JDKs. All following Maven invocations will use this mapping and will be able to find the JDK your project requires.

## How It Works
The Maven Toolchains Bamboo plugin looks for JDK capabilities which use a specific naming scheme:
```
AnythingHere_VENDOR_VERSION
```
Basically it expects the vendor name and the JDK version to be at the end of the capability's name, separated by underscores. The plugin will ignore JDK capabilities which don't conform to this format.

Examples:
* JDK_Oracle_1.5 will be parsed to vendor=Oracle and version=1.5.
* JRE_OpenJDK1.6 will be ignored. The second underscore is missing.
* Custom_Built_JDK_ByMe_1.8 will be parsed to vendor=ByMe and version 1.8

## Parallel Build Support
The Maven home folder is a sub folder of the user's home folder. This means there is only one per user. So what if you setup multiple agents to run on one machine? They will share the same Maven home folder (asumed you run all agents under the same user). But as long as you configure the same capabilites for them, there will be no problem. When a `toolchains.xml` file exists, it will be overwritten. But file will be written in a atomic operation, so even 
if there ara concurrent jobs reading the `toolchains.xml` file, they will not observe a half written file.
