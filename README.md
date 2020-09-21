# Reproducible Builds
The following includes experimental code for testing reproducible builds with Maven and Gradle.

# Maven Reproducible Builds
For reproducible builds maven-jar-plugin, maven-source-plugin and maven-assembly-plugin to version 3.2.0 minimum and [additional pom.xml configuration](https://maven.apache.org/guides/mini/guide-reproducible-builds.html) is required.

pom.xml
```
<properties>
 <project.build.outputTimestamp>2019-10-02T08:04:00Z</project.build.outputTimestamp>
</properties>
```

Additionally, a [maven reproducible build plugin](http://zlika.github.io/reproducible-build-maven-plugin/) is available to be added to existing projects to repackage libraries for reproducible builds.

## Example Maven Reproducible Building Using maven-jar-plugin, maven-source-plugin and maven-assembly-plugin 3.2.0+ 

```
mvn -Drat.ignoreErrors=true clean package -f pom-repBuildConfig.xml
```

## Example Maven Reproducible Building Using [maven reproducible build plugin](http://zlika.github.io/reproducible-build-maven-plugin/)  

```
mvn -Drat.ignoreErrors=true clean package -f pom-repBuildPlugin.xml
```

This example shows a configuration for the use of the maven-assembly-plugin which requires executing twice. When maven-assembly-plugin is not used and a single archive is produced, the configuration executes once.

Non Maven-Assembly-Plugin Configuration
```
...
      <plugin>
        <groupId>io.github.zlika</groupId>
        <artifactId>reproducible-build-maven-plugin</artifactId>
        <version>0.12</version>
        <executions>
          <execution>
            <goals>
              <goal>strip-jar</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
...

```

Maven-Assembly-Plugin Configuration
``
...
      <plugin>
        <groupId>io.github.zlika</groupId>
        <artifactId>reproducible-build-maven-plugin</artifactId>
        <version>0.12</version>
        <executions>
          <execution>
            <id>strip-jar</id>
            <phase>package</phase>
            <goals>
              <goal>strip-jar</goal>
            </goals>
          </execution>
          <execution>
            <id>strip-archive</id>
            <phase>pre-integration-test</phase>
            <goals>
              <goal>strip-jar</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
...
``

## Determining Project Dependencies
```
mvn -Drat.ignoreErrors=true dependency:tree
```

```
e.g.
[INFO] commons-logging:commons-logging:jar:1.2.1-SNAPSHOT
[INFO] +- junit:junit:jar:3.8.1:test
[INFO] +- log4j:log4j:jar:1.2.17:compile
[INFO] +- logkit:logkit:jar:1.0.1:compile
[INFO] +- avalon-framework:avalon-framework:jar:4.1.5:compile
[INFO] \- javax.servlet:servlet-api:jar:2.3:provided
```

## Ensuring Tier 3 Dependencies by Enforcing Approved Repositories
To ensure Tier 3 dependencies, the pom.xml will need to be modified to specify Tier 3 repositories.
```
e.g.
...
  <repositories>
    <repository>
      <id>tier3-dependencies</id>
      <name>Tier3 Dependencies</name>
      <url>http://tier3-repo.jfrog.com</url>
    </repository>
  </repositories>
...
```

## Determining Project Plugin Dependencies
```
mvn -Drat.ignoreErrors=true help:effective-pom -f pom-repBuildConfig.xml
```

## Ensuring Maven Integrity by Enforcing Approved Plugin Repositories
To ensure the integrity of Maven plugins, the pom.xml will need to be modified to specify Tier 3 Maven plugin respositories.
```
e.g.
...
  <pluginRepositories>
    <pluginRepository>
      <id>tier3-plugins</id>
      <name>Tier3 Plugins</name>
      <url>http://tier3-plugins.jfrog.com</url>
    </pluginRepository>
  </pluginRepositories>
  ...
```

## Calculating Checksum
The [Checksum Maven Plugin](http://checksum-maven-plugin.nicoulaj.net/) can be used to calculate checksums from the commandline or configured in the POM.

### Command-line
```
mvn net.nicoulaj.maven.plugins:checksum-maven-plugin:1.9:file -Dfile=the-file-to-process
```

### POM Configuration
```
e.g.
...
  <plugin>
      <groupId>net.nicoulaj.maven.plugins</groupId>
      <artifactId>checksum-maven-plugin</artifactId>
      <version>1.9</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>files</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <fileSets>
          <fileSet>
            <directory>${project.build.directory}</directory>
          </fileSet>
        </fileSets>
      </configuration>
    </plugin>
  ...
```

# Gradle Reproducible Builds
Gradle supports reproducible builds since version 3.4. [Changes](https://docs.gradle.org/current/userguide/working_with_files.html#sec:reproducible_archives) are required to the Gradle build file to ensure build order and address timestamps.

build.gradle
```
tasks.withType(AbstractArchiveTask) {
    preserveFileTimestamps = false
    reproducibleFileOrder = true
}
```

build.gradle.kts
```
tasks.withType<AbstractArchiveTask>().configureEach {
    isPreserveFileTimestamps = false
    isReproducibleFileOrder = true
}
```
## Example Gradle Reproducible Building
```
./gradlew build -c settings-repBuild.gradle
```

## Determining Project Dependencies
```
./gradlew lib:dependencies -c settings-repBuild.gradle
```

## Ensuring Tier 3 Dependencies by Enforcing Approved Repositories
To ensure Tier 3 dependencies, the Gradle build file will need to be modified to specify Tier 3 repositories.

```
  ...
    repositories {
        maven {
            url "https://tier3-gradle.jfrog.com/"
        }
    }
  ...
```

## Determining Project Plugin Dependencies
Plugin dependencies can be listed with the following added to the Gradle build file.
```
task showPlugins {
    project.plugins.each {
        println it.getClass().name
    }
}
```

## Ensuring Gradle Integrity by Enforcing Approved Plugin Repositories
To ensure the integrity of Gradle plugins, the Gradle build file will need to be modified to specify Tier 3 plugin repositories.

```
  ...
  repositories {
    maven {
      url "http://tier3-plugins.jfrog.com"
    }
  }
  dependencies {
    classpath "tier3.myplugin:gradle-plugin:5.5.6"
  }
  ...
```


## Calculating Checksum
Checksum files can be generated with the following added to the Gradle build file. See [build-repBuild.gradle](./test-projects/logger-interceptor-multi-project-gradle-groovy-kotlin/build-repBuild.gradle)

```
jar.doLast { task ->
    ant.checksum file: task.archivePath
}
```

# [Test Projects](./test-projects)
The following is a list of test projects along with an explanation of the reproducible build implementation and any issues encountered.
- [commons-logging-mvn](./test-projects/commons-logging-mvn) - Original project requires Maven2. Updated pom.xml to include _classifier_ for jars in order to build for Maven3.
    - command: mvn -Drat.ignoreErrors=true package
- [logger-interceptor-multi-project-gradle-groovy-kotlin](./test-projects/logger-interceptor-multi-project-gradle-groovy-kotlin) - Android logging library. Multi-project gradle build in Groovy. Kotlin code. Requires ANDROID_SDK_ROOT with API 28,29 or environment variable or by setting the sdk.dir path in your project's local properties.
    - command ./gradlew clean build
         

