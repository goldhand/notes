ES6 Maps and Weakmaps
=====================
ECMA script 2015 introduced maps and weakmaps to native javascript.


`Map` API
---------

* `set(key, value)`
* `get(key)`
* `size`
* `clear()`
* `has(key)`

__Iterators:__
* `keys()`
* `values()`
* `entries()`: iterate over arrays `[key, value]`

```javascript

for (const [key, value] of myMap.entries()) {
    ...
}
```

`WeakMap`
--------
Like `Map` but no refs are held to the keys of the map. This allows it to be available to javascript's automatic garbage collection but because there are no refs to keys, `WeakMap` is innumerable (you can't iterate over it).


`Map()` vs `new Object()`
__Why is this different than just using an object for mapping?__
In the past, objects have been used for mapping. What does `Map` do that cant be done with javascript objects?

* An object has a prototype, so by default, there are keys in the map whether the user has added them or not
* An object key has to be a string, whereas in a map, it can be anything from a function, to an object, to all other types of primitives.
* Maps have methods on them that allow you to easily get the size of and keep track of the size of your map.
