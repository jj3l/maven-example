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
