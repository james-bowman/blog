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

In other languages, plugins are well supported and relatively easy to implement.  In Java for example, concrete implementations of an interface can be dynamically loaded and instantiated by reflection.  Different implementations can be 'plugged' in simply by specifying the fully qualified name of the implementing class during reflection and ensuring it is on the CLASSPATH.  

Taking this one step further, Java has a set of conventions for creating and loading plug-ins called [SPI (Service Provider Interfaces)](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html).  This is used by Java itself to dynamically load XML library implementations.  Using the SPI, it is possible to simply place a new implementation on the CLASSPATH and it will be automatically picked up and used by the application.

## Support in Go

### Statically linked code

### Composition over extension

## Examples

### Out of process calls

### in the Go language

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
