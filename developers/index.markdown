---
layout: default
title: Developer Corner
---

# Setting Up your Development Environment
{:.no_toc}

* Contents
{:toc}

## Requirements

We recommend to use a Linux-based OS as a developer environment (e.g. Debian), without any HTTP proxy.

You need:

* git (`aptitude install git`)
* java (`aptitude install openjdk-7-jdk`)
* scala (`aptitude install scala`)
* sbt ([help](http://www.scala-sbt.org/0.13/tutorial/Setup.html))
* jsvc (`aptitude install jsvc`)
* couchdb (`aptitude install couchdb`)

Couchdb should be automatically started, if not start it (`service couchdb start`). By default, couchdb runs in admin party (no password) on port 5984.

You will also need a mail server (SMTP server), listening on port 12525. For development, you can use the one bundled in python:

```shell
$ python -m smtpd -n -c DebuggingServer localhost:12525
```

This will launch an instance of the python's SMTP debugging server on port 12525 and will display all received messages on the console.


## Compiling \BlueLaTeX

To start working on \BlueLaTeX, you will need to execute these commands:

```shell
$ git clone git@github.com:gnieh/bluelatex.git
$ cd bluelatex
$ sbt compile
```

This will download all necessary dependencies, compile the entire project and should simply work.

## Launching Test Server

The sbt configuration of \BlueLaTeX allows you to start a test instance of the server. This test instance is launched as a daemon on Unix systems using [jsvc](http://commons.apache.org/proper/commons-daemon/jsvc.html) (this should be possible on windows too, but it is not tested yet, any help is welcome).

First, create a file called `build.sbt` and add this line in it:

```scala
launchExe := "/usr/bin/jsvc"
```
Or whatever path is returned by `which jsvc`

If you have your own running couchdb instance, we also recommend to add:

You just need to add these lines in your build.sbt file:

```scala
couchdb := None

couchPort := 5984
```

Then, to start the server, type:

```shell
$ sbt blueStart
```

You can then access the web application by going to [http://localhost:18080/](http://localhost:18080/).

To stop the test server just type:

```shell
$ sbt blueStop
```

## Customize your Building Environment

It is possible to override default settings by creating a local `build.sbt` file at the root of the project.


**Note** it is important to have a [blank line between all commands in this sbt file](http://www.scala-sbt.org/release/docs/Getting-Started/Basic-Def.html#how-build-sbt-defines-settings), forgetting it may lead to some strange behaviors and headaches.

All customization are not described here, and is behind scope of this wiki page, as it would lead to have a sbt tutorial. But you can basically override any settings key in this file.

Every time you change a setting, you must either execute the `reload` command in the sbt console or restart it.

## Troubleshooting

### Jsvc won't start

Starting at some version of jsvc, it is required to invoke the command with full path and an explicit working directory. If you are in this case and see the following error message:

```
JSVC re-exec requires execution with an absolute or relative path
```

you will have to add this line in your `build.sbt`

```scala
launchExe := "/usr/bin/jsvc"
```

Or whatever path is returned by `which jsvc`

### Unable to connect to the server 

If the `blueStart` command was correctly executed but you cannot connect to the server. Have a look at content of file `/tmp/bluelatex.err`. If you see something like the following lines in it:

```
No config.properties found.
```
or

```
ERROR: Unable to create cache directory: ./felix-cache
ERROR: Error creating bundle cache. (java.lang.RuntimeException: Unable to create cache directory.)
```

It means that you have a version of jsvc that requires setting explicitly the working directory.
Once again this can be solved by overriding the setting key configuring the start options in `build.sbt`:

```scala
startOptions <<= (startOptions, bluePack) map { (opt, pack) => "-cwd" :: pack.getCanonicalPath :: opt }
```


This line is a bit more complex than the previous ones but just says: the value of the `startOptions` setting key is whatever its value was **plus** the option `-cwd` followed by the full path of where the application was packaged.
This directory contains the configuration etc (by default it is packed in `target/pack`). stop your current \BlueLaTeX instance by typing `blueStop`  in the sbt console, reload the configuration, and restart. It should work.
