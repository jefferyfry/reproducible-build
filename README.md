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

# Gradle Reproducible Builds
Gradle supports reproducible builds since version 3.4. [Changes](https://docs.gradle.org/3.4/userguide/working_with_files.html#sec:reproducible_archives) are required to the Gradle build file to ensure build order and address timestamps.

build.gradle
```
tasks.withType(AbstractArchiveTask) {
    preserveFileTimestamps = false
    reproducibleFileOrder = true
}
```

# [Test Projects](./test-projects)
The following is a list of test projects along with an explanation of the reproducible build implementation and any issues encountered.
- commons-logging 
    - Maven 
        - Original project requires Maven2. Updated pom.xml to include _classifier_ for jars in order to build for Maven3.
    - Gradle 
        - Converted from Maven project using _gradle init_.