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

## OWASP Dependency check

The [OWASP Dependency check](https://www.owasp.org/index.php/OWASP_Dependency_Check) checks all dependencies for 
known security risks.

Add to `pom.yml`:

```yml
build:
  plugins:
  - groupId: org.owasp
    artifactId: dependency-check-maven
    version: 3.0.2
    executions:
    - goals:
      - check
```

See https://jeremylong.github.io/DependencyCheck/dependency-check-maven/ and [Automatisierte Überprüfung von
Sicherheitslücken in Abhängigkeiten von Java-Projekten](
https://www.triology.de/blog/automatisierte-ueberpruefung-von-sicherheitsluecken-in-abhaengigkeiten-von-java-projekten)
(german).

## Enforce project model

Certain aspects of the project model [can be enforced](https://maven.apache.org/enforcer/maven-enforcer-plugin/).
There are even more aspect which could be enforced but are not supported by the plugin.

* Normalization of the POM (to reduce merge conflicts, make POM better readable). Partly solved by the 
  [tidy-maven-plugin](http://www.mojohaus.org/tidy-maven-plugin/).
* Deprecated/vulnerable dependencies/plugins could be bannend.
* Version is compliant to [semantic versioning](http://semver.org/).
* There is a changelog compliant to [keep a changelog](http://keepachangelog.com/en/1.0.0/).

Add to `pom.yml`:

```yml
build:
  plugins:
  - groupId: org.apache.maven.plugins
    artifactId: maven-enforcer-plugin
    version: 3.0.0-M1
    executions:
    - id: no-duplicate-declared-dependencies
      goals:
      - enforce
      configuration:
        rules:
          banDuplicatePomDependencyVersions: ''
    - id: enforce
      goals:
      - enforce
      configuration:
        rules:
          dependencyConvergence: ''
```

##  Add license informations

[Add license informations](http://www.mojohaus.org/license-maven-plugin/) to each file and add a license file.

Add to `pom.yml`:

```yml
build:
  plugins:
  - groupId: org.codehaus.mojo
    artifactId: license-maven-plugin
    version: 1.14
    configuration:
      licenseName: apache_v2
      addJavaLicenseAfterPackage: false
      canUpdateCopyright: true
      canUpdateDescription: true
      canUpdateLicense: true
      emptyLineAfterHeader: true
      extraFiles:
        DockerFile: properties
      processStartTag: 'LICENSE_START'
      processEndTag: 'LICENSE_END'
      sectionDelimiter: '**'
    executions:
    - id: first
      goals:
      - update-file-header
      phase: process-sources
    - id: second
      goals:
      - update-project-license      
```

It would be appreciated to remove `processStartTag`, `processEndTag` and `sectionDelimitersectionDelimiter` (the 
first section is not required). Trailing spaces should be removed. A `LICENSE.md` would be preferred over 
`LICENSE.txt`. The placeholder `Copyright [yyyy] [name of copyright owner]` will not be replaced.

## Release process

One of the most powerful features of Maven is the repository concept to store packaged Java artefacts like JAR
files. The default repository where dependencies are looked up is known as [Maven Central Repository](
https://repo.maven.apache.org/maven2). By default the Maven Central Repository is available at
https://repo.maven.apache.org/maven2. There are multiple mirrors like https://repo1.maven.org/maven2 or (local)
company repositories.

### Publish to Maven Central Repository

To publish artefacts to Maven Central some [requirements](http://central.sonatype.org/pages/requirements.html)
need to be met.

1. Choose a `groupId` as your global unique namespace. Similar to the [Java package naming convention](
   http://docs.oracle.com/javase/tutorial/java/package/namingpkgs.html) it reuses the DNS in a reversed manner.
   This means you need a domain name you control. This could be any registered second level Domain. But a third
   level domain from your GitHub account like `jj3l.github.com` will be [sufficient](
   https://issues.sonatype.org/browse/OSSRH-36150). Having a `groupId` you have to use the `groupId` in your POM
   and to [signup](https://issues.sonatype.org/secure/Signup!default.jspa) for Sonatype Jira and [register](
   https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134) the `groupId` to be used publishing
   artefacts in the Maven Central Repository with.
 1. Make your Sonatype Jira credentials available to Maven as they are used for OSSRH as well. Add to
    `~/.m2/settings.xml`:
    ```xml
    <settings>
      <servers>
        <server>
          <id>ossrh</id>
          <username>jjaekel</username>
          <password>1password</password>
        </server>
      </servers>
    </settings>
    ```
 2. The Sonatype Jira credentials will be referenced by the id `ossrh`. One time from the `maven-deploy-plugin`
    via the `distributionManagement` releasing snapshot releases and one time from the 
    `nexus-staging-maven-plugin` configuration releasing production releases (see [Snapshot Artefact vs. Release 
    Artefacts](#snapshot-artefact-vs-release-artefacts) for details). 
    Add to `pom.yml`:
    ```yml
    distributionManagement:
      snapshotRepository:
        id: ossrh
        url: https://oss.sonatype.org/content/repositories/snapshots
    build:
      plugins:
      - groupId: org.sonatype.plugins
        artifactId: nexus-staging-maven-plugin
        version: 1.6.7
        extensions: true
        configuration:
          serverId: ossrh
          nexusUrl: https://oss.sonatype.org/
          autoReleaseAfterClose: true
    ```
 3. Add to `pom.yml`:
    ```yml
    build:
      plugins:
      - groupId: org.apache.maven.plugins
        artifactId: maven-release-plugin
        version: 2.5.3
        configuration:
          autoVersionSubmodules: true
          useReleaseProfile: false
          releaseProfiles: release
          goals: deploy
    ```
2. Generate a GPG key pair and publish the pubic key so everybody can verify the artefacts you signed with your
   private key.
 1. Install [GPG Suite](https://gpgtools.org/) with MacGPG. The Maven GPG plugin does not have a Java GPG
    implementation and needs the `gpg` executeable on the `PATH`.
 2. Generate GPG key pair:
    ```bash
    gpg --batch --gen-key << EOF
    Key-Type: RSA
    Key-Length: 4096
    Subkey-Type: RSA
    Subkey-Length: 4096
    Name-Real: Jonathan Jäkel
    Name-Email: j@j3l.de
    Expire-Date: 0
    Passphrase: <1password>
    EOF
    ```
  3. Get key ID:
     ```bash
     gpg --list-keys
     pub   rsa4096 2017-12-03 [SCEA]
           183303E02CCE0907450FE9370046FC88CB105E68 <-- Key ID
     ```
  4. Publish public key
     ```bash
     gpg --keyserver hkp://pgp.mit.edu --send-keys 183303E02CCE0907450FE9370046FC88CB105E68
     ```
  5. Configure the Maven GPG Plugin in your `pom.yml`:
     ```yml
     build:
       plugins:
       - groupId: org.apache.maven.plugins
         artifactId: maven-gpg-plugin
         executions:
         - id: sign-artifacts
           phase: verify
           goals:
           - sign
           configuration:
             keyname: ${gpg.keyname}
             passphraseServerId: ${gpg.keyname}
     ```
  6. To sign the GPG key passphrase has to specified as command line argument
     `mvn verify -Dgpg.passphrase=thephrase` or it will be prompted for. Even better would be to define the
     passphrase  in the `settings.xml`. Add to `~/.m2/settings.xml`:
     ```xml
     <settings>
       <servers>
         <server>
           <id>183303E02CCE0907450FE9370046FC88CB105E68</id>
           <passphrase>1password</passphrase>
         </server>
       </servers>
       <profiles>
         <profile>
           <activation>
             <activeByDefault>true</activeByDefault>
           </activation>
           <properties>
             <gpg.keyname>183303E02CCE0907450FE9370046FC88CB105E68</gpg.keyname>
           </properties>
         </profile>
       </profiles>
     </settings>
     ```
     You can sign explicitly by running `JAVA_HOME=$(/usr/libexec/java_home -v 1.8) mvn clean verify` but you will
     sign regularly implicit using `JAVA_HOME=$(/usr/libexec/java_home -v 1.8) mvn release:perform` later.
     You will find the generated signatures in `target/maven-example-*.*.asc`.
3. Using [semantic versioning](http://semver.org/) is not required but [recommended](
   http://central.sonatype.org/pages/choosing-your-coordinates.html).

### Snapshot Artefact vs. Release Artefacts

Maven distinguish between snapshot and production releases. Production releases use common versioning schemes (
[semantic versioning](http://semver.org/) in best case). Snapshot releases use the same versioning scheme but add
a `-SNAPSHOT` suffix. A snapshot release prepares the production release with the same prefix. Whereas a 
production release denotes a immutable version with certain assertion like QA process or semantic versioning the 
snapshot release can be assigned to multiple versions as floating tag.

### Publish Snapshot Artefacts

Artifact build on a snapshot version can be deployed to the [Sonatype snapshot repository](
https://oss.sonatype.org/content/repositories/snapshots/) but will not be synchronized to Maven Central. The 
version in the `pom.yml` will not be changed and no tag at GitHub will be set. For the artifacts stored in the 
repository the `-SNAPSHOT` suffix will be replaced with a timestamp. Maven references the latest timestamp for a 
given `SNAPSHOT` automatically.

For snapshot releases the default Maven process is used. The snapshot repository is configured under 
`distributionManagement` which will be looked up by the `maven-deploy-plugin`.
    
To deploy a snapshot version run:
```bash
JAVA_HOME=$(/usr/libexec/java_home -v 1.8) mvn clean deploy
```

### Publish Release Artefacts

To issue a production release some more steps are required as it must be switched from current snapshot version to 
a production release version (remove the `-SNAPSHOT` suffix) and the next snapshots version must be bumped. 
Additionally there may be rules like reviews and QA for the release process as for Maven Central. The QA is based 
on the final release artifacts and can not be done before having the release artifacts available. But if the QA 
fails the release artifacts should not be published.

For this reason it's not possible to publish into the Maven Central repository directly. Rather you have to 
publish your production release artifacts into the Sonatype Open Source Software Repository Hosting (OSSRH). The
OSSRH make use of [staging releases](https://help.sonatype.com/repomanager2/staging-releases) as a Nexus 
Repository Pro feature. A staging release is a temporary staging repository with the artifacts of a release 
candidate. The release candidate artifacts can be reviewed and discarded or promoted to the final release 
repository. Only the final release repository will be synchronized to the Maven Central Repository.

This makes it difficult to deploy production release artefacts to Maven Central using the `maven-deploy-plugin`.
Nexus introduces the `nexus-staging-maven-plugin` for this case([reference](
https://help.sonatype.com/repomanager2/staging-releases/configuring-your-project-for-deployment#ConfiguringYourProjectforDeployment-DeploymentwiththeNexusStagingMavenPlugin
)). Unfortunately the `nexus-staging-maven-plugin` does not recognize the `distributionManagement`. So the URL of 
the OSSRH repository must be configured with the plugin configuration (as shown above).

To deploy a production version run:
```bash
git diff-index --quiet HEAD -- || echo 'There are local modifications!'
sed -i '' -E 's/^(version: ([0-9]{1,}\.){1,}[0-9]{1,})(.*)/\1/' pom.yml
grep '\-SNAPSHOT' pom.yml && echo 'There are SNAPSHOT dependencies!'
JAVA_HOME=$(/usr/libexec/java_home -v 1.8) mvn test
JAVA_HOME=$(/usr/libexec/java_home -v 1.8) mvn site-deploy
git add pom.yml
git commit -m 'Create production release'
git push
git tag "v$(grep '^version:' pom.yml|sed -E 's/^version: (.*)/\1/')"
git push origin "v$(grep '^version:' pom.yml|sed -E 's/^version: (.*)/\1/')"
JAVA_HOME=$(/usr/libexec/java_home -v 1.8) mvn deploy
# Change version number to next relase manually.
sed -i '' -E 's/^version: .*/&-SNAPSHOT/' pom.yml
git add pom.yml
git commit -m 'Bump to next snapshot release'
git push
```

### The disqualification of the maven-release-plugin

The `maven-release-plugin` does not work together with polyglot Maven([reference](
https://github.com/takari/polyglot-maven#limited-plugin-support)).

There is the criticism the `maven-release-plugin` tries to make things simpler than they are([reference](
https://dzone.com/articles/why-i-never-use-maven-release)). A release process has a intrinsic complexity which can 
not be eliminated by a tool.

Releases have to be carefully planned. They have to be announced and there may be strategies required to handle 
incompatible changes. And there may be tradeoffs to be considered (e.g tollerate downtimes vs. migration costs).
Last but not least multiple conventions apply (like semantic versioning) and a QA process is required.

So what you want to do would be probably something like this:

1. Checkout a clean copy of the current `master` branch.
2. Ensure there are no local modifications.
3. Remove the `-SNAPSHOT` suffix from the version in the POM. Commit and push the changes.
4. Ensure there are no other snapshot dependencies as they are floating tags and you can't guarantee the released
   software will later behave as you tested.
5. Run tests.
6. Build and publish the project website.
7. Tag a GitHub release.
8. Upload the production release artifacts to the production release repository.
9. Bump to a new snapshot version. Commit and push the changes.

### Links

* https://maven.apache.org/repository/
* https://blog.sonatype.com/2010/10/new-official-maven-central-repository-in-europe/
 * http://central.sonatype.org/pages/ossrh-guide.html
 * http://central.sonatype.org/pages/choosing-your-coordinates.html
* http://central.sonatype.org/pages/apache-maven.html
* http://blog.sonatype.com/2010/01/how-to-generate-pgp-signatures-with-maven/.
* https://maven.apache.org/plugins/maven-gpg-plugin/usage.html
* https://maven.apache.org/plugins/maven-release-plugin/
* https://maven.apache.org/plugins/maven-jarsigner-plugin/

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

## CircleCI

Run tests at CircleCI. See [`.circleci/config.yml`](https://circleci.com/docs/2.0/configuration-reference/) for 
CircleCI configuration.
