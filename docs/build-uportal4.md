# Building [jasig/uPortal](https://github.com/jasig/uPortal)

This is a guide to cloning and successfully configuring Apereo's uPortal repository on Linux and macOS.

<b>Note:</b> This guide assumes you use <code>apt</code> as a package manager but also works with others and <code>brew</code> on macOS.

You can install <code>brew</code> on macOS with the following command:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## 1. Ensure your system is up to date
* Run <code>sudo apt-get update</code> and install any available package updates with <code>sudo apt-get upgrade</code>.

## 2. Prepare the uPortal directory
* Create a new directory in your home directory titled <code>uportal</code>.
This will be where you extract and configure the dependencies needed to run uPortal.

## 3. Install required dependencies
* Install Git: <code>sudo apt-get install git</code>
* Install Java: <code>sudo apt-get install openjdk-8-jdk</code>
* Install PostgreSQL: <code>sudo apt-get install postgresql</code>
* Download the latest tar.gz binary of [Apache Ant](https://ant.apache.org/bindownload.cgi) and extract it into <code>~/uportal/ant</code>.
* Download the latest tar.gz binary of [Apache Maven](https://maven.apache.org/download.cgi) and extract it into <code>~/uportal/maven</code>.
* Download the latest tar.gz binary of [Apache Tomcat 8](https://tomcat.apache.org/download-80.cgi) and extract it into <code>~/uportal/tomcat</code>.

* Clone the jasig/uPortal repository into a directory titled <code>uportal</code> nested within
the one you just created:

```bash
git clone https://github.com/jasig/uportal.git ~/uportal/uportal
```

The resulting directory structure should look like this (along with other files):

```bash
uportal
├── ant
│   └── bin
├── maven
│   └── bin
├── tomcat
│   └── bin
└── uportal
    └── [git subdirectories]
```

## 4. Set environment variables
* Open <code>~/.bashrc</code> (or <code>~/.zshrc</code> for zshrc users) in a text editor. Add the following code to the end of the file.

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin

export ANT_HOME=/home/$USER/uportal/ant
export PATH=$PATH:$ANT_HOME/bin

export M2_HOME=/home/$USER/uportal/maven
export M2=$M2_HOME/bin
export PATH=$PATH:$M2

export TOMCAT_HOME=/home/$USER/uportal/tomcat
export PATH=$PATH:$TOMCAT_HOME
```

<b>Make sure you source your <code>.bashrc</code> or <code>.zshrc</code> whenever you make any changes!</b>

.bashrc users:

```bash
. ~/.bashrc
```

.zshrc users: 

```bash
source ~/.zshrc
```

## 5. Helper Scripts

* Add the following helper script to your <code>~/.bashrc</code> (or <code>~/.zshrc</code>) for starting and stopping Tomcat. This lets you use commands such as <code>t start</code> and <code>t tail</code>.

```bash
# Tomcat Script
function t {
    for i in "$@"; do
        if [[ $i == "start" ]]; then
            $TOMCAT_HOME/bin/startup.sh
        elif [[ $i == "stop" ]]; then
            $TOMCAT_HOME/bin/shutdown.sh
            sleep 5
        elif [[ $i == "kill" ]]; then
            pkill -f "tomcat"
        elif [[ $i == "restart" ]]; then
            $TOMCAT_HOME/bin/shutdown.sh
            echo "Restarting..."
            sleep 10
            $TOMCAT_HOME/bin/startup.sh
        elif [[ $i == "clean" ]]; then
            rm -rf $TOMCAT_HOME/webapps/*
            rm -rf $TOMCAT_HOME/work/*
            rm -rf $TOMCAT_HOME/temp/*
        elif [[ $i == "cleanlogs" ]]; then
            rm -rf $TOMCAT_HOME/logs/*
        elif [[ $i == "s" || $i == "status" ]]; then
            ps aux | grep 'tomcat'
        elif [[ $i == "tail" ]]; then
            tail -f $TOMCAT_HOME/logs/catalina.out
        elif [[ $i == "help" || $i == "h" ]]; then
            echo "Tomcat commands:"
            echo "start: starts Tomcat"
            echo "stop: stops Tomcat"
            echo "kill: force stops Tomcat"
            echo "restart: restarts Tomcat"
            echo "clean: clears Tomcat configuration files"
            echo "cleanlogs: clears Tomcat logs"
            echo "status: displays Tomcat status"
            echo "help: displays this help text"
        else
            echo "Unrecognized command. Type \"t help\" for a list of commands."
        fi
    done
}
```

## 6. Configure uPortal build properties
Copy <code>uportal/uportal/build.properties.sample</code> to a new file called <code>build.properties</code> in the same directory.

In build.properties, change

```javascript
server.home=@server.home@
```
to

```javascript
server.home=/home/(your username)/uportal/tomcat
```

<b>Note:</b> Use the FULL path to your Tomcat directory here. Don't use the environment variable, $USER, or ~.

## 7. Configure Tomcat
* Navigate to <code>uportal/tomcat/conf</code>.

* Set the <code>shared.loader</code> value in <code>catalina.properties</code> to:

```bash
${catalina.base}/shared/lib/*.jar
```

* Find the following code in <code>server.xml</code>:

```xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"/>
```

* Change it to read the following:

```xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               compression="on" compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript"/>
```

* Find the <code>Engine</code> and <code>Host</code> tags in <code>server.xml</code>:

```xml
<Engine name="Catalina" defaultHost="localhost">
```

```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
```

* Add <code>startStopThreads="2"</code> to each tag as shown below. (Computers with more powerful processors can use a higher number such as 4.)

```xml
<Engine name="Catalina" defaultHost="localhost" startStopThreads="2">
```

```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true" startStopThreads="2">
```
* Edit the <code>Context</code> tag in <code>context.xml</code> and add a <code>Resources</code> tag below it:

```xml
<Context sessionCookiePath="/">
    <Resources cachingAllowed="true" cacheMaxSize="100000" />
```
* Replace the <code>\<tomcat-users\></code> tag in <code>tomcat-users.xml</code> with the following. (You can choose your own username and password if you'd like.)

```xml
<tomcat-users>

  <role rolename="probeuser" />
  <role rolename="poweruser" />
  <role rolename="poweruserplus" />
  <role rolename="manager" />
  <role rolename="manager-gui" />

  <user username="admin" password="t0psecret" roles="manager, manager-gui" />

</tomcat-users>
```

## 8. Configure PostgreSQL dependencies
* Open <code>uportal/uportal/pom.xml</code>.
* Use [search.maven.org](https://search.maven.org) to find the latest version of <code>org.postgresql.postgresql</code> hosted on The Central Repository.
* Add this version to the list of dependencies:

```xml
<!-- Project Dependency Version Properties -->
<postgres.version>number</postgres.version> 
```

* Open <code>uportal/uportal/uportal-db/pom.xml</code>.
* Uncomment the postgresql dependency by changing this code:

```xml
<!--
<dependency>
<groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>${postgres.version}</version>
    <scope>compile</scope>
</dependency>
<dependency>
```

to this:

```xml
<dependency>
<groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>${postgres.version}</version>
    <scope>compile</scope>
</dependency>
<!--
<dependency>
```

## 9. Configure PostgreSQL Database

* Open <code>uportal/uportal/filters/local.properties</code>.
* Find <code>## Database Connection Settings</code> and replace the code beneath it with the following:

```properties
environment.build.hibernate.connection.driver_class=org.postgresql.Driver
environment.build.hibernate.connection.url=jdbc:postgresql://localhost:5432/uPortal
environment.build.hibernate.connection.username=uportal
environment.build.hibernate.connection.password=uportal
environment.build.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
environment.build.hibernate.connection.validationQuery=select version();
```

* Run these commands to create the 'uportal' user and 'uPortal' database:

```bash
sudo -u postgres -i
createuser uportal -P
```

* When prompted, enter <code>uportal</code> as the new database user's password.

* Create the uPortal database and set uportal as its owner:

```bash
createdb uPortal -O uportal
```

<b>Note:</b> The casing here is <b>extremely</b> important. The user is 'uportal' while the database is 'uPortal'.

* Press <code>Ctrl + D</code> (or type <code>logout</code>) to exit postgres.

## 10. Build uPortal

* Run the following command from <code>uportal/uportal</code> to test your database setup:

```bash
ant dbtest
```

* If the database test succeeds, run the following command to build uPortal:

```bash
ant clean initportal
```

* Finally, start Tomcat. uPortal should appear in your browser at [localhost:8080/uPortal](http://localhost:8080/uPortal).

```bash
t start
```

# Troubleshooting

## Problem: Tomcat directory not found

```
checkForTomcat:

BUILD FAILED
[...]build.xml:164: The following error occurred while executing this line:
[...]build.xml:1161: server.base build property must be set.
```

### Solution
Check the following:

* Make sure you set the <code>server.home</code> property in <code>build.properties</code> correctly.
* Make sure you set <code>TOMCAT_HOME</code> to the correct path in your <code>.bashrc</code> or <code>.zshrc</code> and sourced it.
* Make sure Tomcat is extracted to the correct directory (<code>uportal/tomcat</code>) and properly configured.

## Problem: Postgres database not found

```
[java] org.postgresql.util.PSQLException: FATAL: database "uPortal" does not exist
```

### Solution
* Make sure you initialized the postgres database and user correctly in step 9. <b>Double check your casing.</b>
* The commands <code>dropdb name</code> and <code>dropuser name</code> drop a database or a user titled

## Problem: PostgreSQL dependency issue

```
JDBC Driver class not found: org.postgresql.Driver
```

### Solution
Check the following:
* Make sure you correctly modified the <code>pom.xml</code> dependencies as described in step 8.
* Make sure the version number is valid and correct: use [search.maven.org](https://search.maven.org) to find the latest version of postgreSQL hosted on The Central Repository.

## Problem: Maven dependency issues

```
[exec] [ERROR] Failed to execute goal on project uportal-war: Could not resolve dependencies for project org.jasig.portal:uportal-war:war:5.0.0-SNAPSHOT: The following artifacts could not be resolved: [...]
```

### Solution
* Delete your local gradle distribution. It will be rebuilt by uPortal when you run an ant build command.

```bash
rm -rf ~/.gradle
```