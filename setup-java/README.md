# zing-actions/setup-java

<p align="left">
  <a href="https://github.com/actions/setup-java"><img alt="GitHub Actions status" src="https://github.com/actions/setup-java/workflows/Main%20workflow/badge.svg"></a>
</p>

This action sets up a Zing java environment for use in actions by:

- Downloading and caching a requested version of java by version and adding to PATH. Downloads are populated from the [Zing distribution CDN](https://cdn.azul.com/zing/releases/tgz/)
- registering problem matchers for error output

# Usage

See [action.yml](action.yml)

## Basic
```yaml
steps:
- uses: actions/checkout@v2
- uses: actions/setup-java@v1
  with:
    java-version: '8.0.252' # The JDK version to make available on the path.
    java-package: jdk # (jre, jdk, or jdk+fx) - defaults to jdk
- run: java -cp java HelloWorldApp
```
Examples of version specifications that the java-version parameter will accept:

- A major Java version

  e.g. ```7, 8, 11, ...```
 
- A semver Java version specification

  e.g. ```8.0.252, 7.0.262, 11.0.7```
  
  e.g. ```8.0.x, >7.0.212, >=11.0.1, <8.0.262```
  
- 1.x syntax

    e.g. ```1.8``` (same as ```8```)
    
    e.g. ```1.8.0.252``` (same as ```8.0.252```)

## Matrix Testing
```yaml
jobs:
  build:
    runs-on: ubuntu-18.04
    strategy:
      matrix:
        # test against latest update of each major Java version, as well as specific updates of LTS versions:
        java: [ 7, 7.0.262, 8, 8.0.252, 11.0.x, 11.0.7 ]
    name: Java ${{ matrix.java }} sample
    steps:
      - uses: actions/checkout@v2
      - name: Setup java
        uses: actions/setup-java@v1
        with:
          java-version: ${{ matrix.java }}
      - run: java -cp java HelloWorldApp
```

## Publishing using Apache Maven
```yaml
jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8

    - name: Build with Maven
      run: mvn -B package --file pom.xml

    - name: Publish to GitHub Packages Apache Maven
      run: mvn deploy
      env:
        GITHUB_TOKEN: ${{ github.token }} # GITHUB_TOKEN is the default env for the password

    - name: Set up Apache Maven Central
      uses: actions/setup-java@v1
      with: # running setup-java again overwrites the settings.xml
        java-version: 1.8
        server-id: maven # Value of the distributionManagement/repository/id field of the pom.xml
        server-username: MAVEN_USERNAME # env variable for username in deploy
        server-password: MAVEN_CENTRAL_TOKEN # env variable for token in deploy

    - name: Publish to Apache Maven Central
      run: mvn deploy 
      env:
        MAVEN_USERNAME: maven_username123
        MAVEN_CENTRAL_TOKEN: ${{ secrets.MAVEN_CENTRAL_TOKEN }}
```

The two `settings.xml` files created from the above example look like the following.

`settings.xml` file created for the first deploy to GitHub Packages
```xml
<servers>
    <server>
      <id>github</id>
      <username>${env.GITHUB_ACTOR}</username>
      <password>${env.GITHUB_TOKEN}</password>
    </server>
</servers>
```

`settings.xml` file created for the second deploy to Apache Maven Central
```xml
<servers>
    <server>
      <id>maven</id>
      <username>${env.MAVEN_USERNAME}</username>
      <password>${env.MAVEN_CENTRAL_TOKEN}</password>
    </server>
</servers>
```

***NOTE: The `settings.xml` file is created in the Actions $HOME directory. If you have an existing `settings.xml` file at that location, it will be overwritten. See below for using the `settings-path` to change your `settings.xml` file location.***	

See the help docs on [Publishing a Package](https://help.github.com/en/github/managing-packages-with-github-packages/configuring-apache-maven-for-use-with-github-packages#publishing-a-package) for more information on the `pom.xml` file.

## Publishing using Gradle
```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Set up JDK 1.8
      uses: actions/setup-java@v1

    - name: Build with Gradle
      run: gradle build

    - name: Publish to GitHub Packages
      run: gradle publish
      env:
        USERNAME: ${{ github.actor }}
        PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

***NOTE: The `USERNAME` and `PASSWORD` need to correspond to the credentials environment variables used in the publishing section of your `build.gradle`.***	

See the help docs on [Publishing a Package with Gradle](https://help.github.com/en/github/managing-packages-with-github-packages/configuring-gradle-for-use-with-github-packages#example-using-gradle-groovy-for-a-single-package-in-a-repository) for more information on the `build.gradle` configuration file.

## Apache Maven with a settings path

When using an Actions self-hosted runner with multiple shared runners the default `$HOME` directory can be shared by a number runners at the same time which could overwrite existing settings file. Setting the `settings-path` variable allows you to choose a unique location for your settings file.

```yaml
jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 1.8 for Shared Runner
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
        server-id: github # Value of the distributionManagement/repository/id field of the pom.xml
        settings-path: ${{ github.workspace }} # location for the settings.xml file

    - name: Build with Maven
      run: mvn -B package --file pom.xml

    - name: Publish to GitHub Packages Apache Maven
      run: mvn deploy -s $GITHUB_WORKSPACE/settings.xml
      env:
        GITHUB_TOKEN: ${{ github.token }}
```

# License

The scripts and documentation in this project are released under the [MIT License](LICENSE)

# Contributions

Contributions are welcome!  See [Contributor's Guide](docs/contributors.md)
