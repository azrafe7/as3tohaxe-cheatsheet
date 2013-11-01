**NB:** I'm writing this while trying to port a project from **AS3** to **[Haxe 3.0](http://haxe.org/)** __+__ **[OpenFL 1.1.0](http://www.openfl.org/)**.

**NOT COMPLETE: needs updating**

as3tohaxe-cheatsheet
====================

A rough list of what to be aware of when porting as3 code to haxe 3.

Read through this first: 

 - [http://haxe.org/doc/start/flash/as3migration](http://haxe.org/doc/start/flash/as3migration)
 - [http://www.openfl.org/developer/documentation/actionscript-developers/](http://www.openfl.org/developer/documentation/actionscript-developers/)

Some things may be repeated here, but I like to have the most meaningful info in one place.


converters
----------
You can take a look at the converters listed on the sites mentioned above to automate the work and get an approximate port of as3 code to haxe.

They're particularly useful in better understanding how to map as3 to haxe, but many of them (although I've only tested a few), will require you to skim through the generated output afterwards, and adjust/fix parts of it (well... unless your original code was fairly basic).

simple things
-------------
Haxe is very similar to as3 and some things can be easily ported just using a 
_find/replace_ (maybe with some wise use of regexps in your editor of choice):

| Description           | AS3                 | Haxe 3             |
| ----------------------|:-------------------:|:------------------:|
| void                  | :void               | :Void              |
| boolean               | :Boolean            | :Bool              |
| integer               | :int                | :Int               |
| unsigned integer      | :uint               | :UInt              |
| floating p. number    | :Number             | :Float             |
| untyped object        | :Object (or :*)     | :Dynamic           |

arrays/vectors
------------
Other things _may_ take some more attention:

 - Untyped array:

 | AS3                             | Haxe 3                        |
 | ------------------------------- |:-----------------------------:|
 | :Array                          | :Array<Dynamic>               |

 - Typed array:

 | AS3                             | Haxe 3                        |
 | ------------------------------- |:-----------------------------:|
 | :Vector.<T>                     | :Array<T>                     |

 - Untyped dictionary:

 | AS3                             | Haxe 3                        |
 | ------------------------------- |:-----------------------------:|
 | :Dictionary                     | :Map<Dynamic, Dynamic>        |

- Typed dictionary:

 | AS3                             | Haxe 3                        |
 | ------------------------------- |:-----------------------------:|
 | :Dictionary                     | :Map<K, T>                    |


loops
-------------

In Haxe the **for** loops are quite different. They use `iterators` and work much like a **for each** construct in AS3, so you have to take particular attention when working with them (it's also a reason why most conversion tools automatically convert them to `while` loops).

Specifically 
In AS3:
```AS3
	
	for (
```

In Haxe 3:

 - `package` doesn't expect a `{`, just end the line with `;`
 - `class` doesn't need (/WANT) a `public` accessor (just delete it)
 - the constructor is called `new(...)` and not as the name of the class:
`AS3`
	...
	public class Entity {
		puclic function Entity(...) { }
	...
	}

vs.

Haxe
	...
	class Entity {
		puclic function new(...) { }
	...
	}


 - be sure to call a `super(...)` constructor in your extend classes
 - if you extend a class you need a constructor: be sure to add a super(...) to your

rest parameters
---------------


what to avoid (for performance reasons)
-------------------------------------
