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

## Create sources JAR

Add to `pom.yml`:

```yml
build:
  plugins:
  - groupId: org.apache.maven.plugins
    artifactId: maven-source-plugin
    executions:
    - id: attach-sources
      goals:
      - jar
```

## Create Java-Doc JAR

Add to `pom.yml`:

```yml
build:
  plugins:
  - groupId: org.apache.maven.plugins
    artifactId: maven-javadoc-plugin
    executions:
    - id: attach-javadocs
      goals:
      - jar
```

## Deploy project website to GitHub repository

You need an OAuth2 access token to allow Maven to publish the project website to GitHub on a behalf of you. Each
commit will be done in [your name](https://github.com/jj3l/maven-example/commits/gh-pages). Consider using a 
dedicated "bot" user for this.

1. Login at https://github.com/ with your user.
2. [Go to "Settings" -> "Developer settings" -> "Personal access tokens"](https://github.com/settings/tokens)
3. Click on "Generate new token".
4. Choose a token description like "Maven bot to publish project website."
5. Select at least `public_repo` and `user:emailuser:email`  scope. This limits the access to the minimum needed 
   to publish the website at GitHub ([Documentation](
   https://developer.github.com/apps/building-integrations/setting-up-and-registering-oauth-apps/about-scopes-for-oauth-apps/)).

Write the token into your [`settings.xml`](https://maven.apache.org/settings.html):

```bash
cat << 'EOF' > ~/.m2/settings.xml
<settings
  xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd"
>
  <servers>
    <server>
      <id>github</id>
      <password>GENERATED_TOKEN</password>
    </server>
  </servers>
</settings>
EOF
```

There are two things important here:
1. The credentials will be referenced by the ID `github`.
2. [Through the absence of an `username` the `password` is interpreted as OAuth2 token](
   https://github.com/github/maven-plugins/blob/8d6d49393d1453be709ecc4cfe87f41f2081f5c5/github-core/src/main/java/com/github/maven/plugins/core/GitHubProjectMojo.java#L349). 

See https://developer.github.com/apps/building-integrations/setting-up-and-registering-oauth-apps/registering-oauth-apps/ 
for details to register an oauth2 app.

The `maven-site-plugin` is not able to to push the generated site into a git repository. But GitHub provides a 
plugin for this. Skip site deployment by the `maven-site-plugin` and configure the GitHub plugin to push the site 
to GitHub in `pom.yml`:

```yml
  - groupId: org.apache.maven.plugins
    artifactId: maven-site-plugin
    version: 3.4
    configuration:
      skipDeploy: true
  - groupId: com.github.github
    artifactId: site-maven-plugin
    version: 0.12
    executions:
    - goals:
      - site
      phase: site-deploy
      configuration:
        server: github
        message: Create project website for ${project.version}.
```

The `maven-site-plugin` is available at Maven Central and [refers](
https://github.com/github/maven-plugins/blob/master/pom.xml#L219) 
https://repo.eclipse.org/content/repositories/egit-releases/ as external repository [which is discouraged](
http://central.sonatype.org/pages/requirements.html) and needs a workaround on CircleCI. As a workaround the 
external repository must be referenced by the project using the `maven-site-plugin`. So we have to add 
additionally in `pom.yml`:

```yml
# Required by com.github.github:github-maven-plugins-parent
# https://github.com/github/maven-plugins/blob/master/pom.xml
repositories:
- id: egit
  name: Eclipse egit
  url: https://repo.eclipse.org/content/repositories/egit-releases/
```

Unfortunately GitHub plugin does not use the `/project/distributionManagement/site` information like the 
`maven-site-plugin`. Instead the project URL, the SCM metadata or an explicit configuration is used (first wins)

For the project URL this pattern should be used: `https://github.com/REPOSITORY_OWNER/REPOSITORY_NAME`
But better would be to use the SCM metadata and point the project URL to 
`https://REPOSITORY_OWNER.github.io/REPOSITORY_NAME`. This is the location there the generated project website 
will be found.

Weblinks:
* https://help.github.com/articles/user-organization-and-project-pages/
* https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/

## Create Site Descriptor

Run `mvn site:effective-site` to view the default site descriptor:

```xml
<project
  xsi:schemaLocation="http://maven.apache.org/DECORATION/1.7.0 http://maven.apache.org/xsd/decoration-1.7.0.xsd"
  name="maven-example"
  xmlns="http://maven.apache.org/DECORATION/1.7.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 >
  <bannerLeft>
    <name>maven-example</name>
  </bannerLeft>
  <publishDate />
  <version />
  <body>
    <links>
      <item name="maven-example" href="./" />
    </links>
    <menu ref="parent" />
    <menu ref="reports" />
  </body>
</project>
```

Add a [contemporary skin](https://maven.apache.org/skins/maven-fluido-skin/) built on [Bootstrap](
https://getbootstrap.com/):

```xml
  <skin>
    <groupId>org.apache.maven.skins</groupId>
    <artifactId>maven-fluido-skin</artifactId>
    <version>1.5</version>
  </skin>
```

There is [another  skin](https://github.com/andriusvelykis/reflow-maven-skin) built on [Bootstrap](
https://getbootstrap.com/). But even this one was last edited three years ago and has [some issues with TLS
resources](https://github.com/andriusvelykis/reflow-maven-skin/issues/50). So best for now will be to choose
`maven-fluido-skin`.

See [Site Descriptor](https://maven.apache.org/doxia/doxia-sitetools/doxia-decoration-model/decoration.html) 
documentation for further details.

Since v3.3 of the `maven-site-plugin` Markdown syntax is supported without declaring a [Doxia](
https://maven.apache.org/doxia/references/) dependency explicitly. Add a Markdown document to `src/site/markdown`.
With an additional suffix `.vm` filtering is supported (resolve e.g. ${project.name}).
