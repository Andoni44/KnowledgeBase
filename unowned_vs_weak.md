# Unowned vs weak

Todos sabemos lo importante que es evitar ciclos de retención, es decir, que un objeto *A* tenga una referencia fuerte hacia un objeto *B* y viceversa.  Para conseguir esto, Swift nos facilita dos herramientas muy parecidas, **unowned** y **weak**.

Cuando nombramos un objeto como débil usando weak o unowned, por ejemplo, dentro de una relación de composición(la referencia débil siempre la tendrá el objecto que es poseído hacia el poseedor), lo que estamos consiguiendo es que no aumente el contador de referencias, con lo cual, si el objeto padre es borrado de memoria, también podrá ser borrado de forma automática el objeto hijo. Pero, ¿cuál es la diferencia real entre estas dos instrucciones? La principal ventaja de usar **unowned** es que no tenemos que tratar con *optionals*, por lo tanto el código se vuelve más limpio, pero también más peligroso y es que tenemos que tener en cuenta que de una variable **unowned** siempre se espera que haya valor, es decir, que nunca valga *nil*. Esto último se traduce en que cuando tengamos una relación de composición debemos pensar siempre que el objeto hijo vaya a durar en momoria siempre lo mismo o menos que el padre, por lo tanto, nunca se dará la situación de tener un objeto hijo con una referencia nil hacia su padre.  Por otro lado, usando weak debemos tratar con optionals haciendo los unwrapping necesarios, pero también nos aseguramos de que el programa no rompa por intentar acceder a una variable sin valor.

En resumen, se debería usar siempre **weak** para romper ciclos de retención a no ser que tengamos claro que no tiene sentido que pueda existir un objeto hijo sin su padre, con lo que podríamos usar unowned y evitar ensuciar el código con opcionales.

A continuación un ejemplo de relación de composición con **unowned** y con **weak**. Se han definido dos clases, *Car* y *Driver*. Como se ha definido que no puede existir un conductor sin un coche asociado pero sí un coche sin conductor, o sea *Car* posee a *Driver*, podremos usar perfectamente **unowned** dado que *Driver* dejará de existir cuando lo haga *Car*. También podemos usar weak, pero para acceder a valores del coche desde conductor, necesitaríamos hacer unwrappings.

## Example using unowned

```Swift
// Owns a Driver
final class Car {
    
    let plateNumber: String
    let brand: String
    let model: String
    var color: String?
    var driver: Driver?
    
    init(plateNumber: String, brand: String, model: String) {
        self.plateNumber = plateNumber
        self.brand = brand
        self.model = model
    }
    
    deinit {
        print("Car \(plateNumber) erased")
    }
}

// Owned by a Car
final class Driver {
    
    let name: String
    let idNumber: String
    unowned let car: Car
    
    init(name: String, idNumber: String, car: Car) {
        self.name = name
        self.idNumber = idNumber
        self.car = car
    }
    
    deinit {
        print("Driver \(idNumber) erased")
    }
}

var car: Car? = Car(plateNumber: "1980JTB", brand: "Opel", model: "Ascona")
car?.driver = Driver(name: "Laura", idNumber: "778899RK", car: car!)
car = nil
/* Prints: 
* Car 1980JTB erased
* Driver 778899RK erased
*/
```

## Excample using weak

```Swift
// Owns a Driver
final class Car {
    
    let plateNumber: String
    let brand: String
    let model: String
    var color: String?
    var driver: Driver?
    
    init(plateNumber: String, brand: String, model: String) {
        self.plateNumber = plateNumber
        self.brand = brand
        self.model = model
    }
    
    deinit {
        print("Car \(plateNumber) erased")
    }
}

// Owned by a Car
final class Driver {
    
    let name: String
    let idNumber: String
    weak var car: Car?
    
    init(name: String, idNumber: String, car: Car) {
        self.name = name
        self.idNumber = idNumber
        self.car = car
    }
    
    deinit {
        print("Driver \(idNumber) erased")
    }
}

var car: Car? = Car(plateNumber: "1980JTB", brand: "Opel", model: "Ascona")
car?.driver = Driver(name: "Laura", idNumber: "778899RK", car: car!)
car = nil
/* Prints:
* Car 1980JTB erased
* Driver 778899RK erased
*/

```
