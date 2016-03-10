## What is a Design Pattern?

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
The Observer is a design pattern where an object (the *subject*) mantains a list of objects depending on it (*observers*), automatically notifying them of any changes to state.

If somethign interesting happens, the subject can notify all observers, broadcasing a notification which can include data related to the topic of the notification.

If an observer does no longer wish to be notified of changes, the subject can remove it from its list of observers.

#### Gang of Four (GoF) definition:
> "One or more observers are interested in the state of a subject and register their interest with the subject by attaching themselves. When something changes in our subject that the observer may be interested in, a notify message is sent which calls the update method in each observer. When the observer is no longer interested in the subject's state, they can simply detach themselves."

### Implementing the Observer Pattern
The Observer pattern can be implemented with the following components
* *Subject*: Mantains the list of observers, adds, or removes them.
* *Observer*: Provides an update interface for objects that need to be notified of a change of state in a Subject, i.e. a function that can be called when the state thanges *callback*
* *ConcreteSubject*: Broadcasts notifications to observers on changes of state, stores the state of ConcreteObservers.
* *ConcreteObserver*: Stores a reference to the ConcreteSubject, implements an update interface for the Observer to ensure state is consistent with the Subject's

#### Observer list
```javascript
function ObserverList() {
  this.observerList = [];
}

ObserverList.prototype.add = function(obj) {
  return this.observerList.push(obj);
};

ObserverList.prototype.count = function() {
  return this.observerList.length;
};

ObserverList.prototype.get = function(index) {
  if (index > -1 && index < this.observerList.length) {
    return this.observerList[index];
  }
};

ObserverList.prototype.indexOf = function(obj, startIndex) {
  var i = startIndex;
  while (i < this.observerList.length) {
    if (this.observerList[i] === obj) {
      return i;
    }
    i++;
  }
  return -1;
};

ObserverList.prototype.removeAt = function(index) {
  this.observerList.splice(index, 1);
};
```

We have enabled the observerList to add, remove and get an observer. Now we will make it part of the Subject:
```javascript
function Subject() {
  this.observers = new ObserverList();
}

Subject.prototype.addObserver = function(observer) {
  this.observers.add(observer);
};

Subject.prototype.removeObserver = function(observer) {
  this.observers.removeAt(this.observers.indexOf(observer, 0));
};

Subject.prototype.notify = function(context) {
  var observerCount = this.observers.count();
  for (var i = 0; i < observerCount; i++) {
    this.observers.get(i).update(context);
  }
};

```

The skeleton for our observers will look as this:
```javascript
function Observer() {
  this.update = function() {
    // ...
  }
}
```

### The Publish/Subscribe Pattern
This is a common variation on the Observer pattern commonly used in the Javascript world.

The Observer parttern requires the observer wishing to receive topic notifications to subscribe this interest to the object emitting the events (subject).

The Publish/Subscribe (PubSub for short) uses a *topic/event* channel which sits between the objects wishing to receive notifications (subscribers) and the object firing the events (the publisher). This allows us to avoid dependencies between the subscriber and the publisher.

Any subscriber can then register for and receive topic notifications broadcast by the publisher by implementing an appropriate event handler.

```javascript
var unreadCount = 0;

// we can have one subscriber listening to new messages
var subs1 = subscribe("inbox/newMessage", function(topic, data) {
  // * do something with the message
  renderMessage(data);
})

var subs2 = subscribe("inbox/newMessage", function(topic, data) {
  // maybe do something else
  updateMessageCounter(++unreadCount);
});

// and somewhere else in the code
publish("inbox/newMessage", [
  {
    sender: 'hello@google.com',
    body: 'hey there! how is it going?'
  }
])
```

### Advantages
* One of the most important patterns in JavaScript development
* Makes us think about the relationships between different parts of our app
* Helps us identify which layers containing direct relationships can be replaced with subjects + observers
* Break systems down into smaller, loosely coupled blocks.

### Disadvantages
* Harder to guarantee parts of the system function as we expect them to, if we make them too losely coupled
* Subscribers don't know if publishers are working correctly

## Prototype Pattern
Creates objects based on a template of an existing object, through cloning.

In prototypal inheritance we *avoid classes altogether* - We simply create copies of an existing object.

Using the prototype pattern in Javascript is advantageous because prototypal inheritance is a *native feature* of the language. This can aid in performance, as functions are created by reference, rather than creating individual copies on each child.

The standard way to create an object from a prototype in ECMAScript 5 is to use `Object.create`:
```javascript
var myCar = {
  name: "Ford Escort",

  drive: function() {
    console.log("driving");
  },

  panic: function() {
    console.log("stop!");
  }

}
// use Object.create to instantiate a new car
var yourCar = Object.create(myCar);
```

It is possible for objects to inherit from other objects, by using the second argument of `Object.create` which takes an object literal of properties that we want to initialise on the object:

```javascript
var vehicle = {
  getModel: function() {
    console.log("model of the vhicle", this.model);
  }
};

var car = Object.create(vehicle, {
  id: {
    value: MY_GLOBAL.nextId(),
    // writable: false, configurable: false //these are false by default
    enumerable: true
  },

  model: {
    value: "Ford Escort",
    enumerable: true
  }
})
```

This is a rather complex way of establishing prototypal relationships. A simpler way involves overriding the `prototype` property of a function we return from another function:

```javascript
var vehiclePrototype = {
  init: function (carModel) {
    this.model = carModel;
  },

  getModel: function() {
    console.log('our vehicle model is', this.model);
  }
};

function vehicle(model) {
  function F() {};
  F.prototype = vehiclePrototype;

  var f = new F();

  f.init(model);
  return f;
}

var car = vehicle("ford escort");
car.getModel()
```

here `vehicle` emulates a constructor, since the prototype pattern does not include any notion of initialisation beyond linking an object to a prototype.

## Command Pattern
The command pattern encapsulates method invocation, requests or operations into a single object.
It gives us the ability to parameterize and pass method calls around that can be executed at our discretion.
It also enables us to decouple objects invoking the action from the objects implementing them, giving us a greater degree of flexibility in swapping out concrete *classes*

It provides us a means to seaparate the responsabilities of issuing commands from executing commands, enabling us to delegate this responsability.

In the implementation, a simple command binds together an action and the object wishing to invoke the action. All command objects with the same interface can be easily swapped as needed, which is the best benefit of this pattern.

Let's create a car purchasing service
```javascript
(function() {
  var carManager = {
    // request Information
    requestInfo: function (model, id) {
      return "the information for " + model + " width ID " + id + " is foobar";
    },

    // purchase the car
    buyVehicle: function (model, id) {
      return "You have successfully purchased Item " + id + ", a " + model;
    },

    arrangeViewing: function (model, id) {
      return "you have successfully booked a viewing of " + model + " (" + id + ")";
    }
  };
})();
```

While we could execute the methods in `carManager` directly, if the API behind it changes, we would require modifying all of the objects accessing it. Instead, we can abstract the API further

We can expand our `carManager` so that our application of the Command pattern allows us to accept any named methods that can be performed on the `carManager` object, passing along any data to be used:

```javascript
carManager.execute("buyVehicle", "Ford Escort", "234556");
```

In order to do this, we must define an `execute` method as follows:
```javascript
carManager.execute = function (name) {
  return carManager[name] && carManager[name].apply(carManager, [].slice.call(arguments, 1));
}
```

That would allow us to do calls as follows:
```javascript
carManager.execute("arrangeViewing", "Ferrari", "14523");
carManager.execute("requestInfo", "Ford Mondeo", "54323");
carManager.execute("requestInfo", "Ford Escort", "34232");
carManager.execute("buyVehicle", "Ford Escort", "34232");
```

## Mixin Pattern
Mixins are classes which offer funcitonality that can be easily inherited by a subclass or group of subclasses for the purpose of funciton reuse.

In JS, inheriting from Mixins can be seen as a way to collect functionality through extension. With prototypes, we can define properties for any number of object instances. We can use this to promote function re-use.

Mixins allow objects to borrow functionality from them with a minimal amount of complexity, and we can implement this using Javascript's Object prototypes.

Mixins can be seen as objects with attributes and methods than can be shared easily across a number of other object prototypes.
```javascript
var myMixins = {
  moveUp: function() {
    console.log("move up");
  },

  moveDown: function() {
    console.log("move down");
  },

  stop: function() {
    console.log("stop!")
  }
}
```

We can extend the prototype of *existing* constructor functions to include this behaviour using `Object.assign`:
```javascript
// a skeleton carAnimator constructor
function CarAnimator() {
  this.moveLeft = function() {
    console.log("move left");
  };
}

// a skeleton personAnimator constuctor
function PersonAnimator() {
  this.moveRandomly = function () {
    /* ... */
  }
}

// extend our prototypes with our mixin
Object.assign(CarAnimator.prototype, myMixins);
Object.assign(PersonAnimator.prototype, myMixins);

// create a new instance of car Animator
var myAnimator = new CarAnimator();
myAnimator.moveLeft();
myAnimator.moveRight();
myAnimator.stop();
```

this allows us to *mix* in common behaviour into object constructors really easily.

### Advantages
* Mixins allow us to decrease functional repetition and increase function reuse in a system.
* We can avoid duplication when shared behaviour is required across object instances

### Disadvantages
* Some developers feel injecting functionality into a prototype is a bad idea (prototype pollution)
* Strong documentation can assist in minimizing this

## Flyweight Pattern

