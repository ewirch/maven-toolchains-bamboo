# Maven Toolchains Bamboo Plugin
Software projects evolve. We update dependencies to never versions. But we still have to maintain older release brances wich still use older dependencies. Maven makes the dependency manegement for each build easy. You reference an artifact and it's exact version. Maven cares for fetching the correct binary and linking to it.

But there are some dependencies which are not so easily maintained. One of those is the Java Runtime used to compile the project. Maven uses the same Java runtime for compilation under which it runs. So if your project has to support Java 7 runtime environments you not only have to set the `maven-compiler-plugin` configuration option to use 1.7 target runtime, you also have to start Maven using Java 7. Only this way you can make sure your code does not use any Java runtime classes only available in later releases.

Getting back to your project. When you need to build your project using Java 7, you configure your bulid plan to use a Java 7 runtime. Easy. But once you make the next step and use Java 8 in the next release you get problems. You still have to maintain (and bulid) the older realease using Java 7. So one branch of your project has to be build using Java 8 and the other branch has to be build using Java 7. How do you do this in Bamboo? The easy but cumbersome solution is to duplicate your build plan. Let the one build plan checkout the older branch and use Java 7, and the other build plan check out the newer branch and use Java 8. This is not [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)! If there are changes to apply to the bulid plan, you have to do it twice.

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
