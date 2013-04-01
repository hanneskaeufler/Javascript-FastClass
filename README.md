Javascript-fastClass
====================
A faster and easier Javascript Inheritance 


## Performance tests
*  Among <a href="http://jsperf.com/js-inheritance-performance/36" target="_blank"><code>popular libraries define + usage</code></a> - 3 classes and 3 methods * 500 instances each
*  <a href="http://jsperf.com/js-inheritance-performance/35" target="_blank"><code>Fastest libraries define + usage</code></a> - 3 classes and 3 methods * 500 instances each
*  <a href="http://jsperf.com/js-inheritance-performance/34" target="_blank"><code>Fastest libraries define only</code></a> - 3 classes and 3 methods * 1 instances each

<div align="center">
<img src="../../wiki/images/NugetIcon.png"/>
</div>

## Why yet another library?
Native Javascript Inheritance is a pin in the ass. Even if you understand it perfectly it still requires some hideous repetivie code.

There are a lot of libraries which aims to help you with such that, but the main question is:

What is <a href="http://jsperf.com/js-inheritance-performance/36" target="_blank"><code>the fastest</code></a> vs <a target="_blank" href="https://github.com/njoubert/inheritance.js/blob/master/INHERITANCE.md"><code>most convenient</code></a> to create <a href="http://msdn.microsoft.com/en-us/magazine/ff852808.aspx" target="_blank"><code>Prototypal Inheritance</code></a> with?

## When to use it?
You do need this library when you can't use a language that <a href="https://github.com/jashkenas/coffee-script/wiki/List-of-languages-that-compile-to-JS" target="_blank"><code>compiles</code></a> into javascript.

e.g. <a href="https://developers.google.com/closure/" target="_blank">Google Closure</a>, <a href="http://www.typescriptlang.org/Playground/" target="_blank">TypeScript</a>, <a href="http://arcturo.github.com/library/coffeescript/03_classes.html" target="_blank">Coffee Script</a> etc.

## What is it? 
FastClass is a very tiny library (<0.5KB minified and gzipped) that helps you quickly derrive your `classes` so to speak. 
It comes in two flavours:
* [`Function.prototype.fastClass(creator)`](#fastclass-flavour) - fastest, does not iterate the members when creates the derrived function
```javascript
function(base, baseCtor) { this.somePrototypeMethod1 =  ...; this.somePrototypeMethod2 =  ...; } }
```

* [`Function.prototype.inheritWith(creator)`](#inheritwith-flavour) 
It makes usage of __proto__ on all new browsers (which makes it blazing fast) except `Internet Explorer` and maybe other ancient browser where it fallbacks to `for (var key in obj)` statement.
__proto__ will become standard in ECMAScript 6
```javascript
function(base, baseCtor) { return { somePrototypeMethod1: ..., somePrototypeMethod2: ...} }
```
whereas `baseCtor` is the function we want to inherit and base is it's prototype *(`baseCtor.prototype` that is).*

## How to use it?

### The base "class"
```javascript
var Figure = function(name) {
    this.name = name;
}
Figure.prototype.draw = function() { console.log("figure " + name); }
```

A classical example to use inheritance when you have a base class called `Figure` and a derrived class called `Square`.

#### `.fastClass` flvaour

To define the `derrived class` Square:
```javascript
var Square = Figure.fastClass(function(base, baseCtor) {
    this.constructor = function(name, length) { 
      this.length = length;
      baseCtor.call(this, name);//calls the bacse ctor
    }
    this.draw = function() {//redefines the draw function
      console.log("square with length " + this.length);
      base.draw.call(this);//calls the base class' draw function
    }
})
```

#### `.inheritWith` flavour

To define the `derrived class` Square:
```javascript
var Square = Figure.inheritWith(function(base, baseCtor) {
    return { 
        constructor:  function(name, length) { 
            this.length = length;
            baseCtor.call(this, name);//calls the bacse ctor
        },
        draw: function() {//redefines the draw function
            console.log("square with length " + this.length);
            base.draw.call(this);//calls the base class' draw function
        }
    }
})
```

As you can see in both cases the definition is pretty simple and very similar. 

However the `.inheritWith` flavour comes with about 5-10% <a href="http://jsperf.com/js-inheritance-performance/35" target="_blank"><code>performance cost</code></a> depending on the actual browser and number of members.

#### Usage

Whichever flavour you choose the code usage is the same. Firstly you need to instantiate the Constructor with the `new` operator:
```javascript
var figure = new Figure("generic");
figure.draw();
//figure generic
var square = new Square("10cm square", 10);
figure.draw(); 
//square with length 10
//figure 10cm square
```

## Another clasical example

#### Example

```javascript
// Animal base class
function Animal() {
    // Private
    function private1(){}
    function private2(){}

    // Privileged - on instance
    this.privileged1 = function(){}
    this.privileged2 = function(){}
}.define({ 
    // Public - on prototype
    method1: function(){}
});
```

Alternatively the above can be defined as:
```javascript
var Animal = Function.fastClass(function(){
    // Private 
    function private1() {}
    function private2() {}
    return {
        constructor: function() {
            // Privileged - on instance
            this.privileged1 = function(){}
            this.privileged2 = function(){}
        },
        method1: function() {}
    };
});
```

The `function Animal` method acts as the constructor, which is invoked when an instance is created:

```javascript
var animal = new Animal(); // Create a new Animal instance
```

### Inheritance

#### Usage

`[[constructor]].inheritWith( function(base, baseCtor) { return {...} } )` - that function should `return` methods for the derrived prototype


```javascript
// Extend the Animal class with inheritWith flavor
var Dog = Animal.inheritWith(function(base, baseCtor) {
    //derrived class containing some private method(s)
    function someOtherPrivateMethod() { }
    
    return {
        // Override base class `method1`
        method1: function(){
            someOtherPrivateMethod.call(this);
            console.log('dog::method1');
        },
        scare: function(){
            console.log('Dog::I scare you');
        },
        //some new public method(s)
        method2: function() { }
    }
});
```

Create an instance of `Dog`:

```javascript
var husky = new Dog();
husky.scare(); // "Dog::I scare you'"
```

#### Accessing parent prototype

#### Usage

`[[constructor]].inheritWith( function(base, baseCtor) { this.m = function {...} ... } )` - that function `populates` the derrived prototype
Every class definition has access to the parent's prototype via the first argument passed into the function. The second argument is the base Class itself (constructor):

```javascript
// Same as above but Extend the Animal class using fastClass flavor
var Dog = Animal.fastClass(function(base, baseCtor) {
    //private functions
    function someOtherPrivateMethod() {};
    
    //since we don't set this.constructor, automatically will be added one for us which will call the baseCtor with all the provided arguments
    //this is importat so we can set the new prototype into it
    
    // Override base class `method1`
    this.method1 = function(){
        someOtherPrivateMethod.call(this);
        // Call the parent function
        base.method1.call(this);
    };
    this.scare = function(){
        console.log('Dog::I scare you');
    };
    //some more public functions
    this.method2 = function() {};
});
```

### Some gems
Although it could hurt `hard-core performance` in real live it doesn't feel much of a difference betweeen `.innerWith` and `.fastClass` upon same browser. 
Between different browsers is a whole different story.

However, for those cases where you don't count every performance bit then you can also define de primary function like this:
```javascript
var A = function(name) { 
    this.name = name; 
}.define({
    draw: function() {
        console.log("figure "+ this.name);
    }
}) 
```

The `define` function copies all the members of the returned object to the `function.prorortpe` *(`A.prototype`)* and returns it *(`A`)*


## Where?
Beside GitHub, you can download it as a <a href="http://nuget.org/packages/Javascript-FastClass/" target="_blank"><code>Nuget package</code></a> in Visual Studio from<a href="http://nuget.org/packages/Javascript-FastClass/" target="_blank"><code>here</code></a>
```javascript
Install-Package Javascript-FastClass
```

## What's next?
Do you have a better & faster way? Share it! We would love to seeing creativity in action!

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/ced79a6263a52ce6aed7515d0cd0b0f3 "githalytics.com")](http://githalytics.com/dotnetwise/Javascript-FastClass)
