**NB:** I'm writing this while porting a project from **AS3** to **[Haxe 3.0](http://haxe.org/)** __+__ **[OpenFL 1.1.0](http://www.openfl.org/)**.

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

arrays/vectors and dictionaries
------------

 - Untyped array:

 | AS3                             | Haxe 3                        |
 | ------------------------------- |:-----------------------------:|
 | :Array                          | :Array\<Dynamic>               |
 | :Vector.<*>                     | :Array\<Dynamic>               |

 - Typed array:

 | AS3                             | Haxe 3                        |
 | ------------------------------- |:-----------------------------:|
 | :Vector.<T>                     | :Array\<T>                     |

[`Array`](http://haxe.org/api/array)s in haxe can't take an optional parameter to specify the initial size.

 - Dictionary:

In Haxe you can use the [`Map`](http://haxe.org/api/map) class, specifying what types you need as _Keys_ and _Values_ respectively. Ex.:

```as3
    var strMap:Map<String, Int> = new Map<String, Int>();
    var dynaMap:Map<Dynamic, Dynamic> = new Map<Dynamic, Dynamic>();
   
    strMap["beast"] = 666;
    trace(strMap.get("beast"));  // 666
```
and access them with `[]` notation or via `get()` and `set()`.

constants
---------
There's no `const` keyword in Haxe, but you can make a constant by declaring a variable as `static inline` (which will tell the compiler that whenever he finds a reference to that var it must substitute it directly with the _literal_ specified value).

AS3:

```as3
	public const PI:Number = 3.14;
```

Haxe:

```as3
    public static inline var PI:Float = 3.14;
```

for loops
-------------

In Haxe the **for** loops are quite different. They use [`Iterator`](http://haxe.org/ref/iterators)s and work much like a **for each** construct in AS3, so you have to take particular attention when working with them (it's also a reason why most conversion tools replace them with `while` loops).

Specifically 

AS3:
```as3
	for (var i:int = 0; i < 10; i++)  // loop from 0 to 9
```

Haxe:
```as3
	for (i in 0...10)  // loop from 0 to 9
```

Things to note here:

 - the upper bound is not included
 - it works with any [`Iterable`](http://haxe.org/api/iterable) type (f.e. Array, List, etc.)
 - you cannot specify a step (like i += 2)
 - the iterator (`i` in the example) lives only within the cycle scope, you cannot access it outside the loop

Highlighting the latter statement:

```as3
    var array:Array<String> = ["one", "two", "three"];
    var item:String = "empty";

    for (item in array) {
        trace(item);
    }

    trace(item);  // "empty"
```

Count down (workaround):

```as3
   for (i in -20...-10) trace(-i);
```

Loop through keys of a `Map` object:

```as3
    for (key in map.keys()) trace(map[key]);
```

Package/Class/Constructor
-------------------------

AS3:

```as3
   package com.engine {

	   public class Entity extends GameObject {
	       
           public function Entity(...) {
               super(...);
               ...
           }
	   }
   }
```

Haxe:

```as3
   package com.engine;                // no curly braces

   class Entity extends GameObject {  // no public
       
       public function new(...) {     // new instead of Entity
           super(...);
           ...
       }
   }
```



other gotchas/notes
-------------------
 - haxe has [`enum`](http://haxe.org/ref/enums)s
 - no `protected` or `internal` access keywords in haxe (use [`typedef`](http://haxe.org/ref/type_advanced#typedef)s, [interfaces](http://haxe.org/ref/oop#interfaces) - or maybe [`@:access`/`@:allow`](http://haxe.org/manual/acl#allowing-access) for extreme cases)
 - `super()` is mandatory in sub classes
 - parentheses when calling constructor with no parameters are mandatory (`var p = new Point;` is invalid - must be `var p = new Point();`)
 - `Float` is not automatically cast to `Int` (use `Std.int()`)
 - no `toFixed()` or `toString()` for number types (but there's `Std.string()`)
 - no automatic conversion to `Bool` (you have to specifically check for a condition - as in `if (entity.count != 0) ... `)
 - again: conditions must be explicit (`if (entity.count != null) ... `)
 - class names (/type names) must begin with an uppercase letter
 - class members initialization only take _compile-time constant values_ (more on this in snippets later)
 - always initialize your variables, especially if you plan to compile for non as3 targets
 - no `indexOf()` on `Array` (there's [`Lambda.indexOf()`](http://haxe.org/doc/cross/lambda) but...)
 - inlined functions must have one and only one `return` point (and cannot be overridden in sub classes)
 - no `Function` type (use `String -> Int` construct)
 - `Reflect.makeVarArgs()` to simulate rest parameters
 - use [`Type`](http://haxe.org/api/type) and [`Reflect`](http://haxe.org/api/reflect) static classes to get info about classes/objects
 - `Map` can't be indexed by `Class` (but you can index by `String` and use `Type.getClassName()`)
 - `haxe.ds` package has a bunch of useful data structures

what to avoid - or do (to improve performance)
---------------------------------------
 - use `Dynamic` as few as possible (type your variables - or again use `typedef`s or `Dynamic<Entity>` construct)
 - use `Int` instead of `UInt` whenever possible
 - try to avoid `Lambda.indexOf()` (seems to leak memory for the flash target)
 - try to avoid `Reflect.getProperty()` (leaks `ReferenceError` objects for the flash target - apparently due to the implementation with `try`/`catch`)
 - inline short functions that get used a lot (and that don't call other functions)

snippets
--------

**Clear an array (multi-target):**
```as3
	public static inline function removeAll(array:Array<Dynamic>):Void
	{
	#if !(cpp || php)
		untyped array.length = 0;
	#else
		array.splice(0, array.length);
	#end
	}
```

**Static initialization (simple method):**
```as3
    class Utils {
        
        private static inline var _inited:Bool = initStaticVars();
        
        public static var lookupTable:Array<Float>;
        public static inline var VERSION:String = "0.1";
        public static var PI:Float;
        
        private static inline function initStaticVars():Bool {
            lookupTable = new Array<Float>();
            ...
            PI = Math.PI;
            
            return true;
        }
        
        ...
    }
```

**Function with rest parameters (via Dynamic property):**
```as3
    public var sum(get, null):Dynamic;
    private function get_sum():Dynamic {
        return Reflect.makeVarArgs(_sum);
    }
    
    private function _sum(args:Array<Dynamic>):Dynamic {
        var res:Float = 0;
        for (i in args) res += i;
        return res;
    }

    trace(sum(1, 2, 3));  // 6
    
    /*
    Equivalent to this AS3 code:
    
	public function sum(...args):Number 
	{
		var res:Number = 0;
		for each (var i:Number in args) res += i;
		return res;
	}
    */
```
