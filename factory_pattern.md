We'll be focusing on a family of design patterns we call Factory Patterns. The group includes the Simple Factory, Factory Method and Abstract Factory patterns:

**Simple Factory:** Strictly speaking, it's not a design pattern, but a technique we use very often. It encapsulates the object instantiation process.
**Factory Method:** Defines an interface for creating an object, but let's the classes that implement the interface decide which class to instantiate. The Factory method lets a class defer instantiation to subclasses.
**Abstract Factory:** Provides an interface for creating families of related or dependent objects without specifying their concrete classes.

### Simple Factory

Dragon Inc. is one of the top toy manufacturers in China. In fact, they're a pioneer in toy manufacturing. They started production at a time when few toys were being produced commercially. Hence, they dominated the market and became the leader in the toy production industry.

Their produceToy() function looked like this:
```c
class ToysFactory {
    public function produceToy($toyName) {
        $toy = null;
        if ('car'==$toyName) {
            $toy = new Car();
        } else if ('helicopter'==$toyName) {
            $toy = new Helicopter();
        }

        $toy->prepare();
        $toy->package();
        $toy->label();

        return $toy;
    }
}
```

Initially, they only manufactured toy Cars and Helicopters. For this simple task the function worked well and everyone was happy. But not long after that, a cool new toy, the “Jumping Frog,” was introduced by the design team. Jumping Frog looked cool and they knew it was going to sell really well, it was time to change the produceToy() function:

```c
class ToysFactory {
    public function produceToy($toyName) {
        $toy = null;

        if ('car'==$toyName) {
            $toy = new Car();
        } else if ('helicopter'==$toyName) {
            $toy = new Helicopter();
        } else if ('jumpingFrog'==$toyName) {
            $toy = new JumpingFrog();
        }

        $toy->prepare();
        $toy->package();
        $toy->label();

        return $toy;
    }
}
```

As the business grew, more and more toys came into production and the CEO was very happy with the business' financial growth. However, the development team's office the nightmare was just beginning. The developers were tasked to modify the produceToy() function with the introduction of every new toy. It has violated the open/close principle. Which states “software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification”. Every new toy brought in modifications of the produceToy() function.


**It was time for refactoring. Let’s take a look at the situation here. What was the real issue with produceToy function? A concrete class was instantiated inside ToysFactory and messed up the produceToy() function. The ToysFactory class was tied to the concrete classes of toys.** Let's the issue.

First, let's create a class to encapsulate the concrete class instantiation.

```c
class SimpleFactory {
    public function createToy($toyName) {
        $toy = null;

        if ('car'==$toyName) {
            $toy = new Car();
        } else if ('helicopter'==$toyName) {
            $toy = new Helicopter();
        }

        return $toy;
    }
}
```


```c
class ToysFactory {

    public $simpleFactory;

    public function __construct(SimpleFactory $simpleFactory) {
        $this->simpleFactory = $simpleFactory;
    }

    public function produceToy($toyName) {
        $toy = null;

        $toy = $this->simpleFactory->createToy($toyName);

        $toy->prepare();
        $toy->package();
        $toy->label();

        return $toy;
    }
}
```

With our brand new class SimpleFactory, things just became uncomplicated. The devs no longer need to touch ToysFactory when a new toy is introduced, and they'll just use SimpleFactory to produce a new toy instead of using the former messy code.

This is the Simple Factory. It's not a real design pattern per se, but it's a useful technique that you can apply to your own needs. With Simple Factory, concrete class instantiation is encapsulated. It decouples the client code from the object creation code.




## Factory Method

>  Defines an interface for creating an object, but lets classes that > implement the interface decide which class to instantiate. The Factory > method lets a class defer instantiation to subclasses.


With the Simple Factory in place, developers are now enjoying their day at Dragon Inc. Despite the thorough discussion of toys, we haven't really looked at the toy classes yet. Toy is an abstract class with the functions prepare(), package() and label().

```c
abstract class Toy {
    public $name  = '';
    public $price  = 0;

    public function prepare() {
        echo $this->name. ' is prepared';
    }

    public function package() {
        echo $this->name. ' is packaged';
    }

    public function label(){
        echo $this->name . ' is priced at '.$this->price;
    }
}
```


The concrete classes Car and Helicopter inherit from super class Toy. They're pretty straightforward.

Car:
```php
class Car extends Toy {
    public $name  = 'Car';
    public $price = 20;
}
```
Helicopter:
```php
class Helicopter extends Toy {
    public $name  = 'Helicopter';
    public $price = 100;
}
```

Back to Dragon Inc. The CEO walks in to the developers' office with a smile on his face, but we know there's bad news coming. The CEO happily announces that Dragon Inc. is going to open several factories in the US. They'll be located in different states, and the first two will be in New York and California. All toys will be produced locally with their own properties, which means that for the same type of toy Car, the ones produced in New York will be NyCar and those from California will be CaCar. Simple Factory will make this task a cinch for the development team. All they need to do is create a location specific SimpleFactory class and a location specific ToysFactory class. Simple Factory simplifies the task and makes the developers' job easy.

For example, for New York we could do:


```c
class NySimpleFactory {
    public function createToy($toyName) {
        $toy = null;

        if ('car'==$toyName) {
            $toy = new NyCar();
        } else if ('helicopter'==$toyName) {
            $toy = new NyHelicopter();
        }

        return $toy;
    }
}

class NyToysFactory {

    public $simpleFactory;

    public function __construct(SimpleFactory $simpleFactory) {
        $this->simpleFactory = $simpleFactory;
    }

    public function produceToy($toyName) {
        $toy = null;
        $toy = $this->simpleFactory->createToy($toyName);
        $toy->prepare();
        $toy->package();
        $toy->label();
        return $toy;
    }
}
```

The developers finish the new code quickly and hand it over to the US factories. After two weeks, the phone starts ringing in the developers’ office because the New York factory was having production issues. It turns out that the NyToysFactory class has been modified by developers at the remote branch because the staff doesn't want to do packaging and labeling work. They've modified produceToy() by removing its label() and package() functions.

It seems like Simple Factory won't work in this scenario. We don't want branches in US to be able to modify produceToy() functions. ProduceToy() should consist of a set of standard procedures and the branches should only be responsible for creating location specific toys. What if they can create an abstract class? And the abstract class they create will have a concrete produceToy() method which will implement a set of standard operating procedurea that all branches have to follow. Inside produceToy(), it calls its own abstract method createToy() to obtain a toy class. This way createToy() is able to encapsulate object creation and, since it's abstract, it delegates the creation to its subclasses.


That sounds like exactly what they need in their case:
```c
abstract class ToysFactory {

    public function produceToy($toyName) {
        $toy = null;
        $toy = $this->createToy($toyName);
        $toy->prepare();
        $toy->package();
        $toy->label();
        return $toy;
    }

    abstract public function createToy($toyName);
}
```
Now in the New York branch, all they need to do is to implement the createToy() method in their the subclass:

```c
class NyToysFactory extends ToysFactory {

    public function createToy($toyName) {
        $toy = null;

        if ('car'==$toyName) {
            $toy = new NyCar();
        } else if ('helicopter'==$toyName) {
            $toy = new NyHelicopter();
        }

        return $toy;
    }
}
```

No the toys factory implements its own createToy function and the produceToy method is in the abstract class ToysFactory.


In the code above, the function createToy() in the ToysFactory class is called a factory method. Factory Method pattern defines an interface(createToy) for creating an object. But it delegates the actual creation to subclasses(NyToysFactory and CaToyFactory).



#### Abstract factory


An abstract Factory provides a common interface for creating families of related objects together.
The client object does not bother to build objects directly, but it calls the methods provided by this common interface.

Below is showed one possible implementation of an abstract Factory and its concrete Factories that implement it.

Suppose we have two categories of games as a model classes:
```ruby
#models.rb
class Game
  attr_accessor :title
  def initialize(title)
    @title = title
  end
end

class Rpg < Game
  def description
    puts "I am a RPG named #{@title}"
  end
end

class Arcade < Game
  def description
    puts "I am an Arcade named #{@title}"
  end
end
```


As we can see, both models derive from a common superclass Game.

Let’s define the Factories delegate to build these objects:

```ruby
#factories.rb
module MyAbstractGameFactory
  def create(title)
    raise NotImplementedError, "You should implement this method"
  end
end

class RpgFactory
  include MyAbstractGameFactory
  def create(title)
    Rpg.new title
  end
end

class ArcadeFactory
  include MyAbstractGameFactory
  def create(title)
    Arcade.new title
  end
end
```

Note that we have defined the abstract factory (MyAbstractGameFactory) as a module: it defines the abstract method that must be implemented by the class that includes it.
RpgFactory and ArcadeFactory represent the two concrete factories responsible to build, respectively, Arcade and RPG games.



#### More on abstract factories:

An Abstract Factory is simply an abstract interface for concrete Factory objects to conform to, this pattern pretty much falls away. This is perhaps easier to show than to explain, so I will go ahead and build out a Ruby version of the example shown in the wikipedia article I've linked to above.

```ruby
module OSXGuiToolkit
  extend self

  def button
    Button.new
  end

  class Button
    def paint
      puts "I'm an OSX Button"
    end
  end
end

module WinGuiToolkit
  extend self

  def button
    Button.new
  end

  class Button
    def paint
      puts "I'm a WINDOWS button"
    end
  end
end

class Application
  def initialize(gui_toolkit)
    button = gui_toolkit.button
    button.paint
  end
end

# this check is a very quick hack, not reliable.
if PLATFORM =~ /darwin/
  Application.new(OSXGuiToolkit)
else
  Application.new(WinGuiToolkit)
end
```

In the above example, I think the Application class is the factory which causes any gui_toolkil module to conform to it. Any gui_toolkit needs to implement a button method which then needs to return the module's own Button class.

In this example, you'll see that we've eliminated the explicit Abstract Factory interface. Instead, what we've done is created two concrete object factories, OSXGuiToolkit and WinGuiToolkit, that implement a common API. We then create a simple Application stub class which shows that the GUI toolkit factory should be injected into the Application.




#### More on FActory pattern:


In Factory pattern, we create object without exposing the creation logic to the client and refer to newly created object using a common interface.

This following examples involves a common interfact called Shape, concrete shape objects, a shape factory which has the creation logic, and a factory class called FactoryPatternDemo that uses the factory and has no exposure to the creation logic and the object all have a common interface.

The interface:

```java
public interface Shape {
   void draw();
}
```

Some shapes:

```java
public class Rectangle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Rectangle::draw() method.");
   }
}
```

```java
public class Square implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Square::draw() method.");
   }
}
```

```java

public class Circle implements Shape {

   @Override
   public void draw() {
      System.out.println("Inside Circle::draw() method.");
   }
}
```

ShapeFactory:

```java
public class ShapeFactory {

   //use getShape method to get object of type shape
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }		
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();

      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();

      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }

      return null;
   }
}
```


FactoryPatternDemo

```java
public class FactoryPatternDemo {

   public static void main(String[] args) {
      ShapeFactory shapeFactory = new ShapeFactory();

      //get an object of Circle and call its draw method.
      Shape shape1 = shapeFactory.getShape("CIRCLE");

      //call draw method of Circle
      shape1.draw();

      //get an object of Rectangle and call its draw method.
      Shape shape2 = shapeFactory.getShape("RECTANGLE");

      //call draw method of Rectangle
      shape2.draw();

      //get an object of Square and call its draw method.
      Shape shape3 = shapeFactory.getShape("SQUARE");

      //call draw method of circle
      shape3.draw();
   }
}
```


#### From wiki:


In class-based programming, the factory method pattern is a creational pattern that uses factory methods to deal with the problem of creating objects without having to specify the exact class of the object that will be created. This is done by creating objects by calling a factory method—either specified in an interface and implemented by child classes, or implemented in a base class and optionally overridden by derived classes—rather than by calling a constructor.

> "Define an interface for creating an object, but let subclasses decide which class to instantiate. The Factory method lets a class defer instantiation it uses to subclasses."(Gang Of Four)


Creating an object often requires complex processes not appropriate to include within a composing object. The object's creation may lead to a significant duplication of code, may require information not accessible to the composing object, may not provide a sufficient level of abstraction, or may otherwise not be part of the composing object's concerns. The factory method design pattern handles these problems by defining a separate method for creating the objects, which subclasses can then override to specify the derived type of product that will be created.

**Example in Java:**

A maze game may be played in two modes, one with regular rooms that are only connected with adjacent rooms, and one with magic rooms that allow players to be transported at random (this Java example is similar to one in the book Design Patterns). The regular game mode could use this template method:


```java
public abstract class MazeGame {
    public MazeGame() {
        Room room1 = makeRoom();
        Room room2 = makeRoom();
        room1.connect(room2);
        this.addRoom(room1);
        this.addRoom(room2);
    }

    abstract protected Room makeRoom();
}
```

In the above snippet, the MazeGame constructor is a template method that makes some common logic. It refers to the makeRoom factory method that encapsulates the creation of rooms such that other rooms can be used in a subclass. To implement the other game mode that has magic rooms, it suffices to override the makeRoom method:

```java
public class MagicMazeGame extends MazeGame {
    @Override
    protected Room makeRoom() {
        return new MagicRoom();
    }
}

public class OrdinaryMazeGame extends MazeGame {
    @Override
    protected Room makeRoom() {
        return new OrdinaryRoom();
    }
}

MazeGame ordinaryGame = new OrdinaryMazeGame();
MazeGame magicGame = new MagicMazeGame();
```

So in this code, the MazeGame implements the factory method pattern. The MazeGame constructor in the class has common logic and it refers to a makeRoom method that does not specify the exact class being created.

MagicMazeGame is a subclass that actually creates the object.

The factory pattern is specified in an interface and implemented by child classes.

In this pattern, when the new MagicMazeGame is called, the constructor MazeGame is called which then calls teh makeRoom method which creates the actual rooms. The constructor MazeGame creates the room objects without having to specify the exact class being created. 
