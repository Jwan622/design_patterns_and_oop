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



#### More on factory pattern:

The key detail that I have been glossing over until now is the one where your code magically knows which class to pick at some critical point.

In this chapter, we will look at both of these GoF patterns: the Factory Method pattern and the Abstract Factory pattern. We will also shine our light on some dynamic Ruby techniques that will help us build factories more effectively.

**A Different Kind of Duck Typing**

It seems like factories need to implement duck typing in the concrete objects because the factory is probably going to then call a common method on the object that is eventually created, regardless of the underlying object.

```ruby
class Duck
  def initialize(name)
    @name = name
  end

  def eat
    puts("Duck #{@name} is eating.")
  end

  def speak
    puts("Duck #{@name} says Quack!")
  end

  def sleep
    puts("Duck #{@name} sleeps quietly.")
  end
end
```


As you can see from this code, ducks—like most animals—eat, sleep, and make noise. But ducks also need a place to live, and for that you build a Pond class:


```ruby
class Pond
  def initialize(number_ducks)
    @ducks = []
    number_ducks.times do |i|
      duck = Duck.new("Duck#{i}")
      @ducks << duck
    end
  end

  def simulate_one_day
    @ducks.each {|duck| duck.speak}
    @ducks.each {|duck| duck.eat}
    @ducks.each {|duck| duck.sleep}
  end
end
```

Running the pond simulation is not much of a challenge:

```ruby
pond = Pond.new(3)
pond.simulate_one_day
```


The preceding code simulates one day in the life of a three-duck pond, and it ­produces the following output:

```
Duck Duck0 says Quack!
Duck Duck1 says Quack!
Duck Duck2 says Quack!
Duck Duck0 is eating.
Duck Duck1 is eating.
Duck Duck2 is eating.
Duck Duck0 sleeps quietly.
Duck Duck1 sleeps quietly.
Duck Duck2 sleeps quietly.
```


Life on the pond continues idyllically until one dark day when you get a request to model a different denizen of the puddle: the frog. Now it is easy enough to create a Frog class that sports exactly the same interface as the ducks:

```ruby
class Frog
  def initialize(name)
    @name = name
  end

  def eat
    puts("Frog #{@name} is eating.")
  end

  def speak
    puts("Frog #{@name} says Crooooaaaak!")
  end

  def sleep
    puts("Frog #{@name} doesn't sleep; he croaks all night!")
  end
end
```


But there is a problem with the Pond class—right there in the initialize method you are explicitly creating ducks:

```ruby
  def initialize(number_ducks)
    @ducks = []
    number_ducks.times do |i|
      duck = Duck.new("Duck#{i}")
      @ducks << duck
    end
  end
```
The trouble is that you need to separate out something that is changing—the specific creatures that inhabit the pond (duck or frog)—from something that is staying the same—the other workings of the Pond class. If only you could somehow excise that Duck.new from the Pond class, then the Pond class could support both ducks and frogs. This dilemma brings us to the central question of this chapter: Which class do you use?


**One solution is to use the template method**

One way to deal with the “which class” problem is to push the question down onto a subclass. We start by building a generic base class, a class that is generic in the sense that it does not make the “which class” decision. Instead, whenever the base class needs a new object, it calls a method that is defined in a subclass. For example, we could recast our Pond class as shown below so that it relies on a method called new_animal to produce the inhabitants of the pond:

```ruby
class Pond
  def initialize(number_animals)
    @animals = []
    number_animals.times do |i|
      animal = new_animal("Animal#{i}")
      @animals << animal
    end
  end

  def simulate_one_day
    @animals.each {|animal| animal.speak}
    @animals.each {|animal| animal.eat}
    @animals.each {|animal| animal.sleep}
  end
end
```

```ruby
class DuckPond < Pond
  def new_animal(name)
    Duck.new(name)
  end
end

class FrogPond < Pond
  def new_animal(name)
    Frog.new(name)
  end
end

```


Now we need simply choose the right kind of pond, and it will be full of the right kind of creatures:
```ruby
pond = FrogPond.new(3)
pond.simulate_one_day
```

And we get all sorts of slimy green goings on:
```
Frog Animal0 says Crooooaaaak!
Frog Animal1 says Crooooaaaak!
Frog Animal2 says Crooooaaaak!
Frog Animal0 is eating.
Frog Animal1 is eating.
...
```


**The GoF called this technique of pushing the “which class” decision down on a subclass the Factory Method pattern. (In this example, Pond is no longer responsible for figuring out which class is instantiated. We push that down to something like FrogPond which then decides the which class decision. So now we call FrogPond(3) which inherits the initialize method from Pond. Now the Pond class is the creator which creates animals but does not decide which kind of class is created. So Pond is now flexible and can create any kind of animal as long as the subclass (something like FrogPond and DuckPond) decides which kind of animal to create. Now we call the FrogPond.new and not Pond.new. The Pond class's creation initialize method is flexible as it now calls a generic method that is implemented in each of the Animal pond classes which now decides which animal to create.)

Figure 13-1 shows the UML diagram for this pattern, which includes two separate class hierarchies. On the one hand, we have the creators, the base and concrete classes that contain the factory methods. On the other hand, we have the products, the objects that are being created. In our pond example, the creator is the Pond class, and the specific types of ponds (like DuckPond and FrogPond) are the concrete creators; the products are the Duck and Frog classes. While Figure 13-1 shows the two products sharing a common base class (Product), our Duck and Frog are not actually blood relatives: They simply share a common type because they implement a common set of methods.**

If you stare at Figure 13-1 long enough, you may discover that the Factory Method pattern is not really a new pattern at all. At its heart, this pattern is really just the Template Method pattern (remember Chapter 3?) applied to the problem of creating new objects. In both the Factory Method pattern and the Template Method pattern, a generic part of the algorithm (in our pond example, its day-to-day aquatic existence) is coded in the generic base class, and subclasses fill in the blanks left in the base class.**With the factory method, those filled-in blanks determine the class of objects that will be living in the pond.**



#### Parameterized Factory Methods

One problem with successful programs is that they tend to attract an ever-increasing pile of requirements. Suppose your pond simulation is so popular that your users start asking you to simulate plants as well as animals. So you wave your magic code wand and come up with a couple of plant classes:

```ruby
class Algae
  def initialize(name)
    @name = name
  end

  def grow
    puts("The Algae #{@name} soaks up the sun and grows")
  end
end

class WaterLily
  def initialize(name)
    @name = name
  end

  def grow
    puts("The water lily #{@name} floats, soaks up the sun, and grows")
  end
end
```


You also modify the Pond class to deal with plants, like this:

```ruby
class Pond
  def initialize(number_animals, number_plants)
    @animals = []
    number_animals.times do |i|
      animal = new_animal("Animal#{i}")
      @animals << animal
    end

    @plants = []
    number_plants.times do |i|
      plant = new_plant("Plant#{i}")
      @plants << plant
    end
  end

  def simulate_one_day
    @plants.each {|plant| plant.grow }
    @animals.each {|animal| animal.speak}
    @animals.each {|animal| animal.eat}
    @animals.each {|animal| animal.sleep}
  end
end
```

You will also need to modify the subclasses to create some flora:

```ruby


class DuckWaterLilyPond < Pond
  def new_animal(name)
    Duck.new(name)
  end

  def new_plant(name)
    WaterLily.new(name)
  end
end

class FrogAlgaePond < Pond
  def new_animal(name)
    Frog.new(name)
  end

  def new_plant(name)
    Algae.new(name)
  end
end
```


**An awkward aspect of this implementation is that we need a separate method for each type of object we are producing: We have the new_animal method to make frogs and ducks and the new_plant method to create lilies and algae. Having a separate method for each type of object that you need to produce is not too much of a burden if you are dealing with only two types, as in our pond example. But what if you have five or ten different types? Coding all those methods can be, well, tedious.**


A different and perhaps cleaner way to go is to have a single factory method that takes a parameter, a parameter that tells the method which kind of object to create. The following code shows yet another version of our Pond class, this time sporting a parameterized factory method—a method that can produce either a plant or an animal, depending on the symbol that is passed in:

```ruby
class Pond
  def initialize(number_animals, number_plants)
    @animals = []
    number_animals.times do |i|
      animal = new_organism(:animal, "Animal#{i}")
      @animals << animal
    end

    @plants = []
    number_plants.times do |i|
      plant = new_organism(:plant, "Plant#{i}")
      @plants << plant
    end
  end

  # ...
end
```

So the factory method here is the new_organism method:

```ruby
class DuckWaterLilyPond < Pond
  def new_organism(type, name)
    if type == :animal
      Duck.new(name)
    elsif type == :plant
      WaterLily.new(name)
    else
      raise "Unknown organism type: #{type}"
    end
  end
end
```

Parameterized factory methods tend to slim down the code, because each subclass needs to define only one factory method. They also make the whole thing a bit easier to extend. Suppose you need to define a new kind of product, perhaps fish to go in your pond. In that case, you need to modify only a single method in the subclasses instead of adding a whole new method—another example of the virtues of separating the things that change from those that don’t.

**Takeaway**
Parameterized factory methods in subclasses that determine which object to create allows you to have slimmer code. The creator class here (pond) now only calls one method and that never needs to change in the creator class and the subclass that determines which object to create only has one method that checks the type (the parameter).



#### Classes are just objects too and you can pass them into Pond

A more significant objection to the Factory Method pattern as we have written it so far is that this pattern requires a separate subclass for each specific type of object that needs to be manufactured. This is reflected in the names of the subclasses in the last version—we have DuckWaterLilyPond and FrogAlgaePond, but we could have just as easily needed a DuckAlgaePond or a FrogWaterLilyPond. Add a few more types of animals and plants, and the number of possible subclasses becomes truly scary. But the only difference between the various flavors of ponds is the class of objects produced by the factory method: In the one case it produces lilies and ducks, and in the other it makes algae and frogs.

The thing to realize is that the Frog, Duck, WaterLily, and Algae classes are just objects—objects that make their living by producing other objects, but objects nevertheless. We can get rid of this whole hierarchy of Pond subclasses by storing the classes of the objects that we want to create in instance variables:

```ruby
class Pond
  def initialize(number_animals, animal_class,
                 number_plants, plant_class)
    @animal_class = animal_class
    @plant_class = plant_class

    @animals = []
    number_animals.times do |i|
      animal = new_organism(:animal, "Animal#{i}")
      @animals << animal
    end

    @plants = []
    number_plants.times do |i|
      plant = new_organism(:plant, "Plant#{i}")
      @plants << plant
    end
  end

  def simulate_one_day
    @plants.each {|plant| plant.grow}
    @animals.each {|animal| animal.speak}
    @animals.each {|animal| animal.eat}
    @animals.each {|animal| animal.sleep}
  end

  def new_organism(type, name)
    if type == :animal
      @animal_class.new(name)
    elsif type == :plant
      @plant_class.new(name)
    else
      raise "Unknown organism type: #{type}"
    end
  end
end
```

```ruby
pond = Pond.new(3, Duck, 2, WaterLily)
pond.simulate_one_day
```


In this case, we get rid of the subclasses and now we're just down to one class again... the pond class. Now we don't need to require a separate subclass for each specific type of object that needs to be manufactured.



#### Extending the Pond class, the creator class to capture more types of creation.

Suppose even more success has befallen your pond simulator, and new requirements are pouring in faster than ever. The most pressing request is to extend this program to model other types of habitats besides ponds. In fact, a jungle simulation seems to be the next order of business.


```ruby
class Tree
  def initialize(name)
    @name = name
  end

  def grow
    puts("The tree #{@name} grows tall")
  end
end

class Tiger
  def initialize(name)
    @name = name
  end

  def eat
    puts("Tiger #{@name} eats anything it wants.")
  end

  def speak
    puts("Tiger #{@name} Roars!")
  end

  def sleep
    puts("Tiger #{@name} sleeps anywhere it wants.")
  end
end
```

You also need to change your Pond class’s name to something more appropriate for jungles as well as ponds. Habitat seems like a good choice:

```ruby
jungle = Habitat.new(1, Tiger, 4, Tree)
jungle.simulate_one_day

pond = Habitat.new( 2, Duck, 4, WaterLily)
pond.simulate_one_day
```

Now our pond class looks like Habitat:

```ruby
class Habitat
  def initialize(number_animals, animal_class,
                 number_plants, plant_class)
    @animal_class = animal_class
    @plant_class = plant_class

    @animals = []
    number_animals.times do |i|
      animal = new_organism(:animal, "Animal#{i}")
      @animals << animal
    end

    @plants = []
    number_plants.times do |i|
      plant = new_organism(:plant, "Plant#{i}")
      @plants << plant
    end
  end

  def simulate_one_day
    @plants.each {|plant| plant.grow}
    @animals.each {|animal| animal.speak}
    @animals.each {|animal| animal.eat}
    @animals.each {|animal| animal.sleep}
  end

  def new_organism(type, name)
    if type == :animal
      @animal_class.new(name)
    elsif type == :plant
      @plant_class.new(name)
    else
      raise "Unknown organism type: #{type}"
    end
  end
end
```

Other than the name change, Habitat is exactly the same as our last Pond implementation (the one with the plant and animal classes). We can create new habitats in exactly the same way that we did ponds.


#### Bundles of Object creation. How do make sure we don't create Tiger and waterlillies together?

Bundles of Object Creation
One problem with our new Habitat class is that it is possible to create incoherent (not to mention ecologically unsound) combinations of fauna and flora. For instance, nothing in our current habitat implementation tells us that tigers and lily pads do not go together:

```ruby
unstable = Habitat.new( 2, Tiger, 4, WaterLily)
```

This may not seem like much of a problem when you are dealing with just two kinds of things (plants and animals, in this case), but what if our simulation was much more detailed, extending to insects and birds and mollusks and fungi? We certainly don’t want any mushrooms growing on our lily pads or fish floundering away in the boughs of some jungle tree.

We can deal with this problem by changing the way we specify which creatures live in the habitat. Instead of passing the individual plant and animal classes to Habitat, **we can pass a single object that knows how to create a consistent set of products.**

We will have one version of this object for ponds, a version that will create frogs and lily pads. We will have a second version of this object that will create the tigers and trees that are appropriate to a jungle. **An object dedicated to creating a compatible set of objects is called an abstract factory.**

In fact, the Abstract Factory pattern is yet another of those patterns made famous by the GoF. The code below shows two abstract factories for our habitat simulation, one for the jungle and one for the pond:


```ruby
class PondOrganismFactory
  def new_animal(name)
    Frog.new(name)
  end

  def new_plant(name)
    Algae.new(name)
  end
end

class JungleOrganismFactory
  def new_animal(name)
    Tiger.new(name)
  end

  def new_plant(name)
    Tree.new(name)
  end
end
```


After a few simple modifications, our Habitat initialize method is ready to begin using the abstract factory:
```ruby
class Habitat
  def initialize(number_animals, number_plants, organism_factory)
    @organism_factory = organism_factory

    @animals = []
    number_animals.times do |i|
      animal = @organism_factory.new_animal("Animal#{i}")
      @animals << animal
    end

    @plants = []
    number_plants.times do |i|
      plant  = @organism_factory.new_plant("Plant#{i}")
      @plants << plant
    end
  end

  # Rest of the class...
```

We can now feed different abstract factories to our habitat, serene in the knowledge that there will be no unholy mixing of pond creatures with jungle denizens:

```ruby
jungle = Habitat.new(1, 4, JungleOrganismFactory.new)
jungle.simulate_one_day

pond = Habitat.new( 2, 4, PondOrganismFactory.new)
pond.simulate_one_day
```

The Abstract Factory pattern really boils down to a problem and a solution. The problem is that you need to create sets of compatible objects. The solution is that you write a separate class to handle that creation. In the same way that the Factory Method pattern is really the Template Method pattern applied to object creation, so the Abstract Factory pattern is simply the Strategy pattern applied to the same problem.

#### Classes are just objects again

One way to look at the abstract factory is to view it as a sort of super-duper-class object. While ordinary class objects know how to create only one type of object (i.e., instances of themselves), the abstract factory knows how to create several different types of objects (i.e., its products). This suggests a way to simplify our Abstract Factory pattern implementation: We can make it a bundle of class objects, with one class for each product. This is exactly the same “classes are just objects” insight that helped us simplify the Factory Method pattern

Let's simplify so we don't need to create a different class for different combinations of objects. Let's parameterize the object being created so we don't need to keep creating classes like JungleOrganismFactory for every combination of objects needed to be created.

```ruby
class OrganismFactory
  def initialize(plant_class, animal_class)
    @plant_class = plant_class
    @animal_class = animal_class
  end

  def new_animal(name)
    @animal_class.new(name)
  end

  def new_plant(name)
    @plant_class.new(name)
  end
end
```

With this class-based abstract factory, we can create a new instance of the factory for each compatible set of objects that we need:

```ruby
jungle_organism_factory = OrganismFactory.new(Tree, Tiger)
pond_organism_factory = OrganismFactory.new(WaterLily, Frog)

jungle = Habitat.new(1, 4, jungle_organism_factory)
jungle.simulate_one_day

pond = Habitat.new( 2, 4, pond_organism_factory)
pond.simulate_one_day
```

This all may seem a bit circular. After all, didn’t we originally create the abstract factory to avoid specifying the individual classes? Now we're specifying the classes again. And with our latest abstract factory implementation, aren’t we right back to being able to create a pond full of tigers or a jungle overrun by algae simply by passing those two classes into the abstract factory? Not really. The important thing about the abstract factory is that it encapsulates the knowledge of which product types go together. You can express that encapsulation with classes and subclasses, or you can get to it by storing the class objects as we did in the code above. Either way, you end up with an object that knows which kind of things belong together which is the point.



#### Leveraging Ruby Names

Another way that we can simplify the implementation of abstract factories is to rely on a consistent naming convention for the product classes. This approach won’t work for our habitat example, which is populated with things like tigers and frogs that have unique names, but imagine you need to produce an abstract factory for objects that know how to read and write a variety of file formats, such as PDF, HTML, and PostScript files. Certainly we could implement IOFactory using any of the techniques that we have discussed so far. But if the reader and writer class names follow some regular pattern, something like HTMLReader and HTMLWriter for HTML and PDFReader and PDFWriter for PDF, we can simply derive the class name from the name of the format. That’s exactly what the following code does:

```ruby
class IOFactory
  def initialize(format)
    @reader_class = self.class.const_get("#{format}Reader")
    @writer_class = self.class.const_get("#{format}Writer")
  end

  def new_reader
    @reader_class.new
  end

  def new_writer
    @writer_class.new
  end
end
```

```ruby
html_factory = IOFactory.new('HTML')
html_reader = html_factory.new_reader

pdf_factory = IOFactory.new('PDF')
pdf_writer = pdf_factory.new_writer
```

The const_get method used in IOFactory takes a string (or a symbol) containing the name of a constant (remember that class names in Ruby are constants) and returns the value of that constant. For example, if you pass const_get the string "PDFWriter", you will get back the class object of that name, which is exactly what we want in this case.
