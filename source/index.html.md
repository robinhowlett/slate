---
title: Snap Development Documentation

language_tabs:

toc_footers:
  - <a href='http://doc.snaplogic.com/home'>Product Documentation</a>
  - <a href='http://www.snaplogic.com'>SnapLogic website</a>

includes:

search: true
---

# Introduction

This documentation guides a developer through the steps necessary to develop **Snaps** for the [SnapLogic Elastic Integration Platform](https://www.snaplogic.com/).

## Snaps and Snap Packs

SnapLogic [Snaps](https://www.snaplogic.com/snaps) are modular collections of integration components built for a specific application or data source. Snaps shield both business users and developers from much of the complexity of the underlying application, data model, and service.

![Transform Snap Pack](http://dl.dropboxusercontent.com/u/3519578/Screenshots/WmuJ.png)

Snap Packs logically organize Snaps and are the deployable unit when adding/modifying Snaps to the SnapLogic Elastic Integration Platform. For instance, in the above example, the Aggregate Snap is part of the Transform Snap Pack. 

Snaps may be related by functionality or share common code. A Snap Pack may contain a single Snap or multiple Snaps.</img>

## Snap Fundamentals

Snaps are streaming data processors. They can consume and/or produce Binary or Document data through input and output views, and can report error Documents to an optional error view. 

![Snap Anatomy](http://dl.dropboxusercontent.com/u/3519578/Screenshots/iKCt.png)

The have metadata that define their view settings and configuration. They can read/poll from an endpoint, batch execute or process each input individually, and may write to another endpoint.

Snaps can provide design-time Suggest/lookup and Preview/validate data capabilities, or run in full execution mode.

![Snap Execution](http://dl.dropboxusercontent.com/u/3519578/Screenshots/Yjfr.png)

## Snaps vs Scripts

SnapLogic's [Script Snap](http://doc.snaplogic.com/com-snaplogic-snaps-script-script_2) executes a Javascript, Jython, or JRuby script using the JVM `ScriptEngine` mechanism. 

Consider the following when deciding whether to develop a custom Snap vs using the Script Snap:

Custom Snap | Script Snap
--------- | -----------
Supports easy configuration without coding | Working directly with code
Supports pipeline `_parameters` | Does not support pipeline `_parameters`
Supports Binary or Document input/output | Supports only Document input/output
Supports Accounts, simplify impersonation |	Does not support Accounts
Supports debugging during development | Often awkward to debug
Supports unit testing during/after development | Cannot be unit tested
Java | JavaScript, Jython, or JRuby
Need to build/deploy/etc | No deployment, quick & simple

# Prerequisites

* Java 8 (JDK)
* Maven 3
* A Snaplex, including the `keys.properties` and `global.properties` files/data provided by [SnapLogic Support](mailto:support@snaplogic.zendesk.com?subject=Requesting%20Snap%20Development%20Credentials) upon request for Snap development

<aside class="notice">
We also recommend using an Java IDE like <a href="https://www.jetbrains.com/idea">IntelliJ IDEA</a>, <a href="https://eclipse.org/ide">Eclipse</a>, or <a href="https://netbeans.org/">NetBeans</a>.
</aside>

# Setting up the Snaplex

A [Snaplex](http://doc.snaplogic.com/snaplex) is the data processing engine of the [SnapLogic Elastic Integration Platform](https://www.snaplogic.com/why-snaplogic/how-it-works). A locally-installed/on-premises Snaplex (also known as a "Groundplex" or "JCC") will be used throughout this guide to develop, execute and debug a Snap.

## Downloads

A Snaplex may be installed on:

* Linux
* Microsoft Windows
* Mac OS X (for development only)

<aside class="notice">
The current list of Snaplex downloads is maintained at <a href="http://doc.snaplogic.com/snaplex-downloads">http://doc.snaplogic.com/snaplex-downloads</a>.
</aside>

## Installation

### Linux & Microsoft Windows 

Please visit the official SnapLogic documentation for detailed instructions for installing a Snaplex on [Linux](http://doc.snaplogic.com/snaplex-installation-on-linux) or [Microsoft Windows](http://doc.snaplogic.com/installing-snaplex-on-windows). 

### Mac OS X

<aside class="warning">
Use of a Snaplex in a Mac OS X environment is supported only for developing Snaps.
</aside>

```shell
$ cd ~
$ wget https://s3.amazonaws.com/elastic.snaplogic.com/snaplogic-sidekick...rpm
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ brew install rpm2cpio
$ rpm2cpio.pl snaplogic-sidekick...rpm | cpio -idvm
$ echo 'export SL_ROOT=/opt/snaplogic' >> ~/.bash_profile
$ source ~/.bash_profile
```

> If successful, the output of the `cpio` command will list the Snaplex files populating the newly created `~/opt/snaplogic` directory, for example:

```shell
$ rpm2cpio.pl snaplogic-sidekick-4.mrc244-x86_64.rpm | cpio -idvm
./opt/snaplogic/bin/functions
./opt/snaplogic/bin/jcc.sh
./opt/snaplogic/bin/logstash.sh
./opt/snaplogic/etc/global.properties
./opt/snaplogic/etc/keys.properties
./opt/snaplogic/etc/logstash.conf
./opt/snaplogic/ldlib/libsapjco3.jnilib
./opt/snaplogic/ldlib/libsapjco3.so
./opt/snaplogic/pkgs/jre1.8.0_45/COPYRIGHT
...
./opt/snaplogic/run/lib/jcc.war
./opt/snaplogic/run/lib/logstash.jar
./opt/snaplogic/run/log
1044673 blocks
```

1. `cd` to your home directory
1. Download the latest RHEL/CentOS/Amazon RPM file from the [Snaplex downloads page](http://doc.snaplogic.com/snaplex-downloads).
1. Download and install [Homebrew](http://brew.sh/)
1. Install the `rpm2cpio` library
1. Extract the downloaded Snaplex `.rpm` file
1. Export the `SL_ROOT` environment variable to your `~/.bash_profile`

## Configuration

```shell
$ cat ~/opt/snaplogic/etc/keys.properties
cc.admin_uri = https://elastic.snaplogic.com:443
cc.username = rhowlett@snaplogic.com
cc.api_key = JDAFBDAD....9dAg43
cc.ssl_verify = True
```

For approved customers, SnapLogic Support will provide the necessary instructions for authentication and configuration of the Snaplex.

```shell
$ cat ~/opt/snaplogic/etc/global.properties
jcc.location = sidekick
jcc.sldb_uri = https://elastic.snaplogic.com:443
...
# The subscriber ID assigned for the customer. This will be available after
# the customer account is created.
jcc.subscriber_id = snaplogic

# This is the runtime environment in which the Snaplex should be started.
jcc.environment = devsnaplex
```

The `~/opt/snaplogic/etc/keys.properties` file contains the information required for authenticating to the SnapLogic Elastic Integration Platform APIs to run the Snaplex and deploy the developed Snaps.

The `~/opt/snaplogic/etc/global.properties` file contains the information required for general configuration of the Snaplex.

## Running

```shell
$ cd ~/opt/snaplogic/run/lib
$ java -jar jcc.war jcc
```

> If successful, the output of the `java` command will be similar to this:

```shell
...
	+- file:/Users/snapdev/opt/snaplogic/run/lib/jcc/zookeeper-3.4.6.jar
	+- sun.misc.Launcher$AppClassLoader@42a57993
		+- file:/Users/snapdev/opt/snaplogic/run/lib/jcc.war
		+- sun.misc.Launcher$ExtClassLoader@14ae5a5
2016-07-01T00:59:04,505 I main	[  ] Server listening for incoming connections
```
The simplest way to start the Snaplex is to run the packaged `jcc.war` file.

To check if the Snaplex started correctly, login to the [SnapLogic Dashboard](https://elastic.snaplogic.com/sl/dashboard.html):

![SnapLogic Dashboard Snaplex Health](http://dl.dropboxusercontent.com/u/3519578/Screenshots/1Z0t.png)

<aside class="success">
A green check mark beside the Snaplex entry will indicate a healthy instance.
</aside>

# Snap Anatomy 101

```java
import com.snaplogic.api.ConfigurationException;
import com.snaplogic.api.ExecutionException;
import com.snaplogic.api.Snap;
import com.snaplogic.common.properties.builders.PropertyBuilder;
import com.snaplogic.snap.api.PropertyValues;
import com.snaplogic.snap.api.SnapCategory;
import com.snaplogic.snap.api.capabilities.Category;
import com.snaplogic.snap.api.capabilities.General;
import com.snaplogic.snap.api.capabilities.Version;
import com.snaplogic.snap.api.capabilities.Inputs;
import com.snaplogic.snap.api.capabilities.Outputs;
import com.snaplogic.snap.api.capabilities.Errors;
import com.snaplogic.snap.api.capabilities.ViewType;

@General(title = "Snap Name", purpose = "Description", 
		author = "Company Name", docLink = "http://www.docs.com/mysnap")
@Inputs(min = 0, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class MySnap implements Snap {

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
    }

    @Override
    public void execute() throws ExecutionException {
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

At its most basic, a Snap class declares the required name, view, version and category metadata annotations and implements the `Snap` interface.

### Metadata annotations

* `@General`: name, description, author, and documentation link for the Snap.
* `@Inputs`: specifies the minimum and maximum number of Input Views and the type of input they accept (JSON Documents or Binary data).
* `@Outputs`: specifies the minimum and maximum number of Output Views and the type of output they produce (JSON Documents or Binary data).
* `@Errors`: specifies whether an Error Views should be written to
* `@Version`: version number for the Snap (not to be confused with its build number).
* `@Category`: categorizes the Snap and determines the icon/color of the Snap within the SnapLogic Designer.

### Snap interface

The `com.snaplogic.api.Snap` interface that should be implemented by the Snap developers to convert their business logic into entity that can be used inside SnapLogic Pipelines.
 
**`defineProperties`**

Defines the Snap properties using the given `PropertyBuilder`. This method is called during the `compile` phase and generates the `settings` Snap schema property that the SnapLogic Designer uses to build the user interface.

**`configure`**

Configures the Snap with the `PropertyValues` provided by the user's input to the Snap Settings. Input validation should be done here - if there is an issue with the provided input, a `ConfigurationException` should be thrown.

**`execute`**

Executes the Snap's business logic. Should an error be encountered, an `ExecutionException` may be thrown.

**`cleanup`**

Cleans up any suitable resources after the Snap execution. Again, should an error be encountered, an `ExecutionException` may be thrown.

<aside class="notice">
Both <code>ConfigurationException</code> and <code>ExecutionException</code> are implementations of the root <code>SnapException</code> interface.

See the <a href="#exceptions-and-error-views">Exceptions and Error Views</a> section for more information.
</aside>

# Getting Started

```shell
$ mkdir ~/opt/snaplogic-dev
$ echo 'export SNAP_HOME=~/opt/snaplogic-dev' >> ~/.bash_profile
$ echo 'export JCC_DEBUG_PORT=9000' >> ~/.bash_profile
$ source ~/.bash_profile
```

If a healthy Snaplex is running locally, Snap Development may begin. 

Make a directory where Snap Development will occur, export the location as the `SNAP_HOME` environment variable and set the port to use for debugging.

## Debugging

```shell
$ cd ~/opt/snaplogic/run/lib
$ java -agentlib:jdwp=transport=dt_socket,server=y,address=${JCC_DEBUG_PORT},suspend=n -jar jcc.war jcc
```

> This will wait to launch the JCC until you have attached the debugger.

The Snaplex/JCC supports debugging a Snap's execution with your chosen IDE. Simply launch the JCC with the additional `agentlib` parameter with shown value, and attach to it using your IDE's remote debugging capability.

## Snap Maven Archetype

```shell
$ cd $SNAP_HOME
$ mvn archetype:generate -DarchetypeCatalog=http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources @ standalone-pom >>>
[INFO] 
[INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources @ standalone-pom <<<
[INFO] 
[INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
Choose archetype:
1: http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/ -> com.snaplogic.tools:SnapArchetype (This is a sample snap project.)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): : 1
Choose com.snaplogic.tools:SnapArchetype version: 
1: 1.0
2: 1.1
3: 1.2
4: 1.3
5: 1.4
6: 1.5
Choose a number: 6: 
Define value for property 'groupId': : com.snaplogic.snaps
Define value for property 'artifactId': : demosnappack
Define value for property 'version':  1.0-SNAPSHOT: : 
Define value for property 'package':  com.snaplogic.snaps: : 
Define value for property 'organization': : snaplogic
Define value for property 'assetPath':  /snaplogic/shared: : /snaplogic/robin/shared
Define value for property 'snapPack':  demosnappack: : Demo Snap Pack
Define value for property 'user': : rhowlett@snaplogic.com
Confirm properties configuration:
groupId: com.snaplogic.snaps
artifactId: demosnappack
version: 1.0-SNAPSHOT
package: com.snaplogic.snaps
organization: snaplogic
assetPath: /snaplogic/robin/shared
snapPack: Demo Snap Pack
user: rhowlett@snaplogic.com
 Y: : Y
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: SnapArchetype:1.5
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.snaplogic.snaps
[INFO] Parameter: artifactId, Value: demosnappack
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.snaplogic.snaps
[INFO] Parameter: packageInPathFormat, Value: com/snaplogic/snaps
[INFO] Parameter: package, Value: com.snaplogic.snaps
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: user, Value: rhowlett@snaplogic.com
[INFO] Parameter: organization, Value: snaplogic
[INFO] Parameter: assetPath, Value: /snaplogic/robin/shared
[INFO] Parameter: groupId, Value: com.snaplogic.snaps
[INFO] Parameter: snapPack, Value: Demo Snap Pack
[INFO] Parameter: artifactId, Value: demosnappack
[INFO] project created from Archetype in dir: /Users/snapdev/opt/snaplogic-dev/demosnappack
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

A [Maven Archetype](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html) is a Maven project templating toolkit. SnapLogic provides `SnapArchetype` for quickly starting Snap development.

<aside class="notice">
The Snap archetype catalog is available at <a href="http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/">http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/</a>
</aside>

`SnapArchetype` ships with eight sample Snaps for demonstration purposes:

Sample | Description
--------- | -----------
**Single Doc Generator** | *Outputs a single document*
**Doc Consumer** | *Consumes documents*
**Doc Generator** | *Outputs `n` documents, where `n` is specified by the user as an expression*
**Suggest** | *Demonstrates suggest setting capabilities*
**Property Types** | *Shows creating a variety of different Snap UI settings*
**Two Inputs** | *Consuming from two different input sources*
**Two Inputs Two Outputs** | *Consuming from two different input sources, and outputting two different Documents*
**Schema Example** | *Demonstrating input and output schemas for IO validation*

<aside class="warning">
If you choose a <code>groupId</code>/<code>package</code> value other than <code>com.snaplogic.snaps</code>, you must provide a HTTP URL value to the <code>docLink</code> parameter of a Snap's <code>@General</code> annotation on a Snap.
</aside>

Once `SnapArchetype` has been generated, it can be imported as a Maven project into your IDE ([IntelliJ IDEA](https://www.jetbrains.com/help/idea/2016.1/importing-project-from-maven-model.html), [Eclipse](https://books.sonatype.com/m2eclipse-book/reference/creating-sect-importing-projects.html), [Netbeans](http://wiki.netbeans.org/MavenBestPractices)).

## Project Structure

> ![Project Structure](https://dl.dropboxusercontent.com/u/3519578/Screenshots/uzoL.png)

### `src/main/assembly`

`final.xml` and `snap.xml` are used by the `maven-assembly-plugin` to build the Snap Pack zip file that can be uploaded to the SnapLogic Manager. You should not need to alter these files.

### `src/main/config`

**directives**

`directives` contains basic Snap Pack metadata. 

The `NAME` property is the label of Snap Pack. This is visible when the SnapLogic Designer is grouping by Snap Pack e.g. the Snaps related to SQL Server are grouped under "SQL Server".

![SQL Server Snap Pack label](https://dl.dropboxusercontent.com/u/3519578/Screenshots/AUH9.png)

The `ASSET_DIR_PATH` property instructs the Snap Pack Installer to deploy the Snap Pack to the specified asset path.

A customer will be deploying to an Organization root, Project, or Project Spaces asset path e.g. `/snaplogic/shared`, `/snaplogic/projects/rhowlett`, and `/snaplogic/robin/shared` respectively (where "snaplogic" is the Organization name assigned by SnapLogic to your account, "rhowlett" is the Project folder name, and "robin" is the Project Spaces. 

This value can be changed dynamically at build time using the `-Dasset_path` Maven parameter (especially useful with a continuous integration tool, or if your company has multiple Organizations or wishes to deploy to multiple asset path locations).

### `src/main/java` and `src/main/resources`

Files and folders within these directories will be on the classpath, as per Maven convention. This is where you will write your Snap code.

### `src/test/java` and `src/test/resources`

Files and folders within these directories will be on the test classpath, as per Maven convention. Unit and integration tests (and supporting resources) are located in these directories.

### `pom.xml`

This is the Maven POM file. You will change this file when registering new Snaps and Accounts, when new dependencies or plugins are required, or when needing to modify the build properties.

# Developing Snaps

```xml
<!-- This identifies the classes which represent the actual Snaps
	(and become accessible on the Snaplex/JCC after deployment).
	
	After creating a new Snap class, add it here.
-->
<snap.classes>
	com.snaplogic.snaps.DocConsumer,
	com.snaplogic.snaps.DocGenerator,
	com.snaplogic.snaps.PropertyTypes,
	com.snaplogic.snaps.SchemaExample,
	com.snaplogic.snaps.SingleDocGenerator,
	com.snaplogic.snaps.Suggest,
	com.snaplogic.snaps.TwoInputs,
	com.snaplogic.snaps.TwoInputsTwoOutputs
</snap.classes>
<!-- After creating a new Account class, add it here -->
<account.classes>
</account.classes>
```

> pom.xml

SnapLogic provides the **JSDK**, a library of Java APIs to assist with building Snaps. 

The [Snap Maven Archetype](#snap-maven-archetype) ships with sample Snaps that we can use to highlight the JSDK's capabilities.

## Basic Snap Implementation

```java
// Set the Snap title, description etc
// Also, use the "docLink" parameter to set a link to the documentation
@General(title = "Single Doc Generator", purpose = "Generates one document and completes",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
// There is no input view for this Snap
@Inputs(min = 0, max = 0)
// This Snap has exactly one document output view
// Snaps can also have binary output i.e., offers={ViewType.BINARY}
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
// This Snap has an optional document error view
@Errors(min = 0, max = 1, offers = {ViewType.DOCUMENT})
// Version number of the Snap
@Version(snap = 1)
// This Snap belongs to SnapCategory.READ as is does not require input.
@Category(snap = SnapCategory.READ)
public class SingleDocGenerator implements Snap {
    private static final Logger log = LoggerFactory.getLogger(SingleDocGenerator.class);
    
    int counter = 0;
    
    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues)
            throws ConfigurationException {
    }

    @Override
    public void execute() throws ExecutionException {
        counter++;
        log.debug("counter=" + counter);

        /*
         * Write a single document to outputView. The next Snap in the pipe will
         * only ever see a single document from this Snap.
         */
        Map<String, String> data = new LinkedHashMap<String, String>() {{
            put("key", "value");
        }};
        outputViews.write(documentUtility.newDocument(data));
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

`SingleDocGenerator` is one of the sample Snaps included in the Snap Archetype. 

It is an uncomplicated Snap that generates a single document, without requiring any input data or user input. It implements the `Snap` interface.

![SingleDocGeneratorPipeline](https://dl.dropboxusercontent.com/u/3519578/Screenshots/o6o1.png)

[Snap Anatomy 101](#snap-anatomy-101) explains the purpose of each annotation and the overridden methods. The comments above each annotation describe their specific usage for this Snap.

<aside class="notice">
For simplicity, the "Single Doc Generator" Snap does not permit any input views. However, for general Snap development, there should **almost always** be an input view. See <a href="#reading-documents-from-an-input-view">Reading Documents from an Input View</a> for more.
</aside>

### Snap Lifecycle

At build time, `defineProperties` is called to generate the Snap Schema. This will construct the Snap's settings tab.

At runtime, the Snap is initialized by the Snaplex/JCC. Next, [Guice](https://github.com/google/guice) injects the various declared dependency instances (`documentUtility` and `outputViews` in this Snap).

The Snaplex's runtime then instructs the Snap's `configure` method to be called to validate and/or operate on any provided `PropertyValues` created from a user's interaction with the Snap's settings.

Next, the `execute` method is called. This is where the majority of the Snap's business logic will initiate.

Finally, the `cleanup` method is called to clean-up any resources used.

Due to no user input being required, nor clean-up needed, the `defineProperties`, `configure()`, and `cleanup` methods have no implementation for this Snap. 

The `execute` method increments an `int` member variable, logs a debug statement, creates data, and generates a new Document with this data to be subsequently written to the output view.

## Logging

```shell
$ brew install lnav
$ lnav -i https://snaplogic.box.com/v/lnav-log-format
$ lnav ~/opt/snaplogic/run/log/jcc.json

2016-07-13T13:10:35.072 DEBUG [Single Doc Generator[5d64230b-fa22-4696-b736-05b599b06ba9 -- 9a3a88ef-d5b1-4715-945d-0c6a595f7940]] (SingleDocGenerator.java:77) - counter=1                                                                                      │
  snrd: 9a3a88ef-d5b1-4715-945d-0c6a595f7940                                                                                                                                                                                                                     │
  com.snaplogic.logging.id: 57868ceb64cabf28873d20a7.5d64230b-fa22-4696-b736-05b599b06ba9.9a3a88ef-d5b1-4715-945d-0c6a595f7940                                                                                                                                   │
  plrd: 5d64230b-fa22-4696-b736-05b599b06ba9                                                                                                                                                                                                                     │
  snlb: Single+Doc+Generator                                                                                                                                                                                                                                     │
  ccid: 57868ceb64cabf28873d20a7                                                                                                                                                                                                                                 │
  snii: 898c5459-0419-455c-9493-a0d6fbae9c8a                                                                                                                                                                                                                     │
  xid: a83bcfdd-2bc7-480e-bf1e-5fbdfef3c726   
```

In ([the "Single Doc Generator" sample Snap](#basic-snap-implementation)) above, an SLF4J `Logger` static class variable is declared and initialized and is used to write debug messages to the Snaplex/JCC's log.

For example, when previewing or executing the "Single Doc Generator" Snap in the Designer, the value of the `counter` variable is written to the log. 

This entry can then be viewed in the Snaplex's `jcc.json` log file (we recommend using [LNAV, the log file navigator](http://www.snaplogic.com/blog/lnav/) created by SnapLogic's own Timothy Stack):

## Creating Documents with DocumentUtility

```java
package com.snaplogic.snap.api;

public interface DocumentUtility {
	...
	/**
	 * Returns a new document with the given data for the given incoming document.
	 *
	 * @param incomingDocument
	 * @param data
	 * @return document
	 */
	Document newDocumentFor(Document incomingDocument, Object data);

	/**
	 * Returns a new document object with the given data and merges the original data into the
	 * new returned document.
	 * This allows to "pass through" the data from an incoming document and merge it into the new
	 * document being written to the output. The new document will have an {@literal original}
	 * key holding the original document to avoid data being overwritten when merging the data
	 * with original data
	 *
	 * @param document     as the document being written
	 * @param originalData as original data being merged with the document.
	 *
	 * @return document
	 * @throws UnsupportedOperationException only data and originalData of type Map is currently
	 *                                       supported
	 */
	Document newDocument(Document document, Object originalData);
	...
}
```

Guice's `@Inject` annotation will instruct the Snaplex/JCC to inject an instance of `DocumentUtility` and assign it to `documentUtility` ([see "Single Doc Generator" Snap above](#basic-snap-implementation)).

Documents may be created with given data. 

<aside class="success">
Data should be created either as a single <code>LinkedHashMap</code> instance (to preserve field ordering) or a <code>List</code> of <code>LinkedHashMap</code> instances. Avoid data being just a primitive data type or <code>null</code>.
</aside>

### Lineage

For Snaps that have input views and process documents, you should preserve [data lineage](https://en.wikipedia.org/wiki/Data_lineage) (required by [Ultra Pipelines](https://www.snaplogic.com/ultra-pipelines)) between the new output document to be written and the incoming original document by using one of the highlighted `DocumentUtility` methods:

*Establishes lineage between the new document and the incoming original document, but discards the incoming data:*<br />
`Document newDocumentWithLineage = documentUtility.newDocumentFor(originalDocument, data);`

*Establishes lineage between the new document and the incoming original document and "passes through" the incoming document's data by adding a key named "original" (containing the incoming original document's data) to the new document:*<br />
`Document seed = documentUtility.newDocument(newData);`<br />
`Document newDocumentWithLineage = documentUtility.newDocument(seed, originalDocument);`

<aside class="success">
We recommend that you do pass through the original document for greatest flexibility.
</aside>

## Writing to an Output View

```java
package com.snaplogic.snap.api;

/**
 * OutBoundViews is the base interface for all the output views - {@link OutputView} and
 * {@link com.snaplogic.snap.view.ErrorView}
 */
public interface OutBoundViews<V extends OutputView> extends Views<V> {
	...
	/**
	 * Writes the given document to all the outbound views of the Snap.
	 *
	 * @param document
	 */
	void write(Document document);

	/**
	 * Writes a new document object with the given data and merges the original data into the
	 * new returned document.
	 * This allows to "pass through" the data from an incoming document and merge it into the new
	 * document being written to the output. The new document will have an {@literal original}
	 * key holding the original document to avoid data being overwritten when merging the data
	 * with original data.
	 *
	 * @param document     as the document being written
	 * @param originalData as original data being merged with the document.
	 *
	 * @return document
	 * @throws UnsupportedOperationException only data and originalData of type Map is currently
	 *                                       supported
	 */
	void write(Document document, Object originalData);
	...
}
```

Guice's `@Inject` annotation will instruct the Snaplex/JCC to inject an instance of `OutputViews` and assign it to `outputViews` ([see "Single Doc Generator" Snap above](#basic-snap-implementation)).

### Lineage

Similarly to [DocumentUtility](#creating-documents-with-documentutility), output views can establish lineage between incoming and outgoing documents. This is useful when an outgoing Document has been created without lineage but the incoming original document is still accessible:

`outputViews.write(documentWithoutLineage, originalDocument);`

## Unit Testing Snaps with SnapTestRunner

```java
import com.snaplogic.snap.test.harness.OutputRecorder;
import com.snaplogic.snap.test.harness.SnapTestRunner;
import com.snaplogic.snap.test.harness.TestFixture;
import com.snaplogic.snap.test.harness.TestResult;

/**
 * Tests that the {@link SingleDocGenerator} Snap sent one Document to the output view.
 */
@RunWith(SnapTestRunner.class)
public class SingleDocGeneratorTest {

    @TestFixture(snap = SingleDocGenerator.class,
            outputs = "output0")
    public void testSingleDocGeneratorFunctionality(TestResult testResult)
            throws Exception {
        assertNull(testResult.getException());
        OutputRecorder outputRecorder = testResult.getOutputViewByName("output0");
        assertEquals(1, outputRecorder.getRecordedData().size());
    }
}
```

The `SingleDocGeneratorTest` class outlines basic usage of **jtest**, a JUnit-based Snap unit testing framework.

jtest provides `SnapTestRunner`, a custom JUnit test runner which simulates the targeted Snap being executed within a pipeline. 

Test methods are annotated with the `@TestFixture` annotation, which specifies what Snap implementation to use, input/output/error views configured, property settings specified, accounts associated etc.

The test method should take a single `TestResult testResult` argument, which can be used within the test to determine whether the correct output was written (using an `OutputRecorder`), if errors occurred etc.

In the `SingleDocGeneratorTest` sample provided, we can see that the test method focuses on examining that the `TestResult` did not receive any thrown exceptions, and that the declared output view registered the expected number of documents processed.

The ["jtest: Snap Unit Testing Framework"](#jtest-snap-unit-testing-framework) section contains much more information about testing Snaps, including working towards a purely declarative style of Snap unit testing using just the test method's `@TestFixture` annotation.

## Reading Documents from an Input View

```java
@General(title = "Doc Consumer", purpose = "Consumes the incoming documents",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 1, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 0, max = 0)
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.WRITE)
public class DocConsumer implements Snap {
    private static final Logger log = LoggerFactory.getLogger(DocConsumer.class);
    private AtomicInteger count = new AtomicInteger(0);

    @Inject
    private InputViews inputViews;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
    }

    @Override
    public void execute() throws ExecutionException {
        InputView inputView = inputViews.get();
        Iterator<Document> documentIterator = inputViews.getDocumentsFrom(inputView);
        while (documentIterator.hasNext()) {
            Document doc = documentIterator.next();
            log.debug("Received Document {}", doc.toString());
            count.getAndIncrement();
        }
    }

    @Override
    public void cleanup() throws ExecutionException {
        log.debug("Consumed {} documents", count.get());
    }
}
```

The `DocConsumer` sample Snap that ships with the [Snap Archetype](#snap-maven-archetype) demonstrates how to read from a Snap's input view.

Similarly to [Writing to an Output View](#writing-to-an-output-view), Guice will inject an instance of `InputViews` and assign it to the `inputViews` variable.

`inputViews.get()` returns the default view associated with the Snap. If the Snap has more than one view, a `ViewException` is thrown.

An `Iterator` of `Document` objects can then be acquired by asking the `InputViews` instance for the `Document` objects associated with the specified `InputView` instance.

Iterating over the `Document` objects, the Snap logs that a document was consumed and increments its `AtomicInteger` counter.

The `cleanup()` method, which is called after the Snap has completed its execution phase, logs the total number of documents consumed.

![Consuming 3 Documents](http://dl.dropboxusercontent.com/u/3519578/Screenshots/KZdv.png)

<aside class="notice">
SnapLogic's Platform will enforce that all input documents have been fully consumed, otherwise an error will occur and the pipeline will fail.
</aside>

## Binary Input/Output Views

TK

### Headers


## Snap Categories

The [Doc Consumer Snap's](#reading-documents-from-an-input-view) category (`SnapCategory.WRITE`) differs from [SingleDocGenerator's](#basic-snap-implementation) `SnapCategory.READ` category. 

This difference between category types manifests in three ways:

* the color and icon of the Snaps are different,
* the "Doc Consumer" Snap inherits an "Execute During Preview" setting to toggle whether the Snap will execute when the Pipeline is validating/previewing,
* the Snaps are separated when "Group By Type" is chosen.

![Snap Category Difference](http://dl.dropboxusercontent.com/u/3519578/Screenshots/hEPI.png)

Snap Category | Description
--------- | -----------
**READ** | Sources of data in the pipeline.<br />*Example: File Reader*
**WRITE** | Data destinations or sinks in the pipeline. They also inherit an "Execute during Preview" setting.<br />*Example: File Writer* 
**PARSE** |	Parse unstructured input data into structured output data.<br />*Example: CSV Parser*
**FORMAT** | Change data format.<br />*Example: CSV Formatter*
**TRANSFORM** | Modify data significantly.<br />*Example: Aggregate, Join*
**FLOW** | Change the direction or output of data in the pipeline.<br />*Example: Filter, Router*

As you develop Snaps, choose the appropriate Snap Category for your use cases.

## Bootstrapping with SimpleSnap

```java
/**
 * This Snap accepts two inputs and outputs to a single output. To use it, feed
 * it at least one document in each view. It will output a single stream which consists
 * of the combination of the two inputs.
 *
 * @author <you>
 */
@General(title = "Two Inputs", purpose = "Accepts two inputs, merges them",
        author = "Your Company Name" docLink = "http://yourdocslinkhere.com")
@Inputs(min = 2, max = 2)
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 0, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
/*
 * This Snap extends {@link SimpleSnap} (instead of implementing {@link Snap}).
 * This means that instead of having a method called 'execute' which is called once,
 * it has a method called 'process' which is called for every document the Snap receives.
 * This means you do not have to iterate over incoming documents as 'process' does that
 * for you.
 */
public class TwoInputs extends SimpleSnap {

    private static final Logger log = LoggerFactory.getLogger(TwoInputs.class);
    private int count = 0;
    @Inject
    private OutputViews outputViews;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues)
            throws ConfigurationException {
    }

    @Override
    public void process(Document document, String inputViewName) {
        // Demonstrates how to create a new copy of the document
        Document newdoc = document.copy();

        // get a map representation of the document
        // NOTE: we are *not* duplicating the data.
        // We will *not* have to convert the map back to a doc.
        // It's just a pointer representation.
        Map<String, Object> data = newdoc.get(Map.class);

        // assign a value to a field of the map
        // In this case, we are just assigning 'processed=True'
        // to signal that the document has been processed.
        data.put("processed", "True");

        count++;
        // log current document number
        log.debug("count=" + count);

        // log current document
        log.debug("document: " + newdoc.toString());

        // send new document to next Snap
        outputViews.write(newdoc, document);
    }

    @Override
    public void cleanup() throws ExecutionException {
        // Log final number of documents processed
        log.debug("Final count=" + count);
    }
}
```

Now that we understand the basics of a Snap, we can extend the `SimpleSnap` class to have some of the heavy lifting done for us.

`SimpleSnap` provides all the necessary hooks for writing a new Snap. A Snap author can write a new Snap by simply extending `SimpleSnap`, adding the desired metadata annotations (`SimpleSnap` already declares an `@Errors` view that can be overridden), and providing the implementation for the `process` method (and optionally, for the `defineProperties` and `configure` methods too).

`SimpleSnap` takes care of iterating over the incoming documents (if any) and allowing you to process each document at a time. It also handles the dependency injection for the `Input/Output/ErrorViews`, the `DocumentUtility`, and the `ExecutionUtil` (which handles iterating over the documents in the incoming input view(s) on your behalf, and writing any thrown `SnapDataException` instances to the error view).

![Two Inputs](https://dl.dropboxusercontent.com/u/3519578/Screenshots/IVTO.png)

## Accepting User Input with PropertyBuilder

```java
@General(title = "Doc Generator", purpose = "Generates documents based on the configuration",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 0, max = 0)
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 0, max = 0)
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class DocGenerator implements Snap {
    private static final String COUNT = "count";

    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;

    private int count;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(COUNT, "Number of documents to create")
                .type(SnapType.INTEGER).required().add();
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
        BigInteger countValue = propertyValues.get(COUNT);
        count = countValue.intValue();
    }

    @Override
    public void execute() throws ExecutionException {
        for (int i = 0; i < count; i++) {
            Map<String, String> data = new LinkedHashMap<>();
            data.put("key", "value" + (i + 1));
            outputViews.write(documentUtility.newDocument(data));
        }
    }

    @Override
    public void cleanup() throws ExecutionException {
        // NOOP
    }
}
```

The `DocGenerator` Snap is similar to the [`SingleDocGenerator`](#basic-snap-implementation) Snap, except the number of documents generated is configurable through user input:

![Doc Generator](http://dl.dropboxusercontent.com/u/3519578/Screenshots/lJqe.png)

<aside class="success">
All Snaps will have a required <strong>Label</strong> property for customizing the display label of the Snap within a Pipeline.
</aside>

To accept user input, the `defineProperties` method is used.

[As described earlier](#snap-anatomy-101), `defineProperties` is called when the Snap is being built. It constructs the Snap Schema which the Designer UI interprets and assembles into the Snap's Settings tab.

In `DocGenerator.defineProperties()`, a required `count` property of type `SnapType.INTEGER` is described and added to the provided `PropertyBuilder` instance.

Once the user has input their data and the Snap is executing, the `configure` method can access the data through the provided `PropertyValues` instance. As shown, this could be stored in an instance variable and used in the Snap's `execute()` method.

See the [PropertyBuilder Reference](#propertybuilder-reference) section for more examples.

## Validating User Input

The SnapLogic Platform will attempt to provide some basic type validation on user input:

![Auto validation](http://dl.dropboxusercontent.com/u/3519578/Screenshots/mi8k.png)

If more advanced validation is required, provide the implementation in the `configure` method and throw a `ConfigurationException` when invalid input is encountered. See [Exceptions and Error Views](#exceptions-and-error-views) for more.

## Expression-enabled Properties

```java
public class DocGenerator implements Snap {
    private static final String COUNT = "count";
    private static final Document NO_DOCUMENT = null;

    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;
    
	private final ;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(COUNT, "Number of documents to create")
                .type(SnapType.INTEGER).required().expression().add();
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
        ExpressionProperty countValueExp = propertyValues.getAsExpression(COUNT);

    	// evaluate the expression directly (without the context of a Document)
		BigInteger countValue = countValueExp.eval(NO_DOCUMENT);
		
		count = countValue.intValue();
    }

    @Override
    public void execute() throws ExecutionException {
        for (int i = 0; i < count; i++) {
            Map<String, String> data = new LinkedHashMap<>();
            data.put("key", "value" + (i + 1));
            outputViews.write(documentUtility.newDocument(data));
        }
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

Snaplogic's [Expression Language](http://doc.snaplogic.com/expression-language) is an incredibly powerful tool available to Snaps. Using a JavaScript-like syntax, users have access to functions and properties to dynamically set property values.

Comparing to the [previous implementation](#accepting-user-input-with-propertybuilder) of `DocGenerator`, we can see that `.expression()` has been added to the `propertyBuilder()`. 

The `configure` method's `BigInteger countValue` local variable has been replaced by the `ExpressionProperty countValueExp` local variable, which is retrieved from the `PropertyValues` with the `getAsExpression` method, passing the field name "count" in. The `countValueExp` expression property is evaluated against `null` (as there is no incoming document to evaluate against e.g. when referencing a key in the incoming document), so the expression is evaluated directly. Since a number value was evaluated, we assign it to a local variable of type `BigInteger`, and then get its `intValue()`.

Within `execute()`, we can then use the `count` in the loop.

To demonstrate this new capability, we can use a "count" [Pipeline Property](http://doc.snaplogic.com/pipeline-properties) and then reference this in the Snap within a `parseInt` expression function:

![Doc Generator with Expression](https://dl.dropboxusercontent.com/u/3519578/Screenshots/Zin9.png)

### @PlatformFeature(coerceAndValidateExpressions = true)

TK

## Suggesting Property Values

```java
/**
 * This Snap has one output and writes one document that contains the suggested
 * value. This Snap demonstrates the suggest value functionality that uses the
 * partial configuration information to suggest a property value.
 *
 * @author <you>
 */
@General(title = "Suggest", purpose = "Demo suggest functionality.",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 0, max = 0)
@Outputs(min = 1, max = 1)
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class Suggest implements Snap {

    public static final String PROP_NAME = "name";
    public static final String PROP_ECHO = "echo";
    
    private String valueToWrite;
    
    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;

    @Override
    public void defineProperties(final PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(PROP_NAME, PROP_NAME)
                .add();
        propertyBuilder.describe(PROP_ECHO, PROP_ECHO)
                .withSuggestions(new Suggestions() {
                    @Override
                    public void suggest(SuggestionBuilder suggestionBuilder,
                            PropertyValues propertyValues) {
                        String name = propertyValues.get(PropertyCategory.SETTINGS, PROP_NAME);
                        suggestionBuilder.node(PROP_ECHO).suggestions(name);
                    }
                }).add();
    }

    @Override
    public void configure(final PropertyValues propertyValues) throws ConfigurationException {
        valueToWrite = propertyValues.get(PROP_ECHO);
    }

    @Override
    public void execute() throws ExecutionException {
        Map<String, String> data = new LinkedHashMap<String, String>() {{
            put("key", valueToWrite);
        }};
        outputViews.write(documentUtility.newDocument(data));
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

A Snap can make "suggestions" to the user about property values. This is useful to lookup appropriate values using either another data source or the input value of a different property.

In the "Suggest" sample Snap, the user enters in a value for the "name" property. The "echo" property's suggest action then reads that value and allows the user to select it.

![Suggest](https://dl.dropboxusercontent.com/u/3519578/Screenshots/e2S6.png)

The suggest capability is enabled through use of the the `PropertyBuilder.withSuggestions(Suggestions suggestions)` method. Implementations of the `Suggestions` interface should be provided to it.

<aside class="notice">
Notice that the UI also added the <strong>"="</strong> expression toggle button - suggestible properties <a href="#expression-enabled-properties">should always be expression-enabled too</a>. 

This was not done in the <code>Suggest</code> example above. The work required to support expressions for the field is left as an exercise to the reader.
</aside>

The `Suggestions` instance (often an anonymous instance) implements the single `suggest` method. It provides, as method arguments, references to the `SuggestionBuilder` and the `PropertyValues`. 

Using the `SuggestionBuilder` instance, we can configure simple String varargs for `suggestions()` for a selected `node()` that will result in a list of suggested values.

<aside class="success">
Alternatively, use <code>objectSuggestions()</code> to add a list of values, with each entry representing a suggestion. The entry must be a map which provides a <strong>"label"</strong> key.

The label will be displayed by the UI as the selectable entry, the rest of the object definition will be hidden.
</aside>

### Suggestions within Tables and Composites

To add suggestions to a Table element property, select the table node and use the `over()` builder method:

<div class="inline-code">
SnapProperty echo = propertyBuilder.describe(PROP_ECHO, PROP_ECHO)
	.type(SnapType.STRING)
	.withSuggestions(new Suggestions() {
		@Override
		public void suggest(SuggestionBuilder suggestionBuilder, 
				PropertyValues propertyValues) {
			suggestionBuilder
				.node("table")
				.over("echo")
				.suggestions("X", "Y");
		}
	}).build();

propertyBuilder.describe("table", "table")
	.type(SnapType.TABLE)
	.withEntry(echo)
	.add();
</div>

![Table Suggest](http://dl.dropboxusercontent.com/u/3519578/Screenshots/NlQw.png)

<aside class="warning">
Suggestions are not supported within Composite properties.
</aside>

## Input/Output View Schemas

```java
/**
 * A Pass-through Snap to provide schema on its input and output view.
 * If the incoming document matches the expected schema, document is written
 * to the output view. If the document does not match the schema, it is written
 * to the error view.
 */
@General(title = "View Schema", purpose = "Demonstrates view schema definition",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 1, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.TRANSFORM)
public class SchemaExample extends SimpleSnap implements InputSchemaProvider,
        OutputSchemaProvider {
    private static final String COL_A = "colA";
    private static final String COL_B = "colB";
    private static final String COL_C = "colC";
    private static final String INPUT_VIEW_NAME = "input0";
    private static final String OUTPUT_VIEW_NAME = "output0";

    @Override
    public void defineInputSchema(final SchemaProvider provider) {
        Schema colA = provider.createSchema(SnapType.STRING, COL_A);
        Schema colB = provider.createSchema(SnapType.STRING, COL_B);
        Schema colC = provider.createSchema(SnapType.STRING, COL_C);
        provider.getSchemaBuilder(INPUT_VIEW_NAME)
                .withChildSchema(colA)
                .withChildSchema(colB)
                .withChildSchema(colC)
                .build();
    }

    @Override
    public void defineOutputSchema(SchemaProvider provider) {
        Schema colA = provider.createSchema(SnapType.STRING, COL_A);
        Schema colB = provider.createSchema(SnapType.STRING, COL_B);
        Schema colC = provider.createSchema(SnapType.STRING, COL_C);
        provider.getSchemaBuilder(OUTPUT_VIEW_NAME)
                .withChildSchema(colA)
                .withChildSchema(colB)
                .withChildSchema(colC)
                .build();
    }

    @Override
    public void defineProperties(final PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues)
            throws ConfigurationException {
    }

    @Override
    protected void process(Document document, final String inputViewName) {
        // Validate incoming data against the schema
        validate(document);
        // Pass the document to the downstream Snap in the pipeline.
        outputViews.write(document);
    }

    private void validate(Document document) {
        Map<String, Object> content = document.get(Map.class);
        for (String key : new String[]{COL_A, COL_B, COL_C}) {
            if (!content.containsKey(key)) {
                // Throwing a SnapDataException instructs the SnapLogic platform
                // to forward this document to the error view with the
                // provided error message.
                throw new SnapDataException(document,
                        String.format("Data map does not contain key:%s", key));
            }
        }
    }
}
```

Snap View Schemas allow Snaps to declare the structure of the data they are expecting to consume and/or generate. 

When understanding how to use a Schema, you must first understand how declaring an input or output schema affects the Snaps in the pipeline immediately preceding and following, respectively.

A Snap's input view schema becomes the Target Schema of the Snap (e.g. Mapper) immediately preceding it in the pipeline. This allows the user the opportunity to shape the data to the desired form:

![Input Schema](http://dl.dropboxusercontent.com/u/3519578/Screenshots/M3WA.png)

In the `SchemaExample` sample (which creates the **"View Schema"** Snap), an input schema is declared by implementing the `InputSchemaProvider` interface. This interface has a `defineInputSchema` method, where the `SchemaProvider` argument can be used to create child schemas and add them to the provider through a `SchemaBuilder`.

Similarly the Snap's output view schema becomes the Input Schema of the Snap immediately following it in the pipeline, allowing users to anticipate the shape of the data exiting the Snap:

![Output Schema](http://dl.dropboxusercontent.com/u/3519578/Screenshots/gqxF.png)

### Enforcing a Schema

Defining the schema as above will aid users in understanding what should be the structure of the data entering and exiting the Snap. However, the Snap will not reject data that does not conform to the schema.

If you wish to be strict about enforcing a schema, then you may provide your own validation of the input and output data during `execute`/`process`.

In the `SchemaExample` sample above, the `validate` method is called before the data is written to the output view. It checks that the incoming document has the properties declared in the schema present (`colA`, `colB`, and `colC`).

If the input data contains those keys, the document is written to the output view:

![Valid Input](http://dl.dropboxusercontent.com/u/3519578/Screenshots/fiAa.png)

If the input data does not contain all of those keys, a `SnapDataException` is thrown and the document is written to the error view:

![Invalid Input](http://dl.dropboxusercontent.com/u/3519578/Screenshots/f4sZ.png)

## Exceptions and Error Views

```java
package com.snaplogic.api;

/**
 * This is the root exception for the jsdk module.
 *
 * All the other exceptions in jsdk module should derive from this exception. This exception
 * should not be used directly by the Snap author. Snap author should use
 * {@link com.snaplogic.api.ConfigurationException} or
 * {@link com.snaplogic.api.ExecutionException}
 */
public class SnapException extends RuntimeException {
    private static final long serialVersionUID = 1L;
    
    private String message;
    private String reason;
    private String resolution;

    private static Pattern STACK_TRACE_PATTERN = Pattern.compile("\n[ \t]*at[ \t]+");

    public SnapException(String message) {
        super();
        this.message = stripStackTrace(message);
    }

    public SnapException(String message, String reason) {
        this(message);
        this.reason = stripStackTrace(reason);
    }

    public SnapException(Throwable cause, String message) {
        this(message);
        super.initCause(cause);
    }

    public SnapException(Throwable cause, String message, String... params) {
        this(cause, stringFormat(message, params));
        this.reason = this.getMessage();
    }

    /**
     * Applies the given parameter values to the message template string.
     */
    public SnapException formatWith(Object... params) {
        String message = getMessage();
        this.message = stripStackTrace(stringFormat(message, params));
        return this;
    }
    ...
    /**
     * Sets the exception message for the user.
     */
    public final <T extends SnapException> T  withReason(final String reason) {
        this.reason = stripStackTrace(reason);
        return (T) this;
    }

    /**
     * Sets the resolution for fixing this exception.
     */
    public final <T extends SnapException> T withResolution(String resolution) {
        this.resolution = stripStackTrace(resolution);
        return (T) this;
    }

    public final <T extends SnapException> T  withResolutionAsDefect() {
        this.resolution = Messages.PLEASE_FILE_A_DEFECT_AGAINST_SNAP;
        return (T) this;
    }
    ...
```

`SnapException` is the root exception type and extends `RuntimeException`. However, you should instead use one of the following implementations instead:

Class | Where to Use | Description
--------- | ----------- | -----------
`ConfigurationException` | `configure()` | Throw when the Snap's settings are invalid.
`ExecutionException` | `execute()` | Throw for internal, non-data-related ex errors. 
`SnapDataException` | `execute()` | Throw for data-related errors.

<aside class="notice">
If your Snap extends <code>SimpleSnap</code>, thrown <code>SnapDataException</code>s will be automatically written to the error view.
</aside>

### Constructing Exceptions

At it's most basic, a `SnapException` subclass instance just requires a failure message to be communicated to the user:

<div class="inline-code">
@Override
public void execute() throws ExecutionException {
	throw new ExecutionException("Message");
}
</div>

This results in the following user experience:

![Basic Exception](https://dl.dropboxusercontent.com/u/3519578/Screenshots/eX3K.png)

When creating exceptions, we recommend providing a failure message, a reason, and a resolution:

Parameter | Description
--------- | -----------
Message | High level explanation of the issue (what was the Snap trying to do).
Reason | Detailed explanation of the issue (defaults to the root cause message).
Resolution | What can the user do to fix the issue

<div class="inline-code">
@Override
public void execute() throws ExecutionException {
	String deptName = "deptName";

	String message = "The \"%s\" field was missing.";
	String reason = "This Snap requires all fields to be present in the data source.";
	String resolution = String.format("Contact your DBA to ensure all model fields " +
			"are present in the data source.", deptName);

	throw new ExecutionException(message)
		.formatWith(deptName)
		.withReason(reason)
		.withResolution(resolution);
}		
</div>

![Reason and Resolution](https://dl.dropboxusercontent.com/u/3519578/Screenshots/FjIr.png)

### ConfigurationException

`ConfigurationException` instances should be thrown during `configure()`, when the pipeline is validating:

<div class="inline-code">
@Override
public void configure(PropertyValues propertyValues) throws ConfigurationException {
	throw new ConfigurationException("Error Message for User")
		.withReason("A good reason.")
		.withResolution("What to do next.");
}
</div>

![Config Exception](https://dl.dropboxusercontent.com/u/3519578/Screenshots/JAB9.png)

### Writing Exceptions to the Error View

Throwing a new `SnapDataException` and writing it to the error view along with the input document that generated the data error, will display both the exception stacktrace and the original data. For example:

<div class="inline-code">
@Override
public void execute() throws ExecutionException {
	try {
		throw new IllegalArgumentException("Illegal Argument.");
	} catch (IllegalArgumentException e) {
		SnapDataException snapDataException = new SnapDataException(e, "Error Message.")
			.withResolution("Provide the correct argument.");

		errorViews.write(snapDataException, document);
	}
}
</div>

will result in the following content written to the error view:

![Error View Exception](http://dl.dropboxusercontent.com/u/3519578/Screenshots/DmKE.png)

If your Snap extends `SimpleSnap`, it is even easier - just throw the `SnapDataException` and it will be written to the error view as above:

<div class="inline-code">
@Override
public void process(Document document, String inputViewName) throws ExecutionException {
	try {
		throw new IllegalArgumentException("Illegal Argument.");
	} catch (IllegalArgumentException e) {
		throw new SnapDataException(e, "Error Message.")
			.withResolution("Provide the correct argument.");
	}
}
</div>

<aside class="success">
SnapLogic recommends always enabling a Document error view:

<p><code>@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})</code></p>
</aside>

## Customizing Input/Output Views

```java
/**
 * This Snap demonstrates two inputs and two outputs.
 *
 * It will output two streams, one for only males and another for females. 
 * Unknowns will get sent to both.
 *
 * @author <you>
 */
@General(title = "Two Ins/Outs", purpose = "Accepts two inputs, sends to two outputs.",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 2, max = 2)
@Outputs(min = 2, max = 2, offers = {ViewType.DOCUMENT})
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class TwoInputsTwoOutputs extends SimpleSnap implements ViewProvider {
    private static final Logger log = LoggerFactory.getLogger(TwoInputsTwoOutputs.class);
    
    private final static String INPUT_0 = "input0";
    private final static String INPUT_1 = "input1";
    private final static String MALE_VIEW = "output_male";
    private final static String FEMALE_VIEW = "output_female";
    
    @Inject
    private OutputViews outputViews;
    
    @Inject
    private ErrorViews errorViews;

    @Override
    public void defineViews(final ViewBuilder viewBuilder) {
        viewBuilder.describe(INPUT_0)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.INPUT);
        viewBuilder.describe(INPUT_1)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.INPUT);

        viewBuilder.describe(MALE_VIEW)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.OUTPUT);
        viewBuilder.describe(FEMALE_VIEW)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.OUTPUT);
    }

    ...

    @Override
    public void process(Document document, String inputViewName) {
        Document newdoc = document.copy();
        Map<String, Object> data = newdoc.get(Map.class);

        // Add a new field "processed"
        data.put("processed", "True");

        // send males to male output view
        if (data.get("gender").equals("male")) {
            outputViews.getDocumentViewFor(MALE_VIEW).write(newdoc);
        } else if (data.get("gender").equals("female")) {
            // send females to female output view
            outputViews.getDocumentViewFor(FEMALE_VIEW).write(newdoc);
        } else {
            // send unknowns to error view
            errorViews.write(newdoc, document);
        }
    }
    
    ...
}
```

The `TwoInputsTwoOutputs` sample accepts documents from two input views and routes to a particular output view based on the value of the "gender" field:

![TwoInOut](http://dl.dropboxusercontent.com/u/3519578/Screenshots/oHpC.png)

### Reading from Multiple Input Views

As it extends `SimpleSnap`, the "Two Ins/Outs" Snap benefits from already knowing how to read from multiple input views. `ExecutionUtil` iterates over the available input views and send each document to the `process()` method, [as shown previously](#bootstrapping-with-simplesnap). If you can guarantee that each input view will have documents sent to it, this is an acceptable solution. 

<aside class="success">
If you cannot guarantee that all input views will be written to, use <code>InputViews.select()</code> to poll the input views in a non-blocking manner. SnapLogic recommends this approach for most multiple-input view scenarios.
</aside>

### Configuring Optional Views

It is rarely a good idea to prevent an input and output views for a Snap. For one, it prevents using that Snap as unlinked input at the beginning of [a Task pipeline](http://doc.snaplogic.com/tasks), and restricts its ability to be used mid-pipeline.

However, if you wish to allow the user to remove all input and output views, it's best to declare the views as optional, to open them by default, and then have the user remove them through the **Views** tab of the Snap.

This can be done by implementing the `ViewProvider` interface and using the `defineViews()` method's `ViewBuilder` argument:

<div class="inline-code">
@Inputs(min = 0, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 0, max = 1, offers = {ViewType.DOCUMENT})
...
// this creates the two open input views and the two named output views
@Override
public void defineViews(final ViewBuilder viewBuilder) {
	viewBuilder.describe(INPUT_0)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.INPUT);
	viewBuilder.describe(INPUT_1)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.INPUT);

	viewBuilder.describe(MALE_VIEW)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.OUTPUT);
	viewBuilder.describe(FEMALE_VIEW)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.OUTPUT);
}
</div>

![Remove Views](http://dl.dropboxusercontent.com/u/3519578/Screenshots/nFE8.png)

### Writing to Multiple Output Views

`OutputViews.getDocumentViewFor(viewName)` can be used to return the document output view for the given view name:

<div class="inline-code">
// send males to male output view
if (data.get("gender").equals("male")) {
	outputViews.getDocumentViewFor(MALE_VIEW).write(newdoc);
} else if (data.get("gender").equals("female")) {
	// send females to female output view
	outputViews.getDocumentViewFor(FEMALE_VIEW).write(newdoc);
} else {
	// send unknowns to error views
	errorViews.write(newdoc, document);
}
</div>

## Preview-specific Behavior

`SuggestExecutionProvider` (confusing name)

`configureForSuggest` and `executeForSuggest`

# Deploying Snap Packs

TK

## Using Maven Goal snappack:deploy

By default, will deploy to ASSET_DIR_PATH, no binaries

### asset_path

### binaries=true

keys.properties and pom.xml needs to line up together

## Uploading ZIP File through Manager UI

Select ZIP file in Snap /target directory

![Manager](https://dl.dropboxusercontent.com/u/3519578/Screenshots/WhVl.png)

### Beware clashing Snap Schemas

# Authenticating with Accounts

```java
package com.snaplogic.account.api;

/**
 * Interface for Account classes which can be used by Snap classes
 *
 * @param <T> as the return type for {@link #connect}
 */
public interface Account<T> {
    /**
     * Defines the properties of the account using the provided {@link PropertyBuilder}
     * The property category is set to {@link PropertyCategory#ACCOUNT} and should not be
     * changed.
     *
     * @param propertyBuilder as the builder
     */
    void defineProperties(final PropertyBuilder propertyBuilder);

    /**
     * Sets the property values on the account
     *
     * @param propertyValues as the property values
     */
    void configure(final PropertyValues propertyValues);

    /**
     * Allows to connect to the endpoint
     *
     * @return an optional value that might be needed to access the session
     * @throws ExecutionException
     */
    T connect() throws ExecutionException;

    /**
     * Allows to disconnect from the endpoint
     *
     * @throws ExecutionException
     */
    void disconnect() throws ExecutionException;
}
```

Snap Accounts allow the re-use of authentication credentials and properties across Snaps. 

They also permit storing, encrypting, and obfuscating sensitive information like passwords, secret keys etc.

## Account Configuration

```java
@General(title = "Example Snap Account")
@Version(snap = 1)
@AccountCategory(type = AccountType.CUSTOM)
public class ExampleAccount implements Account<String> {

    protected static final String USER_ID = "userId";
    protected static final String PASSPHRASE = "passphrase";

    private String userId;
    private String passphrase;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(USER_ID, "User ID", "ID of the user")
                .required()
                // for Enhanced Account Encryption; indicate to the SnapLogic Platform
                // that Medium/High Sensitivity-configured Organizations should encrypt
                // this data
                .sensitivity(SnapProperty.SensitivityLevel.MEDIUM)
                .add();

        propertyBuilder.describe(PASSPHRASE, "Passphrase", "The user's passphrase")
                .required()
                .obfuscate() // masks user's input and sets SensitivityLevel to HIGH
                .add();
    }

    @Override
    public void configure(PropertyValues propertyValues) {
        // Exercise: sanitize and validate
        userId = propertyValues.get(USER_ID);
        passphrase = propertyValues.get(PASSPHRASE);
    }
    
    @Override
    public String connect() throws ExecutionException {
        /*
        Return a String that conforms to this simple hash-token scheme:

        base64(userId + ":" + expirationTimestamp + ":" +
             md5(userId + ":" + expirationTimestamp + ":" passphrase))
         */
        long expiration = DateTime.now().plusDays(1).getMillis();
        byte[] md5;

        try {
            MessageDigest md5MessageDigest = MessageDigest.getInstance("MD5");
            md5 = md5MessageDigest.digest(
                    (getUserId() + ":" + expiration + ":" + getPassphrase()).getBytes(UTF_8));
        } catch (NoSuchAlgorithmException e) {
            throw new ExecutionException(e, "Unable to get MD5 MessageDigest instance")
                    .withResolution("Contact the Snap Developer");
        }

        return encodeBase64String(
                (userId + ":" + expiration + ":" + new String(md5)).getBytes(UTF_8));
    }

    @Override
    public void disconnect() throws ExecutionException {
        // no-op
    }
    
    ...
}
```

At their most basic, Accounts will implement the `Account` interface and provide a Generic Type that is used as the return type of the `Account.connect()` method.

Similarly to Snaps, you may use the `defineProperties()` method to build the Settings tab UI, and the `configure()` method to bind/validate the user's input. 

<aside class="warning">
Unlike Snaps, you cannot <a href="#expression-enabled-properties">expression-enable</a> Account properties.
</aside>

![Create Custom Account](https://dl.dropboxusercontent.com/u/3519578/Screenshots/fNOT.png)

<aside class="notice">
Don't forget to declare the new Account in the Snap POM's <code>&lt;account.classes&gt;</code> element e.g.
<br /><code>&lt;account.classes&gt;com.snaplogic.snaps.accounts.ExampleAccount&lt;/account.classes&gt;</code>
</aside>

### Encrypting and Obfuscating User Input

It can also be particular useful to mark particular properties as containing sensitive information and/or requiring input masking. 

The `sensitivity()` builder method allows setting a `LOW`, `MEDIUM`, or `HIGH` `SensitivityLevel`, which instructs the SnapLogic Platform's [Enhanced Account Encryption](http://doc.snaplogic.com/account-encryption) feature to encrypt the marked account data.

The `obfuscate()` builder method masks user input and automatically sets the property to `SensitivityLevel.HIGH`.

### Connecting the Account

The `Account` interface also defines a `connect()` method (whose return type is determined by the [Generic Type](https://docs.oracle.com/javase/tutorial/java/generics/types.html) provided by the implementing class) and a `disconnect()` method.

For demonstration purposes, we've chosen to implement a simple hash-token scheme for creating an authentication String to be used within a Snap to connect to some service.

## Using the Account in the Snap

```java

```

### Exposing the Account properties in the Snap

To begin with, you may wish to expose the properties of the Account in the Settings of the Snap to which the Account has been associated with.

To do this, have the Account implement the `AccountVariableProvider` interface and its `getAccountVariableValue()` method e.g.

<div class="inline-code>
@General(title = "Example Snap Account")
@Version(snap = 1)
@AccountCategory(type = AccountType.CUSTOM)
public class ExampleAccount implements Account<String>, AccountVariableProvider {
	
	...
	
    @Override
    public Map<String, Object> getAccountVariableValue() {
        return new ExpressionVariableAdapter() {
            @Override
            public Set<Entry<String, Object>> entrySet() {
                return new ImmutableSet.Builder<Entry<String, Object>>()
                        .add(entry(USER_ID, getUserId()))
                        .add(entry(PASSPHRASE, getPassphrase()))
                        .build();
            }
        };
    }
}
</div>

The `$account.userId` and `$account.passphrase` expression values can then be used within the Snap's expression-enabled properties.



### Account Types

In the `ExampleAccount` sample provided, the Account has been marked with the `@AccountCategory` annotation, whose `type` argument value is `AccountType.CUSTOM`    /**

The list of provided Account Types are:

* `NONE`
* `CUSTOM`
* `BASIC_AUTH`
* `SSH`
* `SSL`
* `DATABASE`
* `OAUTH2`
* `OAUTH1`
* `AWS_S3`
* `SAP`

## Validation

ValidatableAccount

ExtendedValidatableAccount

## Multiple Accounts

MultiAccountBinding

# jtest: Snap Unit Testing Framework

TK

## TestFixture

test

### snap, input, output, errors

### inputHeaders, expectedOutputPath, expectedErrorPath

### properties, propertyOverrides

### suggestProperty

### account, accountProperties

### injectorModule

### data files, dataFilesSupplier

# PropertyBuilder Reference

```java
@General(title = "Property Types", purpose = "Demonstrates use of different property types",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Category(snap = SnapCategory.READ)
@Version(snap = 1)
@Inputs(min = 0, max = 0)
@Outputs(min = 1, max = 1)
public class PropertyTypes extends SimpleSnap {
    private static final String PASSWORD_PROP = "password_prop";
    private static final String FILE_BROWSER_PROP = "file_browser_prop";
    private static final String CHILD_FILE_BROWSER_PROP = "child_file_browser_prop";
    private static final String CHILD_PROP = "child_prop";
    private static final String PARENT_PROP = "parent_prop";
    private static final String COLUMN_FILE_BROWSER_PROP = "column_file_browser_prop";
    private static final String COLUMN_PROP = "column_prop";
    private static final String TABLE_PROP = "table_prop";
    private static final String PASSWORD_PROP_VALUE = "abc";

    @Inject
    private DocumentUtility documentUtility;

    @Override
    public void defineProperties(final PropertyBuilder propBuilder) {
        // Password (obfuscate)
        propBuilder.describe(PASSWORD_PROP, PASSWORD_PROP)
                .required()
                .defaultValue(PASSWORD_PROP_VALUE)
                .obfuscate()
                .add();
                
        // File Browsing
        propBuilder.describe(FILE_BROWSER_PROP, FILE_BROWSER_PROP)
                .required()
                .defaultValue(PASSWORD_PROP_VALUE)
                .fileBrowsing()
                .add();

        // Composite Property
        SnapProperty fileBrowsingChild = propBuilder.describe(CHILD_FILE_BROWSER_PROP,
                CHILD_FILE_BROWSER_PROP)
                .required()
                .fileBrowsing()
                .build();
        SnapProperty child = propBuilder.describe(CHILD_PROP,
                CHILD_PROP)
                .required()
                .build();
        propBuilder.describe(PARENT_PROP, PARENT_PROP)
                .type(SnapType.COMPOSITE)
                .required()
                .withEntry(fileBrowsingChild)
                .withEntry(child)
                .add();
                
        // Table Property
        SnapProperty fileBrowsingColumn = propBuilder.describe(COLUMN_FILE_BROWSER_PROP,
                COLUMN_FILE_BROWSER_PROP)
                .required()
                .fileBrowsing()
                .build();
        SnapProperty column = propBuilder.describe(COLUMN_PROP,
                COLUMN_PROP)
                .required()
                .build();
        propBuilder.describe(TABLE_PROP, TABLE_PROP)
                .type(SnapType.TABLE)
                .withEntry(fileBrowsingColumn)
                .withEntry(column)
                .add();
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
    }

    @Override
    public void process(Document document, String inputViewName) {
        Map<String, String> data = new LinkedHashMap<String, String>() {{
            put("key", "value");
        }};
        outputViews.write(documentUtility.newDocument(data), document);
    }
}
```

add10

![Property Values](http://dl.dropboxusercontent.com/u/3519578/Screenshots/qJXX.png)

## SnapType Reference

```java
NUMBER("number", BigDecimal.class),
INTEGER("integer", BigInteger.class),
STRING("string", String.class),
DATETIME("date-time", DateTime.class),
LOCALDATETIME("local-date-time", LocalDateTime.class),
BOOLEAN("boolean", Boolean.class),
DATE("date", LocalDate.class),
TIME("time", LocalTime.class),
BINARY("binary", Byte.class),
TABLE("array", List.class),
ANY("any", Object.class),
COMPOSITE("object", Map.class),
BYTES("bytes", String.class);
```