# Design Patterns

## Que es un Design Pattern?

Patterns (patrones) son soluciones a problemas que pueden ser reutilizadas fácilmente, como plantillas o recetas para problemas. Sin embargo, no son soluciones exactas. Nos dan sin embargo un plan de solución.

Ventajas
* Evitan problemas comunes
* Proveen soluciones generalizadas
* Disminuyen el tamaño total de nuestro código
* Añaden al vocabulario de un developer
* Los patterns evolucionan y mejoran


## Patrón contra Anti-Patrón
Patterns son "practicas estandard", Anti-Patterns son lecciones aprendidas.

* Describen una mala solución a un problema la cual creó una mala situación.
* Describen cómo salir de la situación y llegar a una buena solución.

> Un mal diseño que merece la pena documentar

### Anti-Patterns en JavaScript
* Polución del scope global
* Usar strings como argumentos a `setInterval` para invocar `eval()` internamente
* Modificar el prototipo de `Object` 💩 NUNCA HAGAS ESTO
* Usar JS en un formulario inline
* Usar `document.write` en lugar de `document.createElement`

## Cagetorias de Design Pattern
* Creational: Patterns que se encargan de la creación de objetos: (Constructor, Factory, Abstract, Prototype, Singleton, Builder)
* Structural: Patterns que se encargan de la composición de objetos: (Decorator, Facade, Flyweight, Adapter, Proxy)
* Behavioural: Patterns que se encargan de la comunicación entre objetos y su comportamiento: (Iteratior, Mediator, Observer, Visitor)


# Design Patterns

A continuación veremos una selección de los Patterns mas importantes.

## Constructor
Un Constructor es un método usado para inicializar un objecto una vez que se ha creado un espacio en memoria para el objeto. Javascript no tiene clases de manera nativa, pero tiene funciones especiales que trabajan con objetos.

```javascript
var newObject = {};

var newObject = Object.create(Object.prototype);

var newObject = new Object();
```

El último ejemplo es un constructor, que se llama con el keyword `new`

Hay 4 maneras de definer las propiedades de un objeto en Javascript
```javascript
// 1. Usando el 'dot' syntax:
newObject.myKey = 'someValue';

// 2. Usando los []
newObject['myKey'] = 'someValue';

// 3. Usando el metodo Object.defineProperty (ES5+)
Object.defineProperty(newObject, 'myKey', {
  value: 'someValue',
  writable: true,
  enumerable: true,
  configurable: true
});

// 4. O en bloque usando Object.defineProperties (ES5+)
Object.defineProperties(newObject, {
  'myKey' : {
    value: 'someValue',
    writable: true
  },

  'anotherKey' : {
    value: 'someOtherValue',
    writable: false
  }
});
```

### Constructor Básico
Si llamamos a una funcion con la keyword `new`, le decimos a JS que queremos que la función se comporte como un *constructor*. Dentro del constructor, la palabra clave `this` hace referencia al objecto que estamos creando.

```javascript
function Coche (modelo, año, km) {

  this.modelo = modelo;
  this.año = año;
  this.km = km;

  this.toString = function () {
    return this.modelo + " tiene " + this.km + " km";
  };
}

// Uso:

// Podemos crear nuevas instancias de 'Coche'
var civic = new Coche("Honda Civic", 2009, 20000);
var mondeo = new Coche("Ford Mondeo", 2010, 5000);

civic.toString(); // -> "Honda Civic tiene 20000 km"
mondeo.toString(); // -> "Ford Mondeo tiene 5000 km"
```


### Constructor con Prototype
El anterior ejemplo redefine el método `toString` cada vez que creamos un objeto. Sería mejor compartir esta funcionalidad con todas las instancias de `Coche`

En javascript, las funciones, como la mayoría de objetos, contienen un objeto llamado `prototype`. Cuando invocamos un constructor en JS para crear un Objeto, todas las propiedades del objeto estarán disponibles en el nuevo objeto.

```javascript
function Coche (modelo, año, km) {
  this.modelo = modelo;
  this.año = año;
  this.km = km;
}

Coche.prototype.toString = function() {
  return this.modelo + ' tiene ' + this.km + ' km';
}
```

En este caso, todos las las instancias de Coche compartirán una referencia a *la misma* función `toString`

## Module
Se definen como una manera de crear *encapsulación* privada y pública, comparable a las clases en lenguajes convencionales.

Como JS no tiene clases de manera nativa, usamos módulos para emularlas.

# Usando Scope para emular Privacidad
Un Módulo encapsula la idea de "privacidad", estado y organización mediante *closures*. De esta manera, solo exponemos una API Pública, manteniendo el resto de métodos privados.

Sin embargo Javascript, a diferencia de otros lenguajes orientados a objetos, no tiene concepto de `private` o `public`, así que usamos scope para emularlo:

```javascript
var myNamespace = (function() {

  // esta variable es privada
  var myPrivateVar = 0;

  // este método también
  var myPrivateMethod = function(foo) {
    console.log(foo);
  };

  return {

    // Una variable pública:
    myPublicVar: 'foo',

    // un método publico que usa cosas privadas
    myPublicFunction: function(bar) {

      // Incrementa el contador privado
      myPrivateVar++;

      // llama a nuestro método privado usando bar
      myPrivateMethod(bar);
    }
  };

})(); //<-- Invocamos la función directamente
```

### Ventajas:
* Facilita la transición a JS para programadores de otros lenguajes Orientados a objetos que echan de menos las Classes.
* Provee soporte a datos privados

### Desventajas
* Los miembros privados y públicos se acceden de manera distinta, por tanto si queremos cambiar la visibilidad de un campo, necesitamos cambiar todas las referencias en las que éste se use.
* No podemos acceder a los miembros privados en métodos añadidos a nuestro módulo despés de su creación.
* No es posible escribir unit tests para miembros privados.
* Los miembros privados no pueden ser parcheados, así que si hay que reparar un bug, debemos modificar todos los miembros públicos.

[más información sobre Module 🇬🇧](http://www.adequatelygood.com/2010/3/JavaScript-Module-Pattern-In-Depth)

## Singleton Pattern
Restringe la instanciación de una clase a *un solo objeto*. Esto se implementa creando una clase con un método que crea una nueva instancia de la clase si la instancia no existe. Si la instancia ya ha sido creada, retornamos una referencia a ella.

Los singletons son útiles cuando necesitamos retrasar la inicialización de un objeto hasta que recibamos información requerida

```javascript
var mySingleton = (function () {
  // esta variable guarda una referencia al Singleton
  var instance;

  function init() {

    // Singleton
    // Metodos privados
    function privateMethod() {
      console.log("soy Privado");
    }

    var privateVariable = "yo también soy privada";
    var privateRandom = Math.random();

    return  {
      // Métodos públicos y variables
      publicMethod: function () {
        console.log( "el público me puede ver!");
      },
      publicProperty: "y a mi también",

      getRandomNumber: function() {
        return privateRandomNumber;
      }
    };
  };

  return {
    // aqui esta la clave =
    // Retornamos una referencia a la instancia del
    // singleton si ya existe. Si no, creamos un nuevo
    // singleton
    getInstance: function() {
      if (!instance) {
        instance = init();
      }
      return instance;
    }
  };

})();
```

Obtenemos acceso a los métodos del singleton llamando al m´todo `MySingleton.getInstance()`. La diferencia entre una instancia estática de una clase (un objeto) y un Singleton reside en que mientras un singleton puede ser implementado como una instancia estática; tambien puede ser construido *lazily*, osea, sólo en el momento en el que sea necesario.

Los Singletons son útiles cuando necesitamos *exactamente un objeto* para coordinar otros objetos en un system.

De todas maneras, si necesitas un Singleton, puede ser que tengas que reevaluar tu diseño.


[Más información sobre Singletons 🇬🇧](http://www.ibm.com/developerworks/webservices/library/co-single/index.html)

## Observer Pattern
Un Observer es un patrón en el que un objeto (llamado *Sujeto*) mantiene una lista de objetos que dependen de él (llamados *Observadores*), notificándoles automáticamente de cualquier cambio en el estado.

Si pasa algo interesante, el Sujeto puede notificar a todos los Observadores, emitiendo una notificación que puede incluir datos relacionados con el tema de la notificación.

Cuando un Observador ya no necesita (o desea) ser notificado de los cambios en el estado del Sujeto, éste último puede quitar al Observador de su lista de Observadores.

#### Gang of Four (GoF) definition:
> "One or more observers are interested in the state of a subject and register their interest with the subject by attaching themselves. When something changes in our subject that the observer may be interested in, a notify message is sent which calls the update method in each observer. When the observer is no longer interested in the subject's state, they can simply detach themselves."

### Implementando el Observer Pattern
Para implementar este patrón, necesitamos los siguientes componentes:
* **Subject**: Mantiene una lista de Observadores, y los añade o quita.
* **Observer**: Nuestro objeto tiene que implementar una interfaz para recibir notificaciones de cambio de estado en un Sujeto, por ejemplo, un *callback*
* **ConcreteSubject**: Emite notificaciones a los Observadores sobre los cambios de estado, y guarda el estado de los ConcreteObservers
* **ConcreteObserver**: Guardia una referencia al ConcreteSubject e implementa una interfaz de `update` que sea consistente con la del Sujeto

#### Observer List
```javascript
// lo primero que debemos construir es una lista de Observadores que nos permita añadir, quitar y buscar observadores:

function ObserverList() {
  this.observerList = [];
}

ObserverList.prototype.add = function(obj) {
  return this.observerList.push(obj);
}

ObserverList.prototype.count = function() {
  return this.observerList.length;
}

ObserverList.prototype.get = function(index) {
  if (index > -1 && index < this.observerList.length) {
    return this.observerList[index]:
  }
}

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

Nuestro `ObserverList` nos permite añadir, quitar y obtener nuestros `Observers`. Ahora lo haremos parte de nuestro `Subject`:
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
  for (var i = 0; i < observercount; i++) {
    this.observers.get(i).update(context);
  }
};
```

El esqueleto de nuestros Observers funciona de esta manera:
```javascript
function Observer() {
  this.update = function() {
    // ...
  }
}
```

### Publish/Subscribe
Esta es la variante más popular de el patron Observer en el mundo de Javascript.

El patrón de Observer require que cualquier Observador que desee recibir notificaciones de un tópico se tenga que subscribir al objeto que emita los eventos (Sujeto).

El patron Publish/Subscribe (PubSub) usa un canal de *topic/event* que se sitúa entre los objetos que desean recibir notificaciones (los *subscriptores*) y el objeto que emite los eventos (el *publicador*). Esto nos permite evitar dependencias entre los subscriptores y los publicadores.

Cualquier subscriptor se puede entonces registrar y recibir notificaciones emitidas por el publicador implementando un event handler apropiado:

```javascript
var noLeidos = 0;

// podemos tener un subscriptor atento a cuando lleguen nuevos mensajes
// y que se encargue de renderizarlos en la pantalla:
var subs1 = subscribe("inbox/mensajeNuevo", function(sujeto, datos) {
  // ** haz algo con el mensaje
  renderizaMensaje(datos);
});

// y otro subscriptor que se encargue de actualizar el contador de
// mensajes no leidos
var subs2 = subscribe("inbox/mensajeNuevo", function(sujeto, datos) {
  // actualiza el contador
  actualizarContador(++unreadCount);
});

// y luego en otra parte de nuestro código:
publish("inbox/mensajeNuevo", [
  {
    sender: 'hello@google.com',
    body: 'hola! que tal?'
  }
]);
```

### Ventajas
* Es uno de los patrones más importantes en Javascript
* Nos ayuda a pensar en las relaciones entre las diferentes partes de nuestro app
* Nos ayuda a identificar qué partes de nuestro sistema que contienen relaciones directas entre objetos pueden ser reemplazadas con sujetos + observadores
* Nos ayuda a dividir nuestro sistema en partes más pequeñas, testables y no co-dependientes.

### Desventajas
* Es más dificil garantizar que partes del sistema funcionen de manera correcta si las hacemos demasiado independientes.
* Los subscriptores no saben si los publicadores estan funcionando correctamente


## Prototype Pattern
Crea objetos basados en un patrón de un objeto existente, clonandolo.

En prototypal inheritance evitamos las clases totalmente. En lugar de clases, simpelmente creamos copias de un objeto ya existente.

Prototypal Inheritance es una propiedad nativa de Javascript, asi que usar el Prototype pattern es fácil. Esto nos puede ayudar con el rendimiento de nuestras apps, ya que las funciones pueden ser creadas por referencia en lugar de crear copias individuales.

La manera estándar de crear objetos a partir de un prototipo en ECMAScript 5 es usando `Object.create`:
```javascript
var miCoche = {
  nombre: "Ford Escort",

  conducir: function() {
    console.log("conduciendo!");
  },

  frenar: function() {
    console.log("stop!");
  },
}
// usa Object.create para instanciar un nuevo coche
var tuCoche = Object.create(miCoche);
```

Es posible hacer que objetos hereden de otros objetos, usando el segundo argumento de `Object.create`, el cual toma un objeto literal con las propiedades que queremos inicializar en el objeto:

```javascript
var vehiculo = {
  getModelo: function() {
    console.log("Modelo del vehículo", this.modelo);
  }
};

var coche = Object.create(vehiculo, {
  id: {
    value: MY_GLOBAL.nextId(),
    // writable: false, configurable: false <-- false by default
    enumerable: true
  },

  modelo: {
    value: "Ford Escort",
    enumerable: true
  }
});
```

Esta es una manera un tanto compleja de establecer relaciones de prototipo. Una manera más simple (y popular) es cambiar la propiedad `prototype` de una función que retornamos desde otra función:
```javascript
var vehiculoPrototipo = {
  init: function(modelo) {
    this.modelo = modelo;
  },

  getModelo: function() {
    console.log("Modelo del vehículo: " + this.model);
  }
};

function vehiculo(modelo) {
  function F() {}
  F.prototype = vehiculoPrototipo;

  var f = new F();

  f.init(modelo);
  return f;
}

// lo usamos asi:
var coche = vehiculo("Ford Escort");
coche.getModelo();
```

En este caso `vehiculo` emula a un constructor, ya que el Prototype Pattern no include ninguna noción de inicialización mas alla de unir un objeto a un prototipo.

## Command Pattern
El patrón de Comando encapsula la invocación de un método, request o operación dentro de un sólo objeto. Nos da la habilidad de parametrizar y pasar llamadas a métodos que puedan ser ejecutadas a nuestra discrección.

Tambien nos permite desenlazar el objeto que invoca una accion del objeto que la implementa, lo que nos da un mayor grado de flexibilidad a la hora de cambiar clases concretas (e.j. objetos).

Nos aporta un medio de separar las responsabilidades de emitir comandos de la responsabilidad de ejecutarlos, permitiendonos delegar esta responsabilidad.

En nuestra implementación, un comando simple une una acción con el objeto que la quiere invocar. Todos los objetos de comando con la misma interfaz pueden ser cambiados como se necesite. Esta es la principal ventaja de este pattern.

Como ejemplo vamos a imaginarnos que estamos haciendo un juego. En todos los juegos hay un trozo de código que lee inputs del usuario (botones, eventos de teclado, clicks, etc). Toma cada input y lo traduce a una acción concreta en el juego:

![input-image](http://gameprogrammingpatterns.com/images/command-buttons-one.png) (from Game Programming Patterns)

Una implementación simple de esto sería:

```javascript
handleInput() {
  if      (isPressed(BUTTON_X)) jump();
  else if (isPressed(BUTTON_Y)) fireGun();
  else if (isPressed(BUTTON_A)) swapWeapon();
  else if (isPressed(BUTTON_B)) crouch();
}
```

Esta función se llama típicamente una vez por frame en nuestro [game loop](http://gameprogrammingpatterns.com/game-loop.html). El problema con esto es que no podemos configurar el mapeado de los botones.

Para cambiar esto, podemos convertir nuestras llamadas directas a `jump()` y `fireGun()` en cosas que podamos cambiar (i.e. asignar, como una variable).

Para ello, definimos una interfaz `Command`:
```javascript
var Command = function() {
  this.execute = function() {};
}
```

Por ejemplo, podemos crear comandos para cada una de las acciones de nuestro juego, que implementen la misma interfaz (i.e. que tengan la propiedad 'execute')
```javascript
function JumpCommand() {
  this.execute = function() {
    jump();
  }
}

function FireCommand() {
  this.execute = function() {
    fireGun();
  }
}
```

Entonces, podemos modificar nuestro `inputHandler` para tener, para cada botón, un pointer a un comando:
```javascript
function InputHandler () {

  var botones = {
    X : new Command(),
    Y : new Command(),
    A : new Command(),
    B : new Command()
  };

  return {
    handleInput() {
      if      (isPressed(BUTTON_X)) botones.X.execute();
      else if (isPressed(BUTTON_Y)) botones.Y.execute();
      else if (isPressed(BUTTON_A)) botones.A.execute();
      else if (isPressed(BUTTON_B)) botones.B.execute();
    },

    setButton(boton, comando) {
      // nos aseguramos de que `comando` implementa la interfaz
      // de Command
      if (!comando.hasOwnProperty('execute')) return null
      botones[boton] = comando;
    }
  }
}
```

Como podeis ver, hemos desencajado los botones de las funciones del juego, lo que nos da una capa de abstraccion.

El patrón de Command es básico si queremos implementar la opción de deshacer `undo`. Para eso extendemos la interfaz de command para tener una función `undo` que desahaga la acción del comando.
```javascript
function Command () {

  this.execute = function() {};
  this.undo = function() {};

}
```

Por ejemplo si estas implementando un editor, podriamos manterer una lista de comandos, y si quieres deshacer, pues simplemente ejecutas la función de deshacer del ultimo comando:
```javascript
function CommandHistory() {
  this.undoHistory = [];
  this.redoHistory = [];

  this.performCommand = function(command) {
    undoHistory.push(command);
    command.execute();
  }

  this.undo = function() {
    var lastCommand = undoHistory.pop();
    if (!lastCommand) return null;
    lastCommand.undo();
    redoHistory.push(lastCommand);
  }

  this.redo = function() {
    var lastCommand = redoHistory.pop();
    if (!lastCommand) return null;
    lastCommand.execute();
    undoHistory.push(lastCommand);
  }

}
```
