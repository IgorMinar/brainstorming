# <a name="1"/>1 Motivating examples:

## <a name="1.1"/>1.1 Conditional implementation 

Conditional code generation:

```TypeScript
class Debug {  
    @conditional("debug")  
    static assert(condition: boolean, message?: string): void;  
}  
  
Debug.assert(false); // if window.debug is not defined Debug.assert is replaced by an empty function
```

## <a name="1.2"/>1.2 Observable and computed properties

Consider the Ember.js alias-like definition:

```TypeScript
class Person {  
    constructor(public firstName: string, public lastName: string) { }  
  
    @computed('firstName', 'lastName', (f, l) => l + ', ' + f)  
    fullName: string;  
}  
  
var david = new Person('David', 'Tang');  
david.fullName; /// Tang, David
```

## <a name="1.3"/>1.3 Dynamic Instantiation/Dependency Injection (composition)

Consider Angular 2.0 DI implementation example:

```TypeScript
class Engine {  
}  
  
class Car {  
    constructor(@Inject(Engine) engine: Engine) {}  
}  
  
var inj = new Injector([Car, Engine]);  
  
// AtScript compilation step adds a property “annotations” on Car of value [ new Inject(Engine) ].  
// At runtime, a call to inj.get would cause Angular to look for annotations, and try to satisfy dependencies.  
// in this case first by creating a new instance of Engine if it does not exist, and use it as a parameter to Car’s constructor  
var car = inj.get(Car);
```

## <a name="1.4"/>1.4 Attaching Meta data to functions/objects

Metadata that can be queried at runtime, for example:

```TypeScript
class Fixture {  
    @isTestable(true)  
    getValue(a: number): string {  
        return a.toString();  
    }  
}  
  
// Desired JS  
class Fixture {  
    getValue(a) {  
        return a.toString();  
    }  
}  
Fixture.prototype.getValue.meta.isTestable = true;  
  
// later on query the meta data  
function isTestableFunction(func) {  
    return !!(func && func.meta && func.meta.isTestable);  
}
```

## <a name="1.5"/>1.5 Design-time extensibility

An extensible way to declare properties on or associate special behavior to declarations; design time tools can leverage these associations to produce errors or produce documentation. For example:

Deprecated, to support warning on use of specific API’s:

```TypeScript
interface JQuery {  
    /**  
     * A selector representing selector passed to jQuery(), if any, when creating the original set.  
     * version deprecated: 1.7, removed: 1.9  
     */  
    @deprecated("Property is only maintained to the extent needed for supporting .live() in the jQuery Migrate plugin. It may be removed without notice in a future version.", false)  
    selector: string;  
}
```

Suppress linter warning:

```TypeScript
@suppressWarning("disallow-leading-underscore")   
function __init() {  
}
```

# <a name="2"/>2 Proposal

A decorator is defined as a function accepting the decorator’s target as an argument:

```TypeScript
// A simple decorator  
@annotation  
class MyClass { }  
  
function annotation(target) {  
   // Add a property on target  
   target.annotated = true;  
}
```

Alternatively a decorator factory can be specified accepting additional arguments:

```TypeScript
@isTestable(true)  
class MyClass { }  
  
function isTestable(value) {  
   return function decorator(target) {  
      target.isTestable = value;  
   }  
}
```

A decorator can also alter the target:

```TypeScript
class C { 
  @fail  
  func() { console.log("original"); }  
}

function fail(target) {  
   // return a new function instead of target  
   return function() { throw new Error("fail");}  
}
```

A decorator decorating a class or object literal members and accessor operate on the descriptor:

```TypeScript
class C {  
    @enumerable(false)  
    method() { }  
}  
  
function enumerable(value) {  
    return function (target, key, propertyDescriptor) {  
       propertyDescriptor.enumerable = value;  
       return propertyDescriptor;  
    }  
}
```

# <a name="3"/>3 Transformation details:

## <a name="3.1"/>3.1 Introduction

This section shows the desugaring/compilation of the annotation forms to their corresponding ES5 and ES6.

## <a name="3.2"/>3.2 Class Declaration

### <a name="3.2.1"/>3.2.1 Syntax

```TypeScript
@F("color")  
@G  
class Foo {  
}
```

### <a name="3.2.2"/>3.2.2 Desugaring (ES6)

```TypeScript
var Foo = (function () {  
    class Foo {  
    }  
  
    Foo = F("color")(Foo = G(Foo) || Foo) || Foo;  
    return Foo;  
})();
```

### <a name="3.2.3"/>3.2.3 Desugaring (ES5)

```TypeScript
var Foo = (function () {  
    function Foo() {  
    }  
  
    Foo = F("color")(Foo = G(Foo) || Foo) || Foo;  
    return Foo;  
})();
```

## <a name="3.3"/>3.3 Function Expression

### <a name="3.3.1"/>3.3.1 Syntax

```TypeScript
var funcExpr = @F("color") @G function() { }
```

### <a name="3.3.2"/>3.3.2 Desugaring

```TypeScript
var funcExpr = (function () {  
    var _t = function() {}  
    _t = F("color")(_t = G(_t) || _t) || _t;  
    return _t;  
})();
```

## <a name="3.4"/>3.4 Class Parameter Declaration

### <a name="3.4.1"/>3.4.1 Syntax

```TypeScript
class Foo {  
    constructor(@F("color") @G param0) {  
    }  
}
```

### <a name="3.4.2"/>3.4.2 Desugaring (ES6)

```TypeScript
var Foo = (function () {  
    class Foo {  
        constructor(param0) {  
        }  
    }  
  
    F("color")((G(Foo, 0), Foo), 0);  
    return Foo;  
})();
```

### <a name="3.4.3"/>3.4.3 Desugaring (ES5)

```TypeScript
var Foo = (function () {  
    function Foo(param0) {  
    }  
  
    F("color")((G(Foo, 0), Foo), 0);  
    return Foo;  
})();
```



## <a name="3.5"/>3.5 Class Method Declaration

### <a name="3.5.1"/>3.5.1 Syntax

```TypeScript
class Foo {  
    @F("color")  
    @G  
    bar() { }  
}
```

### <a name="3.5.2"/>3.5.2 Desugaring (ES6)

```TypeScript
var Foo = (function () {  
    class Foo {  
        bar() { }  
    }  
  
    var _temp;  
    _temp = F("color")(Foo.prototype, "bar",   
        _temp = G(Foo.prototype, "bar",   
            _temp = Object.getOwnPropertyDescriptor(Foo.prototype, "bar")) || _temp) || _temp;  
    if (_temp) Object.defineProperty(Foo.prototype, "bar", _temp);  
    return Foo;  
})();
```

### <a name="3.5.3"/>3.5.3 Desugaring (ES5)

```TypeScript
var Foo = (function () {  
    function Foo() {  
    }  
    Foo.prototype.bar = function () { }  
  
    var _temp;  
    _temp = F("color")(Foo.prototype, "bar",   
        _temp = G(Foo.prototype, "bar",   
            _temp = Object.getOwnPropertyDescriptor(Foo.prototype, "bar")) || _temp) || _temp;  
    if (_temp) Object.defineProperty(Foo.prototype, "bar", _temp);  
    return Foo;  
})();
```

### <a name="3.5.4"/>3.5.4 By-hand for annotations not requiring descriptors (ES5)

```TypeScript
var Foo = (function () {
    function Foo() {
    }
    Foo.prototype.bar = function () { }

    G(Foo.prototype, "bar");
    F("color")(Foo.prototype, "bar");

    return Foo;
})();
```

## <a name="3.6"/>3.6 Class Accessor Declaration

### <a name="3.6.1"/>3.6.1 Syntax

```TypeScript
class Foo {  
    @F("color")  
    @G  
    get bar() { }  
    set bar(value) { }  
}
```

### <a name="3.6.2"/>3.6.2 Desugaring (ES6)

```TypeScript
var Foo = (function () {  
    class Foo {  
        get bar() { }  
        set bar(value) { }  
    }  
  
    var _temp;  
    _temp = F("color")(Foo.prototype, "bar",   
        _temp = G(Foo.prototype, "bar",   
            _temp = Object.getOwnPropertyDescriptor(Foo.prototype, "bar")) || _temp) || _temp;  
    if (_temp) Object.defineProperty(Foo.prototype, "bar", _temp);  
    return Foo;  
})();
```

### <a name="3.6.3"/>3.6.3 Desugaring (ES5)

```TypeScript
var Foo = (function () {  
    function Foo() {  
    }  
    Object.defineProperty(Foo.prototype, "bar", {  
        get: function () { },  
        set: function (value) { }  
        enumerable: true, configurable: true  
    });  
  
    var _temp;  
    _temp = F("color")(Foo.prototype, "bar",   
        _temp = G(Foo.prototype, "bar",   
            _temp = Object.getOwnPropertyDescriptor(Foo.prototype, "bar")) || _temp) || _temp;  
    if (_temp) Object.defineProperty(Foo.prototype, "bar", _temp);  
    return Foo;  
})();
```

## <a name="3.7"/>3.7 Object Literal Method Declaration

### <a name="3.7.1"/>3.7.1 Syntax

```TypeScript
var o = {  
    @F("color")  
    @G  
    bar() { }  
}
```

### <a name="3.7.2"/>3.7.2 Desugaring (ES6)

```TypeScript
var o = (function () {  
    var _obj = {  
        bar() { }  
    }  
  
    var _temp;  
    _temp = F("color")(_obj, "bar",   
        _temp = G(_obj, "bar",   
            _temp = void 0) || _temp) || _temp;  
    if (_temp) Object.defineProperty(_obj, "bar", _temp);  
    return Foo;  
})();
```

### <a name="3.7.3"/>3.7.3 Desugaring (ES5)

```TypeScript
var o = (function () {  
    var _obj = {  
        bar: function () { }  
    }  
  
    var _temp;  
    _temp = F("color")(_obj, "bar",   
        _temp = G(_obj, "bar",   
            _temp = void 0) || _temp) || _temp;  
    if (_temp) Object.defineProperty(_obj, "bar", _temp);  
    return Foo;  
})();
```

## <a name="3.8"/>3.8 Object Literal Accessor Declaration

### <a name="3.8.1"/>3.8.1 Syntax

```TypeScript
var o = {  
    @F("color")  
    @G  
    get bar() { }  
    set bar(value) { }  
}
```

### <a name="3.8.2"/>3.8.2 Desugaring (ES6)

```TypeScript
var o = (function () {  
    var _obj = {  
        get bar() { }  
        set bar(value) { }  
    }  
  
    var _temp;  
    _temp = F("color")(_obj, "bar",   
        _temp = G(_obj, "bar",   
            _temp = void 0) || _temp) || _temp;  
    if (_temp) Object.defineProperty(_obj, "bar", _temp);  
    return Foo;  
})();
```

### <a name="3.8.3"/>3.8.3 Desugaring (ES5)

```TypeScript
var o = (function () {  
    var _obj = {  
    }  
    Object.defineProperty(_obj, "bar", {  
        get: function () { },  
        set: function (value) { }  
        enumerable: true, configurable: true  
    });  
  
    var _temp;  
    _temp = F("color")(_obj, "bar",   
        _temp = G(_obj, "bar",   
            _temp = void 0) || _temp) || _temp;  
    if (_temp) Object.defineProperty(_obj, "bar", _temp);  
    return Foo;  
})();
```


# <a name="4"/>4 Examples

### <a name="4.1"/>4.1 Inject

```TypeScript
// TypeScript Souce
function Inject(@type type: Function) {
    return function (constructor) {
        constructor.annotate = constructor.annotate || [];
        constructor.annotate.push(new inject(type));
    }
}

class Engine {
}

class Car {
    constructor(@Inject() engine: Engine) {
    }
}


// ES3/ES5 
var Car = (function () {
    function Car(engine) {
    }

    Inject(/* @type */ Engine)(Car, /* paramterIndex */ 0);

    return Car;
})();

// ES7 decorator proposal
class Car {
    constructor(@Inject(Engine) engine) {
    }
}
```

### <a name="4.2"/>4.2 Component

```TypeScript
// TypeScript Souce
function Component(options, @parameterTypes types?: Function[]) {
    return function (constructor) {
        constructor.annotate = constructor.annotate || [];
        constructor.annotate.push(new component(options));
        constructor.parameters = types;
    }
}

@Component({
    selector: 'b'
})
class MyComponent {
    constructor(server: Server, service: Service) { }
}


// ES3/ES5 
var MyComponent = (function () {
    function MyComponent(server, service) {
    }

    MyComponent = Component({ selector: 'b' }, /* @parameterTypes */[Server, Service])(MyComponent) || MyComponent;

    return MyComponent;
})();

// ES7 decorator proposal
@Component({
    selector: 'b'
}, [Server, Service])
class MyComponent {
    constructor(server, service) { }
}
```

# <a name="A"/>A Grammar

## <a name="A.1"/>A.1 Expressions

&emsp;&emsp;*DecoratorList*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp; *Decorator*<sub> [?Yield]</sub>

&emsp;&emsp;*Decorator*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;`@`&emsp;*AssignmentExpression*<sub> [?Yield]</sub>

&emsp;&emsp;*PropertyDefinition*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*IdentifierReference*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*CoverInitializedName*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*PropertyName*<sub> [?Yield]</sub>&emsp; `:`&emsp;*AssignmentExpression*<sub> [In, ?Yield]</sub>  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;*MethodDefinition*<sub> [?Yield]</sub>

&emsp;&emsp;*CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;`[`&emsp;*Expression*<sub> [In, ?Yield]</sub>&emsp;`]`

NOTE	The production *CoverMemberExpressionSquareBracketsAndComputedPropertyName* is used to cover parsing a *MemberExpression* that is part of a *Decorator* inside of an *ObjectLiteral* or *ClassBody*, to avoid lookahead when parsing a decorator against a *ComputedPropertyName*. 

&emsp;&emsp;*PropertyName*<sub> [Yield, GeneratorParameter]</sub>&emsp;:  
&emsp;&emsp;&emsp;*LiteralPropertyName*  
&emsp;&emsp;&emsp;[+GeneratorParameter] *CoverMemberExpressionSquareBracketsAndComputedPropertyName*  
&emsp;&emsp;&emsp;[~GeneratorParameter] *CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [?Yield]</sub>

&emsp;&emsp;*MemberExpression*<sub> [Yield]</sub>&emsp; :  
&emsp;&emsp;&emsp;[Lexical goal *InputElementRegExp*] *PrimaryExpression*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;*CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;`.`&emsp;*IdentifierName*  
&emsp;&emsp;&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;*TemplateLiteral*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*SuperProperty*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*NewSuper*&emsp;*Arguments*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;`new`&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;*Arguments*<sub> [?Yield]</sub>

&emsp;&emsp;*SuperProperty*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;`super`&emsp;*CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;`super`&emsp;`.`&emsp;*IdentifierName*

&emsp;&emsp;*CallExpression*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;*Arguments*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*SuperCall*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*CallExpression*<sub> [?Yield]</sub>&emsp;*Arguments*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*CallExpression*<sub> [?Yield]</sub>&emsp;*CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [In, ?Yield]</sub>  
&emsp;&emsp;&emsp;*CallExpression*<sub> [?Yield]</sub>&emsp;`.`&emsp;*IdentifierName*  
&emsp;&emsp;&emsp;*CallExpression*<sub> [?Yield]</sub>&emsp;*TemplateLiteral*<sub> [?Yield]</sub>

## <a name="A.4"/>A.4 Functions and Classes

&emsp;&emsp;*FormalRestParameter*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;*BindingRestElement*<sub> [?Yield]</sub>

&emsp;&emsp;*FormalParameter*<sub> [Yield, GeneratorParameter]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;*BindingElement*<sub> [?Yield, ?GeneratorParameter]</sub>

&emsp;&emsp;*FunctionExpression*&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`function`&emsp;*BindingIdentifier*<sub> opt</sub>&emsp;`(`&emsp;*FormalParameters*&emsp;`)`&emsp;`{`&emsp;*FunctionBody*&emsp;`}`

&emsp;&emsp;*GeneratorExpression*&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`function`&emsp;*BindingIdentifier*<sub> [Yield]opt</sub>&emsp;`(`&emsp;*FormalParameters*<sub> [Yield, GeneratorParameter]</sub>&emsp;`)`&emsp;`{`&emsp;*GeneratorBody*<sub> [Yield]</sub>&emsp;`}`

&emsp;&emsp;*ClassDeclaration*<sub> [Yield, Default]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`class`&emsp;*BindingIdentifier*<sub> [?Yield]</sub>&emsp;*ClassTail*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;[+Default] *DecoratorList*<sub> [?Yield]opt</sub>&emsp;`class`&emsp;*ClassTail*<sub> [?Yield]</sub>

&emsp;&emsp;*ClassExpression*<sub> [Yield, GeneratorParameter]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`class`&emsp;*BindingIdentifier*<sub> [?Yield]opt</sub>&emsp;*ClassTail*<sub> [?Yield, ?GeneratorParameter]</sub>

&emsp;&emsp;*ClassElement*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;*MethodDefinition*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`static`&emsp;*MethodDefinition*<sub> [?Yield]</sub>

## <a name="A.5"/>A.5 Scripts and Modules

&emsp;&emsp;*ExportDeclaration*&emsp;:  
&emsp;&emsp;&emsp;`export`&emsp;`*`&emsp;*FromClause*&emsp;`;`  
&emsp;&emsp;&emsp;`export`&emsp;*ExportClause*&emsp;*FromClause*&emsp;`;`  
&emsp;&emsp;&emsp;`export`&emsp;*ExportClause*&emsp;`;`  
&emsp;&emsp;&emsp;`export`&emsp;*VariableStatement*  
&emsp;&emsp;&emsp;`export`&emsp;*LexicalDeclaration*  
&emsp;&emsp;&emsp;*DecoratorList*<sub> opt</sub>&emsp;`export`&emsp;[lookahead ≠ `@`]&emsp;*HoistableDeclaration*  
&emsp;&emsp;&emsp;*DecoratorList*<sub> opt</sub>&emsp;`export`&emsp;[lookahead ≠ `@`]&emsp;*ClassDeclaration*  
&emsp;&emsp;&emsp;*DecoratorList*<sub> opt</sub>&emsp;`export`&emsp;`default`&emsp;[lookahead ≠ `@`]&emsp;*HoistableDeclaration*<sub> [Default]</sub>  
&emsp;&emsp;&emsp;*DecoratorList*<sub> opt</sub>&emsp;`export`&emsp;`default`&emsp;[lookahead ≠ `@`]&emsp;*ClassDeclaration*<sub> [Default]</sub>  
&emsp;&emsp;&emsp;`export`&emsp;`default`&emsp;[lookahead  { `function`, `class`, `@` }]&emsp;*AssignmentExpression*<sub> [In]</sub>&emsp;`;`

# <a name="B"/>B Decorator definitions

```TypeScript
interface TypedPropertyDescriptor<T> {  
    enumerable?: boolean;  
    configurable?: boolean;  
    writable?: boolean;  
    value?: T;  
    get?: () => T;  
    set?: (value: T) => void;  
}  
  
interface DecoratorFunction<TFunction extends Function> {  
    (target: TFunction): TFunction | void;  
}  
  
interface ArgumentDecoratorFunction {  
    (target: Function, parameterIndex: number): void;  
}  
  
interface MemberDecoratorFunction<T> {  
    (target: Function | Object, propertyKey: PropertyKey, descriptor: TypedPropertyDescriptor<T>): TypedPropertyDescriptor<T> | void;  
}  
  
interface PropertyDecoratorFunction {  
    (target: Function | Object, propertyKey: PropertyKey): void;  
}  
  
interface DecoratorFactory<TFunction extends Function> {  
    (...args: any[]): DecoratorFunction<TFunction>;  
}  
  
interface ArgumentDecoratorFactory {  
    (...args: any[]): ArgumentDecoratorFunction;  
}  
  
interface MemberDecoratorFactory<T> {  
    (...args: any[]): MemberDecoratorFunction<T>;  
}  
  
interface PropertyDecoratorFactory {  
    (...args: any[]): PropertyDecoratorFunction;  
}
```

# <a name="C"/>C TypeScript decorators

## <a name="C.1"/>C.1 Exposing types

TypeScript compiler will honor special decorator names and will flow additional information into the decorator factory parameters annotated by these decorators. The types provided are in a serialized form. Serialization logic is descriped in C.2

```TypeScript
@F("color")  
class Foo {  
    constructor(a: Object, b: number, c: { a: number }, d: C2) {  
    }  
}  
  
function F(tag: string, @paramterTypes types?: Function[]) {  
    return function (target) {  
        target.paramterTypes = types; // [Object, Number, Object, C2]  
    }  
}
```

## <a name="C.2"/>C.2 List of supported decorators

@type – The serialized form of the type of the decorator target

@returnType – The serialized form of the return type of the decorator target if it is a function type, undefined otherwise

@paramterTypes – A list of serialized types of the decorator target’s arguments if it is a function type, undefined otherwise

@name – The name of the decorator target 

## <a name="C.3"/>C.3 Type Serialization:

### <a name="C.3.1"/>C.3.1 Example

```TypeScript
class C { }  
interface I { }  
enum E { }  
module M { }
```

Formal parameter list in a call signature like so:

```TypeScript
(a: number, b: boolean, c: C, i: I, e: E, m: typeof M, f: () => void, o: { a: number; b: string; })
```

Serializes as:

```TypeScript
[Number, Boolean, C, Object, Number, Object, Function, Object]
```

### <a name="C.3.2"/>C.3.2 Details

* number serialized as Number
* string serialized as String
* boolean serialized as Boolean
* any serialized as Object
* void serializes as undefined
* Array serialized as Array
* If a Tuple, serialize as Array
* If a class serialize it as the class constructor
* If an Enum serialize it as Number
* If has at least one call signature, serialize as Function
* Otherwise serialize as Object

### <a name="C.3.3"/>C.3.3 Open issues

* Do we want to enable more elaborate serialization? E.g. `type O = { a: string; b: number }` as `{ a: String, b: Number }` instead of just `Object`.

## <a name="C.4"/>C.4 Defining a decorator

* Valid decorators must be annotated by the @decorator -- details below
* A decorator function must return a value that is assignable to what it is decorating to be considered valid.
* @decorator parameters define the usage of the decorator and are enforced by the compiler
* @decorator is defined in lib.d.ts and loaded by the checker when it is initialized, some constraints will be checked like that it is defined as a class with one constructor signature and one argument
* Decorators are only allowed on functions, parameters, or members. Thus, modules, enums, and variable declarations are not valid decorator targets.
* Constructors are not valid decorator targets. Containing classes should be used instead.
* Decorators are emitted in declaration files attached to their declarations, normal visibility rules applies to decorators and their arguments.

```TypeScript


/**  
  * Built-in decorator annotation. Used to define a decorator and its properties.  
  */  
declare function decorator(options?: {  
    /**  
      * Valid targets for this decorator.  
      *  
      * default: DecoratorTargets.All  
      */  
    allowOn?: DecoratorTargets;  
      
    /**  
      * Indicates whether multiple applications are allowed on the same declaration or not.  
      *  
      * default: true  
      */  
    allowMultiple?: boolean;  
  
    /**  
      * True indicates that the decorator is for design-time only, and should not be  
      * added to the emitted code.   
      *  
      * default: false  
    */  
    ambient?: boolean;  
});  
  
/**  
  * Valid decorator targets  
  */  
declare const enum DecoratorTargets {  
    Class,  
    Interface,  
    Function,  
    Method,  
    Property,  
    Accessor,  
    Parameter,  
    All = Class | Interface | Function | Method | Property | Accessor | Parameter  
}
```

## <a name="C.5"/>C.5 Run-time Decorators

* Runtime decorators are designated by setting { ambient: false } in the options of the @decorator decorator
* Runtime decorators follow their target, decorators targeting ambient declarations are not emitted
* Runtime decorators are not allowed on interfaces

## <a name="C.6"/>C.6 Ambient decorators

* Design-time (Ambient) decorators are designated by setting { ambient: true } in the options of the @decorator decorator
* These are decorators that are not emitted. This category include built-in decorator (decorator, conditional, deprecated, etc..)
* For design-time decorator, arguments are restricted to constant values. Variables would not be observable by the compiler at compile time. Here is the set of possible values:
  * string literal,
  * number literal, 
  * regexp literal,
  * true keyword,
  * false keyword,
  * null keyword,
  * undefined symbol,
  * const enum members,
  * array literals of one of the previous kinds,
  * object literal with only properties with values of one of the previous kinds
* Examples: @conditional, @deprecated, @profile

# <a name="D"/>D Function declaration support

## <a name="D.1"/>D.1 Function Declaration

Since decorator evaluation involved expressions, decorated function declarations cannot be hoisted to the containing scope. Rather they should be treated similar to ES6 classes where only the function’s symbol is hoisted to the top but remains undefined until the definition is encountered. 

Note that decorators applied to a function declaration would necessitate a TDZ for the declaration. The desugared emit below illustrates this as the function declaration is evaluated inside an IIFE

### <a name="D.1.1"/>D.1.1 Syntax

```TypeScript
@F("color")  
@G  
class Func() {  
}
```

### <a name="D.1.2"/>D.1.2 Desugaring

```TypeScript
var Func = (function () {  
    function Func() {  
    }  
    Func = F("color")(Func = G(Func) || Func) || Func;  
    return Func;  
})();
```

## <a name="D.2"/>D.2 Function Parameter Declaration

### <a name="D.2.1"/>D.2.1 Syntax

```TypeScript
function Func(@F("color") @G param0) {  
}
```

### <a name="D.2.2"/>D.2.2 Desugaring

```TypeScript
var Func = (function () {  
    function Func(param0) {  
    }  
  
    F("color")((G(Func, 0), Func), 0);  
    return Func;  
})();
```
### <a name="D.3"/>D.3 Grammar

&emsp;&emsp;*FunctionDeclaration*<sub> [Yield, Default]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`function`&emsp;*BindingIdentifier*<sub> [?Yield]</sub>&emsp;`(`&emsp;*FormalParameters*&emsp;`)`&emsp;`{`&emsp;*FunctionBody*&emsp;`}`  
&emsp;&emsp;&emsp;[+Default] *DecoratorList*<sub> [?Yield]opt</sub>&emsp;`function`&emsp;`(`&emsp;*FormalParameters*&emsp;`)`&emsp;`{`&emsp;*FunctionBody*&emsp;`}`

&emsp;&emsp;*GeneratorDeclaration*<sub> [Yield, Default]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`function`&emsp;`*`&emsp;*BindingIdentifier*<sub> [?Yield]</sub>&emsp;`(`&emsp;*FormalParameters*<sub> [Yield, GeneratorParameter]</sub>&emsp;`)`&emsp;`{`&emsp;*GeneratorBody*<sub> [Yield]</sub>&emsp;`}`  
&emsp;&emsp;&emsp;[+Default] *DecoratorList*<sub> [?Yield]opt</sub>&emsp;`function`&emsp;`*`&emsp;`(`&emsp;*FormalParameters*<sub> [Yield, GeneratorParameter]</sub>&emsp;`)`&emsp;`{`&emsp;*GeneratorBody*<sub> [Yield]</sub>&emsp;`}`

