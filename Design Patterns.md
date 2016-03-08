## What is a pattern?

Patterns are proven solutions to a problem that can be easily reused, like templates to solve a problem.

However, they are *not* an exact solution, they provide us instead with a *solution scheme*

Other advantages:
* Prevent minor issues
* Provide generalised solutions
* Decrease overall code size (DRY)
* Add to dev's vocabulary
* Patterns evolve and get improved

## Pattern vs Anti-Pattern
Patterns are best practices, Anti-Patterns are lessons learnt.

* They describe a bad solution to a problem, and created a bad situation
* They describe how to get out of that situation into a good solution

> A bad design that's worth documenting

### JS Antipatterns
* Polluting the global scope
* Using strings on `setInterval` to trigger `eval()` internally
* Modifying the `Object` prototype 💩 never do this
* Using JS in an inline form
* Using `document.write` instead of `document.createElement`


## Categories of Design Pattern
* Creational Design Patterns: Handling object creation. (Constructor, Factory, Abstract, Prototype, Singleton & Builder)
* Structural Design Patterns: Dealing with object composition. (Decorator, Facade, Flyweight, Adapter and Proxy)
* Behavioral Design Patterns: Streamlining object communication. (Iterator, Mediator, Observer and Visitor)


# Design Patterns

## Constructor Pattern
A method used to initialise an object once memory has been created for it. Javascript does not support classes, but it supports special constructor functions that work with objects.

In JS: three ways
```javascript
var newObject = {};

var newObject = Object.create(Object.prototype);

var newObject = new Object();
```

The last one is called a constructor.

There are 4 ways to define properties on an object
```javascript
//1. Dot Syntax:
newObject.myKey = 'someValue';

//2. Square Bracket syntax:
newObject['myKey'] = 'someValue';

//3. Object.defineProperty (ES5+)
Object.defineProperty(newObject, 'myKey', {
  value: 'someValue',
  writable: true,
  enumerable: true,
  configurable: true
});

//4. Object.defineProperties (ES5+, like 3 but more than one)
Object.defineProperties(newObject, {
  'myKey' : {
    value: 'someValue',
    writable: true
  },

  'anotherKey' : {
    value: 'someOtherValue',
    writable: false
  }
})
```

### Basic Constructor
By prefixing a function with the keyword `new` we tell JS we want the function to behave like a constructor. Inside the constructor, the keyword `this` references the object being created.

```javascript
function Car (model, year, miles) {

  this.model = model;
  this.year = year;
  this.miles = miles;

  this.toString = function () {
    return this.model + " has done " + this.miles + " miles";
  };
}

// Usage:

// We can create new instances of the car
var civic = new Car("Honda Civic", 2009, 20000);
var mondeo = new Car("Ford Mondeo", 2010, 5000);

civic.toString(); // -> "Honda Civic has done 20000 miles"
mondeo.toString(); // -> "Ford Mondeo has done 5000 miles"
```

### Constructor with prototype
The previous example redefines the `toString` function everytime we create an object. Would be better to share this amongst all the instances of `Car`

In JS, Functions, like most objects, contain a `prototype` object. When we call a JS constructor to create an object, all the properties of the constructor's prototype are made available to the new object.

```javascript
function Car (model, year, miles) {
  this.model = model;
  this.year = year;
  this.miles = miles;
}

Car.prototype.toString = function() {
  return this.model + ' has done ' + this.miles + ' miles';
}
```

Now a single instance of `toString` will be shared amonst all the Car objects.

## Module Pattern
Defined as a way to provide both private and public *encapsulation* for classes in conventional sofware engineering.

In JS, the Module pattern *emulates* the concept of clases.

### Privacy
The module pattern encapsulates "privacy", state and organisation using *closures*. Only a Public API is returned, keeping everything else private.

Privacy doesn't exist in javascript, so we use function scope to emulate this:
```javascript
var myNamespace = (function () {

  var myPrivateVar, myPrivateMethod;

  // A private counter variable
  myPrivateVar = 0;

  // A private function which logs any arguments
  myPrivateMethod = function( foo ) {
      console.log( foo );
  };

  return {

    // A public variable
    myPublicVar: "foo",

    // A public function utilizing privates
    myPublicFunction: function( bar ) {

      // Increment our private counter
      myPrivateVar++;

      // Call our private method using bar
      myPrivateMethod( bar );

    }
  };

})();
```

### Advantages:
* Makes it easier for OO developers (i.e people from a non-functional background) who miss Classes, rather than using trully functional encapsulation.
* It supports private data
### Disadvantages:
* Different way of accessing public and private members, If we want to change visibility we need to change everywhere the member was used
* Can't access private members in methods added to the object at a later point.
* Unable to write unit tests for private members.
* Additional complexity when bugfixing: Not possible to patch private members, all public methods need to be overrid

[more about the module pattern](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth)

## Singleton Pattern
Restricts Instantiation of a class to a single Object. This is implemented by creating a class with a method that creates a new instance of the class if one doesn't exist. If it does exist, then returns a reference to that object.

We can delay the initialisation of singletons, usually because we don't have some required information.

```javascript
var mySingleton = (function () {

  // Instance stores a reference to the Singleton
  var instance;

  function init() {

    // Singleton

    // Private methods and variables
    function privateMethod(){
        console.log( "I am private" );
    }

    var privateVariable = "Im also private";

    var privateRandomNumber = Math.random();

    return {

      // Public methods and variables
      publicMethod: function () {
        console.log( "The public can see me!" );
      },

      publicProperty: "I am also public",

      getRandomNumber: function() {
        return privateRandomNumber;
      }

    };

  };

  return {

    // Get the Singleton instance if one exists
    // or create one if it doesn't
    getInstance: function () {

      if ( !instance ) {
        instance = init();
      }

      return instance;
    }

  };

})();
```
We get the singleton by calling `MySingleton.getInstance()`. The difference between a static instance of a class (object) and a Singleton is that while a singleton can be implemented as a static instance, it can also be constructed lazily, without the need for resources or memory until they're needed.

Singletons are useful when exactly one object is needed to coordinate others across a system.

If you find you need a singleton, perhaps you might have to reevaluate your design.

[More about singletons](http://www.ibm.com/developerworks/webservices/library/co-single/index.html)


## Observer Pattern
The Observer is a design pattern where an object (the subject) mantains a list of objects depending on it (observers), automatically notifying them of any changes to state.






* Command
* Flyweight

Constructor
