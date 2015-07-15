+++
categories = [ "Development" ]
date = "2015-07-14T07:47:59+01:00"
draft = true
tags = [ "development", "go", "go-lang", "java", "plugins" ]
title = "Go plug-ins"

+++

I am really enjoying developing in [Go] but very occasionally come across things that aren't really possible, or considered idiomatic, in Go.  Go is a very opinionated language which is a good thing as it keeps the language and tool chain very simple but also means if you need to do something unusual, it can sometimes feel like you are fighting the language.  One example of this is developing [plug-in]s.

[Plug-in]s are a way of allowing third parties to extend or customise the behaviour of an existing piece of software.  This can be useful in a variety of contexts but, most recently, whilst I was working on [Talbot] I wanted to use plug-ins to allow people to easily extend the bot's capabilities without having to change the existing [Talbot] code.

## Support in other languages

In other languages, plugins are well supported and relatively easy to implement.  In Java for example, concrete implementations of an interface can be dynamically loaded and instantiated by reflection as shown in the snippet below.  Different implementations can be 'plugged' in simply by specifying the fully qualified name of the implementing class during reflection and ensuring it is defined on the CLASSPATH.  

``` java
    Class clazz = Class.forName(fullyQualifiedClassName);				
    Object product = clazz.newInstance();
```

Taking this one step further, Java has a set of conventions for creating and loading plug-ins called [SPI (Service Provider Interface)](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html).  This is used by Java itself to dynamically load XML library implementations.  Using the SPI, it is possible to simply place a new implementation on the CLASSPATH and it will be automatically picked up and used by the application dynamically at runtime.

## Support in Go

The Go compiler statically links libraries at compilation time which results in a single, fat, executable binary without external dependencies.  This massively simplifies deployment across environments but means that unfortunately it is not possible to load alternative implementations from dynamically linked libraries at runtime as with the Java SPI approach.

### Composition over extension



## Examples

Supporting extensibility through plug-ins is something a lot of people have attempted in Go and a number of different approaches exist including but not limited to:

- Out of process RPC calls
- Shelling out to other processes
- Embedded scripting
- Compiled-in extensions.

Each of these approaches have their own strengths and weaknesses which I will discuss over the next few sections.  

### Out of process RPC calls

This is the approach taken by Hasicorp's [Packer](https://www.packer.io/docs/extend/plugins.html) tool.  Essentially the 'plug-in' runs as a separate process and then the existing application communicates with the Plug-in via RPC (Remote Procedure Calls).  This means that the plug-in does not need to be compiled into the existing application or statically linked but can be integrated dynamically at runtime.  Other examples of this approach are [Pie](http://npf.io/2015/05/pie/) and [Pingo](https://github.com/dullgiulio/pingo).  Pie is interesting because instead of using RPC over TCP/IP it communicates with the plug-in process via stdin and stdout.  Some of the pros and cons of this approach are as follows:

#### Pros

- Truly dynamic integration at runtime
- Plug-ins are technology independent i.e. the plug-in does not need to be written in the same language as the existing application.
- Process isolation - the plug-in and existing application are running in separate processes so if the plug-in crashes, the impact on the existing application is limited.

#### cons

- Added complexity through additional runtime processes, binaries to be deployed and dependencies between them.
- Relatively higher latency for out of process (RPC) communication

### Shelling out to other processes



### Embedded scripting


### Compiled-in extensions



## Solution

### an interface or function type

### a plugin registry

### anonymous imports 

``` go
import _ "plugins"
```

### the init() function

``` go
package "plugins"

func init() {
	
}
```



[Go]: https://golang.org/
[plug-in]: https://en.wikipedia.org/wiki/Plug-in_(computing)
[Talbot]: https://github.com/james-bowman/talbot
