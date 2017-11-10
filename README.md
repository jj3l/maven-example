# Maven Example

This is an example project to demonstrate [Maven](https://maven.apache.org/) usage and configuration.

## Project setup
* Create `README.md`.
* [Generate](https://www.gitignore.io/api/java,macos,maven,eclipse) `.gitignore`
* Make initial commit with `README.md` and `.gitignore`.

## Generate Maven project
```bash
mvn \
  archetype:generate \
  -DgroupId=com.github.jj3l.maven-example \
  -DartifactId=maven-example \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false
```

## Move to polyglot Maven with pom.yml

Replace `pom.xml` with less noisy `pom.yml`. This requires the [Polyglot Maven](
https://github.com/takari/polyglot-maven) [core extension](https://takari.io/2015/03/19/core-extensions.html) as 
available with Maven v3.3.1.

```bash
mvn \
  io.takari.polyglot:polyglot-translate-plugin:translate \
  -Dinput=pom.xml \
  -Doutput=pom.yml
rm pom.xml
mkdir -p .svn
cat << 'EOF' > .mvn/extensions.xml
<?xml version="1.0" encoding="UTF-8"?>
<extensions>
  <extension>
    <groupId>io.takari.polyglot</groupId>
    <artifactId>polyglot-yaml</artifactId>
    <version>0.2.1</version>
  </extension>
</extensions>
EOF
```

The last line from `pom.yml` (`pomFile: {}`) must be removed.

## Minimal project configurations

Maven is not able to run with [minimal POM](
https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Minimal_POM) without errors and 
warnings. The following adjustments to the [super POM](
https://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Super_POM) are necessary:
* Set Java version to 1.6 for the Java Compiler.
* Specify the Version of the project info report plugin.

```yml
build:
  plugins:
  - groupId: org.apache.maven.plugins
    artifactId: maven-project-info-reports-plugin
    version: 2.8.1
  - groupId: org.apache.maven.plugins
    artifactId: maven-compiler-plugin
    version: 3.1
    configuration:
      source: 1.6
      target: 1.6
```


