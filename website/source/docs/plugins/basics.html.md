---
layout: "docs"
page_title: "Plugin Basics"
sidebar_current: "docs-plugins-basics"
description: |-
  This page documents the basics of how the plugin system in Vault works, and how to setup a basic development environment for plugin development if you're writing a Vault plugin.
---

# Plugin Basics

This page documents the basics of how the plugin system in Vault
works, and how to setup a basic development environment for plugin development
if you're writing a Vault plugin.

~> **Advanced topic!** Plugin development is a highly advanced
topic in Vault, and is not required knowledge for day-to-day usage.
If you don't plan on writing any plugins, we recommend not reading
this section of the documentation.

## How it Works

The plugin system for Vault is based on multi-process RPC. Every
provider, provisioner, etc. in Vault is actually a separate compiled
binary. You can see this when you download Vault: the Vault package
contains multiple binaries.

Vault executes these binaries in a certain way and uses Unix domain
sockets or network sockets to perform RPC with the plugins.

If you try to execute a plugin directly, an error will be shown:

```
$ vault-provider-aws
This binary is a Vault plugin. These are not meant to be
executed directly. Please execute `vault`, which will load
any plugins automatically.
```

The code within the binaries must adhere to certain interfaces.
The network communication and RPC is handled automatically by higher-level
Vault libraries. The exact interface to implement is documented
in its respective documentation section.

## Installing a Plugin

To install a plugin, put the binary somewhere on your filesystem, then
configure Vault to be able to find it. The configuration where plugins
are defined is `~/.vaultrc` for Unix-like systems and
`%APPDATA%/vault.rc` for Windows.

An example that configures a new provider is shown below:

```javascript
providers {
  privatecloud = "/path/to/privatecloud"
}
```

The key `privatecloud` is the _prefix_ of the resources for that provider.
For example, if there is `privatecloud_instance` resource, then the above
configuration would work. The value is the name of the executable. This
can be a full path. If it isn't a full path, the executable will be looked
up on the `PATH`.

## Developing a Plugin

Developing a plugin is simple. The only knowledge necessary to write
a plugin is basic command-line skills and basic knowledge of the
[Go programming language](http://golang.org).

-> **Note:** A common pitfall is not properly setting up a
<code>$GOPATH</code>. This can lead to strange errors. You can read more about
this [here](https://golang.org/doc/code.html) to familiarize
yourself.

Create a new Go project somewhere in your `$GOPATH`. If you're a
GitHub user, we recommend creating the project in the directory
`$GOPATH/src/github.com/USERNAME/vault-NAME`, where `USERNAME`
is your GitHub username and `NAME` is the name of the plugin you're
developing. This structure is what Go expects and simplifies things down
the road.

With the directory made, create a `main.go` file. This project will
be a binary so the package is "main":

```go
package main

import (
  "github.com/hashicorp/vault/plugin"
)

func main() {
  plugin.Serve(new(MyPlugin))
}
```

And that's basically it! You'll have to change the argument given to
`plugin.Serve` to be your actual plugin, but that is the only change
you'll have to make. The argument should be a structure implementing
one of the plugin interfaces (depending on what sort of plugin
you're creating).

Vault plugins must follow a very specific naming convention of
`vault-TYPE-NAME`. For example, `vault-provider-aws`, which
tells Vault that the plugin is a provider that can be referenced
as "aws".
