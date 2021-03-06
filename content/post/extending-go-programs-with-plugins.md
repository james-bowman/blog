+++
categories = [ "Development" ]
date = "2015-07-14T07:47:59+01:00"
draft = false
tags = [ "development", "go", "golang", "java", "plugins", "RPC", "scripting" ]
title = "Extending Go programs with plugins"
description = "A look at some of the options for extending and customising Golang programs with plug-ins"
aliases = ["/post/extending-go-programs-with-plug-ins/"]
author = "James Bowman"
image = "/post/Gopher-plugin.png"
imageURL = "https://github.com/ashleymcnamara/gophers/blob/master/GOPHER_AVATARS.jpg"
imageTitle = "'GOPHER_AVATARS' with added plugs"
imageCreator = "Ashley McNamara"
imageLicenceURL = "http://creativecommons.org/licenses/by-nc-sa/4.0/"
imageLicenceName = "Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International license"
+++

I am really enjoying programming in Google's [Go] language (Golang for search engines) but very occasionally come across things that aren't really possible, or considered idiomatic, in Go.  Go is a very opinionated language which is a good thing as it keeps the language and tool chain very simple but also means if you need to do something unusual, it can sometimes feel like you are fighting the language.  One example of this is developing [plugin]s.

[Plugin]s are a way of allowing third parties to extend or customise the behaviour of an existing piece of software.  This can be useful in a variety of contexts but, most recently, whilst I was working on a [chat bot][talbot] I wanted to use plugins to allow people to easily extend the bot's capabilities without having to change the existing [code][talbot].

## Support in other languages

In other languages, plugins are well supported and relatively easy to implement.  In Java for example, concrete implementations of an interface can be dynamically loaded and instantiated by reflection as shown in the snippet below.  Different implementations can be 'plugged' in simply by specifying the fully qualified name of the implementing class during reflection and ensuring it is defined on the CLASSPATH.  

``` java
Class clazz = Class.forName(fullyQualifiedClassName);				
Object product = clazz.newInstance();
```

Taking this one step further, Java has a set of conventions for creating and loading plugins called [SPI (Service Provider Interface)](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html).  This is used by Java itself to dynamically load XML library implementations.  Using the SPI, it is possible to simply place a new implementation on the CLASSPATH and it will be automatically picked up and used by the application dynamically at runtime.

## Support in Go

The Go compiler statically links libraries at compilation time which results in a single, fat, executable binary without external dependencies.  This massively simplifies deployment across environments but means that unfortunately it is not possible to load alternative implementations from dynamically linked libraries at runtime as with the Java SPI approach.

Given the language's constraints around dynamic linking, supporting extensibility through plugins is something a lot of people have attempted in Go and a number of alternative approaches exist including but not limited to:

1. [Out of process communication]({{< ref "#1-out-of-process-communication" >}})
1. [Embedded scripting]({{< ref "#2-embedded-scripting" >}})
1. [Compiled-in extensions]({{< ref "#3-compiled-in-extensions" >}})

Each of these approaches have their own strengths and weaknesses which I will discuss over the next few sections.  

### 1. Out of process communication

This is the approach taken by Hashicorp's [Packer](https://www.packer.io/docs/extend/plugins.html) tool.  Essentially the 'plugin' runs as a separate process and then the existing application communicates with the Plug-in via RPC (Remote Procedure Calls).  This means that the plugin does not need to be compiled into the existing application or statically linked but can be integrated dynamically at runtime.  Other examples of this approach are [Pie](http://npf.io/2015/05/pie/), [Pingo](https://github.com/dullgiulio/pingo) and [Drone](https://github.com/drone/drone).  Pie and Drone are interesting because instead of using RPC over TCP/IP the communication with the plugin process occurs via stdin and stdout.  Drone even takes the separation between application and plugin further by having the plugin process running within its own discrete Docker container.  Some of the pros and cons of this approach are as follows:

Pros:

- Plug-ins are technology independent i.e. the plugin does not necessarily need to be written in the same language as the existing application (although they will need to share a common protocol for RPC).
- Process isolation - the plugin and existing application are running in separate processes so if the plugin crashes, the impact on the existing application is limited.

Cons:

- Added complexity through additional runtime processes, binaries to be deployed and dependencies between them.
- Relatively higher latency for out of process (RPC) communication

### 2. Embedded scripting

This approach uses an embedded scripting engine to execute scripts in process.  As the scripts are interpreted rather than compiled, their implementation can be extended and changed without modifying the existing application.  Some examples include [Agora](https://github.com/PuerkitoBio/agora), [Anko](https://github.com/mattn/anko), [Otto](https://github.com/robertkrimen/otto) and [Go-Lua](https://github.com/Shopify/go-lua).  The syntax of both Agora and Anko bear a striking resemblence to Go whilst Otto and Go-Lua provide bindings for JavaScript and Lua script respectively.

Pros:

- Dynamic invocation at runtime.
- Relatively simple - no additional runtime processes.
- Language bindings allow in process communication and data passing between existing application and scripts

Cons:

- Usually require knowledge of another language e.g. Lua, JavaScript.
- Possible reduced performance of interpreted code relative to compiled code.

### 3. Compiled-in extensions

This is the approach that the Go language uses internally to support extensions e.g. [image formats](http://blog.golang.org/go-image-package#TOC_5.) and [database drivers](http://golang.org/pkg/database/sql/).  Essentially, third parties develop their own plugins and compile the existing application themselves along with the source for their plugin without needing to modify the code for the existing application.  This was the approach I ended up adopting for Talbot and will discuss it in more depth in next sections.

Pros:

- Relatively simple - no additional runtime processes, extra deployment artifacts or dependencies.
- Simple deployment - one fat binary deployable.
- All code written in Go.

Cons:

- Statically linked - requires third parties to compile existing application themselves.
- Behaviour cannot be changed/extended without re-compiling the whole application.

## Compiled-in extensions - In more depth

I ended up choosing the compiled-in extensions approach for [my chat bot][talbot]. The compiled-in approach means that third-parties can clone or fork the [repository][talbot], add their plugins/extensions into the appropriate package and then compile the application (with the plugins included).  This approach relies on a couple of Go language features which are worth considering in more detail.

### The init() function

Whenever a package is imported, any functions called `init()` in that package will be implicitly executed.  If there are more than one, they will all be executed, although Go provides no guarantees as to the order in which they will be called. 

``` go
package "mypackage"

func init() {
	
}
```

### Anonymous (underscore) imports 

The Go compiler will fail to compile code containing any unused imports in code.  Personally, I find this helpful as it forces me to clear up any imports that are redundant or no longer needed.  However, it does mean that in order for an import to be valid, a member of the package must be explicitly referenced and used somewhere in the code.  

Unfortunately, we need to import a package in order for the Go compiler to execute any `init()` functions in that package, but often, we don't actually want to explicitly call any of the code within the package.  Luckily, the Go language includes a construct to cater for just this scenario - anonymous imports.  Anonymous imports are a way of telling the compiler to import a package that will not be explicitly referenced or used in the code.  The syntax is shown below - note the underscore between the import keyword and the package name.

``` go

import _ "mypackage"

```

### Putting it all together

A simple plugin architecture using the compile-in extension approach could be established using 3 packages as follow:

    repo/
    repo/plugins/
    repo/plugins/third-party/

With this structure, the `repo/` folder would be the `main` package for the application.  The `repo/plugins/` package would contain generic code to support third party plugins and the `repo/plugins/third-party/` package would be where the actual plugins would be placed by third parties.

In one of the files inside `repo/` there should be an anonymous import for `repo/plugins/third-party` as shown in the code snippet below:

``` go
package "main"

import "repo/plugins"
import _ "repo/plugins/third-party"

```

The `repo/plugins/` package should contain a file containing the following code:

``` go
package "repo/plugins"

// define type for plugin - in this case it is just a function but it could be an interface
type plugInFunc func(param int) string

// map to store registered plugins
var registry map[string]plugInFunc

// Register plugins
func Register(name string, plugIn plugInFunc) {
	registry[name] = plugIn
}
```

This code essentially defines a type that plugins must conform to.  This could be an interface but in this case I have defined a function type.  The `registry` is simply a map to store plugins registered using the exported `Register(string, plugInFunc)` function.

Finally new plugins should be placed within the `repo/plugins/third-party/` package and should be structured as in the following code snippet:

``` go
package "repo/plugins/third-party"

import "repo/plugins"

func init() {
	plugins.Register("myPlugin", myFunction)
}

func myFunction(param int) string {
	// plugin functionality
}
```

[Go]: https://golang.org/
[plugin]: https://en.wikipedia.org/wiki/Plug-in_(computing)
[Talbot]: https://github.com/james-bowman/talbot
