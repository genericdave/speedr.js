Speedr.js
====
Improved javascript objects.

* [Motivation](#a)
* [Features](#b)
* [Usage](#c)
	* [Construction](#c1)
	* [Getting, Setting and Inserting](#c2)
	* [Removing](#c3)
	* [Length](#c4)
	* [Iterating](#c5)
	* [Gratuitous Optimization](#c6)
	* [Sorting](#c7)
* [Installation](#d)
	* [Node.js](#d1)
	* [Browser](#d2)
* [Tests and Benchmarks](#e)

<a name='a' />
Motivation
----
Javascript is built on objects.  You can use them to make functions or nifty class-like constructs, but mostly we know them from when we {use: 'them', like: {hash: 'maps'}}.

Javascript objects work well as maps in many cases, but there are big gaps in their implementation when used as such.  For example:

* What if I need sorting?
* Do I really need to loop through and count manually in order to get the number of elements I've defined in an object?
* What If I need *fast* iteration on Node.js or Chrome?  For small objects, speed might not be an issue, but as an object grows in size, iteration speeds can become troublesome.

Speedr addresses these issues and then some by introducing a new Map class.  

<a name='b' />
Features
----
* Blazing fast iteration on V8 platforms like Chrome and Node.js.  Huge speedups over the normal for..in object iteration are common.
* No-overhead .length attribute.  Just like with Array.length, Speedr maps keep track of their lengths internally.  Iteration should only be necessary when you're actually doing something useful with the data.
* Fast sorting using binary search.
* Duplicate or unique keys for sorted maps.
* No external dependencies.

<a name='c' />
Usage
----
Speedr currently introduces three new 'Map' classes:

* Map
	* An unsorted hash map that can only store one value per key (i.e. keys are *unique*).
* SortedMap
	* A hash map that sorts its keys upon insertion.  
* SortedMultiMap
	* Sorted map that allows multiple entries to be inserted under the same key.
	
<a name='c1' />
### Construction
Maps can be constructed in two ways

```javascript
// From a native javascript object.
var map = new speedr.Map({a: 1, b:2});

// Or by passing a series of arrays as [key, value] pairs.
var sortedMap = new speedr.SortedMap(['a', 1], ['b', 2]);

// Construction works the same way for all speedr maps.
var sortedMultiMap = new speedr.SortedMultiMap({a: 1, b: 2});
```

The reason for the array style of construction is that object literal keys are always strings.  i.e. the key in {1: 'a'} is *not* the number 1, but the string '1'.  Thus, if you want to preserve the type of numbered keys, you must use the `new Map([1, 'a'], [2, 'b'])` style of construction rather than passing in an object.

<a name='c2' />
### Getting, Setting and Inserting
```javascript
console.log(map.get('a')); // 1

// Setting follows the same rules as construction.
map.set({a: 3});
console.log(map.get('a')); // 3

// The set function can also create new entries.
map.set([10, 'ten'], [20, 'twenty']);
console.log(map.get(10)); // 'ten'
console.log(map.get(20)); // 'twenty'

// Map and SortedMap both use the set function.
// Since SortedMultiMap doesn't overwrite old keys, it uses the 
// insert function instead.
sortedMultiMap.insert({a: 3, b: 4});

// Since keys in SortedMultiMap are not associated with any one 
// value, there is no get function.  To access entries, we must 
// iterate over them.
sortedMultiMap.each(function(key, val) {
	console.log(key + ' ' + val);
});
// a 1
// a 3
// b 2
// b 4

// Notice that the entries are sorted by key when we iterate and 
// that there are two 'a' entries and two 'b' entries.

// Let's try this with SortedMap:
sortedMap.set(['a', 3], ['b', 4]);
sortedMap.each(function(key, val) {
	console.log(key + ' ' + val);
});
// a 3
// b 4

// As expected, the entries get replaced rather than inserted.
```

<a name='c3' />
### Removing
Todo...

<a name='c4' />
### Length
```javascript
// Maps keep track of their length and expose it as a .length
// attribute.
console.log(map.length);            // 4
console.log(sortedMap.length);      // 2
console.log(sortedMultiMap.length); // 4
```

<a name='c5' />
### Iterating
All maps have a variety of iteration methods based on two main ones: each and iter.

```javascript
// The each function takes a function that accepts key value pairs
map.each(function(key, val) {
	if (key == 'a') { console.log('"a" contains ' + val) }
});
// "a" contains 3

// each optionally takes start, end and step arguments
// (default to start = 0, end = this.length and step = 1)
map.each(function(key, val) {
	console.log(key);
}, 1, map.length, 2);
// b
//20

// The iter function is what each uses internally.  It works as 
// an iterator, returning values based on a numerical index.
for (var i = 0; i < map.length; i++) {
	// iter returns a [key, value] array for each item
	var item = map.iter(i);
	var key = item[0];
	var val = item[1];
	console.log(key);
}
// a
// b
// 10
// 20
```

<a name='c6' />
### Gratuitous Optimization
For those times when you need to cut out all the cruft, you can use the Key/Val versions of the iteration functions in order to only retrieve those components of the entries.

```javascript
map.eachKey(function(key) {
	// This will be slightly faster than a normal each() call
});

for (var i = 0; i < map.length; i++) {
	// Returns the value by itself (not in an array)
	map.iterVal(i)
}
```

<a name='c7' />
### Sorting
Both SortedMap and SortedMultiMap sort their entries by key automatically upon insertion using an optimized binary search.  In order to use sorting effectively, however, it's necessary to be aware of a couple things:

**Important:**
* Sorted maps can only sort using keys that allow comparison operations (<, ==, ===, >).
* Mixing key types can lead to strange behavior.  You should be able to mix integer and floating point numbers without a problem, but mixing strings and numbers is discouraged.  Feel free to experiment, however, as your results may vary.

<a name='d' />
Installation
----
<a name='d1' />
### Node.js
* [Install npm.](http://nodejs.org/#download) (it comes with Node.js)
* Install speedr:
	* In your project directory: `$ npm install speedr`
	* Or as an npm dependency in your package.json file: `"dependencies":{ "speedr":"*" }` followed by `$ npm install`
* Require it: `var speedr = require('speedr');`

<a name='d2' />
### Browser
* Download [speedr.min.js.](https://raw.github.com/genericdave/speedr.js/master/speedr-min.js)
* Include it: `<script src='lib/speedr.min.js'></script>`

<a name='e' />
Tests and Benchmarks
----
* Go try out the [jsPerf benchmarks!](http://jsperf.com/speedr-js-vs-normal-object-iteration/8)
* To run the tests on Node.js:
	* Get Coffeescript: `$ npm -g install coffee-script`
	* cd into your tests directory: `$ cd speedr/tests`
	* Install test dependencies: `$ npm install`
	* Run `$ coffee tests-node.coffee` or `$ coffee bench-node.coffee`
