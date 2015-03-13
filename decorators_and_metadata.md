# Design Goals

* Effect of decorators is predictable.  
* Metadata persists as decorators are applied.  Even if a decorator returns a new object, metadata persists.
* Clean ES6 and ES5

# Examples

## TypeScript

```TypeScript
@decorator2
@decorator1
class C { 
  constructor(x: number) { }
}
```

## ES6 (by hand)

```JavaScript
import { ParamTypes } from “annotations”;
class C {
  constructor(x) { }
}
C = Reflect.decorate([decorator2, decorator1, ParamTypes(Number)], C);
```

## ES5 (by hand)

```JavaScript
define(['annotations'], function (annotations) {
  var ParamTypes = annotations.ParamTypes;

  var C = function(x) { };
  C = Reflect.decorate([decorator2, decorator1, ParamTypes(Number)], C);
}
```

# Implementation

```TypeScript
//Example decorator
function decorator1(target) {
  Reflect.setMetadata(target, “myAnnotation”, “rocks!”);
}


//pseudo-code for Reflect API
Reflect.decorate = function(decorators, target) {
  //for each decorator in reverse
  for (var i = decorators.length - 1; i >= 0; i--) {
    var new_target = decorator(target);
    //merge metadata to persist metadata between decorator calls
    if (new_target != null && new_target !== target) { 
      if (typeof Reflect.mergeMetadata === “Function”) {
        Reflect.mergeMetadata(new_target, target);
      }
      target = new_target;
    }
  }
  return target;
}

//pseudo-code for ParamTypes helper function
function ParamTypes(...args) {
  return function(target) {
    Reflect.setMetadata(target, “param types”, args);
  }
}
```

# Requirements

Reflect API:
* decorate(decorators, target, key?, descriptor?)
* setMetadata(target, key, value)
* mergeMetadata(target1, target2)
* getMetadata(target, key)
* getOwnMetadata(target, key)
* getMetadataKeys(target)

Helper API:
* ParamTypes(...types)
* ReturnType(type)
* Type(type)


