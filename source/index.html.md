---
title: Snap Development Documentation

language_tabs:

toc_footers:
  - <a href='http://www.snaplogic.com'>SnapLogic</a>

includes:

search: true
---

# Introduction

This documentation guides a developer through the steps necessary to develop **Snaps** for the [SnapLogic Elastic Integration Platform](https://www.snaplogic.com/).

## Snaps and Snap Packs

SnapLogic [Snaps](https://www.snaplogic.com/snaps) are modular collections of integration components built for a specific application or data source. Snaps shield both business users and developers from much of the complexity of the underlying application, data model, and service.

![Transform Snap Pack](http://dl.dropboxusercontent.com/u/3519578/Screenshots/WmuJ.png)

Snap Packs logically organize Snaps and are the deployable unit when adding/modifying Snaps to the SnapLogic Elastic Integration Platform. Snaps may be related by functionality or share common code. A Snap Pack may contain a single Snap or multiple Snaps.</img>

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

# Snaplex Installation

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
> cd ~
> wget https://s3.amazonaws.com/elastic.snaplogic.com/snaplogic-sidekick...rpm
> /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
> brew install rpm2cpio
> rpm2cpio.pl snaplogic-sidekick...rpm | cpio -idvm
```

> If successful, the output of the `cpio` command will list the Snaplex files populating the newly created `~/opt/snaplogic` directory, for example:

```shell
> rpm2cpio.pl snaplogic-sidekick-4.mrc244-x86_64.rpm | cpio -idvm
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

## Configuration

```shell
> cat ~/opt/snaplogic/etc/keys.properties
cc.admin_uri = https://elastic.snaplogic.com:443
cc.username = robin@acme.com
cc.api_key = JDAFBDAD....9dAg43
cc.ssl_verify = True
```

For approved customers, SnapLogic Support will provide the necessary instructions for authentication and configuration of the Snaplex.

```shell
> cat ~/opt/snaplogic/etc/global.properties
jcc.location = sidekick
jcc.sldb_uri = https://elastic.snaplogic.com:443
...
# The subscriber ID assigned for the customer. This will be available after
# the customer account is created.
jcc.subscriber_id = acme

# This is the runtime environment in which the Snaplex should be started.
jcc.environment = devsnaplex
```

The `~/opt/snaplogic/etc/keys.properties` file contains the information required for authenticating to the SnapLogic Elastic Integration Platform APIs to run the Snaplex and deploy the developed Snaps.

The `~/opt/snaplogic/etc/global.properties` file contains the information required for general configuration of the Snaplex.

## Running

```shell
> cd ~/opt/snaplogic/run/lib
> java -jar jcc.war jcc
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

# Snap Development

```java
@General(docLink = "http://www.docs.com/mysnap", title = "Snap Title", purpose = "Description")
@Inputs(min = 1, max = 1)
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 0, max = 1, offers = {ViewType.DOCUMENT})
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

`@General`: this that other

 @General
title and purpose for the snap.  The title is the name of the snap, as it will appear in the snap catalog.  The purpose is the text which pops up when the user hovers over the snap in the catalog.
@Inputs
defines this snap as supporting no Input Views
@Outputs
defines exactly one OutputView which will offer JSON documents
@Errors
defines an optional Error View which will offer JSON documents
@Version
version number for the snap (not to be confused with its build number)
@Accounts
allows us to define what account type(s) this snap supports
@Category
determines the icon displayed on the snap.


# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

