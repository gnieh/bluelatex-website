---
layout: default
title: Configuration
---

\BlueLaTeX Configuration
========================

* Contents
{:toc}

Configuration File Layout
-------------------------

The \BlueLaTeX server is organized in a bunch of [OSGi](http://www.osgi.org/Main/HomePage) bundles making it really modular.
The configuration of each bundle is made in its own directory, with respect to this modular architecture.

The directory layout for configuration files is organized under one directory for each bundle.
The directory in which the configuration of a bundle lies is simply the bundle identifier.
Standard bundles directories are:

 - `org.gnieh.blue.common` the common utility bundle (http server, database configuration, ...),
 - `org.gnieh.blue.core` the core \BlueLaTeX features (user management, paper management, ...),
 - `org.gnieh.blue.sync` the standard \BlueLaTeX synchronization server,
 - `org.gnieh.blue.compile` the \LaTeX compilation bundle,
 - `org.gnieh.blue.web` the standard web client.

The configuration file format used is [HOCON](https://github.com/typesafehub/config/blob/master/HOCON.md).
We detail below the default configuration of each standard bundle. Any configuration key may be overridden by placing a file named `application.conf` in the bundle configuration directory.

Common Bundle `org.gnieh.blue.common`
-------------------------------------

This bundle contains all the common services and features.
This is the biggest piece of configuration available as it provides a lot of configuration possibilities.
Theses configuration keys are listed and (hopefully) explained below with their default values.

```scala
http {

  # the port the http server listens to
  port = 8080

  # the host to bind the http server to
  host = localhost

}

recaptcha {
  # no ReCaptcha key by default
  # the private key
  # private-key = "my super private key"
  # the public key (for clients)
  # public-key = "my awesome public key"
}

# \BlueLaTeX service configuration. This section contains the generic
# configuration keys of the service. Each bundle can use these configuration keys
blue {

  # the base URL hosting the \BlueLaTeX service.
  # this is used in emails sent to users or to generate links to the service.
  base-url = "http://${http.host}:${http.port}/"

  # the kind of registration confirmation process, one of:
  #  - `email-confirmation` set a default password and send a confirmation email
  #    asking the newly registered user to set a new password
  #  - `email-summary` the user chooses his password upon registration and
  #    then receives an information email with summary of registration
  #   `no-email` the user chooses his password upon registration and receives no
  #   email
  registration-confirmation = "no-email"

  # the base directory where the working data are saved
  data = "/var/lib/blue"

  # the session timeout
  session-timeout = 15 minutes

  # Paper specific keys
  paper {

    # the directory where the paper data are saved
    directory = ${blue.data}/papers

    # the directory containing the cls files
    classes = ${blue.data}/classes

  }

  # Template specific keys
  template {

    # the base directory where the templates are located
    directory = ${blue.data}/templates

  }

  # Rest API specific configuration keys
  api {

    # the prefix of this \BlueLaTeX installation
    # if it is installed at the root of the domain, then leave this string empty
    path-prefix = "api"

  }

  # All configuration keys related to the permission model
  # There are 5 predefined roles in \BlueLaTeX:
  #  - Author
  #  - Reviewer
  #  - Guest
  #  - Other
  #  - Anonymous
  # However permissions assigned to each role can be tuned for each paper.
  # The list of valid permissions is:
  #  - `edit` allows for collaboratively editing the paper,
  #  - `compile` allows for compiling the paper,
  #  - `configure` allows for configuring the paper settings like compiler settings for example (if any),
  #  - `publish` allows for using the publication feature (like creating a tag or sending to arXive),
  #  - `download` allows for downloading the source files of the paper,
  #  - `read` allows for reading the paper source but not for editing it,
  #  - `view` allows for viewing the rendered paper but not the source,
  #  - `comment` allows for commenting on a paper (either the rendered view or the source),
  #  - `chat` allows for interacting with other connected users in the chat,
  #  - `fork` allows for creating a derivative of this paper,
  #  - `change-phase` allows for changing the current phase of the paper.
  permissions {

    # Defines the basic permissions to assign to a private paper, this can be changed later
    # in any existing paper. If no permission entity exists, these permission will be
    # taken whenever a permission check is performed.
    private-defaults {
      # authors can do anything
      author = [ edit, compile, configure, publish, download, read, view, comment, chat, fork, change-phase ]
      # reviewers may read and comment
      reviewer = [ compile, view, comment ]
      # guests may only read
      guest = [ compile, view ]
      # other people cannot do anything
      other = []
      # anonymous people cannot do anything either
      anonymous = []
    }

    # Defines the basic permissions to assign to a public paper, this can be changed later
    # in any existing paper. If no permission entity exists, these permission will be
    # taken whenever a permission check is performed.
    public-defaults {
      # authors, reviewers, guest, other and anonymous can do anything
      author = [ edit, compile, configure, publish, download, read, view, comment, chat, fork, change-phase ]
      reviewer = ${blue.permissions.public-defaults.author}
      guests = ${blue.permissions.public-defaults.author}
      other = ${blue.permissions.public-defaults.author}
      anonymous = ${blue.permissions.public-defaults.author}
    }

  }

}

# The email configuration. For possible configuration keys,
# see https://javamail.java.net/nonav/docs/api/
mail {

  smtp {

    # the smtp server host name
    host = localhost

    # the smtp server port
    port = 25

    # additional configuration keys specific to \BlueLaTeX
    # when a user and a password are required by the smtp server
    # user = username
    # password = my_password

  }

  # the address from which users receive emails
  from = "no-reply@bluelatex.gnieh.org"

}

# CouchDB connectivity configuration keys
couch {

  # the host name where the couchdb instance runs
  hostname = "localhost"

  # the couchdb server port
  port = 5984

  # should we connect to couchdb via ssl
  ssl = false

  # the timeout when waiting for the server response
  timeout = 20 seconds

  # the couchdb database administrator name
  admin-name = "admin"

  # the couchdb database administrator password
  admin-password = "admin"

  # define the standard database names used by the core \BlueLaTeX service.
  # other bundles can use this configuration as well
  database {

    # the name of the database in which the \BlueLaTeX specific user data are saved
    blue_users = "blue_users"

    # the name of the database in which the papers data are saved
    blue_papers = "blue_papers"

  }

  # the design specific keys
  design {

    # the directory containing the design documents
    dir = ${blue.data}/designs

  }

  # user specific configuration
  user {

    # the default roles assigned to a user when created */
    roles = [ "blue_user" ]

    # the password reset token validity
    token-validity = 1 day

  }

}
```

Core Bundle `org.gnieh.blue.core`
---------------------------------

The core bundle is responsible for handling core features of \BlueLaTeX, as its name states quite clearly.
For the moment, there is no specific configuration for this bundle.

Compilation Bundle `org.gnieh.blue.compile`
------------------------------------------

This bundle contains the \LaTeX compilation features. It provides standard system compiler to the user.

```
# The *TeX system process configuration keys
tex {

  # compilation processes are launched in a pool of available processes.
  # the size of the available process pool is configurable here.

  # the minimum amount of available processes in the process pool
  min-process = 5

  # the maximum amount of *TeX processes that can run in parallel
  max-process = 100

}

# The default compiler configuration keys
compiler {

  # the default compiler used when creating \LaTeX paper
  default = pdflatex

  # whther SyncTeX data are generated by default
  synctex = true

  # the timeout after which compilation process for the paper must be aborted
  timeout = 30 seconds

  # interval of time to wait between to background runs of the compiler
  interval = 15 seconds

}
```

Synchronization Bundle `org.gnieh.blue.sync`
--------------------------------------------

This bundle provides the real-time synchronization feature of \BlueLaTeX.
For the moment, there is no specific configuration for this bundle.

Web Client Bundle `org.gnieh.blue.web`
--------------------------------------

This bundle provides the default web client. It is mostly a web frontend written in javascript.
The configuration available in this bundle is listed below.

```
blue {

  client {

    # prefix where the web application is located
    # an empty prefix means that the web client is accessed at the root of the
    # host.
    path-prefix = ""

    # if the string is empty, the link won't be rendered
    issues-url = "https://github.com/gnieh/bluelatex/issues/new"

    # if the string is empty, the link won't be rendered
    clone-url = "https://github.com/gnieh/bluelatex/"

  }

}
```
